# 다음 사이클 키워드 백로그

**운영 규칙**: 다음 사이클은 🔥 **hot**에서만 고른다. hot = *지금 타는 실의 **연관된 다음 칸들 = 하나의 관심사 arc***로, **1~2 사이클에 그 arc를 닫을 수 있게** 묶는다 — 흩어진 픽 금지(수렴하는 묶음만). arc가 닫히면 🧊 **cold**에서 *다음 arc를 통째로* 끌어올린다. 진행 중 새로 떠오른 씨앗은 *로그 본문에만* 적는다(BACKLOG 줄로 안 올림). cold는 **arc·클러스터별 보관함** — 평소엔 안 보고, 실을 갈아탈 때나 가끔 청소할 때만 들춘다. 백로그는 "갚을 빚"이 아니라 "메뉴".

## 🔥 hot — arc '토스 step2: 타임아웃 방어 + 멱등 재시도' **닫힘** (req1~4 + 진입점 + 리팩토링 완료, log_44·45, 2026-06-18). 다음 arc는 🧊 cold '결제 — 능동적 실패/보상'에서.

- [x] **req1 — RestClient connect/read 타임아웃** — ✅ 적용 완료 log_44. 타임아웃은 저수준 factory에. **factory=apache(HttpComponents)** — simple(HttpURLConnection)이 401 응답 바디를 인증처리 중 삼켜 토스 에러매핑 테스트 2개(UNAUTHORIZED_KEY/INVALID_API_KEY)를 깬 걸 *코딩 중 발견*해 전환(httpclient5 의존성 +). `ClientHttpRequestFactoryBuilder.httpComponents()` + `ClientHttpRequestFactorySettings`, 값은 `TossProperties`(Duration) yml 2s/5s. 전체 246 그린.
- [x] **req2 — 타임아웃 예외 처리** ✅ 완료 605c7938 (log_44 §7). 전송예외→도메인예외(root-cause 분류·안전기본값 "모름"), read→`NEEDS_CHECK`(catch→commit, reaper 제외), connect→502. MockWebServer로 미션표 반증·root-cause 전환 확인. 249 그린. **통찰**: 멱등키=능동적 재시도 보호 / NEEDS_CHECK=수동적 reaper 보호 → 다른 길목이라 안 수렴.
- [x] **req3 — Idempotency-Key 코드** ✅ 완료 6692383f (전용 `idempotency_key` 컬럼=SRP[orderId=정체성 / 멱등키=요청멱등성]; `OrderService.create` 1회 발급; 헤더 전송 테스트). log_45 §1.
- [x] **req4 — 주문/결제 내역 페이지** ✅ 완료 c4e30188 (읽기전용 `PaymentHistoryService` 분리·클라 머지) + **진입점 메움**(PENDING→결제하기 / NEEDS_CHECK→결제확인=서버 재confirm B, b873eabe) + **리팩토링**(PaymentService→PaymentAbandonmentService[리네임 d7d519af], TossGateway 번역 추출, 2a5a54df) + **DTO/패키지 정리**(web/dto·payment.service·exception·toss.dto, 1141f02d·d7d519af). log_45.
- [ ] **(파킹) jdk 팩토리가 응답 바디 지연을 read timeout으로 못 잡는 이유** — 예고 log_44 / 종류: 흐름 파악

## 🗂 직전 arc 'PENDING 미완료(abandonment) 정리' **닫힘** (reaper + 무료승격 차단/정리 완료, 2026-06-17)

> log_37로 PENDING이 들어가자, *결제 없이 떠 있는 반쪽 상태*가 슬롯을 영구 점유하고 대기까지 왜곡(리뷰 high). "PENDING을 제대로 청소·취급"이 한 arc — **돈은 안 움직인** 케이스라 환불/보상(arc B)과는 별개로 묶는다.

