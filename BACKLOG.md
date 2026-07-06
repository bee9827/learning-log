# 다음 사이클 키워드 백로그

**운영 규칙**: 다음 사이클은 🔥 **hot**에서만 고른다. hot = *지금 타는 실의 **연관된 다음 칸들 = 하나의 관심사 arc***로, **1~2 사이클에 그 arc를 닫을 수 있게** 묶는다 — 흩어진 픽 금지(수렴하는 묶음만). arc가 닫히면 🧊 **cold**에서 *다음 arc를 통째로* 끌어올린다. 진행 중 새로 떠오른 씨앗은 *로그 본문에만* 적는다(BACKLOG 줄로 안 올림). cold는 **arc·클러스터별 보관함** — 평소엔 안 보고, 실을 갈아탈 때나 가끔 청소할 때만 들춘다. 백로그는 "갚을 빚"이 아니라 "메뉴".

## 🔥 hot — (비어 있음, 2026-07-06 arc 닫힘. 다음 arc는 cold 메뉴에서 통째로 선택)

## 🗂 직전 arc '동시성 손끝 증명 — lock-free(CAS) + 진짜 멀티스레드' **닫힘** (log_55~56, 2026-07-06). log_52의 synchronized에서 이어진 실 — CAS로 틈 무해화 + Testcontainers MySQL 진짜 경합 증명(하다가 gap 락 데드락 결함 발견·수정).

- [x] **compareAndSet — check-then-act를 락 없이 닫기** — ✅ 닫힘 log_55 (실전 판단까지). CAS 루프(읽기→판정[CAS 앞!]→CAS; false=정보 낡음→즉시 재시도) = 틈의 제거가 아니라 **무해화**. 2필드는 불변 객체+AtomicReference. 판단: 경합 낮으면 sync 비용 ~0 → **안 바꿈**(DB 비관 락과 가격표가 다름 — 결정 축이 단순성으로). AtomicLong 명예회복(루프 패턴 하나가 모자랐던 것).
- [x] **동시성에서 정말 직렬화되나 — 진짜 멀티스레드 증명** — ✅ 닫힘 log_56, **arc 종료**. Testcontainers MySQL: 상태 CAS 10 스레드 승자 1 증명 / 예약 가드 성공 1인데 패자가 데드락(500) → InnoDB 현장 기록 해부: **gap 락은 입장을 직렬화 못 하면서 서로의 INSERT만 막는다** + **FOR UPDATE는 JOIN으로 읽은 행 전부 잠금**(times 행 경유 고리). 수정: 중복체크 FOR UPDATE 제거(UNIQUE 백스톱이 진실, log_43·47 동형) + 대기열 JOIN 없는 exists. 덤: TIMESTAMP 2038(→DATETIME), H2 `>=1` 사과문 `==1`로 조임. 290 그린.

## 🗂 직전 arc 'step3 Rate Limit (호출량 상한)' **닫힘** (log_49~54, 2026-07-03). 토큰 버킷·인바운드/아웃바운드·429 백오프(PR #640) + 거부 정책 bounded wait 전환 구현.

- [x] **capacity↑(순간 버스트) vs refillPerSec↑(평균 처리량)** — ✅ 닫힘 log_50. capacity=순간 버스트(잔고 상한), refill=지속 평균(회복률). 시나리오(cap10/refill1, 20폭주 → 10통과·이후 초당1)로 분리.
- [x] **인바운드 한도 ≠ 아웃바운드 한도** — ✅ 닫힘 log_50. 둘 다 우리 버킷이나 *다른 제약*(인바운드=우리 용량 / 아웃바운드=토스 허용). "작아야/커야" 비교규칙 없음(상황 뒤집기로 폐기). 인바운드는 아웃바운드를 *못 묶음* — 재시도·`@Scheduled` 워커(log_46·48)가 인바운드와 무관하게 토스를 부르므로 → 아웃바운드 버킷이 독립 존재해야 하는 이유.
- [x] **fail-fast 거부 vs 토큰 찰 때까지 블로킹 대기** — ✅ 닫힘 log_52. 결정 공식 = **대기시간 × 동시대기자수(유계/무계)**: 인바운드=fail-fast 유지 / 아웃바운드=**bounded wait**로 전환 구현(`tryConsume(Duration maxWait)` 오버로드, 2s, 마감 시 거절 폴백 + fail-early). 부수: check-then-act→synchronized 첫 대면(AtomicLong 탈락법), 잠은 락 밖·while 재확인(herd 미니어처), BackoffSleeper→common 이사, 나노 헬퍼 리팩터링(학습자 제안).
- [x] **read timeout 재시도("이미 처리됐을 수도") vs 429 재시도("아직 처리 안 됨")** — ✅ 닫힘 log_53, **인출만으로**(새 설명 0 — 학습자가 먼저 "닫힌 거 아니야?" 감지). 스펙트럼 = "토스가 어디까지 아나": 아웃바운드 거부(존재도 모름) → 429(받았지만 미처리, "토스 내부 인바운드") → read timeout(모름 → 멱등키+워커 수렴) → 200. 멱등키의 진짜 가치 = 재시도를 특수 케이스가 아니게(같은 성공 응답 재생). 429는 발화자 둘(우리→손님/토스→우리) 주의.
- [x] **Rate Limit(양) vs 서킷 브레이커(연속 실패 차단)** — ✅ 닫힘 log_54, **arc 종료**. 죽은 토스 시나리오(20/s × read timeout 5s = 동시 100 스레드)로 RL 사각 도출: RL=양(평시 quota 약속) / CB=건강(장애 시 스레드 보호) — 재는 축이 달라 둘 다. 3상태(closed/open/half-open)·트립(실패율 AND 최소 표본) 학습자 재발명, Resilience4j 1:1 대조. **코드 적용은 안 함** — 씨앗은 log_54 본문(직전 arc '재시도 정책' 칸과 한 가족).

