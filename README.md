# Config Hub 가이드 문서

본 레포지토리는 *프로젝트: Where Is My Parcel*의 모든 설정 파일들을 중앙에서 관리하기 위한 Config Repository입니다. 

각 마이크로서비스는 개별 모듈로 개발되지만, 실행에 필요한 공통 설정과 서비스별 설정은 본 레포지토리에서 관리합니다. Config Server는 본 레포지토리의 설정 파일을 읽어 각 서비스에 제공합니다.

---

## 역할

본 레포지토리는 다음 설정을 관리합니다.

- 전체 서비스 공통 설정
- profile별 공통 설정
- 서비스별 기본 설정
- 서비스별 profile 설정

민감정보는 본 레포지토리에 저장하지 않습니다.
DB 비밀번호, JWT Secret, 외부 API Key 등은 `.env` 및 배포 환경의 Secret 관리 시스템을 통해 주입합니다.

---

## 파일 네이밍 규칙

Spring Cloud Config는 `{application}`, `{profile}`, `{label}` 값을 기준으로 설정 파일을 조회합니다.

```text
GET config서버ip주소/{application}/{profile}/{label}
```

예를 들어 `user-service`가 `dev` profile로 실행되고, `main` label의 설정을 요청하는 경우 다음 요청이 발생합니다.

```text
/user-service/dev/main
```

이때 Config Server는 본 레포지토리에서 다음 파일들을 조합하여 설정을 제공합니다.

```text
application.yml
application-dev.yml
user-service.yml
user-service-dev.yml
```

---

## 권장 파일 구조

초기 단계에서는 Config Server의 기본 탐색 규칙을 사용하기 위해 모든 설정 파일을 레포지토리 루트에 올려둡니다.

```text
config-hub
├── README.md
├── application.yml
├── application-dev.yml
├── application-prod.yml
├── api-gateway.yml
├── api-gateway-dev.yml
├── api-gateway-prod.yml
├── user-service.yml
├── user-service-dev.yml
├── user-service-prod.yml
└── ...
```

---

## 파일별 역할

| 파일                        | 역할                                |
| ------------------------- | --------------------------------- |
| `application.yml`         | 모든 서비스에 적용되는 공통 기본 설정             |
| `application-dev.yml`     | dev profile에서 모든 서비스에 적용되는 공통 설정  |
| `application-prod.yml`    | prod profile에서 모든 서비스에 적용되는 공통 설정 |
| `{service-name}.yml`      | 특정 서비스에 적용되는 기본 설정                |
| `{service-name}-dev.yml`  | 특정 서비스의 dev profile 설정            |
| `{service-name}-prod.yml` | 특정 서비스의 prod profile 설정           |

---

## 작성 원칙

- `spring.application.name`과 설정 파일 이름을 일치시킵니다.
    - 예: `spring.application.name=user-service` → `user-service.yml`
- profile은 초기 단계에서 `dev`, `prod`만 사용합니다.
- `main` 브랜치의 변경은 직접 push하지 않고 PR을 사용합니다.
- 민감 정보는 설정 파일에 직접 작성하지 않습니다.
- 환경별로 달라지는 민감값은 환경변수 placeholder로 작성합니다.

    - 예시:
    ```yaml
    spring:
      datasource:
        username: ${USER_DB_USERNAME}
        password: ${USER_DB_PASSWORD}
    ```

---

## 설정 변경 반영

본 프로젝트는 설정 변경 사항을 수동 refresh 방식으로 반영합니다.

변경 사항을 반영하려면 대상 서비스의 `/actuator/refresh` endpoint를 호출합니다.

```bash
curl -X POST http://유저서비스IP주소/actuator/refresh
```

이를 위해 `dev` profile에서는 각 Config Client는 Actuator의 `health`, `refresh` endpoint를 노출합니다.

```yaml
management:
  endpoints:
    web:
      exposure:
        include:
          - health
          - refresh
```

`/actuator/refresh`는 설정 강제 갱신 endpoint입니다. 초기 개발 환경에서는 수동 refresh 테스트를 위해 endpoint를 노출하지만, 운영 환경에서는 인증 또는 내부망 접근 제한을 적용할 예정입니다.