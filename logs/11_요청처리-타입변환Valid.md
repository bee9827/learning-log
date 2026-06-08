# 학습 로그 #11

**시간**: 05/18 (약 __ 분)
**학습 범위**: Spring 요청 처리 흐름 — 타입 변환과 @Valid의 관계

분류: [[Spring웹-요청처리]]  #Spring웹-요청처리

## 1. 막힌 것의 종류

- [ ] 개념 자체를 모르겠다
- [ ] 개념은 알겠는데 코드로 어떻게 쓰는지 모르겠다
- [ ] 코드는 돌아가는데 이게 맞는 건지 모르겠다
- [x] 기타: "사소한 궁금"에서 출발 — MethodArgumentTypeMismatchException이 왜 필요한가?

## 2. 이번 타임의 학습 전략

- 이전에 바꾸기로 한 전략은 무엇이었고, 실행했는가?
  1. "왜"까지 연결하기 — ConversionService가 왜 필요한지, @Valid가 왜 그 이후인지 근거까지 연결했다 ✅
  2. 의도적으로 의문을 키워드로 남기기 — "사소한 궁금"을 학습 의문으로 인식해서 시작점으로 삼았다 ✅

### 학습 과정

#### 의문의 시작

`MethodArgumentNotValidException`, `HttpMessageNotReadableException`만 있으면 되지 왜 `MethodArgumentTypeMismatchException`이 필요한가?

#### Spring 요청 처리 흐름

```
HttpServletRequest
  → DispatcherServlet
  → HandlerMapping (Handler 조회)
  → HandlerAdapter
    → ArgumentResolver
      → @RequestBody: HttpMessageConverter (JSON → 객체) → @Valid
      → @PathVariable: ConversionService (String → 타입) → @Valid
    → Handler 실행
    → ReturnValueHandler → HttpMessageConverter (객체 → JSON)
```

#### 핵심 — HttpMessageConverter vs ConversionService

**HttpMessageConverter**
- JSON 바디를 Java 객체로 변환
- `@RequestBody`에 관여

**ConversionService**
- URL 경로/쿼리 파라미터를 타입으로 변환
- `@PathVariable`, `@RequestParam`에 관여
- `String "abc"` → `Long` 변환 실패 시 `MethodArgumentTypeMismatchException` 발생

#### @Valid와의 관계

```
@PathVariable 흐름:
ConversionService (타입 변환) → 실패: TypeMismatchException (여기서 종료)
                              → 성공: @Valid (값 검증)
```

타입 변환 실패는 `@Valid`까지 도달하지 못한다. 단계가 다르기 때문에 별도 예외가 필요하다.

## 3. 전략 평가

- 효과적이었던 것과 그 이유
  "사소한 궁금"을 학습 의문으로 인식한 것이 효과적이었다.
  log_01에서 DispatcherServlet 흐름을 배웠는데, 오늘 `ConversionService`와 `@Valid` 순서를 연결하면서 한 층 깊어졌다. 나선형 학습이 제대로 작동한 사례다.

- 비효과적이었던 것과 그 이유
  `ConversionService`가 정확히 어떤 구현체를 쓰는지, 커스텀 타입 변환은 어떻게 등록하는지는 다루지 않았다.

- 막힌 것의 종류(1번)와 전략의 궁합은 어땠는가?
  "사소하다"고 느꼈던 의문이 실제로 흐름의 핵심을 짚고 있었다. 의문의 크기와 학습 가치는 비례하지 않는다.

## 4. AI 피드백

오늘 두 가지 "바꿀 것"이 동시에 작동했다. 특히 "사소한 궁금"을 학습 의문으로 인식한 순간이 중요하다. 앞으로 "이건 그냥 궁금한 거야"라고 넘어가려 할 때, 한 번 더 "이게 어떤 흐름과 연결되는 건지?" 를 물어보는 습관이 생기면 의도적인 사이클이 더 자주 만들어진다.

## 5. 다음 타임에 바꿀 것

- 유지할 것과 그 이유
  트레이드오프를 짚어주고 내 생각을 끄집어내는 방식 — 다음 학습 키워드를 능동적으로 정할 수 있었다.

- 바꿀 것과 그 이유
  5번 작성 시 AI가 추상적으로 묻지 않고 비율 기반으로 접근하기.
  - 30% 이하: 체득 우선, 평가 강요 없음
  - 50% 내외: 구체적 순간을 짚어서 끌어냄
  - 70% 이상: 컴포트존 신호, 변화 유도

  다음 과제: 비율을 확인하는 방법 자체를 체계화하기 — 지금은 주관적 판단에 의존하므로 다음 사이클에서 기준을 만들어본다.

---

## 다음 사이클 재방문 키워드

1. **@RequestParam** — @PathVariable과 같은 ConversionService를 쓰는가, 다른 흐름인가
   → 종류: 흐름 파악
2. **@JsonFormat** — HttpMessageConverter 단계에서 처리되는가
   → 종류: 흐름 파악
3. **@Valid 내부 동작** — 어떤 Validator가 실행되는가
   → 종류: 흐름 파악 → 코드 적용
4. **ConversionService 커스텀 타입 변환** — 직접 만든 타입(Name 등)을 PathVariable로 받으려면 어떻게 등록하는가
   → 종류: 코드 적용
5. **checked vs unchecked 예외와 롤백** — (log_04에서 이월)
   → 맥락: @Transactional이 걸린 메서드에서 checked 예외가 발생했을 때 롤백이 안 되는 시나리오
6. **트랜잭션 전파와 중첩 트랜잭션** — (log_04에서 이월)
   → 맥락: AdminService와 ReservationService가 서로 호출할 때 어떻게 되는가
