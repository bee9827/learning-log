# 학습 로그 (Learning Log)

우테코 여러 프로젝트에 걸쳐 흩어져 있던 학습 로그를 한 곳에 모아 관리한다.
기술 딥다이브와 사이클별 학습법 회고("바꿀 것")가 프로젝트를 넘어 한 줄로 이어진다.

## 왜 이 저장소를 운영하나

### 1. 재미를 잃었다

대학교 4학년, 팀 프로젝트와 취업 준비를 위해 주입식으로 공부하며 개발의 재미를 잃었다. "이 일을 평생 할 수는 없겠다"는 생각이 들었고, **재미를 되찾기 위해** 우아한테크코스 프리코스에 지원했다.

### 2. 우테코 안에서도 같은 문제를 만났다 — 이번엔 AI였다

- **Level 1**: AI가 학습을 방해할 것 같아 아예 쓰지 않았다.
- **Level 2**: 생각을 바꿔 AI를 써 봤다. 어느새 AI에게 다 맡기고 주도권을 빼앗겨 끌려가고 있었다. 코드를 이해하지 못한 채 작성만 했고, 다시 흥미를 잃어 갔다.

AI를 배제하는 것도, AI에 맡기는 것도 답이 아니었다. 본래 목적으로 돌아가 **"AI를 쓰면서도 주도권을 내가 쥐고, 더 효율적으로 학습하려면 어떻게 해야 하나"** 를 고민했다. 이 저장소는 그 시도의 기록이다.

### 3. PiFSR과 나선형 학습

**출처**

- **PiFSR — 효과적인 학습 5원칙.** 우아한테크코스 8기 프론트엔드 코치 시지프의 레벨1 수업 자료로, 백엔드 레벨2 학습법 미션에서 소개되었다.
  - **P**roblem-Driven — 목적과 문제에서 시작한다
  - **i+1** — 문제를 쪼개고 딱 한 단계 어려운 것에 도전한다 (원 개념: Stephen Krashen의 입력 가설)
  - **F**eedback Loop — Do → Fail → Feedback → Iterate, Output First
  - **S**tructural Thinking — 조각이 아니라 흐름·관계·역할로 연결한다
  - **R**esourceful — 책·사람·AI 등 모든 자원을 비순차적으로 활용한다
- **나선형 학습(Spiral Learning).** 같은 주제를 시간 간격을 두고 다시 만나되 매번 더 깊고 넓게 다루는 방식. Jerome Bruner가 *The Process of Education*(1960)에서 제안한 나선형 교육과정에서 왔다. 핵심은 같은 수준의 **반복(Repetition)** 이 아니라 한 층 위에서 다시 만나는 **반복적 심화(Iteration)** 다.

**내 해석과 적용**

- PiFSR은 원래 내가 공부하던 방식과 거의 같았다. 하나를 **깊게** 파고드는 데 강하지만, 범위가 넓고 제한 시간이 있는 Spring 미션에서는 한 곳만 깊이 팔 수 없었다.
- 그래서 둘을 이렇게 나눠 이해한다: **나선형 학습은 최소치를 끌어올리고, PiFSR은 깊이를 더한다.** 넓게 한 바퀴 돌며 최소치를 확보하고, 재방문할 때마다 PiFSR로 한 층 더 들어간다. 이 저장소의 모든 로그는 그 둘을 함께 적용한 기록이다.

| 원칙 | 이 저장소에서 실제로 하는 것 |
|---|---|
| Problem-Driven | 막힌 지점을 한 문장의 문제로 정의하고 시작한다 |
| i+1 | 어려우면 문제를 분할해 한 단계로 만든다. 다음 키워드도 "지금보다 딱 한 단계"(키워드 이해 → 흐름 파악 → 코드 적용 → 실전 판단) |
| Feedback Loop | 로그 작성 뒤 퀴즈로 체득을 점검하고, 사이클마다 "다음에 바꿀 것"을 회고한다 |
| Structural Thinking | 정답을 받아 적지 않고 후보를 스스로 탈락시키며, 시간 흐름(서사)으로 묶어 내 말로 정리한다 |
| Resourceful | 공식 문서·코드·테스트·AI를 모두 쓴다 — 단, AI는 정답 공급원이 아니라 **사고를 확장하는 도구**로 |

