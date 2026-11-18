# READMEweek02 — 다음 차례 2주차 실습 보고서

> 프로젝트: **다음 차례** — 받은 만큼 돌려주는 생일선물 장부  
> 주차: 2주차 / 총 4주  
> 범위: **1주차 미완료 작업 보완·테스트 + 2주차 신규 기능 구현**  
> 기술 스택: Java, Spring Boot 4.x, Spring Web MVC, Spring Data JPA, Hibernate, MySQL, Gradle, Jakarta Bean Validation

---

## 2. 1주차 미완료 작업 보완 및 실행

1주차 보고서에는 사용자·친구·선물 기록 API의 설계와 구현 내용이 정리되어 있지만, 테스트 확인 항목은 미체크 상태다. 따라서 2주차 신규 기능을 시작하기 전에 1주차 기능을 먼저 실행하고 누락된 부분을 보완한다.

### 2-1. 1주차 완료 대상

| 순서 | 기능 | Method / URL | 성공 코드 |
|---|---|---|---:|
| 1 | 사용자 가입 | `POST /api/users` | 201 |
| 2 | 친구 등록 | `POST /api/friends` | 201 |
| 3 | 친구 목록(생일순) | `GET /api/friends` | 200 |
| 4 | 선물 기록 등록 | `POST /api/friends/{friendId}/gifts` | 201 |
| 5 | 선물 이력 조회 | `GET /api/friends/{friendId}/gifts` | 200 |
| 6 | 선물 기록 삭제 | `DELETE /api/gifts/{giftId}` | 204 |

가입을 제외한 요청에는 `X-User-Id` 헤더를 전달한다. 이 헤더는 1주차 임시 사용자 구분 방식이며 실제 로그인 인증 방식은 아니다.

### 2-2. 프로젝트와 DB 실행 확인

1. IntelliJ에서 기존 프로젝트를 연다.
2. MySQL 서버를 실행한다.
3. Workbench에서 1주차 보고서의 `db/init.sql`을 실행해 `nextturn` 데이터베이스와 계정을 준비한다.
4. IntelliJ의 Run Configuration에 `DB_PASSWORD=실제비밀번호` 환경변수를 설정한다.
5. `DemoApplication`을 실행한다.
6. 콘솔에 애플리케이션 실행 완료 로그가 표시되는지 확인한다.
7. `http://localhost:8080`을 기준으로 API를 테스트한다.

실행이 안 된다면 먼저 JDK 버전, Gradle 동기화, DB 접속 정보, `application.properties` 또는 `application.yml` 설정을 확인한다.

### 2-3. 코드 구조 확인

1주차 보고서에서 제시한 기본 구조는 다음과 같다. 현재 프로젝트의 실제 패키지명과 파일명을 먼저 확인하고, 없는 파일만 추가한다.

```text
com.nextturn.demo
├─ DemoApplication.java
├─ user
│  ├─ User
│  ├─ UserRepository
│  ├─ UserService
│  ├─ UserController
│  └─ 사용자 요청·응답 DTO
├─ friend
│  ├─ Friend
│  ├─ FriendRepository
│  ├─ FriendService
│  ├─ FriendController
│  └─ 친구 요청·응답 DTO
└─ gift
   ├─ GiftRecord
   ├─ Direction
   ├─ GiftRecordRepository
   ├─ GiftRecordService
   ├─ GiftRecordController
   └─ 선물 요청·응답 DTO
```

각 계층의 역할을 확인한다.

| 계층 | 확인할 내용 |
|---|---|
| Entity | `users`, `friends`, `gift_records` 테이블과 매핑 |
| Repository | JPA를 이용한 데이터 저장·조회 |
| Service | 중복 확인, 소유권 확인, 비즈니스 규칙 |
| Controller | HTTP 요청·응답 처리 |
| DTO | 요청값 검증 및 응답 데이터 구성 |

### 2-4. 사용자 가입 API 확인·보완

**요청:** `POST /api/users`  
**성공:** `201 Created`

1주차 보고서에는 가입 API의 경로와 성공 코드만 명시되어 있고, 사용자 가입 요청 JSON의 필드명은 제시되어 있지 않다. 따라서 기존 `User` 엔티티와 가입 요청 DTO에 정의된 필드명을 그대로 사용한다. 새 필드명을 임의로 만들지 않는다.

확인할 구현 흐름:

