# 학습 로그 (Learning Log)

우테코 여러 프로젝트에 걸쳐 흩어져 있던 학습 로그를 한 곳에 모아 관리한다.
기술 딥다이브와 사이클별 학습법 회고("바꿀 것")가 프로젝트를 넘어 한 줄로 이어진다.

> 📍 카테고리별 **지향점·저울 지도**: [CATEGORIES.md](CATEGORIES.md)

## 운영 방식

- **로그**: `logs/NN_핵심키워드.md` — 사이클별 기록.
- **다음 키워드 관리**: [`BACKLOG.md`](BACKLOG.md) — 재방문 대기(열림)/완료(닫힘) 체크리스트. *다음에 뭘 학습할지는 여기서 고른다.*
- **로그 작성 전**: 오늘 키워드를 이 repo에서 먼저 검색한다. 재방문이면 깊이를 더하고(중복 금지), 이전이 얕았으면 명시한다(피드백 루프).

> 옵시디언 그래프(위키링크/태그)로 키워드를 관리하려다 — 유지 부담이 크고 효용은 "구경거리"에 가까워 **과감히 접었다.** 키워드 관리는 BACKLOG.md 하나로 충분하다. (학습 시스템은 학습을 위한 *수단*이지 목적이 아니다.)

## 프로젝트 매핑

| 로그 범위 | 프로젝트 |
|---|---|
| 01 ~ 13 | spring-roomescape-member |
| 14 ~ 22 | spring-roomescape-auth |
| 23 ~ 28 | spring-roomescape-waiting |

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
