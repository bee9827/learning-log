# 다음 사이클 키워드 백로그

**운영 규칙**: 다음 사이클은 🔥 **hot**에서만 고른다. hot = *지금 타는 실의 **연관된 다음 칸들 = 하나의 관심사 arc***로, **1~2 사이클에 그 arc를 닫을 수 있게** 묶는다 — 흩어진 픽 금지(수렴하는 묶음만). arc가 닫히면 🧊 **cold**에서 *다음 arc를 통째로* 끌어올린다. 진행 중 새로 떠오른 씨앗은 *로그 본문에만* 적는다(BACKLOG 줄로 안 올림). cold는 **arc·클러스터별 보관함** — 평소엔 안 보고, 실을 갈아탈 때나 가끔 청소할 때만 들춘다. 백로그는 "갚을 빚"이 아니라 "메뉴".

## 🔥 hot — 현재 arc: PENDING 미완료(abandonment) 정리 (1~2 사이클에 닫기)

> log_37로 PENDING이 들어가자, *결제 없이 떠 있는 반쪽 상태*가 슬롯을 영구 점유하고 대기까지 왜곡(리뷰 high). "PENDING을 제대로 청소·취급"이 한 arc — **돈은 안 움직인** 케이스라 환불/보상(arc B)과는 별개로 묶는다.

- [ ] **PENDING 만료 reaper** — 예고: log_37 / 종류: 코드 적용·설계 (미결제 PENDING이 슬롯 영구 점유[리뷰 high]. created_at TTL + @Scheduled, log_28 아웃박스 패턴 재사용)
- [ ] **대기 정합성 — PENDING 위 대기 금지 / 무료 승격 차단** — 예고: log_37 / 종류: 코드 적용 (findBySlotKeyForUpdate가 status 무시 → 미결제 PENDING 위 대기·무료 승격[리뷰 high]. PENDING을 "확정"으로 오인하는 같은 뿌리)

## 🧊 cold — 보관함 (클러스터별 / 평소엔 안 봄)

### 결제 — arc「능동적 실패 + 보상/재시도」 (돈이 움직인 뒤 실패 → 환불/보상; abandonment와 별개)
- [ ] **트랜잭션 밖 외부호출 분리 + Saga 보상** — 예고: log_31·37 / 종류: 코드 적용 (confirm()이 @Transactional 안에서 토스 호출 → 승인 후 후속단계 실패 시 돈↔상태 불일치[리뷰 high]. 환불 보상 필요)
- [ ] **재시도 정책 — 몇 번·백오프·언제 포기** — 예고: log_36 / 종류: 코드 적용 (log_28 아웃박스 재시도. 위 보상과 한 묶음)

### 결제 — 단독
- [ ] **멱등성 키의 수명·범위 — 토스는 캐시를 얼마나/언제까지** — 예고: log_36 / 종류: 흐름 파악 (키 만료·재사용 충돌·캐시 보관 윈도우)
- [ ] **at-least-once vs exactly-once — 멱등성이 왜 "exactly-once 효과"를 만드나** — 예고: log_36 / 종류: 흐름 파악 (이론적 바닥)

### 구조 / 테스트
- [ ] **ArchUnit으로 ACL 경계 강제** — 예고: log_38 / 종류: 코드 적용 (토스 타입이 payment.toss 밖으로 못 나가게 테스트로 고정. "선이 지켜지나"를 자동 검증)

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

- [x] **갭락** — 예고 log_13 → 학습 log_17·26
- [x] **낙관적 락** — 예고 log_06(개념) → 학습 log_08(JDBC 구현)
- [x] **동시성 race + DB unique index 도입 여부** — 예고 log_25 → 닫힘: waiting UNIQUE 백스톱 + 예약 행 락 직렬화로 정리(roomescape-waiting 리뷰)
- [x] **애그리거트 vs 일급 컬렉션** — 예고 log_25 → 닫힘: Waitings는 식별자·영속 단위가 없어 일급 컬렉션으로 정정(리뷰 #3)
- [x] **Saga 패턴 — 보상 트랜잭션** — 예고 log_30 → 학습 log_31 (흐름 파악: 보상=롤백 아닌 "반대 행동 새 장", rollback vs compensation 구분까지)
- [x] **클라이언트 값 못 믿음 (위조 가능)** — 예고 log_32(JWT Payload) → 재방문 log_35: 토스 amount 검증에 그대로 적용(같은 뼈대 = 서버 secret+진짜값으로 검증)
- [x] **confirm 멱등성 (재시도 안전성)** — 예고 log_31(saga 재시도)·log_35 → 학습 log_36: 응답 유실 딜레마 → paymentKey 중복감지 vs Idempotency-Key, 키는 유실에도 살아남게 클라이언트가 발급