1. `UserController`가 가입 요청 DTO를 받는다.
2. `@Valid`로 필수 입력값을 검증한다.
3. `UserService`가 닉네임 등 중복 여부를 확인한다.
4. `UserRepository`로 사용자를 저장한다.
5. 저장된 사용자 정보를 응답 DTO로 반환한다.
6. 정상 등록은 `201`, 닉네임 중복은 `409`, 입력 오류는 `400`인지 확인한다.

`test.http`에는 현재 가입 DTO에 맞는 실제 필드를 넣어 요청한다.

```http
### 1주차 - 사용자 가입
POST http://localhost:8080/api/users
Content-Type: application/json

{
  // 기존 User 가입 DTO의 필수 필드로 교체
}
```

위 JSON의 주석과 빈 본문은 실행용 데이터가 아니다. 기존 DTO의 필드명을 확인한 후 실제 값으로 채운다.

### 2-5. 친구 등록·목록 조회 API 확인·보완

**친구 등록:** `POST /api/friends` → `201 Created`  
**친구 목록:** `GET /api/friends` → `200 OK`

친구 등록 요청 예시(1주차 보고서 기준):

```http
### 1주차 - 친구 등록
POST http://localhost:8080/api/friends
Content-Type: application/json
X-User-Id: 1

{
  "name": "지현",
  "birthday": "2003-12-05",
  "memo": "대학 동기"
}
```

친구 목록 조회:

```http
### 1주차 - 친구 목록(생일순)
GET http://localhost:8080/api/friends
X-User-Id: 1
```

확인할 구현 흐름:

1. Controller가 `X-User-Id`와 친구 등록 DTO를 받는다.
2. Service가 요청 사용자를 확인한다.
3. 이름·생일 등 필수 입력값을 검증한다.
4. Friend 엔티티에 사용자 관계를 연결한다.
5. Repository로 저장한다.
6. 목록 조회는 현재 사용자 소유의 친구만 가져온다.
7. 생일이 가까운 순으로 정렬해 응답한다.

친구 등록 응답에는 보고서 예시처럼 `id`, `name`, `birthday`, `memo`, `daysUntilBirthday`가 포함되는지 확인한다. `daysUntilBirthday` 계산은 올해 생일이 지났다면 다음 해 생일을 기준으로 해야 한다.

### 2-6. 선물 기록 등록·조회·삭제 API 확인·보완

**등록:** `POST /api/friends/{friendId}/gifts` → `201 Created`  
**이력 조회:** `GET /api/friends/{friendId}/gifts` → `200 OK`  
**삭제:** `DELETE /api/gifts/{giftId}` → `204 No Content`

선물 기록 등록 요청 예시(1주차 보고서 기준):

```http
### 1주차 - 선물 기록 등록
POST http://localhost:8080/api/friends/1/gifts
Content-Type: application/json
X-User-Id: 1

{
  "direction": "RECEIVED",
  "itemName": "립밤 세트",
  "price": 15000,
  "category": "뷰티",
  "occasion": "BIRTHDAY",
  "giftDate": "2026-05-20"
}
```

선물 이력 조회:

```http
### 1주차 - 친구별 선물 이력 조회
GET http://localhost:8080/api/friends/1/gifts
X-User-Id: 1
```

선물 기록 삭제:

```http
### 1주차 - 선물 기록 삭제
DELETE http://localhost:8080/api/gifts/1
X-User-Id: 1
```

확인할 구현 흐름:

1. 등록할 때 친구가 실제로 존재하는지 확인한다.
2. 해당 친구가 요청 사용자의 소유인지 확인한다.
3. `direction`은 `RECEIVED` 또는 `GIVEN`만 허용한다.
4. 가격은 음수가 될 수 없도록 검증한다.
5. 등록한 기록은 해당 친구의 이력 조회에서 확인할 수 있어야 한다.
6. 삭제할 때 선물 기록이 존재하는지 확인한다.
7. 선물 기록이 연결된 친구의 소유자와 요청 사용자가 일치하는지 확인한다.
8. 삭제 성공은 응답 본문 없이 `204`를 반환한다.

### 2-7. 1주차 예외 테스트

아래 테스트는 실제 프로젝트에서 사용하는 필드와 ID에 맞춰 실행한다.

| 테스트 상황 | 기대 결과 |
|---|---:|
| 필수 이름 없이 친구 등록 | 400 |
| 음수 가격으로 선물 등록 | 400 |
| 허용되지 않은 `direction` 전송 | 400 |
| 존재하지 않는 친구 ID로 선물 등록·조회 | 404 |
| 다른 사용자의 친구 또는 선물 기록 접근 | 403 |
| 중복 닉네임으로 사용자 가입 | 409 |
| 정상적인 선물 기록 삭제 | 204 |

