# 62. Testcontainers 하나만 띄우기 — worker JVM·ClassLoader·static·Spring Context Cache

**날짜**: 2026-08-19

**학습 범위**: Mapmory 백엔드 전체 테스트가 4분 20초 걸리고 MySQL 컨테이너가 5개 뜨던 원인을 추적했다. `static` 공유 컨테이너로 바꾸는 과정에서 Gradle, 테스트 worker JVM, ClassLoader, JUnit 테스트 인스턴스, Spring Context의 생명주기를 분리해 이해했다.

---

## 출발점 — `@Bean`인데 왜 MySQL이 5개 뜰까

기존 테스트 설정은 `MySQLContainer`를 Spring Bean으로 등록했다.

```java
@TestConfiguration
class MySqlTestContainerConfig {

    @Bean
    @ServiceConnection
    MySQLContainer<?> mySQLContainer() {
        return new MySQLContainer<>("mysql:8.4");
    }
}
```

처음에는 `@Bean`이면 전체 테스트에서 하나일 거라고 생각했다. 하지만 Spring Bean의 singleton 범위는 JVM 전체가 아니라 **하나의 Spring ApplicationContext 안**이다.

전체 테스트에는 서로 다른 Context 구성이 있었다.

```text
1. 기본 @SpringBootTest
2. @SpringBootTest + @AutoConfigureMockMvc + KakaoApiClient @MockitoBean
3. @SpringBootTest + @AutoConfigureMockMvc + 추가 Security 테스트 설정
4. @SpringBootTest + @AutoConfigureMockMvc + RegionMapSummaryService @MockitoBean
5. @DataJpaTest
```

Spring Test의 Context Cache는 구성이 같은 테스트끼리 Context를 재사용한다. 반대로 테스트 슬라이스, `@Import`, 프로퍼티, `@MockitoBean`, 추가 테스트 설정 등이 달라지면 캐시 키가 달라져 별도의 Context가 만들어진다. 각 Context는 자신이 관리할 Bean을 만들기 때문에 위 `@Bean` 메서드의 `new MySQLContainer<>()`도 Context마다 실행됐다.

핵심은 **`@Import` 자체가 컨테이너를 늘린 것이 아니라, Import한 설정에서 컨테이너를 Context 소유의 Bean으로 만든 것**이다.

```text
서로 다른 Spring Context 5개
└── Context마다 MySQLContainer Bean 1개
    └── MySQL 컨테이너 5개
```

## Gradle과 테스트 worker JVM

Gradle은 프로젝트의 컴파일, 테스트, 패키징 같은 작업을 자동화하는 **빌드 자동화 도구**다. Java 플러그인이 기본 `test` 작업을 제공하고, 다음 코드는 그 작업을 새로 만드는 것이 아니라 이미 존재하는 작업의 테스트 엔진을 설정한다.

```groovy
tasks.named('test') {
    useJUnitPlatform()
}
```

Gradle은 `test` 작업을 실행할 때 테스트를 Gradle 빌드 프로세스와 격리된 별도의 Java 프로세스에서 실행한다. 이것이 테스트 worker JVM이다.

```text
Gradle 빌드 JVM
└── 테스트 worker JVM (Gradle Test Executor 1)
    ├── JUnit 테스트 실행
    ├── Spring Context 생성
    └── Testcontainers 실행
```

처음에는 이를 "운영 JVM과 환경을 분리한다"고 표현했다. 정확히는 운영 서버와의 분리가 아니라 **Gradle 빌드 프로세스와 테스트 실행 프로세스의 격리**다. 같은 OS·JDK·환경변수를 사용해도 프로세스가 다르므로 메모리와 `static` 상태를 직접 공유하지 않는다.

## 테스트 메서드마다 무엇이 새로 만들어지나

JUnit 5의 기본 테스트 인스턴스 생명주기는 `PER_METHOD`다. 테스트 메서드마다 테스트 클래스의 **객체**가 새로 만들어지고, 그 객체의 인스턴스 필드도 다시 초기화된다.

```java
class ExampleTest {
    int instanceValue;
    static int sharedValue;
}
```

```text
ExampleTest 객체 A → 첫 번째 테스트 메서드, instanceValue 별도
ExampleTest 객체 B → 두 번째 테스트 메서드, instanceValue 별도
로딩된 ExampleTest 클래스 → sharedValue 하나
```

여기서 교정한 오개념은 두 가지다.

