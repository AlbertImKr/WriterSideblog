# Fork/Join 프레임워크와 CompletableFuture

## 시작

동시성 프로그래밍은 프로그램에서 매우 중요한 개념입니다. 이 Post에서는 Java에서 동시성 프로그래밍을 위해 사용하는 Fork/Join 프레임워크와 CompletableFuture에 대해 알아보겠습니다.

## 본문

### Fork/Join 프레임워크

Fork/Join 프레임워크는 보이지 않은 스레드 풀을 사용하여 작업을 분할하고 자동으로 스케줄링하고 병렬로 실행하는 프레임워크입니다.

코드로 살펴보겠습니다.

````Java
// RecursiveAction은 ForkJoinTask<Void>의 하위 클래스입니다.
// RecursiveAction은 compute() 메서드를 사용하여 작업을 수행합니다.
public class TransactionSorter extends RecursiveAction {

    // 작업을 분할할 때, 작업의 크기가 이 값보다 작으면 작업을 분할하지 않고 작업을 수행합니다.
    private static final int SMALL_ENOUGH = 32;
    private final Transaction[] transactions;
    private final int start;
    private final int end;
    private final Transaction[] result;

    public TransactionSorter(List<Transaction> transactions) {
        this(transactions.toArray(new Transaction[0]), 0, transactions.size());
    }

    public TransactionSorter(Transaction[] transactions) {
        this(transactions, 0, transactions.length);
    }

    private TransactionSorter(Transaction[] transactions, int start, int end) {
        this.transactions = transactions;
        this.start = start;
        this.end = end;
        this.result = new Transaction[size()];
    }

    public int size() {
        return end - start;
    }

    public Transaction[] getResult() {
        return result;
    }

    @Override
    protected void compute() {
        if (size() < SMALL_ENOUGH) {
            System.arraycopy(transactions, start, result, 0, size());
            Arrays.sort(result, 0, size());
            // 작업이 킅난 쓰레드의 이름과 정렬된 결과를 출력합니다.
            System.out.println(Thread.currentThread().getName() + " : " + Arrays.toString(result));
        } else {
            // 작업을 분할합니다.
            int mid = start + size() / 2;
            TransactionSorter left = new TransactionSorter(transactions, start, mid);
            TransactionSorter right = new TransactionSorter(transactions, mid, end);

            // 작업을 병렬로 실행합니다.
            invokeAll(left, right);
            // 작업을 병합합니다.
            merge(left, right);
        }
    }

    private void merge(TransactionSorter left, TransactionSorter right) {
        int i = 0;
        int l = 0;
        int r = 0;

        while (l < left.size() && r < right.size()) {
            int compare = left.result[l].compareTo(right.result[r]);
            result[i++] = compare < 0 ? left.result[l++] : right.result[r++];
        }

        while (l < left.size()) {
            result[i++] = left.result[l++];
        }

        while (r < right.size()) {
            result[i++] = right.result[r++];
        }
    }
}

// ----------------------------
public class Transaction implements Comparable<Transaction> {

    private static final AtomicLong counter = new AtomicLong(1);
    private final Account sender;
    private final Account receiver;
    private final int amount;
    private final long id;
    private final LocalDateTime time;

    public Transaction(Account sender, Account receiver, int amount, LocalDateTime time) {
        this.sender = sender;
        this.receiver = receiver;
        this.amount = amount;
        this.id = counter.getAndIncrement();
        this.time = time;
    }

    public static Transaction of(Account sender, Account receiver, int amount) {
        return new Transaction(sender, receiver, amount, LocalDateTime.now());
    }

    @Override
    public int compareTo(Transaction o) {
        return Comparator.nullsFirst(LocalDateTime::compareTo).compare(this.time, o.time);
    }

    public Account getSender() {
        return sender;
    }

    public Account getReceiver() {
        return receiver;
    }

    public int getAmount() {
        return amount;
    }

    public long getId() {
        return id;
    }

    public LocalDateTime getTime() {
        return time;
    }

    @Override
    public int hashCode() {
        return Objects.hash(getSender(), getReceiver(), getAmount(), getId(), getTime());
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (!(o instanceof Transaction)) {
            return false;
        }
        Transaction that = (Transaction) o;
        return getAmount() == that.getAmount() && getId() == that.getId() &&
               Objects.equals(getSender(), that.getSender()) &&
               Objects.equals(getReceiver(), that.getReceiver()) &&
               Objects.equals(getTime(), that.getTime());
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Transaction.class.getSimpleName() + "[", "]")
                .add("id=" + id)
                .toString();
    }
}
// ----------------------------

public class Account {

    private int balance;

    public Account(int balance) {
        this.balance = balance;
    }

    public int getBalance() {
        return balance;
    }

    public void deposit(int amount) {
        balance += amount;
    }

    public void withdraw(int amount) {
        balance -= amount;
    }
}

// ----------------------------

public static void main(String[] args) {
    var transactions = new ArrayList<Transaction>();
    var accounts = List.of(new Account(1000), new Account(2000), new Account(3000));

    for (int i = 0; i < 100; i++) {
        var sender = accounts.get(i % accounts.size());
        var receiver = accounts.get((i + 1) % accounts.size());
        var amount = (i + 1) * 10;
        transactions.add(Transaction.of(sender, receiver, amount));
    }

    Collections.shuffle(transactions);
    var sorter = new TransactionSorter(transactions);

    ForkJoinPool.commonPool().invoke(sorter);

    for (Transaction transaction : sorter.getResult()) {
        System.out.println(transaction);
    }
}
````