`test.http`에서 각 요청을 실행하고 실제 응답 코드와 본문을 확인한다. 테스트 결과가 다르면 Service의 검증·소유권 확인 로직과 예외 처리 방식을 확인한다.

### 2-8. 1주차 완료 체크리스트

- [ ] MySQL 연결 및 Spring Boot 실행
- [ ] `users`, `friends`, `gift_records` 테이블 확인
- [ ] 사용자 가입 API 실행
- [ ] 친구 등록 API 실행
- [ ] 친구 목록 조회 및 생일순 정렬 확인
- [ ] 선물 기록 등록 API 실행
- [ ] 선물 이력 조회 API 실행
- [ ] 선물 기록 삭제 API 실행
- [ ] 400 입력 검증 확인
- [ ] 403 소유권 검증 확인
- [ ] 404 대상 없음 확인
- [ ] 409 닉네임 중복 확인

**위 항목을 확인한 뒤에만 1주차 미완료 작업을 완료 처리한다.** 기존 구현이 이미 정상 동작하면 코드를 다시 작성하지 않고 테스트만 수행한다.

---

## 3. 2주차 신규 실습 시작 전 데이터 규칙

1주차 기능이 정상 동작하는 것을 확인한 후 2주차 기능을 진행한다.

## 13. 실습 1 — 친구별 선물 잔액 계산 API

### 3-1. 구현할 API

| 항목 | 내용 |
|---|---|
| Method | `GET` |
| URL | `/api/friends/{friendId}/balance` |
| Header | `X-User-Id: 사용자 ID` |
| 성공 | `200 OK` |
| 목적 | 특정 친구에게 받은 선물 금액과 준 선물 금액의 차이 계산 |

예시 응답:

```json
{
  "friendId": 1,
  "friendName": "지현",
  "receivedTotal": 30000,
  "givenTotal": 20000,
  "balance": 10000
}
```

### 3-2. Repository에 합계 쿼리 추가

기존 `GiftRecordRepository`를 열고 합계 조회 메서드를 추가한다. 이미 같은 기능의 메서드가 있다면 중복으로 만들지 않는다.

아래 코드는 엔티티 이름이 `GiftRecord`, 친구 연관 필드가 `friend`, 가격 필드가 `price`, 방향 필드가 `direction`인 경우의 예시다.

```java
public interface GiftRecordRepository
        extends JpaRepository<GiftRecord, Long> {

    @Query("""
        SELECT COALESCE(SUM(g.price), 0)
        FROM GiftRecord g
        WHERE g.friend.id = :friendId
          AND g.direction = :direction
    """)
    Long sumPriceByFriendIdAndDirection(
            @Param("friendId") Long friendId,
            @Param("direction") Direction direction
    );
}
```

필요한 import:

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
```

`COALESCE`는 조회 결과가 없을 때 `NULL` 대신 `0`을 반환한다. 따라서 선물 기록이 하나도 없는 친구도 계산할 수 있다.

> 엔티티에서 친구 필드가 `friend`가 아니거나 가격 필드명이 다르면 JPQL의 경로를 실제 필드명으로 변경한다. `Direction` enum의 패키지도 현재 프로젝트에 맞춘다.

### 3-3. 응답 DTO 만들기

`gift` 또는 기존 프로젝트의 DTO 패키지에 `BalanceResponse`를 추가한다.

```java
public record BalanceResponse(
        Long friendId,
        String friendName,
        Long receivedTotal,
        Long givenTotal,
        Long balance
) {}
```

### 3-4. Service에서 계산하기

기존 `FriendService` 또는 선물 관련 서비스의 책임 분리 방식에 맞춰 구현한다. 아래는 `FriendRepository`에서 친구를 조회하고 소유자를 확인하는 예시다.

```java
@Transactional(readOnly = true)
public BalanceResponse getBalance(Long userId, Long friendId) {
    Friend friend = friendRepository.findById(friendId)
            .orElseThrow(() -> new ResponseStatusException(
                    HttpStatus.NOT_FOUND, "친구를 찾을 수 없습니다."
            ));

    if (!friend.getUser().getId().equals(userId)) {
        throw new ResponseStatusException(
                HttpStatus.FORBIDDEN, "접근 권한이 없습니다."
        );
    }

    Long received = giftRecordRepository
            .sumPriceByFriendIdAndDirection(friendId, Direction.RECEIVED);

    Long given = giftRecordRepository
            .sumPriceByFriendIdAndDirection(friendId, Direction.GIVEN);

    return new BalanceResponse(
            friend.getId(),
            friend.getName(),
            received,
            given,
            received - given
    );
}
```

주의 사항:

- `friend.getUser()`는 1주차 소유권 관계를 가정한 예시다. 실제 사용자 연결 방식에 맞춰 확인한다.
- 기존 프로젝트에서 예외를 처리하는 방식이 따로 있다면 `ResponseStatusException` 대신 기존 예외 처리 규칙을 사용한다.
- 친구를 먼저 조회하고 소유권을 확인한 후 합계를 계산한다. 다른 사용자의 친구 ID를 이용해 데이터에 접근하지 못하게 해야 한다.

필요한 import 예시:

```java
import org.springframework.http.HttpStatus;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.server.ResponseStatusException;
```

### 3-5. Controller에 경로 추가

기존 `FriendController`에 추가한다. 클래스의 기본 경로가 `/api/friends`인 경우다.

```java
@GetMapping("/{friendId}/balance")
public BalanceResponse getBalance(
        @RequestHeader("X-User-Id") Long userId,
        @PathVariable Long friendId
) {
    return friendService.getBalance(userId, friendId);
}
```

필요한 import:

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestHeader;
```

