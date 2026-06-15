# 다음 사이클 키워드 백로그

재방문 대기 중인 키워드를 한 곳에서 관리한다. 사이클 마무리마다: 재방문한 건 **닫힘**으로 옮기고, 새로 예고한 키워드는 **열림**에 추가한다.
(v1 — 최근 로그에서 시드. 더 과거 로그의 키워드는 점진적으로 채운다.)

## 🔲 열림 (재방문 대기)

- [ ] **B-Tree에서 갭락이 물리적으로 어떻게 구현되는가** — 예고: log_17·29 / 종류: 흐름 파악 (인덱스 리프 노드 사이를 어떻게 잠그는지)
- [ ] **SELECT FOR UPDATE의 스냅샷 우회 — 내부 동작** — 예고: log_29 / 종류: 흐름 파악 (현재 최신 데이터를 읽는 구현 레벨)

- [ ] **insert intention lock이 충돌해 대기하는 상대는 누구인가** — 예고: log_26 / 종류: 흐름 파악
- [ ] **@Transactional propagation — 승격을 REQUIRES_NEW로 떼면 무엇이 달라지나** — 예고: log_27 / 종류: 코드 적용 (join vs 새 트랜잭션 롤백 범위)
- [ ] **새치기 가드가 동시성에서 정말 닫혔나 — 직렬화 증명** — 예고: log_28 / 종류: 코드 적용 (H2는 gap lock 미지원 → Testcontainers MySQL)
- [ ] **DTO 변환을 Slot 단위로** — 예고: log_24 / 종류: 코드 적용 (getter 위임 1단계 → DTO가 slot 직접 수신)
- [ ] **composition vs association vs aggregation** — 예고: log_23·25 / 맥락: Waitings가 List<Waiting>을 가지는 구조의 분류
- [ ] **"DB에 숨은 비즈니스 로직" 찾는 안목** — 예고: log_25 / 맥락: 다른 DAO/Service에서 같은 냄새 스캔

- [ ] **D = Durability, "커밋 시점부터" 영속 보장 + WAL/redo log** — 예고: log_30 / 종류: 키워드 이해→흐름 파악 (오늘 D를 "Duration/저장"으로 흐릿하게 인출)
- [ ] **C의 책임 경계 — DB 제약(FK·UNIQUE·CHECK) vs 애플리케이션 비즈니스 규칙** — 예고: log_30 / 종류: 흐름 파악 (log_24·25 "규칙 도메인 이전"과 직결)
- [ ] **Saga 조율 방식 — choreography vs orchestration** — 예고: log_31 / 종류: 흐름 파악 (장들을 누가 지휘하나)
- [ ] **보상이 불가능/보상 자체가 실패하면** — 예고: log_31 / 종류: 흐름 파악 (못 지우는 이메일→정정메일, 보상의 멱등성·재시도)
- [ ] **Saga 코드 적용** — 예고: log_31 / 종류: 코드 적용 (룸이스케이프엔 외부 PG 없어 미적용 — 어디 적용 가능한가)

- [ ] **refresh token + 짧은 만료 access token** — 예고: log_32 / 종류: 흐름 파악 (확장성·무효화 둘 다 잡는 절충)
- [ ] **세션 공유 — sticky session vs 중앙 저장소(Redis)** — 예고: log_32 / 종류: 흐름 파악 (세션의 확장성 비용 줄이기)
- [ ] **쿠키 보안 — HttpOnly/Secure/SameSite, 세션 하이재킹 방어** — 예고: log_32 / 종류: 흐름 파악

## ✅ 닫힘 (재방문 완료)

- [x] **갭락** — 예고 log_13 → 학습 log_17·26
- [x] **낙관적 락** — 예고 log_06(개념) → 학습 log_08(JDBC 구현)
- [x] **동시성 race + DB unique index 도입 여부** — 예고 log_25 → 닫힘: waiting UNIQUE 백스톱 + 예약 행 락 직렬화로 정리(roomescape-waiting 리뷰)
- [x] **애그리거트 vs 일급 컬렉션** — 예고 log_25 → 닫힘: Waitings는 식별자·영속 단위가 없어 일급 컬렉션으로 정정(리뷰 #3)
- [x] **Saga 패턴 — 보상 트랜잭션** — 예고 log_30 → 학습 log_31 (흐름 파악: 보상=롤백 아닌 "반대 행동 새 장", rollback vs compensation 구분까지)
