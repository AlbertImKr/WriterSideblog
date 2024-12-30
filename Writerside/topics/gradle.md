# Gradle

## 시작

Gradle은 빌드 스크립트의 정보를 기반으로 소프트웨어 빌드, 테스트, 배포를 자동화합니다. 이 포스트에서 그래들에 대해 간단히 알아보겠습니다.

## 본문

### 그래들 설치

`brew`를 사용하여 그래들을 설치할 수 있습니다.

```bash
brew install gradle
```

이렇게 설치하면 `gradle` 명령어를 사용할 수도 있지만 래퍼(wrapper)를 더 권장합니다. 래퍼는 프로젝트에 종속되어 있어 프로젝트의 빌드를 위해 사용하는 그래들 버전을 명시적으로 지정할 수 있습니다.

`gradle wrapper` 명령어를 사용하여 래퍼를 생성할 수 있습니다.

```bash
gradle wrapper
```

### 빌드 스크립트

그래들은 `build.gradle` 파일을 사용하여 빌드 스크립트를 작성합니다. 빌드 스크립트는 Groovy나 Kotlin으로 작성할 수 있습니다.

groovy로 작성한 빌드 스크립트 예시입니다.

```groovy
plugins {
    id 'java'
}

group = 'me.albert'
version = '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    testImplementation platform('org.junit:junit-bom:5.10.0')
    testImplementation 'org.junit.jupiter:junit-jupiter'
}

test {
    useJUnitPlatform()
}
```

### 플러그인

그래들은 마찬가지로 플러그인을 사용하여 빌드 스크립트를 확장할 수 있습니다. 플러그인은 빌드 스크립트에 추가하여 사용할 수 있습니다.

```groovy
plugins {
    id 'java' // Java 플러그인
    id 'org.springframework.boot' version '2.6.0' // 스프링 부트 플러그인
}
```

### 빌드

그래들은 빌드 스크립트를 사용하여 빌드를 수행합니다. 빌드를 수행하려면 `gradle` 명령어를 사용합니다.

```bash
gradle build
```

또는 래퍼를 사용하여 빌드를 수행할 수도 있습니다.

```bash
./gradlew build
```

### 작업 회피(incremental build)

그래들은 작업 회피(incremental build)를 지원합니다. 작업 회피는 이전 빌드에서 변경된 파일만 다시 컴파일하여 빌드 시간을 단축합니다.

```bash
gradle build

# 결과
BUILD SUCCESSFUL in 17s
2 actionable tasks: 2 executed

# 다시 빌드
gradle build

# 결과
BUILD SUCCESSFUL in 520ms
2 actionable tasks: 2 up-to-date
```

그래들은 빌드 캐시(build cache)를 사용하여 빌드 결과를 캐싱하여 빌드 시간을 단축하고 빌드 중복을 방지합니다. `clean` 작업을 수행하면 빌드 캐시가 초기화됩니다.

```bash
gradle clean
```

### 그래들의 의존성 관리

그래들은 의존성 관리를 위해 `dependencies` 블록을 사용합니다. 의존성은 `group`, `name`, `version`으로 구성됩니다.

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web:2.6.0'
    runtimeOnly 'org.springframework.boot:spring-boot-devtools:2.6.0'
    testImplementation 'org.springframework.boot:spring-boot-starter-test:2.6.0'
}
```

Maven과 비교하면 가독성이 좋고, 의존성을 추가하거나 제거할 때 빌드 스크립트를 수정하는 것이 간단합니다. Google, Maven Central 등의 저장소에서 의존성을 가져올 수 있습니다.

`gradle dependencies` 명령어를 사용하여 프로젝트의 의존성 트리를 확인할 수 있습니다.

```bash
gradle dependencies
```

### Java, Kotlin 등 다양한 언어 동시 지원

그래들은 Java 프로젝트에 Kotlin 프로젝트를 추가하는 것과 같이 다양한 언어를 동시에 지원합니다. 또한 다양한 플러그인을 사용하여 빌드 스크립트를 확장할 수 있습니다.

프로젝트 구조는 다음과 같습니다.

```bash
├── settings.gradle.kts
└── src
    ├── main
    │   ├── java
    │   │   └── me
    │   │       └── albert
    │   │           └── App.java  
    │   └── kotlin   # Kotlin 프로젝트 추가
    │       └── me
    │           └── albert
    │               └── App.kt
    └── test
        ├── java
        └── resources
```

스크립트 파일은 다음과 같습니다.

```kotlin
plugins {
    application
    kotlin("jvm") version "2.0.21"
}
```

### 그래들에서 모듈 사용

그래들은 모듈을 사용하여 프로젝트를 구성할 수 있습니다. Java 프로젝트에서는 `module-info.java` 파일을 사용하여 모듈을 정의할 수 있습니다.

`module-info.java` 파일은 다음과 같이 정의할 수 있습니다.

```java
module me.albert {
    requires spring.boot;
    requires spring.boot.autoconfigure;
    requires spring.boot.starter.web;
}
```

`module-info.java` 파일의 위치는 `src/main/java` 디렉토리에 있어야 합니다.

```Bash
└── src
    ├── main
    │   └── java
    │       ├── me
    │       │   └── albert
    │       │       └── App.java
    │       └── module-info.java
    └── test
        └── java
            └── me
                └── albert
                    └── AppTest.java
```

### 사용자 정의 작업

그래들은 사용자 정의 작업을 추가하여 빌드 스크립트를 확장할 수 있습니다. 사용자 정의 작업은 `task` 블록을 사용하여 정의할 수 있습니다.

```groovy
task copyDocument(type: Copy) {
    dependsOn asciidoctor
    from file("build/docs/asciidoc")
    into file("src/main/resources/static/docs")
}
```

위의 코드는 `asciidoctor` 작업이 실행된 후 `copyDocument` 작업이 실행됩니다. `asciidoctor` 작업은 AsciiDoc 파일을 HTML로 변환하는 작업입니다.

## 마무리

이 포스트에서는 그래들에 대해 간단히 알아보았습니다. 그래들은 빌드 스크립트를 사용하여 소프트웨어 빌드, 테스트, 배포를 자동화하는 도구입니다. 그래들은 다양한 플러그인을 사용하여 빌드 스크립트를 확장할 수
있습니다. 또한 래퍼를 사용하여 프로젝트에 종속되어 있는 그래들 버전을 명시적으로 지정할 수 있습니다.

## 참조

- [기본기가 탄탄한 자바 개발자](https://product.kyobobook.co.kr/detail/S000213907278)