1. `new`는 클래스를 새로 만드는 것이 아니라 이미 로딩된 클래스 정의를 기반으로 객체를 만든다.
2. 인스턴스 필드도 객체의 일부이므로 객체와 함께 힙에 있다. 메서드 스택에 있을 수 있는 것은 객체를 가리키는 지역 참조 변수다.

## ClassLoader와 `static`

ClassLoader는 `new`를 실행하는 도구가 아니다. 컴파일된 `.class` 파일에서 클래스 이름, 필드, 메서드 바이트코드, 부모 타입 등의 **클래스 정의를 JVM이 사용할 수 있도록 로딩**한다.

```text
ClassLoader가 로딩한 Example 클래스 정의 1개
├── Example 객체 first
├── Example 객체 second
└── static 필드 1세트
```

`static`은 JVM에 직접 종속된다고 말하기보다 **특정 ClassLoader가 로딩한 특정 클래스에 종속**된다고 말하는 것이 정확하다. 같은 이름의 클래스라도 서로 다른 ClassLoader가 각각 로딩하면 별개의 클래스 정체성과 `static` 값을 가진다.

일반적인 Gradle 전체 테스트에서는 하나의 worker JVM과 테스트 ClassLoader가 여러 테스트 클래스를 실행한다. 따라서 이 환경에서는 `static`의 생명주기가 사실상 worker JVM 생명주기처럼 보인다.

```text
worker JVM 1개 + ClassLoader 1개
└── MySqlTestContainerSupport 클래스 1개
    └── static MYSQL_CONTAINER 1개
```

단, `maxParallelForks` 등으로 worker JVM을 두 개 사용하면 메모리를 공유하지 않으므로 컨테이너도 JVM마다 하나씩, 총 두 개가 생긴다.

### 클래스 로딩·초기화와 JIT는 다른 단계다

ClassLoader가 `.class` 정의를 로딩하는 것은 `new`로 객체를 만드는 것과 다르다. 클래스가 처음 능동적으로 사용되면 JVM이 클래스를 로딩·연결·초기화하고, 이때 `static` 필드 초기화가 실행된다. 객체는 그 뒤 이미 로딩된 클래스 정의를 기반으로 여러 개 만들 수 있다.

JIT는 이 과정의 이름이 아니다. JIT는 프로그램이 실행된 뒤 반복해서 호출되는 바이트코드를 관찰하고 자주 실행되는 코드를 네이티브 코드로 최적화하는 실행 엔진의 별도 메커니즘이다.

```text
.class 로딩·연결·초기화 → 클래스 정의와 static 준비
new                     → 그 정의를 기반으로 객체 생성
JIT                     → 실행 중 뜨거워진 코드를 최적화
```

모든 클래스를 애플리케이션 시작 전에 미리 올리면 사용하지 않는 클래스까지 로딩·초기화하느라 시작 시간과 메모리를 소비한다. 필요한 시점에 로딩하는 이유는 이 초기 비용을 늦추고 불필요한 작업을 피하기 위해서다.

## Spring Context와 ClassLoader는 다른 층이다

둘을 비슷한 개념으로 혼동했지만 책임이 다르다.

```text
ClassLoader    → .class의 정의를 로딩한다
Spring Context → 등록된 객체(Bean)를 생성하고 관리한다
```

하나의 ClassLoader가 `MyService.class`를 한 번 로딩해도 Spring Context A와 B는 그 정의로 `MyService` Bean 객체를 각각 만들 수 있다. 이것이 기존 컨테이너 Bean이 여러 개 생긴 이유이기도 하다.

## 변경 — 컨테이너 소유권을 Context에서 worker JVM으로

공유 컨테이너를 Spring Bean으로 등록하지 않고 공통 테스트 부모 클래스의 `static` 필드로 옮겼다.

```java
public abstract class MySqlTestContainerSupport {

    private static final MySQLContainer<?> MYSQL_CONTAINER
            = startContainer();

    @DynamicPropertySource
    static void registerDataSourceProperties(
            DynamicPropertyRegistry registry
    ) {
        registry.add(
                "spring.datasource.url",
                MYSQL_CONTAINER::getJdbcUrl
        );
        registry.add(
                "spring.datasource.username",
                MYSQL_CONTAINER::getUsername
        );
        registry.add(
                "spring.datasource.password",
                MYSQL_CONTAINER::getPassword
        );
    }

    private static MySQLContainer<?> startContainer() {
        MySQLContainer<?> container
                = new MySQLContainer<>("mysql:8.4");
        container.start();
        return container;
    }
}
```

