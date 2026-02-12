# 🍃 Spring Boot JWT Boilerplate

> **Spring Boot 3 + Spring Security + JWT + Redis + JPA**
> 포트폴리오 및 사이드 프로젝트의 빠른 시작을 위한 백엔드 보일러플레이트입니다.

<br>

## ✨ Key Features

- **Authentication**: JWT (Access Token, Refresh Token) 기반 인증
- **Security**: HttpOnly Cookie를 사용한 Refresh Token 저장 (XSS/CSRF 방지)
- **Token Rotation**: Redis를 이용한 Refresh Token 관리 및 로그아웃 처리
- **Unified Response**: `ApiResponse<T>`를 통한 일관된 JSON 응답 포맷
- **Error Handling**: `GlobalExceptionHandler`와 `ErrorCode`를 통한 전역 예외 처리
- **Logging**: Logback 설정 (Console + File Rolling) 및 요청/SQL 로깅
- **Validation**: `@Valid` 및 DTO 검증 자동화

<br>

## 🛠 Tech Stack

- **Java**: 21 (LTS)
- **Framework**: Spring Boot 3.5.x
- **Database**: MySQL (Prod), H2 (Test), Redis (Token Store)
- **ORM**: Spring Data JPA
- **Build Tool**: Gradle
- **Dependencies**: Spring Security, JJWT, Lombok, Validation

<br>

## 📂 Project Structure

    src/main/java/com/example/spring_boot_jwt_boilerplate
    ├── config              # Security, JWT, WebMvc 설정
    ├── controller          # API 엔드포인트 (Auth 등)
    ├── domain              # JPA Entity (Member, BaseTimeEntity)
    ├── dto                 # 데이터 전송 객체 (Request/Response)
    │   ├── auth            # 로그인, 회원가입 DTO
    │   └── common          # 공통 응답 규격 (ApiResponse)
    ├── exception           # 전역 예외 처리 (GlobalExceptionHandler, ErrorCode)
    ├── repository          # DB 데이터 접근 계층
    └── service             # 비즈니스 로직

<br>

---

## 🚀 Getting Started

### 1. Prerequisites
- **Java 21** 이상
- **Redis** (실행 필수)
- **MySQL** (선택, 기본 설정은 `.env`에서 관리)