- [x] **무료 승격 차단 + 승격 PENDING 정리** — ✅ 구현 완료 2026-06-17 (커밋 0ec4496a·057298e1). 결과: promote()→PENDING / abandon이 enqueuePromotion(공통 코어 cancelPendingReservation) / reaper가 Order 없는 PENDING도 created_at으로 정리(reservations.created_at 추가) / 승격 생성은 ReservationCreator.createFromPromotion으로 위임(권한·새치기 검증 분리). cancel/auth 전면 디커플링은 이 버그엔 불필요로 판단해 생략. (설계 메모는 아래 보존)
  > 정정: "PENDING 위 대기 *금지*"는 오답 — 대기는 받아주는 게 공정. 진짜 결함은 ① 승격이 **무료 BOOKED**(`promote()`→`createByAdmin`) ② 승격 PENDING이 결제 안 되면 reaper 사각지대(Order만 스캔) ③ 정리돼도 다음 대기자 승격 안 됨(큐 멈춤).
  > **설계 4조각:**
  > 1. `Waiting.promote()` → BOOKED 말고 **PENDING 생성**(승격자도 결제해야 확정). 승격 경로만, `createByAdmin`(어드민 직접예약=BOOKED)은 유지.
  > 2. reaper가 **Order 없는 PENDING도** `created_at`으로 줍기(현재 `findExpiredPendingOrderIds`는 Order만 스캔=사각지대). TTL 30분(=결제팝업)과 승격 유예는 *별개 다이얼*.
  > 3. 슬롯 비우는 두 경로(`ReservationService.cancel` / `PaymentService.abandon`)를 **auth 없는 순수 코어로 통합** → 거기서 `enqueuePromotion` 한 번 → 양쪽 승격 보장. (현재 `abandon`에 `enqueuePromotion` 누락 = 큐 멈춤 버그)
  > 4. cancel/auth **디커플링**: 코어는 순수 동작, auth는 **명시적 단일 관문**(각 호출자에 흩지 말 것 — log_41 가드 교훈과 동형: "매 경로마다 기억해서 호출"은 취약).

## 🧊 cold — 보관함 (클러스터별 / 평소엔 안 봄)

### 결제 — arc「능동적 실패 + 보상/재시도」 (돈이 움직인 뒤 실패 → 환불/보상; abandonment와 별개)
- [ ] **불명확(NEEDS_CHECK) 주문 자동 reconciliation** — 예고 log_44 / 종류: 흐름 파악→코드 적용 (사용자가 안 돌아오면 NEEDS_CHECK 림보 → 백그라운드 잡이 토스 결제상태 조회로 확정/실패. step2는 사용자-주도로 갈음)
- [ ] **트랜잭션 밖 외부호출 분리 + Saga 보상** — 예고: log_31·37 / 종류: 코드 적용 (confirm()이 @Transactional 안에서 토스 호출 → 승인 후 후속단계 실패 시 돈↔상태 불일치[리뷰 high]. 환불 보상 필요)
- [ ] **재시도 정책 — 몇 번·백오프·언제 포기** — 예고: log_36 / 종류: 코드 적용 (log_28 아웃박스 재시도. 위 보상과 한 묶음)

### 결제 — 단독
- [ ] **멱등성 키의 수명·범위 — 토스는 캐시를 얼마나/언제까지** — 예고: log_36 / 종류: 흐름 파악 (키 만료·재사용 충돌·캐시 보관 윈도우)
- [ ] **at-least-once vs exactly-once — 멱등성이 왜 "exactly-once 효과"를 만드나** — 예고: log_36 / 종류: 흐름 파악 (이론적 바닥)
- [ ] **reaper TTL 다이얼 분리 — 결제팝업(30분) ≠ 승격 유예** — 예고: log_43 / 종류: 코드 적용 (승격은 알림+복귀 소비시간 필요 → 더 긴/별도 유예. 현재 둘 다 같은 ttlMinutes 사용. 알림 채널 딸려와 큰 작업이라 보류)

### 구조 / 테스트
- [ ] **reservation↔waiting 사이클 끊기** — 예고: log_40·42 / 종류: 코드 적용 (마지막 남은 사이클. payment↔reservation은 log_42에서 끊음). 단일 edge `ReservationCreator→WaitingDao`(새치기 방지 읽기)를 DIP(reservation에 포트 정의 → waiting이 구현)로 역전.
- [ ] **ArchUnit 가드 코드 적용** — 예고: log_38·41 / 종류: 코드 적용 (*왜·정체*는 log_41에서 닫음). 규칙 2종: ①패키지 사이클 금지 ②ACL 경계(토스 타입이 payment.toss 밖으로 못 나감). 이제 사이클이 reservation↔waiting 1건뿐이라 "신규 사이클 금지" 박기 좋은 시점(그 1건은 베이스라인 예외 후 끊기).
- [ ] **테스트 소스 패키지 미러링** — 예고: log_41 / 종류: 코드 적용 (프로덕션은 기능별인데 테스트는 아직 `service/`·`domain/` 레이어별 → 미러링)

### Saga / 분산
- [ ] **Saga 조율 방식 — choreography vs orchestration** — 예고: log_31 / 종류: 흐름 파악 (장들을 누가 지휘하나)
- [ ] **보상이 불가능/보상 자체가 실패하면** — 예고: log_31 / 종류: 흐름 파악 (못 지우는 이메일→정정메일, 보상의 멱등성·재시도)

