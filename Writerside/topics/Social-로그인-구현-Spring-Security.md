# Social 로그인 구현 (Spring Security)

## 개요

Spring Security를 사용하여 Social 로그인을 구현하는 방법에 대해 간단히 알아보겠습니다.

## Google 로그인 구현

Google Oauth2은 Spring Security에서 기본적으로 지원하는 Oauth2 제공자 중 하나입니다. 구현하는 것은 매우 간단합니다.

### Google API Console에서 프로젝트 생성

1. [Google API Console](https://console.developers.google.com/)에 접속합니다.
2. `사용자 인증 정보` > `사용자 인증 정보 만들기`를 클릭합니다.
   ![사용자 인증 정보 만들기.png](사용자 인증 정보 만들기.png)
3. `OAuth 클라이언트 ID`를 선택합니다.
4. `웹 애플리케이션`을 선택하고 `이름`을 입력합니다.
5. `승인된 JavaScript 원본`과 `승인된 리디렉션 URI`를 입력합니다.
    - `승인된 JavaScript 원본`: 브라우저에서 JavaScript를 사용하여 Google API를 호출할 때 사용하는 도메인입니다. (예: `https://localhost:9000`)
    - `승인된 리디렉션 URI`: 사용자가 인증을 완료하면 Google이 사용자를 리디렉션하는 URI입니다. (예: `https://localhost:9000/login/oauth2/code/google`)
6. `만들기`를 클릭합니다.

![OAuth 클라이언트 ID 만들기.png](OAuth 클라이언트 ID 만들기.png)

### Spring Boot 프로젝트 설정

기본 SecurityFilterChain에 Oauth2LoginConfigurer를 추가하여 Google 로그인을 구현합니다.

```java
    @Bean
    public SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
        return http
                .authorizeHttpRequests(request -> request
                        .requestMatchers(HttpMethod.POST, "/api/users", "/login").permitAll()
                        .anyRequest().authenticated())
                // CSRF 보호 기능을 비활성화합니다.
                .csrf(AbstractHttpConfigurer::disable)
                .formLogin(Customizer.withDefaults())
                # ----------------- 추가 -----------------
                .oauth2Login(Customizer.withDefaults())
                # ----------------------------------------
                .build();
    }
```

### application.yml 설정

application.yml 파일에 Google 로그인을 위한 설정을 추가합니다.

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            provider: google
            # Google API Console에서 발급받은 클라이언트 ID
            client-id: { google-client-id }
            # Google API Console에서 발급받은 클라이언트 비밀번호
            client-secret: { google-client-secret } 
```

### Google 로그인 테스트 {#google-login-test}

1. Spring Boot 프로젝트를 실행합니다.
2. 로그인 페이지로 이동합니다. (예: `https://localhost:9000/login`)
3. `Google` 버튼을 클릭하여 Google 로그인을 테스트합니다.

또는

1. Spring Boot 프로젝트를 실행합니다.
2. 다이렉트로 Google 로그인 URL로 이동합니다. (예: `https://localhost:9000/oauth2/authorization/google`)

## Kakao 로그인 구현

Kakao 로그인은 Spring Security에서 기본적으로 지원하지 않습니다. Google 로그인과는 다르게 구현하는 방법이 다소 복잡합니다.

### Kakao Developers에서 프로젝트 생성

1. [Kakao Developers](https://developers.kakao.com/)에 접속합니다.
2. `내 애플리케이션` > `애플리케이션 추가`를 클릭합니다.
3. `애플리케이션 이름`을 입력하고 `애플리케이션 만들기`를 클릭합니다.
4. `플래폼` > `Web`을 선택합니다. `사이트 도메인`을 입력합니다.
    - `사이트 도메인`: 애플리케이션을 사용하는 웹 사이트의 도메인입니다. (예: `https://localhost:9000`)
5. `카카오 로그인` > `활성화 설정`을 `ON`으로 변경합니다.
6. `OpenID Connect` > `활성화 설정`을 `ON`으로 변경합니다.
7. `Redirect URI`를 입력합니다.
    - `Redirect URI`: 사용자가 인증을 완료하면 Kakao가 사용자를 리디렉션하는 URI입니다. (예: `https://localhost:9000/login/oauth2/code/kakao`)
8. `앱 키`에서 `REST API 키`를 확인합니다. `REST API 키`는 클라이언트 ID와 같습니다.
9. `보안` > Client Secret 코드를 생성 및 확인합니다. `Client Secret`는 클라이언트 비밀번호와 같습니다.

### application.yml 설정 {#kakao-application-yml}

application.yml 파일에 Google 로그인을 위한 설정을 추가합니다.

```yaml
spring:
  security:
    oauth2:
      client:
      registration:
        google:
          client-id: { google-client-id }
          client-secret: { google-client-secret }
        # ----------------- 추가 -----------------
        kakao:
          client-id: REST_API_KEY
          client-secret: CLIENT_SECRET
      provider:
        kakao:
          issuer-uri: "https://kauth.kakao.com"
        # ----------------------------------------
```

## Naver 로그인 구현

Naver 로그인도 Spring Security에서 기본적으로 지원하지 않습니다. 뿐만 아니라 Naver 로그인은 OpenID Connect를 지원하지 않습니다. 따라서 Naver 로그인을 구현하려면 더 많은 설정이
필요합니다.

### Naver Developers에서 프로젝트 생성

1. [Naver Developers](https://developers.naver.com/apps/#/register?api=nvlogin)에 접속합니다.
2. `애플리케이션 이름`을 입력합니다.
3. 필요한 `개인정보 수집 항목`을 선택합니다.
4. 로그인 오픈 API 서비스 환경(PC 웹 브라우저)를 선택합니다.
    - `서비스 환경`: 로그인을 사용할 환경을 선택합니다. (예: `PC 웹 브라우저`)
    - `서비스 URL`: 서비스를 사용하는 웹 사이트의 URL입니다. (예: `https://localhost:9000`)
    - `Callback URL`: 사용자가 인증을 완료하면 Naver가 사용자를 리디렉션하는 URI입니다. (예: `https://localhost:9000/login/oauth2/code/NAVER`)
5. 약관을 확인하고 `애플리케이션 등록`을 클릭합니다.
6. `애플리케이션 정보`에서 `Client ID`와 `Client Secret`를 확인합니다.

### application.yml 설정 {#naver-application-yml}

application.yml 파일에 Naver 로그인을 위한 설정을 추가합니다.

```yaml
      client:
        registration:
          google:
            client-name: "Google 로그인"
            client-id: { google-client-id }
            client-secret: { google-client-secret }
          kakao:
            client-name: "KAKAO 로그인"
            client-id: { kakao-client-id }
            client-secret: { kakao-client-secret }
          # ----------------- 추가 -----------------
          NAVER:
            client-name: "NAVER 로그인"
            client-id: { naver-client-id }
            client-secret: { naver-client-secret }
            authorization-grant-type: "authorization_code"
            redirect-uri: "https://localhost:9000/login/oauth2/code/NAVER"
          # ----------------------------------------
        provider:
          kakao:
            issuer-uri: "https://kauth.kakao.com"
          # ----------------- 추가 -----------------
          NAVER:
            authorization-uri: "https://nid.naver.com/oauth2.0/authorize"
            token-uri: "https://nid.naver.com/oauth2.0/token"
            user-info-uri: "https://openapi.naver.com/v1/nid/me"
            user-name-attribute: "response"
          # ----------------------------------------
```

## 마무리

Spring Security를 사용하여 Social 로그인을 구현하는 방법에 대해 간단히 알아보았습니다. Google 로그인은 Spring Security에서 기본적으로 지원하며 Kakao 로그인과 Naver
로그인은 추가 설정이 필요합니다. Social 로그인을 구현하면 사용자가 손쉽게 로그인할 수 있으며 사용자 정보를 가져와서 사용할 수 있습니다.

## 참고

- [Spring Security Reference](https://docs.spring.io/spring-security/site/docs/current/reference/html5/)
- [Google API Console](https://console.developers.google.com/)
- [Kakao Developers](https://developers.kakao.com/)
- [Kakao API Reference](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api)
- [Naver Developers](https://developers.naver.com/apps/#/register?api=nvlogin)
- [Naver Developers API Reference](https://developers.naver.com/docs/login/api/api.md)