### 3-6. 잔액 API 테스트

`test.http`에 추가한다.

```http
### 친구별 잔액 조회
GET http://localhost:8080/api/friends/1/balance
X-User-Id: 1
```

테스트 데이터 예시:

| 방향 | 가격 |
|---|---:|
| RECEIVED | 15,000 |
| RECEIVED | 15,000 |
| GIVEN | 20,000 |
| 결과 잔액 | **10,000** |

확인할 사항:

- [ ] 받은 선물 합계가 정확하다.
- [ ] 준 선물 합계가 정확하다.
- [ ] 잔액이 `받은 금액 - 준 금액`으로 계산된다.
- [ ] 기록이 없는 경우 합계가 0으로 나온다.
- [ ] 존재하지 않는 친구 ID는 404를 반환한다.
- [ ] 다른 사용자의 친구 ID는 403을 반환한다.

---

## 13. 실습 2 — 다가오는 생일 조회 API

### 4-1. 구현할 API

| 항목 | 내용 |
|---|---|
| Method | `GET` |
| URL | `/api/birthdays/upcoming` |
| Header | `X-User-Id: 사용자 ID` |
| Query | `days` — 조회 범위(일), 생략 시 30일 |
| 성공 | `200 OK` |

호출 예시:

```http
GET http://localhost:8080/api/birthdays/upcoming?days=30
X-User-Id: 1
```

응답 예시:

```json
[
  {
    "friendId": 1,
    "name": "지현",
    "birthday": "2003-12-05",
    "nextBirthday": "2026-12-05",
    "daysUntilBirthday": 17
  }
]
```

날짜는 실행 시점에 따라 달라진다. 위 날짜는 응답 형식을 설명하기 위한 예시다.

### 4-2. 응답 DTO 만들기

`BirthdayResponse`를 추가한다.

```java
public record BirthdayResponse(
        Long friendId,
        String name,
        LocalDate birthday,
        LocalDate nextBirthday,
        long daysUntilBirthday
) {}
```

### 4-3. Service에서 생일 계산하기

생일은 연도와 관계없이 매년 반복된다. 저장된 생일의 월·일을 기준으로 올해 생일을 만든다. 올해 생일이 오늘보다 이전이면 내년 생일을 사용한다.

```java
private LocalDate calculateNextBirthday(LocalDate birthday, LocalDate today) {
    LocalDate nextBirthday;

    try {
        nextBirthday = birthday.withYear(today.getYear());
    } catch (DateTimeException e) {
        // 윤년 2월 29일 생일은 평년에는 2월 28일로 처리
        nextBirthday = LocalDate.of(today.getYear(), 2, 28);
    }

    if (nextBirthday.isBefore(today)) {
        try {
            nextBirthday = birthday.withYear(today.getYear() + 1);
        } catch (DateTimeException e) {
            nextBirthday = LocalDate.of(today.getYear() + 1, 2, 28);
        }
    }

    return nextBirthday;
}
```

필요한 import:

```java
import java.time.DateTimeException;
import java.time.LocalDate;
import java.time.temporal.ChronoUnit;
```

이제 로그인한 사용자에게 등록된 친구 목록을 가져와 조회 범위 안에 있는 친구만 반환한다.

