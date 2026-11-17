---
marp: true
theme: default
paginate: true
size: 16:9
style: |
  section {
    font-family: "Pretendard", "Noto Sans KR", sans-serif;
    padding: 48px 64px;
    font-size: 25px;
  }
  h1 { color: #2457A7; font-size: 2.0em; }
  h2 { color: #2457A7; }
  strong { color: #2457A7; }
  table { font-size: 0.72em; }
  section.lead { text-align: center; }
---

<!-- _class: lead -->

# 🎁 다음 차례

## 받은 만큼 돌려주는 생일선물 장부

친구의 생일과 주고받은 선물을 기록하고,
생일이 다가오면 답례 선물을 추천하는 백엔드 서비스

<br>

**작성자:** (이름 입력)  
**진행 상황:** 1주차 / 총 4주

---

# 1. 프로젝트를 기획한 이유

친구에게 생일선물을 받으면 고맙지만, 정작 그 친구의 생일이 다가오면 고민이 시작됩니다.

- 얼마짜리 선물을 받았는지 기억하기 어렵다.
- 받은 만큼 챙기려면 과거 기록을 찾아봐야 한다.
- 친구마다 주고받은 선물이 달라 답례 금액을 정하기 어렵다.

**다음 차례**는 친구별 선물 기록을 모아 이 고민을 줄이는 서비스입니다.

---

# 2. 핵심 기능

1. 친구의 **생일과 취향** 등록
2. 친구에게 받은 선물과 내가 준 선물 기록
3. 선물의 **가격·종류·날짜** 저장
4. 생일이 다가오면 받은 만큼에 맞는 답례 선물 추천

<br>

금융·쇼핑 API를 연결하지 않고, 모든 기록은 사용자가 직접 입력합니다.

---

# 3. 프로젝트의 핵심은 백엔드

화면 자체보다 서버의 데이터 처리와 계산 로직에 집중합니다.

- **잔액 계산:** 받은 금액과 준 금액 비교
- **추천 점수:** 선물 가격과 친구 취향을 반영
- **생일 스케줄러:** 다가오는 생일을 확인하고 알림·추천 준비

즉, 기록을 저장하는 데서 끝나지 않고 저장된 데이터를 활용하는 것이 핵심입니다.

---

# 4. 기술 스택

| 구분 | 사용 기술 |
|---|---|
| Language | Java |
| Framework | Spring Boot 4.x, Spring Web MVC |
| Data | Spring Data JPA (Hibernate), MySQL |
| Validation | Jakarta Bean Validation |
| Build | Gradle |
| 기타 | Lombok, IntelliJ IDEA |

---

# 5. 전체 개발 일정

| 주차 | 개발 내용 | 상태 |
|---|---|---|
| **1주차** | 기획 확정, ERD, API 명세, 프로젝트 세팅, 사용자·친구·선물 기록 API | **이번 발표** |
| 2주차 | 받은 금액 vs 준 금액 잔액 계산, 다가오는 생일 조회, 선물 후보 DB | 예정 |
| 3주차 | 추천 점수 알고리즘, 중복 방지, 생일 D-day 스케줄러, 테스트 코드 | 예정 |
| 4주차 | 간단한 화면, 배포, 최종 발표 | 예정 |

---

<!-- _class: lead -->

# 1주차 개발 결과

기획을 구체화하고 데이터 구조와 기본 API를 설계·구현했습니다.

---

# 6. ERD 설계

전체 **6개 테이블**로 설계했습니다.

| 테이블 | 역할 |
|---|---|
| `users` | 서비스 사용자 |
| `friends` | 사용자가 등록한 친구 |
| `gift_records` | 친구와 주고받은 선물 기록 |
| `friend_tags` | 친구의 취향 태그 |
| `gift_catalog` | 추천할 선물 후보 |
| `recommendations` | 친구별 선물 추천 이력 |

**1주차 구현:** `users`, `friends`, `gift_records`  
**2~3주차 구현 예정:** `friend_tags`, `gift_catalog`, `recommendations`

---

# 7. 테이블 관계

- 한 명의 사용자는 여러 친구를 등록할 수 있습니다.
- 한 명의 친구와 여러 선물을 주고받을 수 있습니다.
- 친구 한 명에게 여러 취향 태그와 추천 이력이 연결됩니다.
- 선물 후보는 여러 추천 이력에서 사용될 수 있습니다.

<br>

**관계 요약**

`users` 1 ─ N `friends` 1 ─ N `gift_records`

`friends` 1 ─ N `friend_tags`

`friends` 1 ─ N `recommendations` N ─ 1 `gift_catalog`

---

# 8. 프로젝트 구조

기능별 패키지를 나누고, 각 기능 안에서 역할을 분리했습니다.

```text
com.nextturn.demo
├─ DemoApplication.java
├─ user
│  └─ User / Repository / Service / Controller
├─ friend
│  └─ Friend / Repository / Service / Controller
└─ gift
   └─ GiftRecord / Direction / Repository
      Service / Controller
```

각 기능에는 요청·응답 DTO도 별도로 둡니다.

---

# 9. 서버 처리 구조

```text
클라이언트
    ↓
Controller
    ↓
Service
    ↓
Repository
    ↓
MySQL
```

| 계층 | 역할 |
|---|---|
| Controller | 요청 수신, `@Valid` 입력값 검증, Service 전달 |
| Service | 소유권 확인, 생일 계산 등 비즈니스 규칙, 트랜잭션 관리 |
| Repository | Spring Data JPA를 이용한 DB 접근 |

---

# 10. API 명세

가입을 제외한 모든 요청은 `X-User-Id` 헤더로 요청 사용자를 구분합니다.  
※ 1주차 임시 인증 방식입니다.

| 기능 | 메서드·경로 | 성공 |
|---|---|---|
| 사용자 가입 | `POST /api/users` | 201 |
| 친구 등록 | `POST /api/friends` | 201 |
| 친구 목록(생일순) | `GET /api/friends` | 200 |
| 선물 기록 | `POST /api/friends/{friendId}/gifts` | 201 |
| 선물 이력 조회 | `GET /api/friends/{friendId}/gifts` | 200 |
| 선물 기록 삭제 | `DELETE /api/gifts/{giftId}` | 204 |

주요 실패 응답: 입력 오류 `400`, 권한 없음 `403`, 대상 없음 `404`, 닉네임 중복 `409`

---

# 11. API 요청·응답 예시: 친구 등록

**요청** `POST /api/friends`

```json
{
  "name": "지현",
  "birthday": "2003-12-05",
  "memo": "대학 동기"
}
```

**응답** `201 Created`

```json
{
  "id": 1,
  "name": "지현",
  "birthday": "2003-12-05",
  "memo": "대학 동기",
  "daysUntilBirthday": 24
}
```

친구 등록 후 다음 생일까지 남은 일수도 함께 반환합니다.

---

# 12. API 요청·응답 예시: 선물 기록

**요청** `POST /api/friends/1/gifts`

```json
{
  "direction": "RECEIVED",
  "itemName": "립밤 세트",
  "price": 15000,
  "category": "뷰티",
  "occasion": "BIRTHDAY",
  "giftDate": "2026-05-20"
}
```

**응답** `201 Created`

```json
{
  "id": 1,
  "direction": "RECEIVED",
  "itemName": "립밤 세트",
  "price": 15000,
  "category": "뷰티",
  "occasion": "BIRTHDAY",
  "giftDate": "2026-05-20"
}
```

받은 선물인지(`RECEIVED`), 준 선물인지(`GIVEN`) 구분해 저장합니다.

---

# 13. 주요 구현 포인트

- **생일 계산:** 올해 생일이 지났다면 내년 생일 기준으로 계산
- **생일순 정렬:** 친구 목록을 생일이 가까운 순으로 정렬
- **소유권 검증:** 다른 사용자의 데이터 접근 시 `403` 반환
- **입력 검증:** 이름 누락, 음수 가격, 잘못된 `direction` 등을 `400`으로 처리
- **DTO 분리:** 엔티티를 직접 노출하지 않고 요청·응답용 `record` 사용
- **비밀번호 관리:** `DB_PASSWORD` 환경변수로 주입해 저장소에 노출하지 않음

---

# 14. 현재 한계와 보완 계획

현재 `X-User-Id` 헤더는 누구나 값을 바꿔 보낼 수 있는 **임시 인증 방식**입니다.

따라서 실제 서비스 수준의 인증·보안을 위해 이후 로그인과 토큰 인증으로 교체할 계획입니다.

또한 1주차에는 기본 데이터 구조와 기록 API 구현에 집중했고, 추천 기능에 필요한 나머지 테이블은 다음 주차에 개발합니다.

---

# 15. 실행 방법

1. **MySQL 준비:** Workbench에서 `db/init.sql` 실행 → `nextturn` DB와 계정 생성
2. **환경변수 설정:** 실행 환경에 `DB_PASSWORD=비밀번호` 추가
   - IntelliJ: Run Configuration → Environment variables
3. **서버 실행:** `DemoApplication` 실행
   - 서버 주소: `http://localhost:8080`
   - JPA가 테이블 자동 생성
4. **API 테스트:** `test.http` 요청을 위에서부터 실행

---

# 16. 테스트 계획 및 확인 항목

`test.http`로 다음 시나리오를 확인합니다.

| 시나리오 | 기대 결과 |
|---|---|
| 사용자 가입 | 201, `id` 발급 |
| 친구 등록 → 목록 조회 | 201, 생일 가까운 순 정렬 |
| 받은 선물 기록 → 이력 조회 | 201, 최신순 조회 |
| 선물 기록 삭제 | 204 |
| 이름 없이 친구 등록 | 400 |
| 없는 친구 번호로 요청 | 404 |
| 다른 사용자가 남의 친구 조회 | 403 |

※ 원본 문서에서 테스트 확인 상태는 미체크(☐)로 표시되어 있습니다.

---

# 17. 트러블슈팅

| 문제 | 원인 및 해결 |
|---|---|
| Spring Initializr에 Boot 3.x가 없음 | 최신 4.x 선택 |
| Gradle 동기화 실패·실행 버튼 미표시 | `build.gradle`의 Java Toolchain 버전을 설치된 JDK에 맞추고 재동기화 |
| 직접 만든 패키지가 인식되지 않을 위험 | `user`, `friend`, `gift`를 `DemoApplication` 하위 패키지에 생성 |
| 테이블 이름이 ERD와 다르게 생성 | `@Table(name = "friends")`로 테이블명 명시 |

---

<!-- _class: lead -->

# 다음 주 개발 계획

## 2주차: 기록을 계산과 추천으로 연결하기

- 친구별 **받은 금액 − 준 금액** 잔액 계산 API
  - `GET /api/friends/{id}/balance`
- **다가오는 생일** 조회 API
  - `GET /api/birthdays/upcoming`
- 추천용 **선물 후보 DB**(`gift_catalog`) 구축
- 친구 **취향 태그** 설정 API

<br>

# 감사합니다