### 2. Installation

    $ git clone [https://github.com/YOUR_USERNAME/spring-boot-jwt-boilerplate.git](https://github.com/YOUR_USERNAME/spring-boot-jwt-boilerplate.git)
    $ cd spring-boot-jwt-boilerplate

### 3. Environment Variables (.env)
프로젝트 루트 경로에 `.env` 파일을 생성하고 아래 내용을 입력하세요.

    # Database
    DB_HOST=localhost
    DB_PORT=3306
    DB_NAME=mydb
    DB_USERNAME=root
    DB_PASSWORD=your_password

    # Redis
    REDIS_HOST=localhost
    REDIS_PORT=6379
    REDIS_PASSWORD=

    # JWT (Base64 Encoded Secret Key recommended)
    JWT_SECRET=your_super_secret_key_should_be_long_enough_for_security_purposes

### 4. Run

    $ ./gradlew bootRun

---

<br>

## 📝 API Documentation

| Method | Endpoint | Description | Auth |
| :--- | :--- | :--- | :---: |
| `POST` | `/api/auth/signup` | 회원가입 | ❌ |
| `POST` | `/api/auth/login` | 로그인 (Access Token 반환, RT는 쿠키) | ❌ |
| `POST` | `/api/auth/reissue` | 토큰 재발급 (Refresh Token 쿠키 필수) | ❌ |
| `GET` | `/api/auth/check-email` | 이메일 중복 확인 | ❌ |
| `GET` | `/api/auth/check-nickname` | 닉네임 중복 확인 | ❌ |
| `GET` | `/api/auth/test` | JWT 인증 테스트 | ✅ |

---


<br>

## 📦 Response Format

모든 API는 아래와 같은 통일된 포맷으로 응답합니다.

**1. 성공 (Success)**

    {
      "success": true,
      "message": "요청 성공",
      "data": {
        "accessToken": "eyJhGci...",
        "nickname": "dev_user"
      }
    }

**2. 실패 (Fail - Error)**

    {
      "success": false,
      "message": "이미 사용 중인 이메일입니다.",
      "data": null
    }

**3. 유효성 검증 실패 (Validation Fail)**

    {
      "success": false,
      "message": "입력값이 올바르지 않습니다.",
      "data": {
        "email": "이메일 형식이 올바르지 않습니다.",
        "password": "비밀번호는 필수 입력값입니다."
      }
    }

<br>

---
<br>

## 🛠 How to Use Custom Exception (Service Layer)

비즈니스 로직 중 발생하는 예외는 `CustomException`을 던져서 처리합니다. 별도의 `try-catch` 없이 예외를 던지면 `GlobalExceptionHandler`가 이를 감지하여 표준화된 에러 응답을 반환합니다.

### 1. 예외 던지기 (Throwing)
서비스 로직에서 특정 조건에 맞지 않을 경우 `ErrorCode`에 정의된 에러 코드를 인자로 사용하여 예외를 발생시킵니다.

    @Service
    public class AuthService {
        public void signup(SignupRequest request) {
            // 이메일 중복 시 CustomException 발생
            if (memberRepository.existsByEmail(request.getEmail())) {
                throw new CustomException(ErrorCode.EMAIL_DUPLICATION);
            }
        }
    }

### 2. 에러 코드 정의 (ErrorCode Enum)
새로운 에러 상황이 필요하면 `exception/ErrorCode.java`에 상태 코드와 메시지를 추가하세요.

    EMAIL_DUPLICATION(HttpStatus.CONFLICT, "이미 사용 중인 이메일입니다."),
    USER_NOT_FOUND(HttpStatus.NOT_FOUND, "사용자를 찾을 수 없습니다."),
    LOGIN_FAILURE(HttpStatus.UNAUTHORIZED, "아이디 또는 비밀번호가 잘못되었습니다.");

### 3. 예외 처리 흐름도



1.  **Service**: `throw new CustomException(ErrorCode.MEMBER_NOT_FOUND)`
2.  **GlobalExceptionHandler**: 예외 낚아채기 (`@ExceptionHandler`)
3.  **ApiResponse**: 에러 정보를 담은 표준 JSON 객체 생성
4.  **Client**: HTTP 404 상태 코드와 함께 에러 메시지 수신

---

<br>

## 🔐 Core Technical Implementation

### 1. Redis Token Management (RT Storage)
보안 강화를 위해 Refresh Token(RT)은 클라이언트의 쿠키와 서버 측 Redis에 이중으로 관리됩니다.

    # Redis 저장 형식
    Key: "RT:{email}"
    Value: "{refreshToken}"
    Expiration: ${jwt.refresh-token-expiration} (현재 설정: 1시간)

- **Token Expiration Policy**:
    - **Access Token**: 1분 (`60,000ms`) - 빈번한 재발급을 통한 보안 강화
    - **Refresh Token**: 1시간 (`3,600,000ms`) - Redis 및 쿠키 만료 시간 동기화
- **Reissue Process**:
    1. 쿠키로 전달된 RT의 유효성 검증
    2. Redis에 저장된 RT와 클라이언트의 RT 일치 여부 확인 (이중 검증)
    3. 검증 성공 시 **Token Rotation** 적용 (AT, RT 둘 다 재발급하여 Redis 갱신 및 쿠키 업데이트)

### 2. JPA Data Integrity & Automation
- **JPA Auditing**: `@EnableJpaAuditing` 설정을 통해 모든 엔티티의 생성 시간과 수정 시간을 자동으로 기록합니다. (`BaseTimeEntity` 상속)
- **Dirty Checking**: `@Transactional` 범위 내에서 엔티티 객체의 필드 값만 변경하면, 별도의 `save()` 호출 없이도 트랜잭션 종료 시점에 DB에 변경 사항이 자동으로 반영됩니다.

<br>

## ⚙️ Configuration Details (application.yaml)

프로젝트의 핵심 설정 값은 `src/main/resources/application.yaml`에서 관리됩니다.

    jwt:
      secret: ${JWT_SECRET}
      access-token-expiration: 60000      # 1분 (테스트 및 보안용)
      refresh-token-expiration: 3600000   # 1시간 (Redis 저장 및 쿠키 만료 시간)

> **Tip**: 실서비스 운영 시에는 Access Token은 30분~1시간, Refresh Token은 7일~14일 정도로 길게 설정하는 것을 권장합니다.
---
<br>

### 3. Logging Strategy (Logback)
애플리케이션의 흐름을 추적하고 문제 발생 시 디버깅을 용이하게 하기 위해 상세한 로깅 정책을 적용했습니다.

- **Log File Structure**:
  - **Active Log**: `./logs/application.log` - 현재 실행 중인 서버의 로그가 계속 기록됩니다.
  - **Archive Logs**: `./logs/archive/application-yyyy-MM-dd-HH.i.log` - 1시간 단위로 쪼개져 저장된 과거 로그입니다.
- **Retention Policy**:
  - **Size-based**: 로그 파일 하나가 10MB를 초과하면 인덱스(`.i`)를 증가시키며 새로 생성합니다.
  - **Time-based**: 최대 30일간 보관하며, 전체 로그 크기가 3GB를 넘지 않도록 관리합니다.
- **Log Levels**:
  - **Console**: ANSI 컬러 패턴을 적용하여 가독성을 높였습니다. (`DEBUG` 레벨까지 노출)
  - **File**: 실서버 운영을 고려하여 색상 코드를 제거한 순수 텍스트 포맷으로 저장합니다.
- **SQL Logging**: `jdbc.sqlonly` 설정을 통해 실행되는 SQL 쿼리를 실시간으로 모니터링할 수 있습니다.

---

<br>

## 🤝 Contribution

1. Fork this repository
2. Create your Feature Branch (`git checkout -b feature/NewFeature`)
3. Commit your changes (`git commit -m 'Add NewFeature'`)
4. Push to the Branch (`git push origin feature/NewFeature`)
5. Open a Pull Request

---
<br>

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