```java
@Transactional(readOnly = true)
public List<BirthdayResponse> getUpcomingBirthdays(Long userId, int days) {
    if (days < 1 || days > 365) {
        throw new ResponseStatusException(
                HttpStatus.BAD_REQUEST, "days는 1~365 사이여야 합니다."
        );
    }

    LocalDate today = LocalDate.now();

    return friendRepository.findAllByUserId(userId).stream()
            .map(friend -> {
                LocalDate next = calculateNextBirthday(
                        friend.getBirthday(), today
                );

                long remaining = ChronoUnit.DAYS.between(today, next);

                return new BirthdayResponse(
                        friend.getId(),
                        friend.getName(),
                        friend.getBirthday(),
                        next,
                        remaining
                );
            })
            .filter(item -> item.daysUntilBirthday() <= days)
            .sorted(Comparator.comparingLong(
                    BirthdayResponse::daysUntilBirthday
            ))
            .toList();
}
```

> `findAllByUserId`는 예시 메서드다. 현재 `FriendRepository`의 사용자별 친구 조회 메서드가 있다면 그것을 사용한다. 없다면 엔티티의 실제 사용자 연관 필드에 맞춰 추가한다. `birthday`가 nullable이 아니고 `LocalDate`인지도 확인한다.

필요한 import:

```java
import java.util.Comparator;
import java.util.List;
import org.springframework.http.HttpStatus;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.server.ResponseStatusException;
```

### 4-4. Controller 추가

기존 구조에 별도의 `BirthdayController`를 만든다. 친구 API와 분리하기 어려운 구조라면 기존 컨트롤러에 추가해도 된다.

```java
@RestController
@RequestMapping("/api/birthdays")
@RequiredArgsConstructor
public class BirthdayController {

    private final FriendService friendService;

    @GetMapping("/upcoming")
    public List<BirthdayResponse> getUpcomingBirthdays(
            @RequestHeader("X-User-Id") Long userId,
            @RequestParam(defaultValue = "30") int days
    ) {
        return friendService.getUpcomingBirthdays(userId, days);
    }
}
```

`@RequestParam`은 URL의 `?days=30` 값을 받는다.

### 4-5. 생일 조회 테스트

```http
### 30일 이내 생일 조회
GET http://localhost:8080/api/birthdays/upcoming?days=30
X-User-Id: 1

### 기본값(30일)으로 조회
GET http://localhost:8080/api/birthdays/upcoming
X-User-Id: 1

### 잘못된 조회 범위
GET http://localhost:8080/api/birthdays/upcoming?days=0
X-User-Id: 1
```

확인할 사항:

- [ ] 오늘 생일인 친구는 0일 후로 표시된다.
- [ ] 이미 올해 생일이 지난 친구는 내년 생일 기준으로 계산된다.
- [ ] 조회 범위 안의 친구만 반환된다.
- [ ] 남은 날짜가 적은 순서로 정렬된다.
- [ ] 다른 사용자의 친구는 조회되지 않는다.
- [ ] `days=0`, `days=366`은 400을 반환한다.
- [ ] 2월 29일 생일의 평년 처리 규칙이 일관된다.

---

## 13. 실습 3 — 선물 후보 데이터베이스 구축

### 5-1. 테이블 목적

`gift_catalog`은 친구에게 추천할 수 있는 선물 후보를 저장하는 테이블이다. 실제 쇼핑몰 API를 연결하지 않고, 이번 주차에는 직접 입력한 데이터를 사용한다.

예시 데이터:

| id | item_name | category | price |
|---:|---|---|---:|
| 1 | 핸드크림 세트 | 뷰티 | 18000 |
| 2 | 텀블러 | 생활 | 25000 |
| 3 | 디저트 세트 | 식품 | 22000 |
| 4 | 무선 충전기 | 디지털 | 30000 |

### 5-2. GiftCatalog 엔티티 만들기

`gift` 패키지 안에 `GiftCatalog`를 추가한다. 프로젝트의 기존 엔티티 작성 방식과 Lombok 사용 여부에 맞춰 생성자와 접근자를 구성한다.

```java
@Entity
@Table(name = "gift_catalog")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class GiftCatalog {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String itemName;

    @Column(nullable = false, length = 50)
    private String category;

    @Column(nullable = false)
    private Long price;

    @Column(length = 500)
    private String description;

    @Column(nullable = false)
    private boolean active = true;

    public GiftCatalog(
            String itemName,
            String category,
            Long price,
            String description
    ) {
        this.itemName = itemName;
        this.category = category;
        this.price = price;
        this.description = description;
    }
}
```

필요한 import:

```java
import jakarta.persistence.*;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;
```

