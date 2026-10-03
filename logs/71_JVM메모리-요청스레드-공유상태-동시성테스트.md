# 71. JVM 메모리와 요청 스레드 — 공유 상태의 경쟁을 테스트하기까지

**날짜**: 2026-09-28

**학습 범위**: Thread 미션의 JVM 메모리 구조·웹 요청 처리·공유 상태·동시성 테스트

**재방문**: [#67 스레드 실행 상태](67_스레드-실행상태-플랫폼-가상스레드.md), [#55 CAS 루프](55_CAS루프-락없이틈무해화-2필드는불변객체참조-경합낮으면sync.md)

**자료**: [Java SE 21 JVM 명세](https://docs.oracle.com/javase/specs/jvms/se21/html/index.html), [JLS 17: Threads and Locks](https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html), [List.copyOf](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html#copyOf(java.util.Collection)), [CountDownLatch](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CountDownLatch.html)

분류: 일반 CS (JVM/동시성) — Thread 미션 코드와 연결

## 메모리 구조를 코드에 연결

한 JVM의 스레드는 Method Area와 Heap을 공유한다. 각 스레드에는 JVM Stack, PC Register, Native Method Stack이 따로 있다. `Counter counter = new Counter()`에서 `Counter` 객체와 필드 `count`는 Heap에, 지역 변수 `counter`의 참조는 main 스레드의 스택 프레임에 놓인다. `increment()`를 두 스레드가 호출하면 지역 변수 `next`는 각자의 스택 프레임에 생기지만 같은 객체의 `count`에 접근한다. `Thread` 객체도 Heap의 객체이며, `start()`가 새 실행 흐름을 시작한다.

두 스레드가 모두 `count == 0`을 읽고 각자 `next == 1`을 계산한 뒤 쓰면 최종값은 1이 될 수 있다. 읽기·계산·쓰기 전체를 같은 락으로 묶거나 `AtomicInteger`의 CAS 재시도로 갱신 유실을 막는다. `volatile`은 다른 스레드가 쓴 값을 볼 수 있도록 하는 가시성 규칙을 제공하지만 이 세 동작을 하나의 원자적 연산으로 만들지는 않는다.

## 미션 서버의 스레드 흐름

`Connector.start()`가 연결 수락용 스레드를 시작한다. 그 스레드의 `accept()`가 연결을 기다리고, 연결마다 `Http11Processor`를 실행할 새 스레드를 만든다. A 요청의 처리 스레드가 I/O로 기다리는 동안 연결 수락 스레드는 B 연결을 받을 수 있다. 현재 코드의 연결당 플랫폼 스레드 생성은 동시 연결이 많을 때 자원 부담이 생긴다. 가상 스레드는 지원되는 blocking I/O에서 캐리어를 놓아줄 수 있지만 공유 객체의 갱신 유실을 해결하지는 않는다.

`Connector.stopped` 같은 종료 플래그에는 가시성이 필요하다. 다만 `volatile`만 붙여도 `accept()`의 블로킹이 풀리지는 않으므로 소켓을 닫는 동작도 필요하다.

## UserServlet의 경쟁과 읽기 전용 반환

`study/src/test/java/thread/stage1/UserServlet.java`의 `if (!users.contains(user)) users.add(user)`는 검사와 추가 사이에 다른 스레드가 끼어들 수 있다. 같은 "gugu"로 가입하는 두 요청이 모두 검사를 통과하면 중복이 생긴다. `join()` 전체를 같은 락으로 보호해야 한다. `size()`와 `getUsers()`처럼 같은 상태를 읽는 경로도 그 락에 맞춰야 한다.

`Collections.unmodifiableList(users)`는 원본을 계속 읽는 뷰다. `List.copyOf(users)`는 일반 가변 리스트의 원소 참조를 별도 목록에 복사해 이후 추가·삭제가 반영되지 않는 읽기 전용 스냅샷을 만든다. 객체 자체를 깊게 복사하지는 않는다. 이 복사도 `join()`과 같은 락 안에서 해야 일관된 목록을 얻는다.

## 경쟁을 재현하는 테스트

기존 `ConcurrencyTest`의 `Thread.join()`은 종료를 기다릴 뿐 실행 순서를 맞추지 않는다. 두 스레드를 동시에 시작하거나 횟수를 늘리면 실패 확률은 높아지지만, 테스트가 통과해도 안전성을 증명하지 못한다.

실패를 확실히 보이려면 테스트용 지점을 `contains()`가 false를 반환한 뒤와 `add()` 사이에 두고 `CountDownLatch`로 다음 순서를 만든다.

```text
첫 번째 스레드: contains=false → add 직전 대기
두 번째 스레드: contains=false → add 완료
첫 번째 스레드: 대기 해제 → add 완료
결과: 중복 가입
```

현재 구현에는 그 지점에 테스트가 개입할 통로가 없다. 코드를 유지하면 동시 출발·반복 실행은 확률적 재현에 그친다. 이번에는 테스트 설계까지 학습했고 미션 소스 수정이나 로그 출력 실습은 하지 않았다.

## 학습 방식 회고

구체적인 실패 순서를 추론하는 질문은 도움이 됐다. 이미 이해한 내용을 다시 묻거나 `CountDownLatch`로 테스트를 설계하는 대화를 디버거 브레이크포인트 설명으로 돌렸을 때 흐름이 끊겼다. 다음에는 질문의 대상을 먼저 정확히 읽고, 문법·도구 사용법을 물으면 바로 구체적인 코드를 보여준다. 학습자가 범위를 물었을 때는 종료 조건을 명확히 말한다.
