# mkcert, Spring Boot를 사용하여 Localhost에서 HTTPS를 사용하는 방법

## 개요

프로젝트 개발 중에 HTTPS를 사용해야 하는 경우가 있습니다. 이때, Localhost에서 HTTPS 사용해야 하는 경우가 있습니다. 이때, mkcert를 사용하면 간단하게 Localhost에서 HTTPS를 사용할
수 있습니다.

## mkcert 무엇인가요?

mkcert는 로컬에서 사용할 수 있는 간단한 인증서를 생성해 주는 도구입니다.

## mkcert 설치 방법 (macOS)

Homebrew를 사용하여 mkcert를 설치합니다.

```bash
brew install mkcert
```

## mkcert 사용 방법 {#mkcert-사용-방법}

local CA를 시스템 신뢰 저장소에 설치합니다.

```bash
mkcert -install
```

mkcert를 사용하여 인증서를 생성합니다. 이때, localhost를 사용할 경우 다음과 같이 입력합니다.

```bash
mkcert localhost
```

local CA를 시스템 신뢰 저장소에서 제거합니다.

```bash
mkcert -uninstall
```

## mkcert 고급 options

출력 경로를 지정하여 인증서를 생성할 수 있습니다.

```bash
mkcert -cert-file FILE
mkcert -key-file FILE
mkcert -p12-file FILE
```

클라이언트 인증서를 생성할 수 있습니다.

```bash
mkcert -client
```

ECDSA 키를 사용하여 인증서를 생성할 수 있습니다.

```bash
mkcert -ecdsa
```

`.p12` PKCS#12 파일을 생성할 수 있습니다.

```bash
mkcert -pkcs12
```

제공된 CSR을 사용하여 인증서를 생성할 수 있습니다.

```bash
mkcert -csr CSR
```

## Spring Boot에서 SSL 설정

Spring Boot에서 keystore를 사용하여 SSL 설정을 하는 방법은 다음과 같습니다.

```yaml
server:
  port: 8443
  ssl:
    key-store: classpath:keystore.p12
    key-store-password: password
```

PEM 파일을 사용하여 SSL 설정을 하는 방법은 다음과 같습니다.

```yaml
server:
  port: 8443
  ssl:
    certificate: "classpath:my-cert.crt"
    certificate-private-key: "classpath:my-cert.key"
    trust-certificate: "classpath:ca-cert.crt"
```

SSL 번들을 사용하여 SSL 설정을 하는 방법은 다음과 같습니다.

```yaml
server:
  port: 8443
  ssl:
    bundle: "example"
```

서버 일름 표시 이름을 사용하여 SSL 설정을 하는 방법은 다음과 같습니다.

```yaml
server:
  port: 8443
  ssl:
    bundle: "web"
    server-name-bundles:
      - server-name: "alt1.example.com"
        bundle: "web-alt1"
      - server-name: "alt2.example.com"
        bundle: "web-alt2"
```

## mkcert를 사용하여 Localhost에서 HTTPS 사용하게 만드는 예시

1. mkcert를 사용하여 Spring Boot 프로젝트에서 `src/main/resources` 디렉토리에 `keystore.p12` 파일을 추가합니다.

```bash
mkcert -pkcs12 -p12-file keystore.p12 localhost
```

2. 생성한 인증서를 시스템 신뢰 저장소에 설치합니다.

```bash
mkcert -install
```

3. `application.yml` 파일에 다음과 같이 SSL 설정을 추가합니다.

```yaml
server:
  port: 8443
  ssl:
    enabled: true
    key-store: classpath:keystore.p12
    key-store-password: changeit # 기본값
```

4. Spring Boot 프로젝트를 실행하고 `https://localhost:8443`에 접속하여 확인합니다.

## 마무리

mkcert를 사용하여 Localhost에서 HTTPS를 사용하는 방법에 대해 알아보았습니다. mkcert를 사용하면 간단하게 Localhost에서 HTTPS를 사용할 수 있습니다.

## 참고

- [mkcert GitHub](https://github.com/FiloSottile/mkcert)
- [SpringBoot configure-ssl](https://docs.spring.io/spring-boot/how-to/webserver.html#howto.webserver.configure-ssl)