### DB 내부 (락)
- [ ] **B-Tree에서 갭락이 물리적으로 어떻게 구현되는가** — 예고: log_17·29 / 종류: 흐름 파악 (인덱스 리프 노드 사이를 어떻게 잠그는지)
- [ ] **SELECT FOR UPDATE의 스냅샷 우회 — 내부 동작** — 예고: log_29 / 종류: 흐름 파악 (현재 최신 데이터를 읽는 구현 레벨)
- [ ] **insert intention lock이 충돌해 대기하는 상대는 누구인가** — 예고: log_26 / 종류: 흐름 파악

### 트랜잭션 / 모델링
- [ ] **@Transactional propagation — 승격을 REQUIRES_NEW로 떼면 무엇이 달라지나** — 예고: log_27 / 종류: 코드 적용 (join vs 새 트랜잭션 롤백 범위)
- [ ] **새치기 가드가 동시성에서 정말 닫혔나 — 직렬화 증명** — 예고: log_28 / 종류: 코드 적용 (H2는 gap lock 미지원 → Testcontainers MySQL)
- [ ] **DTO 변환을 Slot 단위로** — 예고: log_24 / 종류: 코드 적용 (getter 위임 1단계 → DTO가 slot 직접 수신)
- [ ] **composition vs association vs aggregation** — 예고: log_23·25 / 맥락: Waitings가 List<Waiting>을 가지는 구조의 분류
- [ ] **"DB에 숨은 비즈니스 로직" 찾는 안목** — 예고: log_25 / 맥락: 다른 DAO/Service에서 같은 냄새 스캔

### ACID
- [ ] **D = Durability, "커밋 시점부터" 영속 보장 + WAL/redo log** — 예고: log_30 / 종류: 키워드 이해→흐름 파악 (오늘 D를 "Duration/저장"으로 흐릿하게 인출)
- [ ] **C의 책임 경계 — DB 제약(FK·UNIQUE·CHECK) vs 애플리케이션 비즈니스 규칙** — 예고: log_30 / 종류: 흐름 파악 (log_24·25 "규칙 도메인 이전"과 직결)

### 인증 / 세션
- [ ] **refresh token + 짧은 만료 access token** — 예고: log_32 / 종류: 흐름 파악 (확장성·무효화 둘 다 잡는 절충)
- [ ] **세션 공유 — sticky session vs 중앙 저장소(Redis)** — 예고: log_32 / 종류: 흐름 파악 (세션의 확장성 비용 줄이기)
- [ ] **쿠키 보안 — HttpOnly/Secure/SameSite, 세션 하이재킹 방어** — 예고: log_32 / 종류: 흐름 파악

## ✅ 닫힘 (재방문 완료)

- [x] **PENDING 만료 reaper** — 예고 log_37 → 닫힘: `ExpiredOrderWorker`(커밋 d564102a), `created_at` TTL + @Scheduled로 미결제 PENDING 정리 (단, *Order 기준* 스캔이라 승격 PENDING은 못 줍는 한계 → 위 hot 항목 조각2로 이어짐)
- [x] **갭락** — 예고 log_13 → 학습 log_17·26
- [x] **낙관적 락** — 예고 log_06(개념) → 학습 log_08(JDBC 구현)
- [x] **동시성 race + DB unique index 도입 여부** — 예고 log_25 → 닫힘: waiting UNIQUE 백스톱 + 예약 행 락 직렬화로 정리(roomescape-waiting 리뷰)
- [x] **애그리거트 vs 일급 컬렉션** — 예고 log_25 → 닫힘: Waitings는 식별자·영속 단위가 없어 일급 컬렉션으로 정정(리뷰 #3)
- [x] **Saga 패턴 — 보상 트랜잭션** — 예고 log_30 → 학습 log_31 (흐름 파악: 보상=롤백 아닌 "반대 행동 새 장", rollback vs compensation 구분까지)
- [x] **클라이언트 값 못 믿음 (위조 가능)** — 예고 log_32(JWT Payload) → 재방문 log_35: 토스 amount 검증에 그대로 적용(같은 뼈대 = 서버 secret+진짜값으로 검증)
- [x] **confirm 멱등성 (재시도 안전성)** — 예고 log_31(saga 재시도)·log_35 → 학습 log_36: 응답 유실 딜레마 → paymentKey 중복감지 vs Idempotency-Key, 키는 유실에도 살아남게 클라이언트가 발급
- [x] **예약/결제 API 분리 (Q1)** — 예고 log_42 → 닫힘 log_43 (커밋 3938cddc): `POST /reservations`=예약만, `POST /payments/ready`=주문 생성+준비정보. 주문이 "결제 시작" 시점 생성. **부수로**: 인증 default-deny 전환(887f4e73), 중복주문 3중 방어(멱등 재방문 — getOrCreate+cancelPending 멱등+UNIQUE(reservation_id), 0be9e538). 토스 위젯 수동검증 통과(주문 1건).