예(Iteration): 락 — #03 INSERT 경쟁 → #13 S/X락·데드락 → #17 갭락 → #26 INSERT 락 순서 → #29 갭락 재방문 → #56 실제 경합 증명

### 4. 재미가 돌아온 순간

학습 로그를 도입하고, AI에게 **정답을 달라고 하는 대신 내 사고를 확장하는 도구로** 쓰기 시작한 순간부터 다시 재밌어졌다. AI에게 끌려가지 않으려고 만든 규칙:

- **추측 먼저, 설명은 나중** — 모르는 개념도 먼저 추측한다. 틀린 추측도 연결고리를 만든다.
- **개념 먼저, 코드 나중** (#52) — 문제 상황만 받고, 내가 개념 수준의 해법을 말한 뒤에야 이름과 문법을 받는다.
- **구현 전 합의 게이트** (#59) — 새로 등장할 개념을 먼저 목록으로 깔고, 토론이 수렴한 뒤에만 구현한다.
- **유도 질문은 전제부터 선언** (#56~57) — 전제가 무너지면 논증을 버리고 증거(현장 기록)로 돌아간다.

### 5. 결과 — 학습이 코드로 이어진다

| 로그 | 실제 코드 변경 ([spring-roomescape-waiting](https://github.com/bee9827/spring-roomescape-waiting)) |
|---|---|
| #47 상태 CAS(낙관적 직렬화) | [`3e31f6dc`](https://github.com/bee9827/spring-roomescape-waiting/commit/3e31f6dc) 결제 수렴 상태 전이를 CAS로 직렬화 |
| #48 Saga 보상 | [`a87e13ae`](https://github.com/bee9827/spring-roomescape-waiting/commit/a87e13ae) 승인 후 예약 실패 시 환불 보상 |
| #50·#52 Rate Limit | [`ddeb67bc`](https://github.com/bee9827/spring-roomescape-waiting/commit/ddeb67bc) 토큰 버킷 + 토스 429 백오프 |
| #56 실제 경합 증명 | 증명 테스트 중 gap 락 데드락 결함 발견 → [`ebb2e35a`](https://github.com/bee9827/spring-roomescape-waiting/commit/ebb2e35a) 수정 |
| #61 패키지 사이클 | [`48e62568`](https://github.com/bee9827/spring-roomescape-waiting/commit/48e62568) DIP로 사이클 제거 + [`11c46d74`](https://github.com/bee9827/spring-roomescape-waiting/commit/11c46d74) ArchUnit 가드 |

> 📍 카테고리별 **지향점·저울 지도**: [CATEGORIES.md](CATEGORIES.md)

## 운영 방식

- **PR**: 큰 카테고리·학습 arc마다 브랜치와 PR 하나를 만들고, 그 순회를 마칠 때까지 같은 PR을 갱신한 뒤 `master`에 병합한다. 로그마다 새 PR을 만들지 않는다.
- **로그·커밋**: `logs/NN_핵심키워드.md` — 나중에 독립적으로 재방문할 수 있는 개념 묶음 하나. 하루나 원문 하나와 번호를 1:1로 맞추지 않으며, 한 번의 탐구에서 여러 로그가 나올 수 있다.
- **폭 우선**: 자료 순서로 넓게 이동하되 현재 이해에 필요한 인과는 그 자리에서 닫는다. 더 깊은 인접 주제는 씨앗으로 남겨 다음 재방문에서 발전시킨다.
- **다음 키워드 관리**: [`BACKLOG.md`](BACKLOG.md) — 재방문 대기(열림)/완료(닫힘) 체크리스트. *다음에 뭘 학습할지는 여기서 고른다.*
- **로그 작성 전**: 오늘 키워드를 이 repo에서 먼저 검색한다. 재방문이면 깊이를 더하고(중복 금지), 이전이 얕았으면 명시한다(피드백 루프).

> 옵시디언 그래프(위키링크/태그)로 키워드를 관리하려다 — 유지 부담이 크고 효용은 "구경거리"에 가까워 **과감히 접었다.** 키워드 관리는 BACKLOG.md 하나로 충분하다. (학습 시스템은 학습을 위한 *수단*이지 목적이 아니다.)

## 프로젝트 매핑

| 로그 범위 | 프로젝트 |
|---|---|
| 01 ~ 13 | spring-roomescape-member |
| 14 ~ 22 | spring-roomescape-auth |
| 23 ~ 34 | spring-roomescape-waiting |
| 35 ~ 50 | spring-roomescape-waiting — 결제(Toss) 연동 |
| 51 | 일반 CS (JVM/런타임) — 프로젝트 외, 미러 제외 |
| 60 | 일반 CS (네트워크) — 프로젝트 외, 미러 제외 |
| 61 | spring-roomescape-waiting — 패키지 사이클 끊기·ArchUnit 가드 (미러 O) |
| 62 | 2026-Mapmory — Testcontainers worker JVM 공유와 테스트 성능 개선 |
| 63 ~ 67 | 일반 CS (컴퓨터 구조에서 출발한 실행 흐름·가상 메모리·스레드 실행 상태) — 프로젝트 외, 미러 제외 |
| 68 | java-http — Cookie와 JSESSIONID 발급, 세션 식별자와 로그인 상태 구분 |
| 69 ~ 70 | java-http — Coyote·Catalina·Servlet 경계와 어노테이션 기반 라우트 등록 |
| 71 | 일반 CS (JVM/동시성) — Thread 미션 코드와 연결 |
| 72 | 일반 CS (동시성/비동기) — 호출·반환·작업 완료의 순서 |
| 73 | java-mvc — Stream 내부 반복·순수 변환·병렬 분할과 HandlerMapping 등록 경계 (미러 O) |
| 74 | 일반 CS (네트워크/JVM 동시성) — 소켓 읽기·가상 스레드 재개 (미러 제외) |
| 75 ~ 76 | 일반 CS (CPU 캐시·Java 가시성) — 프로젝트 외, 미러 제외 |

> 파일명: `NN_핵심키워드.md` (번호 앞 → 시간순 정렬·상호참조 안정, 키워드 → 그래프 가독성)

## 인덱스

| # | 주제 |
|---|---|
| 01 | Content-Type |
| 02 | Controller 테스트 작성 |
| 03 | 동시성 — INSERT 경쟁 조건과 해결 전략 |
| 04 | JPA 영속성 / @Transactional |
| 05 | 트랜잭션 격리 수준 (Isolation Level) |
| 06 | MVCC / 낙관적 락 / 비관적 락 선택 기준 |
| 07 | @Transactional(readOnly = true) 실제 효과 |
| 08 | JDBC 낙관적 락 직접 구현 |
| 09 | ThreadLocal과 트랜잭션 커넥션 바인딩 |
| 10 | RFC 7807 / ProblemDetail 에러 응답 |
| 11 | Spring 요청 처리 흐름 — 타입 변환과 @Valid의 관계 |
| 12 | Undo Log — 롤백, MVCC, Lost Update |
| 13 | S락/X락, 데드락, INSERT 데드락, SELECT FOR UPDATE |
| 14 | 도메인 예외 설계, HTTP 의존성 분리 |
| 15 | Filter vs Interceptor, OncePerRequestFilter |
| 16 | 세션 vs 토큰 방식 인증 |
| 17 | InnoDB 갭락, Insert Intention Lock, Next-Key Lock |
| 18 | 동시 로그인 방지, Session 관리 구조, JWT 무효화 한계 |
| 19 | Session 전체 흐름 (JSESSIONID, Tomcat 관리) |
| 20 | EventListener, 옵저버 패턴, Java GC 도달 가능성 |
| 21 | PR 리뷰 반영, Filter와 DispatcherServlet 경계 |
| 22 | JWT 구조와 동작, 세션 vs JWT 트레이드오프 |
| 23 | 객체지향 리팩토링 — Slot 값 객체 추출 |
| 24 | 도메인 리팩토링 — WaitingService 규칙의 도메인 이전 |
| 25 | WaitingService 조회 단순화 — DB에 숨은 로직을 도메인으로 |
| 26 | INSERT 락 순서 — intention/record/유니크 S락/데드락 |
| 27 | 트랜잭션 경계 롤백 테스트 — MockitoSpyBean, Mock vs Spy |
| 28 | @Scheduled 워커 + 아웃박스 패턴, 최종 일관성·멱등성 |
| 29 | 갭락 재방문 — MVCC와의 관계, SELECT FOR UPDATE 스냅샷 우회 |
| 30 | ACID 지향점 메타 복습 — DB·트랜잭션 로그를 unique로 한 바퀴 꿰기 |
| 31 | Saga 패턴 — 보상 트랜잭션, Outbox와의 관계, rollback vs compensation |
| 32 | 인증 & 세션 지향점 — stateless 위에 신원 얹기, 세션 vs JWT, 확장성↔무효화 |
| 33 | 복잡한 모델링 — 책임의 위치(정보 전문가), 묶기↔나누기, 행 간 불변식→트랜잭션 |
| 34 | 테스트 — 격리↔실제성 저울, 테스트가 트랜잭션을 쥐면 경계가 가려진다 |
| 35 | 토스 결제 연동 — 인증 vs 승인, 왜 서버가 confirm을 호출하나 |
| 36 | confirm 멱등성 — 재시도 안전성, paymentKey vs Idempotency-Key |
| 37 | 결제 연동 코드 적용 — 아키텍처 추측 인출(엔드포인트·PENDING·포트&어댑터) |
| 38 | ACL은 원칙이 아니라 수단 — 그 "선"은 스펙으로 확인한다 |
| 39 | 설정 외부화 + 프로파일 머지 — 토스 v2 confirm 401(빈 시크릿) 디버깅 |
| 40 | 패키지 구조 — package-by-feature vs by-layer, 기능 경계·의존 방향 |
| 41 | 아키텍처 가드(ArchUnit) — 왜 자동 강제가 필요한가 |
| 42 | 사이클 끊기는 도구가 다르다 + 책임을 제자리에(동기→중립 제3자) |
| 43 | 예약↔결제 API 분리 + default-deny 인증 + 중복 주문 3중 방어(멱등 재방문) |
| 44 | 타임아웃 방어 — connect/read 두 단계, read=모름, 멱등 두 겹(재방문) |
| 45 | 토스 step2 완성 — 멱등키 전용컬럼·read 서비스 분리·책임 분리(코드 적용) |
| 46 | reconciliation — 불명확을 조회로 수렴(아웃박스 폴링) + 멱등을 락 아닌 낙관 가드로 |
| 47 | 앱 가드는 동시성에 못 닫힌다 — 상태 CAS(낙관)로 직렬화 |
| 48 | Saga 보상 — 롤백 못 하는 외부 행동을 반대 행동으로, 아웃박스 한 겹 더 |
| 49 | HTTP 클라이언트 '자동 동작' 함정(401 스트림 소모·429 이중 재시도 끄기) + Rate Limit 토큰버킷 세 방향(인바운드·아웃바운드 reactive/proactive) |
| 50 | Rate Limit 버킷 두 손잡이(capacity·refill) + 인바운드 ≠ 아웃바운드 독립 한도(워커·재시도가 인바운드를 우회) |
| 51 | JVM 실행 엔진(바이트코드·JIT·핫스팟·추론최적화) / GC(도달성·mark&sweep·GC root·STW·lost object·write barrier) — log_20 도달성 심화 재방문, off-arc |
| 52 | 거부 정책 = 대기시간×동시대기자수 — 아웃바운드를 bounded wait로 전환(tryConsume(maxWait)) + synchronized/check-then-act 첫 대면 |
| 53 | 재시도 안전성 = "토스가 어디까지 아나" 스펙트럼(거부→429→timeout→200) + 429 발화자 둘 + 멱등키=재시도를 특수케이스 아니게 — #4를 인출만으로 닫음 |
| 54 | 서킷 브레이커 3상태 재발명(죽은 토스→open/half-open/트립=실패율×최소표본) — RL=양 vs CB=건강, **arc 'step3 Rate Limit' 닫힘** |
| 55 | CAS 루프 = 틈의 무해화(낡으면 거부) + 2필드는 불변객체·AtomicReference + 경합 낮으면 sync가 이긴다(락 가격표 차이) — 새 arc '동시성 손끝 증명' 1칸 |
| 56 | 진짜 경합 증명(Testcontainers MySQL) — gap 락은 입장을 직렬화 못 하고 INSERT만 막는다(데드락 해부·수정) + FOR UPDATE는 읽은 행 전부(JOIN 포함) + TIMESTAMP 2038, **arc 동시성 손끝 증명 닫힘** |
| 57 | 락 발자국 스윕 — 집합 제약=UNIQUE로·카운트 제약=실존 행 앵커 record 락(gap과 다름)·스테일은 안전한 방향만·잠기는 범위는 실행 계획이 정한다, **arc 당일 닫힘** |
| 58 | 재시도 정책 — transient는 가설·바운드는 유효기간 / DLQ를 상태 기계로(DEAD 격리) / 카운터 불변식은 UPDATE 문 안에 / confirm 시간 경계 3층(TTL·패자 재확인·진입 가드) |
| 59 | 서킷 브레이커 적용 — 회로 관점(닫혀야 흐른다)·장부 세 칸은 접촉 증거·CB open↔DEAD 충돌 해소(환경 실패 미계상) + 데드락 패자 @Retry(tx 바깥 증명) + **구현 전 합의 게이트, arc 재시도 정책 닫힘** |
| 60 | URL 한 줄부터 화면까지 — DNS·포트(80/443 기본·8080은 Tomcat이 연 문) / TCP 3-handshake=순서번호 맞추기·ISN 랜덤(주입 방어+유령 패킷) / **Tomcat 두 책임**(연결·번역 + 서블릿 컨테이너), WS vs WAS / **TLS 신뢰 사슬**(비대칭키로 세션키 교환·MITM·인증서+CA 서명[JWT Signature 재사용]·root는 OS/브라우저 내장). 일반 CS(네트워크), 미러 제외 |
| 61 | 패키지 사이클 끊기(DIP) + ArchUnit 가드 — **outbox는 시간적 결합·DIP는 구조적(import) 결합을 끊는다**(런타임 호출은 유지) / 코드 옮기기 ≠ 의존 뒤집기 / 사이클은 클래스 아닌 패키지 단위(모듈러 모놀리스=MSA 전제) / 호기심이 숨은 사이클(reservation↔promotion) 파냄 / **가드가 64개 사이클 폭로**(사람 1 vs 기계 64 = log_41 실증) → 타겟 규칙으로 잠금 + ACL 통과 |
| 62 | Testcontainers 하나만 띄우기 — Gradle test worker JVM·ClassLoader·JUnit 인스턴스·Spring Context Cache의 생명주기 분리 / Context별 `@Bean` 5개 → worker JVM의 `static` 1개 / `@DynamicPropertySource`로 여러 Context를 같은 랜덤 포트 MySQL에 연결 / 통합 테스트는 `@Sql` 정리·Controller는 `@WebMvcTest` / testcase 6.564초와 suite 43.718초의 측정 경계 |
| 63 | 컴퓨터의 구성 — SSD의 실행 파일→RAM의 명령어·데이터→CPU 실행 / 운영체제가 프로세스와 메모리 자원 관리 / 프로그램과 프로세스 분리 / 메모리 격리 필요성. 일반 CS(컴퓨터 구조), 미러 제외 |
| 64 | 가상주소 번역 — 연속 베이스 방식의 한계→페이지·프레임·페이지 테이블 / 10KB→페이지2+2KB→프레임7=30KB / MMU 변환·권한 검사 / TLB는 번역 캐시 / 주소 공간별 같은 가상주소 구분. 일반 CS(운영체제/가상 메모리), 미러 제외 |
| 65 | COW·PTE·페이지 크기 — 읽기 전용 물리 페이지 공유 / 쓰기 순간 한 페이지만 복사하고 PTE 변경·TLB 무효화 / MMU 권한과 OS 정책 분리 / 내부 단편화↔PTE 수의 저울 / 명확화 뒤 자동 재인출 금지 실험. 일반 CS(운영체제/가상 메모리), 미러 제외 |
| 66 | CPU 작동 원리 — PC·MAR·MBR·IR의 인출 흐름 / 제어장치 해독·ALU 실행 / opcode·주소 지정 방식은 기계어에 이미 인코딩 / 함수 복귀 주소 / 인터럽트 번호→벡터 테이블→핸들러. 일반 CS(컴퓨터 구조), 미러 제외 |
| 67 | 스레드 실행 상태 — 프로세스 공유 자원↔스레드별 PC·SP·스택 / 스택 전체를 복사하지 않고 PC·SP·플래그·범용 레지스터 등 실행 문맥 저장·복원 / 플랫폼 스레드 1:1과 풀 재사용 / 가상 스레드·캐리어의 I/O 대기 분리. 일반 CS(운영체제/JVM 동시성), 미러 제외 |
| 68 | Cookie와 JSESSIONID 코드 적용 — 세션 식별자 ≠ 로그인 상태 / 미션의 선발급 규칙과 Servlet 세션 생성 시점 구분 / Cookie 파싱·Optional·빈 객체 / Processor 발급 정책·Response Set-Cookie / 21개 테스트 통과 |
| 69 | Coyote·Catalina·Servlet 요청 표현의 경계 — 낮은 수준 HTTP 요청 → Servlet API 래퍼 → 애플리케이션, Step3 의존성 리뷰 재방문 |
| 70 | 로그인 컨트롤러 응집도 리뷰 → @Controller·@RequestMapping의 발견·등록·호출 / RouteKey → RequestHandler 람다가 객체+Method 포착 / Spring과 현재 구현 비교 |
| 71 | JVM 메모리·요청 스레드·공유 상태 경쟁 / UserServlet의 검사-추가 간섭·스냅샷 반환 / CountDownLatch로 결정적 재현 설계 |
| 72 | 동기·비동기 = 시작 호출과 완료 처리의 관계(대표 시간선: 호출→반환→완료) / 블로킹·논블로킹 = 호출자의 대기 / `tryConsume()`·`Future.get()`·`Object.wait()`에 적용. 일반 CS(동시성/비동기), 미러 제외 |
| 73 | Java Stream — 내부 반복·지연 실행 / uri 포착 람다의 순수 변환과 put 등록의 상태 변경 / Spliterator 분할·ForkJoinPool / for문 선택. java-mvc 미러 O |
| 74 | 소켓 읽기→가상 스레드 중단·캐리어 해제→OS 수신·JDK 재제출→재개 / read·readNBytes·SO_TIMEOUT·TCP 바이트 스트림. 일반 CS, 미러 제외 |
| 75 | CPU 캐시 — 캐시 라인·공간/시간 지역성·L1/L2/L3·write-back / TLB는 번역 캐시, 데이터 캐시는 내용물 / TLB 미스·캐시 미스·페이지 폴트 분리. 일반 CS(컴퓨터 구조), 미러 제외 |
| 76 | Java 가시성 — 코어별 캐시와 스레드별 스택·레지스터 구분 / 캐시 일관성만으로 재읽기·순서 보장 불가 / volatile happens-before와 CAS의 역할 / 컨텍스트 스위치의 레지스터 저장. 일반 CS(동시성), 미러 제외 |

> 다음 새 학습 로그는 **77**부터. (종합·정리 로그는 아래 별도 트랙 — 번호 시퀀스를 쓰지 않는다.)

## 종합·정리 로그 (번호 시퀀스 밖)

기존 카테고리를 종합·정리하는 세션. 새 학습이 아니라 **기존 것의 정리**라 `logs/NN` 번호 대신 `종합_N부_`로 표기한다. 결과물은 [`Level2_정리/`](Level2_정리/) 폴더.

| 로그 | 요약 |
|---|---|
| [종합_1부](logs/종합_1부_실패의생애주기-외부API장애대응신설-흐름기반학습자.md) | 카테고리 종합-인출 1부 — DB 재봉인(집합/카운트 재정정·UNIQUE와 CAS는 같은 편) + 결제 분할 → **외부 API 장애 대응 신설(실패의 생애주기 3장)** + 흐름 기반 학습자 자가 발견 |
| [종합_2부](logs/종합_2부_6칸재봉인-Level2정리폴더신설.md) | 종합-인출 2부 — 6칸(테스트·인증·모델링·아키텍처·Spring웹·학습법) 재봉인 + **`Level2_정리/` 폴더 신설**(카테고리 9칸 파일 + 스킬 스냅샷). 정리물은 요약이 아니라 **인출의 전사**. Spring웹 *미탐구* 탈출 |