### 5-3. Repository 만들기

```java
public interface GiftCatalogRepository
        extends JpaRepository<GiftCatalog, Long> {

    List<GiftCatalog> findAllByActiveTrue();

    List<GiftCatalog> findAllByCategoryAndActiveTrue(String category);
}
```

### 5-4. 조회 DTO 만들기

```java
public record GiftCatalogResponse(
        Long id,
        String itemName,
        String category,
        Long price,
        String description
) {}
```

### 5-5. Service 만들기

```java
@Transactional(readOnly = true)
public List<GiftCatalogResponse> getGiftCatalog(String category) {
    List<GiftCatalog> gifts;

    if (category == null || category.isBlank()) {
        gifts = giftCatalogRepository.findAllByActiveTrue();
    } else {
        gifts = giftCatalogRepository
                .findAllByCategoryAndActiveTrue(category);
    }

    return gifts.stream()
            .map(gift -> new GiftCatalogResponse(
                    gift.getId(),
                    gift.getItemName(),
                    gift.getCategory(),
                    gift.getPrice(),
                    gift.getDescription()
            ))
            .toList();
}
```

### 5-6. Controller 만들기

```java
@RestController
@RequestMapping("/api/gifts/catalog")
@RequiredArgsConstructor
public class GiftCatalogController {

    private final GiftCatalogService giftCatalogService;

    @GetMapping
    public List<GiftCatalogResponse> getCatalog(
            @RequestParam(required = false) String category
    ) {
        return giftCatalogService.getGiftCatalog(category);
    }
}
```

조회 예시:

```http
### 전체 선물 후보
GET http://localhost:8080/api/gifts/catalog

### 뷰티 카테고리만 조회
GET http://localhost:8080/api/gifts/catalog?category=뷰티
```

### 5-7. 초기 데이터 입력

개발용 초기 데이터는 SQL로 넣을 수 있다. 아래 SQL은 해당 테이블이 생성된 뒤 한 번 실행한다.

```sql
INSERT INTO gift_catalog
(item_name, category, price, description, active)
VALUES
('핸드크림 세트', '뷰티', 18000, '향이 좋은 핸드크림 세트', true),
('텀블러', '생활', 25000, '일상에서 사용하기 좋은 텀블러', true),
('디저트 세트', '식품', 22000, '간단하게 선물하기 좋은 디저트', true),
('무선 충전기', '디지털', 30000, '호환 기기를 위한 충전기', true);
```

`spring.jpa.hibernate.ddl-auto` 설정에 따라 테이블 생성 시점이 달라질 수 있다. SQL 실행 전에 `gift_catalog` 테이블이 실제로 생성됐는지 확인한다. 같은 SQL을 반복 실행하면 데이터가 중복될 수 있으므로 한 번만 실행하거나 중복 방지용 초기 데이터 방식을 사용한다.

확인할 사항:

- [ ] `gift_catalog` 테이블이 생성된다.
- [ ] 초기 데이터가 저장된다.
- [ ] 전체 후보 조회가 동작한다.
- [ ] 카테고리별 조회가 동작한다.
- [ ] `active=false`인 후보는 조회되지 않는다.

---

## 13. 실습 4 — 친구 취향 태그 설정

### 6-1. 테이블 목적

친구의 취향을 태그로 저장하면 3주차 선물 추천 점수 계산에 활용할 수 있다.

예시:

| friend_id | tag |
|---:|---|
| 1 | 뷰티 |
| 1 | 향수 |
| 1 | 캐릭터 |
| 2 | 운동 |
| 2 | 디지털 |

### 6-2. FriendTag 엔티티 만들기

```java
@Entity
@Table(
    name = "friend_tags",
    uniqueConstraints = {
        @UniqueConstraint(
            name = "uk_friend_tag",
            columnNames = {"friend_id", "tag"}
        )
    }
)
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class FriendTag {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "friend_id", nullable = false)
    private Friend friend;

    @Column(nullable = false, length = 50)
    private String tag;

    public FriendTag(Friend friend, String tag) {
        this.friend = friend;
        this.tag = tag;
    }
}
```

### 6-3. Repository 만들기

```java
public interface FriendTagRepository
        extends JpaRepository<FriendTag, Long> {

    List<FriendTag> findAllByFriendId(Long friendId);

    boolean existsByFriendIdAndTag(Long friendId, String tag);
}
```

### 6-4. 요청·응답 DTO 만들기

```java
public record FriendTagRequest(
        @NotBlank
        @Size(max = 50)
        String tag
) {}
```

