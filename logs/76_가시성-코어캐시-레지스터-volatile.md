# 76. Java 가시성 — 코어 캐시·레지스터·volatile을 구분하기

**날짜**: 2026-10-03  
**분류**: 일반 CS (Java 메모리 모델/동시성) — 프로젝트 외, 미러 제외  
**재방문**: [#67 스레드 실행 상태](67_스레드-실행상태-플랫폼-가상스레드.md), [#71 JVM 메모리·공유 상태](71_JVM메모리-요청스레드-공유상태-동시성테스트.md), [#75 CPU 캐시](75_CPU캐시-캐시라인-지역성-계층-TLB구분.md)

## 출발 가설과 수정

처음에는 “스레드마다 전용 스택이 있으므로 CPU 캐시도 스레드마다 있고, `volatile`은 캐시를 쓰지 않게 한다”고 생각했다. 스택은 스레드별이지만 L1 캐시는 보통 **코어별**이며, L3는 공유될 수 있다. 스레드는 같은 코어를 번갈아 쓰거나 다른 코어로 옮겨 실행될 수 있다. `volatile`은 캐시 우회 지시가 아니다. [Intel 캐시 계층](https://www.intel.com/content/dam/www/public/us/en/documents/white-papers/cache-allocation-technology-white-paper.pdf)

## 캐시가 일관성을 맞추는데도 왜 가시성이 문제인가

~~~text
스레드 B가 공유 변수 x=0을 읽고 레지스터에 보관
→ 스레드 A가 x=1을 씀
→ 하드웨어 캐시 일관성은 다른 코어의 낡은 캐시 라인을 무효화할 수 있음
→ 그러나 B가 새 읽기 명령을 실행하지 않고 레지스터의 0을 계속 쓸 수 있음
~~~

캐시 일관성은 다른 코어가 **실제로 다시 읽을 때** 올바른 캐시 라인을 얻도록 돕는다. 그것만으로 JVM 컴파일러가 공유 변수를 다시 읽도록 강제하거나 여러 변수의 읽기·쓰기 순서를 Java 코드에 보장하지는 않는다. 동기화가 없는 `while (running) {}`에서 반복마다 필드를 다시 읽을 것이라는 보장이 없다. 실제로 새 값을 볼 수도 있지만 프로그램이 거기에 의존하면 안 된다. [Java 언어 명세 §17](https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html)

## volatile의 약속과 한계

`volatile` 변수에 대한 쓰기는 이후의 해당 변수 읽기와 **happens-before** 관계를 만든다. 이는 가시성과 순서의 약속이지 “항상 RAM까지 직접 읽고 쓴다”는 뜻이 아니다. 예를 들어 A가 `data = 42; ready = true;`를 실행하고 `ready`가 volatile일 때, B가 `ready == true`를 읽었다면 선행한 `data = 42`도 관찰할 수 있다(그 사이 다른 쓰기가 없다고 가정). [Java 언어 명세 §17](https://docs.oracle.com/javase/specs/jls/se21/html/jls-17.html)

단일 volatile 읽기·쓰기가 원자적이어도 `volatile int count; count++`는 읽기→더하기→쓰기의 복합 동작이다. 두 스레드가 모두 0을 읽고 1을 써서 갱신을 잃을 수 있다. 이런 갱신에는 CAS 기반 `AtomicInteger`나 락이 필요하다. [Oracle 원자적 접근](https://docs.oracle.com/javase/tutorial/essential/concurrency/atomic.html)

## 컨텍스트 스위치와 레지스터

레지스터는 CPU 안에 있다. OS가 플랫폼 스레드의 실행을 교체할 때 PC·SP·범용 레지스터 등의 실행 상태를 해당 스레드와 연결된 **커널 메모리**에 보관하고 복원한다. 스택 전체를 매번 복사하는 것은 아니다. 커널 메모리는 OS가 관리하는 메모리 영역이며, 커널은 하나의 일반 프로세스가 아니다. B가 옛 값 0을 레지스터에 갖고 중단됐다면 OS는 그 값도 보존한다. A가 공유 값을 바꾸었다고 B의 저장된 레지스터를 자동으로 고쳐 주지는 않는다. [Linux 커널의 컨텍스트 스위치 설명](https://people.kernel.org/linusw/the-arm32-scheduling-and-kernelspace-userspace-boundary)

## 한 문장 봉인

> 코어별 CPU 캐시는 하드웨어가 일관성을 관리하지만, Java 코드가 공유 변수를 다시 읽고 선행 쓰기를 관찰할 수 있다는 보장은 `volatile`·락·CAS 등의 동기화 관계에서 나온다. `volatile`은 캐시를 끄지 않으며 복합 갱신을 원자적으로 만들지도 않는다.

## 학습 방식 회고와 다음 씨앗

- ‘캐시가 최신인데 왜 못 보나?’라는 반문은 캐시 라인과 레지스터를 분리하면서 풀렸다. “메모리에서 읽는다”는 말은 RAM 직행이 아니라 CPU가 새로운 load를 실행한다는 의미로 고쳐 설명했다.
- 가시성은 캐시만의 문제가 아니라 JVM 최적화와 읽기·쓰기 순서를 포함한다. 다음 재방문에서는 happens-before의 release/acquire를 구체 코드에서 판단해 본다. 현재 hot arc의 다음 칸은 가상 스레드 스케줄러다.