DB가 필요한 테스트가 이 클래스를 상속한다.

```java
@SpringBootTest
class IntegrationTest extends MySqlTestContainerSupport {
}

@DataJpaTest
class RepositoryTest extends MySqlTestContainerSupport {
}
```

Spring Context는 여전히 여러 개일 수 있다. 하지만 모두 같은 worker JVM과 ClassLoader에서 같은 `static MYSQL_CONTAINER`를 참조하므로 `startContainer()`는 한 번만 실행된다.

`MYSQL_CONTAINER`의 정적 초기화가 실행되는 순간 Testcontainers는 Docker를 통해 MySQL을 시작한다. Docker Desktop이나 Docker daemon에 접근할 수 없다면 `@DynamicPropertySource`가 값을 등록하기도 전에 클래스 초기화 단계에서 실패하고, 그 결과 Spring Context도 생성되지 못한다.

여기서 `mysql:8.4`는 YAML의 DB 주소가 아니라 컨테이너가 사용할 Docker 이미지다. 컨테이너 내부 MySQL 포트는 `3306`이지만 호스트 포트는 실행할 때 별도로 결정된다.

## `@DynamicPropertySource` — 여러 Context를 하나의 MySQL로 연결

각 Spring Context는 자신의 `DataSource` Bean을 만든다. 서로 다른 `DataSource`가 같은 컨테이너를 바라보려면 컨테이너의 JDBC URL, 사용자명, 비밀번호를 각 Context의 환경에 전달해야 한다.

Spring Test는 부모 클래스까지 탐색해 `@DynamicPropertySource` 메서드를 찾아 Context를 구성할 때 호출한다. 호출처가 코드에 직접 보이지 않는 것은 애노테이션을 발견한 프레임워크가 리플렉션으로 호출하기 때문이다.

```text
Context A 구성 → registerDataSourceProperties() 호출 ┐
Context B 구성 → registerDataSourceProperties() 호출 ├→ 같은 MYSQL_CONTAINER
Context C 구성 → registerDataSourceProperties() 호출 ┘
```

`application-local.yaml`에도 다음 값이 있다.

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mapmory
    username: root
    password: local
```

하지만 테스트의 동적 프로퍼티가 같은 키를 더 높은 우선순위로 제공하므로, 테스트 `DataSource`는 YAML의 로컬 DB가 아니라 Testcontainer의 접속 정보를 사용한다.

`MySQLContainer` 내부의 MySQL은 `3306`을 사용하지만 호스트 포트는 실행 시 사용 가능한 임의 포트로 매핑된다. 포트를 고정하지 않아 같은 PC나 CI에서 컨테이너가 동시에 떠도 충돌을 피한다. `getJdbcUrl()`은 이 실제 호스트·매핑 포트·DB 이름이 포함된 주소를 제공한다.

클래스의 `static` 초기화가 끝나야 `registerDataSourceProperties()` 본문이 실행된다. 따라서 컨테이너 시작 순서를 보장하는 것은 `Supplier`가 아니라 다음 초기화다.

```java
static MYSQL_CONTAINER = startContainer();
```

`MYSQL_CONTAINER::getJdbcUrl` 같은 `Supplier`는 이미 시작된 컨테이너의 값을 Spring이 프로퍼티를 해석할 때 제공하는 장치다.

## 데이터 격리

컨테이너를 하나로 공유하면 테스트들이 같은 물리 DB를 본다. 실행 속도만 줄이고 테스트 간 데이터가 섞이면 안 된다.

처음에는 공통 `IntegrationTest`에 `@Transactional`을 붙여 테스트 메서드 종료 후 롤백하려 했다. 하지만 이렇게 하면 테스트가 하나의 큰 테스트 관리 트랜잭션 안에서 실행되어 운영 코드의 실제 커밋 경계가 가려질 수 있다. 따라서 통합 테스트는 운영과 같은 트랜잭션 경계로 실행하고, 테스트가 끝난 뒤 `@Sql(AFTER_TEST_METHOD)`로 변경 데이터를 삭제하도록 교정했다.

```java
@SpringBootTest
@Sql(
        scripts = "/db/cleanup-test-data.sql",
        executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD
)
abstract class IntegrationTest extends MySqlTestContainerSupport {
}
```

`@DataJpaTest`는 Repository 슬라이스의 격리 방식으로 제공되는 기본 테스트 트랜잭션 롤백을 그대로 사용한다.

```text
컨테이너 공유 ≠ 데이터 공유 허용
통합 테스트      → 실제 트랜잭션 경계 + 종료 후 SQL 정리
Repository 슬라이스 → @DataJpaTest 기본 롤백
```

## 테스트 경계도 함께 줄이기 — `@WebMvcTest`

`RegionMapSummaryControllerTest`는 Service를 mock으로 대체하면서도 `@SpringBootTest`를 사용해 전체 애플리케이션과 DB 컨테이너를 띄우고 있었다. 검증 대상이 HTTP 매핑·검증·인증 응답이라면 MVC 슬라이스만으로 충분하므로 `@WebMvcTest`로 전환했다.

최신 인증 구조에서는 Controller의 Service에 도달하기 전에 `LoginMemberArgumentResolver`가 `MemberRepository`로 회원을 조회한다.

```text
요청
→ JWT 필터가 memberId 추출
→ LoginMemberArgumentResolver가 MemberRepository 조회
→ Controller 파라미터에 Member 주입
→ Controller가 mock Service 호출
```

따라서 Service를 mock해도 Resolver가 사용하는 Repository까지 자동으로 사라지지는 않는다. 슬라이스 테스트에서는 `RegionMapSummaryService`와 `MemberRepository`를 `@MockitoBean`으로 제공하고, 실제 DB 저장 없이 인증부터 Controller 호출까지 검증했다.

## 세 가지 테스트 시간은 측정 범위가 다르다

IntelliJ가 전체 테스트를 `6.564초`로 표시한 반면, 별도로 실행한 Gradle 전체 테스트는 약 54초가 걸렸다. 같은 실행을 일대일로 잰 수치는 아니지만 차이가 너무 커 측정 범위를 다시 확인했고, 둘 중 하나가 잘못된 것이 아니라 스톱워치를 누른 위치가 다르다는 점을 발견했다.

```text
개별 testcase 시간 합계
  → 각 테스트 메서드가 실행된 시간