```java
public record FriendTagResponse(
        Long id,
        String tag
) {}
```

요청 DTO에 필요한 import:

```java
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
```

### 6-5. Service 로직 작성

태그를 등록하기 전에 친구가 존재하는지, 요청 사용자의 친구가 맞는지 확인한다. 같은 친구에게 같은 태그를 중복 등록하지 않는다.

```java
@Transactional
public FriendTagResponse addTag(
        Long userId,
        Long friendId,
        FriendTagRequest request
) {
    Friend friend = friendRepository.findById(friendId)
            .orElseThrow(() -> new ResponseStatusException(
                    HttpStatus.NOT_FOUND, "친구를 찾을 수 없습니다."
            ));

    if (!friend.getUser().getId().equals(userId)) {
        throw new ResponseStatusException(
                HttpStatus.FORBIDDEN, "접근 권한이 없습니다."
        );
    }

    String tag = request.tag().trim();

    if (friendTagRepository.existsByFriendIdAndTag(friendId, tag)) {
        throw new ResponseStatusException(
                HttpStatus.CONFLICT, "이미 등록된 태그입니다."
        );
    }

    FriendTag saved = friendTagRepository.save(
            new FriendTag(friend, tag)
    );

    return new FriendTagResponse(saved.getId(), saved.getTag());
}
```

태그 목록 조회도 같은 방식으로 친구 소유권을 확인한 뒤 `findAllByFriendId` 결과를 DTO로 변환한다. 삭제 기능은 이번 주차 필수 범위에 포함하지 않아도 된다.

### 6-6. Controller 만들기

```java
@RestController
@RequestMapping("/api/friends/{friendId}/tags")
@RequiredArgsConstructor
public class FriendTagController {

    private final FriendTagService friendTagService;

    @PostMapping
    public FriendTagResponse addTag(
            @RequestHeader("X-User-Id") Long userId,
            @PathVariable Long friendId,
            @Valid @RequestBody FriendTagRequest request
    ) {
        return friendTagService.addTag(userId, friendId, request);
    }
}
```

태그 목록 조회는 다음 메서드를 같은 컨트롤러에 추가한다.

```java
@GetMapping
public List<FriendTagResponse> getTags(
        @RequestHeader("X-User-Id") Long userId,
        @PathVariable Long friendId
) {
    return friendTagService.getTags(userId, friendId);
}
```

### 6-7. 태그 API 테스트

```http
### 친구 취향 태그 등록
POST http://localhost:8080/api/friends/1/tags
Content-Type: application/json
X-User-Id: 1

{
  "tag": "뷰티"
}

### 친구 취향 태그 조회
GET http://localhost:8080/api/friends/1/tags
X-User-Id: 1
```

확인할 사항:

- [ ] 태그가 정상 등록된다.
- [ ] 등록된 태그 목록이 조회된다.
- [ ] 빈 문자열은 400을 반환한다.
- [ ] 같은 태그를 중복 등록하면 409를 반환한다.
- [ ] 존재하지 않는 친구는 404를 반환한다.
- [ ] 다른 사용자의 친구에게 태그를 등록하면 403을 반환한다.

---

## 13. 최종 API 명세 — 2주차 추가분

| 기능 | Method | URL | 성공 응답 |
|---|---|---|---|
| 친구별 잔액 조회 | GET | `/api/friends/{friendId}/balance` | 200 |
| 다가오는 생일 조회 | GET | `/api/birthdays/upcoming?days=30` | 200 |
| 선물 후보 전체 조회 | GET | `/api/gifts/catalog` | 200 |
| 선물 후보 카테고리 조회 | GET | `/api/gifts/catalog?category=뷰티` | 200 |
| 친구 취향 태그 등록 | POST | `/api/friends/{friendId}/tags` | 200 또는 201 |
| 친구 취향 태그 조회 | GET | `/api/friends/{friendId}/tags` | 200 |

모든 사용자 소유 데이터 API는 1주차와 동일하게 `X-User-Id` 헤더를 사용한다. 선물 후보 목록은 공용 카탈로그이므로 사용자 소유권 확인 대상이 아니다.

> 태그 등록 성공 코드는 기존 컨트롤러의 응답 규칙에 맞춘다. `201 Created`를 사용한다면 `ResponseEntity`로 상태 코드를 명시한다.

## 13. 전체 통합 테스트 체크리스트

### 잔액 계산