## 🗂 직전 arc '결제 능동적 실패/보상' (log_46~48). reconciliation·CAS·보상(환불) 완료. 남은 **재시도 정책**은 step3 `RetryAfterInterceptor`로 일부 흡수(429 백오프·maxAttempts·멱등키 유지).

- [x] **불명확(NEEDS_CHECK) 자동 reconciliation** — ✅ 완료 log_46. 토스 `findStatus` 조회로 DONE→확정 / 아니면→실패 수렴, `@Scheduled` 워커(아웃박스 패턴), 멱등은 낙관 상태가드. 257 그린.
- [x] **상태 CAS 직렬화 — 가드는 동시성에 못 닫힌다** ✅ 완료 log_47. status를 version 삼은 낙관 CAS(`UPDATE WHERE status=expected`, 0행=짐), 네 수렴지점(confirm/recheck/reconcile/abandon) 게이트. 258 그린. (이 칸은 *동시성/직렬화* 갈래[#28 가족]였고, 후속 '멀티스레드 증명'은 보상 arc와 결이 달라 cold로 분리 — log_47 "바꿀 것" 적용.)
- [ ] **재시도 정책 — 워커 재시도 횟수·백오프·언제 포기→보상** — 예고 log_46(§1)·48(§5)·#36 / 종류: 흐름 파악→코드 적용 (외부 '모름'은 반환타입으로 못 갈려 바운드 필요. #28 아웃박스 재시도와 한 묶음 — **보상으로 넘어가는 경첩**)
- [x] **결제 보상(Saga) — 예약 확보 실패 시 자동 환불** ✅ 완료 (커밋 a87e13ae, 개념·설계 log_48). 승인 후 예약 확정이 영구 실패(BusinessRule/EntityNotFound)면 예외를 잡아 `NEEDS_REFUND` 커밋(증거 생존) → `PaymentRefundWorker`가 토스 `cancel`(멱등키)로 환불 → CAS로 `FAILED` 수렴(아웃박스 한 겹 더). reconcile의 무한 재시도 루프도 닫음. 263 그린.
  > **주의**: "트랜잭션 밖 외부호출 분리"는 *우회*로 해결 — 토스 호출은 여전히 `@Transactional` 안이고, 대신 예외를 잡아 `NEEDS_REFUND`를 *커밋*(롤백 방지)했다. 설계의 NEEDS_CHECK 재사용 대신 *전용 상태 `NEEDS_REFUND`* 로 분리(의미가 '모름'과 달라서). 재시도 바운드는 미구현(아래 '재시도 정책').

## 🗂 직전 arc '토스 step2: 타임아웃 방어 + 멱등 재시도' **닫힘** (req1~4 + 진입점 + 리팩토링, log_44·45, 2026-06-18)

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
- [ ] **락 발자국 스윕 — JOIN FOR UPDATE 4곳 + 승격 경로 gap-then-insert 안티패턴** — 예고: log_56(§7, 커밋 전 리뷰가 실증) / 종류: 코드 적용 (WaitingService·PromotionService 잔존. 승격 경로는 단일 스레드 워커 가정에 기댄 안전 — 워커 병렬화·다중 인스턴스 순간 재발. "카테고리는 한 번에" 한 패스로)
- [ ] **B-Tree에서 갭락이 물리적으로 어떻게 구현되는가** — 예고: log_17·29 / 종류: 흐름 파악 (인덱스 리프 노드 사이를 어떻게 잠그는지. log_56 §7에서 "스캔한 레코드 단위·인덱스가 gap 위치를 정한다"까지 진입 — 리프 노드 레벨 물리 구현이 남음)
- [ ] **SELECT FOR UPDATE의 스냅샷 우회 — 내부 동작** — 예고: log_29 / 종류: 흐름 파악 (현재 최신 데이터를 읽는 구현 레벨)
- [x] ~~insert intention lock이 충돌해 대기하는 상대는 누구인가~~ — ✅ 닫힘 log_56 (실전 데드락으로): **상대 = 같은 틈의 gap 락 보유자들.** gap끼리는 호환이라 못 막고, insert intention만 gap에 막힌다.

### 트랜잭션 / 모델링
- [ ] **@Transactional propagation — 승격을 REQUIRES_NEW로 떼면 무엇이 달라지나** — 예고: log_27 / 종류: 코드 적용 (join vs 새 트랜잭션 롤백 범위)
- [x] ~~동시성에서 정말 직렬화되나 — 진짜 멀티스레드 증명~~ → **🔥 hot으로 승격** (2026-07-03, arc '동시성 손끝 증명')
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

### JVM / 런타임 (신규 클러스터 — log_51 off-arc 탐험에서 열림)
- [ ] **세대 구분 GC — Minor/Major, Young/Old** — 예고: log_20·51 / 종류: 흐름 파악 (왜 힙을 세대로 나누나 = weak generational hypothesis. log_20이 예고했으나 오늘 미답, STW 트레이드오프에서 이름만 스침)
- [ ] **계층형 컴파일 C1/C2** — 예고: log_51 / 종류: 흐름 파악 (JIT 안에 컴파일러가 여러 단계인 이유 — 왜 한 번에 최고 수준으로 안 하나)
- [ ] **write barrier 구현 깊이** — 예고: log_51 / 종류: 흐름 파악 (SATB vs incremental update 선택 기준, G1/ZGC가 실제로 쓰는 방식)
- [ ] **Java 21 LTS의 JVM 현황 — 현대 GC가 STW를 어떻게 거의 없앴나** — 예고: log_51(세션 끝, 학습자 궁금증) / 종류: 흐름 파악 (오늘 STW·write barrier·동시 GC의 *현재 실제 상태*: ZGC·Generational ZGC·Shenandoah. 곁다리로 가상 스레드[Loom])

## ✅ 닫힘 (재방문 완료)

- [x] **Java GC 동작 방식 심화 (GC 루트·흐름)** — 예고 log_20 → 학습 log_51: 도달성 3줄에서 mark&sweep(루트 정방향 vs 역참조)·GC root·floating garbage·트리거=메모리압력·STW(앱을 멈춤)·lost-object·write barrier(incremental/SATB)까지 심화. *세대 구분(Minor/Major) 부분은 미답 → cold 'JVM/런타임'으로 이월.*

- [x] **PENDING 만료 reaper** — 예고 log_37 → 닫힘: `ExpiredOrderWorker`(커밋 d564102a), `created_at` TTL + @Scheduled로 미결제 PENDING 정리 (단, *Order 기준* 스캔이라 승격 PENDING은 못 줍는 한계 → 위 hot 항목 조각2로 이어짐)
- [x] **갭락** — 예고 log_13 → 학습 log_17·26
- [x] **낙관적 락** — 예고 log_06(개념) → 학습 log_08(JDBC 구현)
- [x] **동시성 race + DB unique index 도입 여부** — 예고 log_25 → 닫힘: waiting UNIQUE 백스톱 + 예약 행 락 직렬화로 정리(roomescape-waiting 리뷰)
- [x] **애그리거트 vs 일급 컬렉션** — 예고 log_25 → 닫힘: Waitings는 식별자·영속 단위가 없어 일급 컬렉션으로 정정(리뷰 #3)
- [x] **Saga 패턴 — 보상 트랜잭션** — 예고 log_30 → 학습 log_31 (흐름 파악: 보상=롤백 아닌 "반대 행동 새 장", rollback vs compensation 구분까지)
- [x] **클라이언트 값 못 믿음 (위조 가능)** — 예고 log_32(JWT Payload) → 재방문 log_35: 토스 amount 검증에 그대로 적용(같은 뼈대 = 서버 secret+진짜값으로 검증)
- [x] **confirm 멱등성 (재시도 안전성)** — 예고 log_31(saga 재시도)·log_35 → 학습 log_36: 응답 유실 딜레마 → paymentKey 중복감지 vs Idempotency-Key, 키는 유실에도 살아남게 클라이언트가 발급
- [x] **예약/결제 API 분리 (Q1)** — 예고 log_42 → 닫힘 log_43 (커밋 3938cddc): `POST /reservations`=예약만, `POST /payments/ready`=주문 생성+준비정보. 주문이 "결제 시작" 시점 생성. **부수로**: 인증 default-deny 전환(887f4e73), 중복주문 3중 방어(멱등 재방문 — getOrCreate+cancelPending 멱등+UNIQUE(reservation_id), 0be9e538). 토스 위젯 수동검증 통과(주문 1건).
