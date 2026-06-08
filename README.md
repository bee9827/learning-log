# 학습 로그 (Learning Log)

우테코 여러 프로젝트에 걸쳐 흩어져 있던 학습 로그를 한 곳에 모아 관리한다.
기술 딥다이브와 사이클별 학습법 회고("바꿀 것")가 프로젝트를 넘어 한 줄로 이어진다.

## 작성 규칙 (v1 — 쓰면서 다듬는다)

옵시디언으로 키워드별 학습 깊이를 추적하기 위한 위키링크 규칙.

1. **키워드 이름은 항상 똑같이.** 같은 개념을 `[[갭락]]`/`[[Gap Lock]]`로 섞으면 다른 노드가 된다. 한글 정식 명칭 하나로 통일한다. (필요하면 `keywords/`에 stub 노트로 이름 고정)
2. **링크를 3곳에 건다** — ① 학습 범위, ② 본문 첫 등장, ③ 다음 사이클 키워드.
3. **방문마다 깊이(종류)를 표시** — `이해 → 흐름 → 코드적용 → 실전`. 키워드 옆에 그 사이클의 종류를 적는다.

```markdown
**학습 범위**: [[INSERT 데드락]] — [[갭락]]과 [[Next-Key Lock]]의 맞물림

... 본문에서 [[갭락]] (종류: 흐름 파악) 을 코드 맥락에서 처음 봤다 ...

## 다음 사이클 키워드
- [[갭락]] → 맥락: SELECT FOR UPDATE가 거는 락 / 종류: 코드 적용
```

> 백링크 수 = 재방문 횟수 = 깊이. 노드를 열면 매 방문의 종류로 "어디까지 학습했나"가 보인다.

**로그 작성 전:** 오늘 키워드를 이 repo에서 먼저 검색한다. 재방문이면 깊이를 더하고(중복 금지), 이전이 얕았으면 명시한다(피드백 루프).

## 프로젝트 매핑

| 로그 범위 | 프로젝트 |
|---|---|
| log_01 ~ 13 | spring-roomescape-member |
| log_14 ~ 22 | spring-roomescape-auth |
| log_23 ~ 28 | spring-roomescape-waiting |

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