testsuite 시간
  → Spring Context·Testcontainers 등 suite 준비와 테스트 실행

Gradle 전체 경과 시간
  → Gradle 구성·컴파일 확인·worker JVM·suite·보고서 생성 전체
```

당시 생성된 XML을 다시 계산하니 88개 테스트 메서드 합계는 `6.564초`, suite 합계는 `43.718초`였다. 예를 들어 `MapmoryBackendApplicationTest`는 suite 전체가 `30.356초`였지만 `loadsApplicationContext()` 테스트 메서드는 `0.615초`였다. 약 29.7초의 Spring Context·컨테이너 준비 시간은 개별 테스트 메서드 합계에 드러나지 않았다.

Testcontainers 개선은 바로 이 준비 시간을 줄이는 작업이므로 IntelliJ의 테스트 메서드 합계만 보면 효과를 놓친다. 컨테이너 시작 횟수, suite 시간, 명령 시작부터 종료까지의 실제 경과 시간을 함께 기록해야 한다.

## 측정

| 구분 | 테스트 수 | MySQL 시작 횟수 | 테스트 스위트 합계 | 실제 경과 시간 |
|---|---:|---:|---:|---:|
| 개선 전 | 106 | 5 | 239.222초 | 260초(4분 20초) |
| 개선 후 | 86 | 1 | 47.669초 | 53.73초 |

개선 전은 기능 브랜치, 개선 후는 `main`에서 분기한 성능 브랜치라 테스트 수가 달라 시간 감소율은 완전히 동일한 조건의 정밀 비교값이 아니다. 확실하게 검증한 지표는 한 번의 전체 테스트에서 MySQL 시작 로그가 5회에서 1회로 줄었고, 86개 테스트가 모두 통과했다는 점이다.

후속 확인에서는 최신 `main`을 반영한 성능 브랜치의 88개 테스트가 54초에 통과했다. 반대로 컨테이너 공유 개선이 아직 없는 PR #95 기반 브랜치에서는 파일 형식 리팩터링 후 120개 테스트가 통과했지만 suite 합계 `232.042초`, 실제 경과 시간 4분 5초가 걸렸고 종료 시 Hikari pool 5개가 관찰됐다. 테스트 수와 코드가 달라 시간 자체를 일대일 비교할 수는 없지만, 여러 Context가 DB 인프라를 각각 소유할 때 준비 시간이 다시 커지는 현상은 재현됐다.

## 한 문장 봉인

> Gradle은 테스트를 별도의 worker JVM에서 실행하고, JUnit은 메서드마다 테스트 **객체**를 새로 만들지만 ClassLoader가 로딩한 **클래스**의 `static`은 유지된다. 기존 컨테이너는 Spring Context 소유의 Bean이라 Context 구성마다 늘어났고, 이를 worker JVM에서 공유하는 `static`으로 옮긴 뒤 `@DynamicPropertySource`로 각 Context를 같은 MySQL에 연결했다.

## 오개념 교정 기록

- worker JVM은 운영 서버와 분리하기 위한 것이 아니라 Gradle 빌드 프로세스와 테스트 실행을 격리한다.
- 테스트 메서드마다 클래스가 다시 만들어지는 것이 아니라 테스트 객체가 다시 만들어진다.
- `static`은 단순히 "힙에 있어서" 공유되는 것이 아니라 로딩된 클래스에 속하기 때문에 공유된다.
- ClassLoader는 객체를 생성하지 않고 클래스 정의를 로딩한다.
- 클래스의 최초 로딩·초기화와 반복 코드를 최적화하는 JIT는 서로 다른 메커니즘이다.
- Spring Context는 ClassLoader가 아니다. ClassLoader는 클래스 정의를, Context는 Bean 객체를 관리한다.
- `@Import` 자체가 문제가 아니라 Import한 설정에서 컨테이너를 Context 소유 Bean으로 만든 것이 문제였다.
- `Supplier`가 컨테이너 시작 순서를 보장하는 것이 아니다. `static` 클래스 초기화가 먼저 컨테이너를 시작한다.

## 학습 피드백

### 잘된 점

- 정답을 받은 뒤 외우는 대신 `테스트 객체 → static → ClassLoader → Spring Context → Context Cache` 순서로 직접 설명하면서 경계를 좁혔다.
- “운영 JVM과 분리”를 “Gradle 빌드 프로세스와 테스트 worker JVM의 분리”로 스스로 교정했다.
- `instanceValue`가 스택에 있다는 첫 추측을 객체 필드는 힙에 있고 스택에는 지역 참조가 놓일 수 있다는 설명으로 수정했다.
- `@Import`가 문제라는 표면적 결론에서 멈추지 않고 Context마다 `@Bean` 메서드가 새 객체를 만든다는 실제 원인까지 도달했다.
- `Supplier`가 컨테이너를 시작한다고 단정하지 않고 static 초기화 순서와 값 조회 시점을 분리했다.
- IntelliJ의 6.564초와 Gradle의 긴 실행 시간이 충돌한다고 느낀 지점에서 직접 의문을 제기했고, XML의 testcase와 testsuite 시간을 비교해 측정 경계를 찾아냈다.

### 보완할 점

- `static은 JVM을 따라간다`보다 `특정 ClassLoader가 로딩한 특정 클래스 정의마다 static 필드 한 세트가 있다`고 말해야 예외 상황까지 설명할 수 있다.
- 성능 수치를 기록할 때는 브랜치·커밋·테스트 수·명령·측정 범위를 함께 남겨야 한다. 106개와 86개를 직접 비교하면서 원인과 개선 효과가 섞였다.
- 테스트를 빠르게 만드는 변경이 운영과 같은 트랜잭션 경계를 훼손하지 않는지도 함께 봐야 한다. 속도와 실제성은 별개의 검증 축이다.
- 슬라이스 테스트에서 예상하지 못한 Repository 의존성이 등장하면 “Service를 mock했는데 왜?”에서 멈추지 말고 요청이 Service에 도달하기 전의 Filter·ArgumentResolver 흐름을 먼저 그린다.

### 다음 인출 질문

1. `maxParallelForks = 2`라면 static MySQL 컨테이너는 왜 두 개가 될 수 있는가?
2. IntelliJ의 6.564초와 testsuite의 43.718초가 동시에 맞을 수 있는 이유는 무엇인가?
3. 공통 `@Transactional`이 통합 테스트를 편하게 만들면서도 운영 트랜잭션 경계를 가릴 수 있는 이유는 무엇인가?
4. Service를 mock한 MVC 슬라이스에서 `MemberRepository`가 필요한 요청 처리 순서를 설명할 수 있는가?

## 다음 씨앗

- Gradle Java 플러그인이 만드는 task와 `tasks.named()`가 기존 task를 설정하는 과정
- 테스트 worker JVM의 fork 수와 `maxParallelForks`
- Spring Test Context Cache의 정확한 캐시 키와 Context 재사용 조건