- [ ] 받은 선물 2건의 합계가 정확하다.
- [ ] 준 선물 1건의 합계가 정확하다.
- [ ] 받은 금액보다 준 금액이 많을 때 음수 잔액이 나온다.
- [ ] 기록이 없을 때 0원이 나온다.
- [ ] 다른 사용자 소유의 친구는 접근할 수 없다.

### 생일 조회

- [ ] 오늘 생일인 친구가 조회된다.
- [ ] 올해 생일이 지난 친구는 내년 날짜로 계산된다.
- [ ] 조회 범위에 해당하는 친구만 나온다.
- [ ] 생일이 가까운 순서로 정렬된다.
- [ ] 사용자별 데이터가 분리된다.

### 선물 후보

- [ ] 테이블과 초기 데이터가 생성된다.
- [ ] 전체 목록이 조회된다.
- [ ] 카테고리별 목록이 조회된다.
- [ ] 비활성 후보가 제외된다.

### 취향 태그

- [ ] 태그 등록과 조회가 된다.
- [ ] 빈 태그가 거절된다.
- [ ] 중복 태그가 거절된다.
- [ ] 소유권 검증이 동작한다.

## 13. 자주 발생할 수 있는 문제

| 문제 | 확인 및 해결 |
|---|---|
| JPQL에서 필드를 찾지 못함 | DB 컬럼명이 아니라 엔티티의 실제 필드명을 사용했는지 확인 |
| `SUM` 결과가 null | `COALESCE(SUM(...), 0)` 사용 |
| `findAllByUserId`를 찾지 못함 | `Friend` 엔티티의 사용자 연관 필드명에 맞춰 Repository 메서드 작성 |
| `friend.getUser()` 오류 | 실제 엔티티의 사용자 연결 필드명과 getter 확인 |
| 날짜 계산 오류 | `LocalDate`, `DateTimeException`, 윤년 처리 확인 |
| 테이블이 생성되지 않음 | 엔티티 패키지 위치, JPA 설정, DB 연결 확인 |
| Lombok 관련 컴파일 오류 | Lombok 의존성과 IntelliJ annotation processing 확인 |
| 중복 태그가 저장됨 | Service 중복 검사와 DB unique constraint를 함께 확인 |
| 다른 사용자의 친구 정보가 노출됨 | 조회·등록 전에 소유권 검증 확인 |

## 13. 2주차 완료 기준

아래 항목을 모두 확인한 뒤 완료로 표시한다.

- [ ] 기존 1주차 API가 계속 정상 동작한다.
- [ ] 잔액 계산 API가 정상 동작한다.
- [ ] 다가오는 생일 조회 API가 정상 동작한다.
- [ ] `gift_catalog` 테이블과 초기 데이터가 준비됐다.
- [ ] 선물 후보 조회 API가 정상 동작한다.
- [ ] `friend_tags` 테이블과 태그 등록·조회 API가 준비됐다.
- [ ] 입력 검증 및 소유권 검증을 확인했다.
- [ ] `test.http`의 2주차 테스트를 실행했다.
- [ ] 테스트 결과와 오류 해결 내용을 기록했다.

## 13. 2주차 실습 결과 보고

### 구현 결과

| 항목 | 결과 | 확인 |
|---|---|---|
| 잔액 계산 | 받은 금액 - 준 금액 | 미확인 |
| 생일 조회 | 조회 기간 내 생일순 정렬 | 미확인 |
| 선물 후보 DB | `gift_catalog` 및 초기 데이터 | 미확인 |
| 취향 태그 | `friend_tags` 등록·조회 | 미확인 |
| 통합 테스트 | 정상·예외 시나리오 | 미확인 |

### 실습 중 발생한 문제

- 문제:
- 원인:
- 해결 방법:

### 최종 소감 및 배운 점

이번 주차에는 단순히 데이터를 저장하는 기능에서 벗어나, 저장된 데이터를 집계하고 날짜를 계산해 서비스 기능으로 연결하는 과정을 실습했다. 또한 선물 후보와 친구 취향 태그를 별도 테이블로 관리해 다음 주차 추천 기능의 기반을 마련했다.

> 위 소감은 보고서 작성용 예시다. 실제로 수행한 내용과 느낀 점에 맞춰 수정한다.

## 13. 다음 주차 개발 계획

3주차에는 2주차에서 준비한 선물 후보와 취향 태그, 선물 잔액을 활용해 추천 기능을 구현한다.

- 친구의 취향 태그와 선물 카테고리 비교
- 잔액을 고려한 가격 적합도 계산
- 추천 점수 알고리즘 구현
- 동일 선물 중복 추천 방지
- 생일 D-day 스케줄러 및 테스트 코드 작성