위 코드는 Fork/Join 프레임워크를 사용하여 Transaction 객체를 정렬하는 코드입니다. 출력 결과는 다음과 같습니다.

````bash
main : [Transaction[id=1], Transaction[id=4], Transaction[id=7], Transaction[id=10], Transaction[id=16], Transaction[id=29], Transaction[id=35], Transaction[id=37], Transaction[id=41], Transaction[id=42], Transaction[id=45], Transaction[id=47], Transaction[id=49], Transaction[id=50], Transaction[id=54], Transaction[id=58], Transaction[id=61], Transaction[id=67], Transaction[id=72], Transaction[id=78], Transaction[id=83], Transaction[id=84], Transaction[id=86], Transaction[id=91], Transaction[id=99]]
ForkJoinPool.commonPool-worker-5 : [Transaction[id=9], Transaction[id=12], Transaction[id=13], Transaction[id=14], Transaction[id=17], Transaction[id=22], Transaction[id=24], Transaction[id=25], Transaction[id=26], Transaction[id=27], Transaction[id=30], Transaction[id=32], Transaction[id=33], Transaction[id=40], Transaction[id=60], Transaction[id=62], Transaction[id=63], Transaction[id=68], Transaction[id=69], Transaction[id=75], Transaction[id=85], Transaction[id=93], Transaction[id=95], Transaction[id=97], Transaction[id=100]]
ForkJoinPool.commonPool-worker-3 : [Transaction[id=11], Transaction[id=15], Transaction[id=20], Transaction[id=21], Transaction[id=31], Transaction[id=36], Transaction[id=39], Transaction[id=44], Transaction[id=46], Transaction[id=48], Transaction[id=56], Transaction[id=59], Transaction[id=66], Transaction[id=70], Transaction[id=73], Transaction[id=74], Transaction[id=76], Transaction[id=77], Transaction[id=79], Transaction[id=80], Transaction[id=82], Transaction[id=87], Transaction[id=94], Transaction[id=96], Transaction[id=98]]
ForkJoinPool.commonPool-worker-7 : [Transaction[id=2], Transaction[id=3], Transaction[id=5], Transaction[id=6], Transaction[id=8], Transaction[id=18], Transaction[id=19], Transaction[id=23], Transaction[id=28], Transaction[id=34], Transaction[id=38], Transaction[id=43], Transaction[id=51], Transaction[id=52], Transaction[id=53], Transaction[id=55], Transaction[id=57], Transaction[id=64], Transaction[id=65], Transaction[id=71], Transaction[id=81], Transaction[id=88], Transaction[id=89], Transaction[id=90], Transaction[id=92]]
Transaction[id=1]
Transaction[id=2]
Transaction[id=3]
Transaction[id=4]
Transaction[id=5]
....
Transaction[id=100]
````

결과에서 볼 수 있듯이 작업이 분할되어 병렬로 스레드에서 실행되고 병합되어 정렬된 결과가 출력됩니다.

> Fork/Join 프레임워크는 work-stealing 알고리즘을 사용하여 작업을 분할하고 병렬로 실행합니다. work-stealing 알고리즘은 스레드가 작업을 끝내면 다른 스레드의 시작하지 않은 작업을
> 가져와서 실행합니다.
> {style="note"}

### CompletableFuture

CompletableFuture는 Future 인터페이스를 확장한 인터페이스로 비동기 작업을 쉽게 처리할 수 있도록 도와줍니다.

코드를 통해 살펴보겠습니다.

````Java
public static void main(String[] args) {
    var future = CompletableFuture.supplyAsync(() -> {
        System.out.println("Starting on: " + Thread.currentThread().getName());
        return "Starting!";
    });

    var future2 = future.thenApply(s -> {
        System.out.println("ThenApply on: " + Thread.currentThread().getName());
        return s + "\nThenApply!";
    });

    var future3 = future2.thenApplyAsync(s -> {
        System.out.println("ThenApplyAsync on: " + Thread.currentThread().getName());
        return s + "\nThenApplyAsync!";
    });

    try {
        System.out.println(future3.get());
    } catch (Exception e) {
        e.printStackTrace();
    }
}
````

위 코드는 CompletableFuture를 사용하여 비동기 작업을 처리하는 코드입니다. 출력 결과는 다음과 같습니다.

````bash
Starting on: ForkJoinPool.commonPool-worker-3
ThenApply on: ForkJoinPool.commonPool-worker-3
ThenApplyAsync on: ForkJoinPool.commonPool-worker-5
Starting!
ThenApply!
ThenApplyAsync!
````

결과에서 볼 수 있듯이 `supplyAsync()` 메서드를 사용하여 비동기 작업을 시작하고 `thenApply()` 메서드를 사용하여 작업을 처리하고 `thenApplyAsync()` 메서드를 사용하여 비동기로
작업을 처리합니다. `async` 메서드를 사용하면 작업이 다른 스레드에서 실행됩니다.

## 마무리

이 Post에서는 Java의 동시성 프로그래밍에 사용하는 Fork/Join 프레임워크와 CompletableFuture에 대해 정리해 보았습니다. Fork/Join 프레임워크는 작업을 분할하고 병렬로 실행한 후
병합하는 프레임워크이며 CompletableFuture는 비동기 작업을 쉽게 처리할 수 있도록 도와주는 인터페이스입니다.

## 참조

- [기본기가 탄탄한 자바 개발자](https://product.kyobobook.co.kr/detail/S000213907278)