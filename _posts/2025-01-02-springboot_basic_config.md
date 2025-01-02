---
title: SpringBoot gradle, application.yml 설정 
date: 2025-01-02 11:58:40 +9000
categories: [MoaMoa, SpringBoot]
tags: [gradle, build, application, yml, yaml, maven, profile]     # TAG names should always be lowercase
---

Spring의 Profile
--
스프링은 ```@Profile```으로 각 Profile에 맞는 Configuration을 적용 할수 있다. 
```@Profile```의 적용 가능 Annotation은 다음과 같다
- ```@Component```
- ```@Configuration```
- ```ConfigurationProperties```

application.yml에서의 Multi profile
--
### 사용할 프로필 설정하기
대표적인 기능으로 ```active```, ```include```와 ```group```가 있다.
- ```active```는 두 profile을 활성화하고 merge하는 것이고 
- ```include```는 기존 프로필에 include된 profile조각을 덧붙이는 느낌이다.
- ```group```을 사용하여 여러 profile을 하나의 ```group```으로 관리할수있다. 
```yaml
spring:
  profiles:
    active: "dev,h2db"
```
```yaml
spring:
  profiles:
    active: "dev"
    include:
      - "common"
      - "local"
```
```yaml
spring:
  profiles:
    group:
      production:
      - "proddb"
      - "prodmq"
```

| **Aspect**              | **`spring.profiles.active`**               | **`spring.profiles.include`**              | **`spring.profiles.group`**                 |
|--------------------------|--------------------------------------------|--------------------------------------------|---------------------------------------------|
| **Purpose**             | Activates specific profile(s).             | Includes additional profiles for merging.  | Defines groups of profiles to activate together. |
| **Activation**          | Activates only specified profiles.         | Merges included profiles into the active ones. | Activates profiles in a logical group when the parent is activated. |
| **Configuration Merge** | Only active profiles are merged.           | Includes additional configurations.        | Group profiles and their children are all merged. |
| **Primary Use Case**     | Explicit environment activation.           | Adding shared/auxiliary configurations.    | Managing logical groups of related profiles.     |
| **Example Usage**        | `spring.profiles.active: prod`             | `spring.profiles.include: shared`          | `spring.profiles.group: { production: [db-prod, security-prod] }` |

> ***주의할점***: 위 설정은 non-profile-specific document에서만 작성돼야한다.
> 즉, ```spring.config.activate.on-profile.```과 공존할수 없다. 

### 프로필 생성하기
프로필을 생성하는 법은 간단하다. 아래코드는 prod라는 프로필을 만드는 코드이다. 
```yaml
spring:
  config:
    activate:
      on-profile: "prod"
```
### Property값 외부 주입받기
```application.yml```에 하드코딩이 아니라 외부에서 주입 받게 하여 보안을 높이자
```${...}```을 사용한다. 
```yaml
app:
  name: "${name}"
  description: "${description}"
```
> Gradle의 ```expand```도 spring placeholder와 같이 ```${...}```을 쓰는데
> 이를 구분하기 위해 spring placeholde은 다음과 같이 역슬래쉬를 붙여 표시하자 ```\${...}```
 
추가적인 내용은 Reference의 2번째 링크 참조

### Springboot에 관련 deep한 Externalized Configuration
다음 사이트 참고
- [Externalized Configuration](https://docs.spring.io/spring-boot/reference/features/external-config.html)

### 많이 쓰이고, 설정되여하는 Propeties 목록
- [Common Application Properties](https://docs.spring.io/spring-boot/appendix/application-properties/index.html)

### build.gradle에 추가할 package정보 찾기
Maven Central Repository에서 검색해서 찾자
- [Maven Central Repository](https://central.sonatype.com)

<details>
<summary>MoaMoa에서 사용한 application.yml</summary>

``` yaml
spring:
  profiles:
    include:
    - "h2db" #원하는 profile 활성화 시키기
    - "default"
---
spring:
  config:
    activate:
      on-profile: "default"
  application:
    name: "moamoa"
server:
  port: 9000
---
spring:
  config:
    activate:
      on-profile: "h2db"
  h2:
    console:
      enabled: true
      settings:
        web-allow-others: true
      path: /h2-console
  datasource:
    url: jdbc:h2:mem:test;mode=mysql  # inmemory mode실행 및 mysql 모드로 실행
    username: sa
    password:
    driver-class-name: org.h2.Driver
  jpa:
    database: h2  # 알아서 해줘서 해줄 필요 없긴 함
    hibernate:
      ddl-auto: create-drop # 매 실행시 db create & 서버 종료시 db drop
    show-sql: true
```
</details>

### Reference
- [Profile](https://docs.spring.io/spring-boot/reference/features/profiles.html)
- [Automatic Property Expansion using gradle](https://docs.spring.io/spring-boot/how-to/properties-and-configuration.html)
- [Common Application Properties](https://docs.spring.io/spring-boot/appendix/application-properties/index.html)
- [Externalized Configuration](https://docs.spring.io/spring-boot/reference/features/external-config.html)
