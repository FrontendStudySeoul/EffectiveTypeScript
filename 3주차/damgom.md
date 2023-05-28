## 타입 추론
- 타입스크립트는 자동으로 변수의 타입을 추론한다.
- 타입 추론을 이용하면 타이핑 해야하는 일을 줄이고 타입스크립트의 타입 시스템을 적극적으로 사용할 수 있다.

### 타입스크립트가 타입 추론을 하는 경우
1. 변수 선언
    - 변수의 초기 값을 기준으로 타입을 추론한다.
    - 객체, 배열을 구조분해할당하는 경우에도 타입을 추론할 수 있다.

2. 함수의 반환 값

    - 함수의 반환 값을 기준으로 타입을 추론한다. 

3. 함수의 매개변수
   - 매개변수의 기본 값
   ``` ts
   function example(a = 1) { ... }
   ```
  
    - 라이브러리의 콜백함수의 매개변수는 주로 자동으로 추론된다

예상과는 다르게 타입을 지정하는 경우도 있다.

- 초기값이 없는 경우: `any`타입의 진화

``` ts
let d; // any

d = 10; // number
d.toFixed() // ✅
d.toUpperCase() // ❌ 오류 발생

d = 'hello' // string
d.toFixed() // ❌ 오류 발생
d.toUpperCase() // ✅
```

- 암묵적 `any` 타입이 지정된다.
- 할당하는 값에 따라 `any` 타입이 다른 타입으로 계속 진화한다.
- 명시적으로 `any` 타입으로 지정하는 것과는 다르다. 



### 타입 추론이 가능하더라도 명시적으로 타입 선언을 작성하는 것이 필요한 상황

1. 함수의 매개변수

``` ts
function func (param) { ... }  // param 매개변수에는 암시적으로 'any' 형식이 포함됩니다.
```

2. 객체 리터럴
   - 타입 구문 사용: 객체를 **선언**하는 곳에서 오류 발생
   - 타입 구문 미사용: 객체를 **사용**하는 곳에서 오류 발생 

3. 함수의 반환 값에 타입을 명시
   - 타입추론이 가능해도 함수 호출한 곳까지 오류가 영향을 미치지 않도록하기 위해 타입 구문을 명시하는 것이 좋다


> 추론 가능한 타입에 타입을 명시할 경우 error를 표시하는 eslint rule이 있다.
>
> https://typescript-eslint.io/rules/no-inferrable-types/


---

### Conditional Type으로 타입 추론하기

- 제네릭 타입 인자 꺼내오기
  - `Promise<number>`에서 `number `타입 꺼내오기


``` ts
type PromiseType<T> = T extends Promise<infer U> ? U : never;

type A = PromiseType<Promise<number>> // A: number
```

Conditional Type
- X extends Y: X 타입이 Y 타입에 할당될 수 있는지에 따라 참 값이 평가
  - true extends boolean: true는 boolean에 할당될 수 있으므로 참으로 평가된다.

- 조건식이 참으로 평가되면 infer 키워들르 사용할 수 있다. 

``` ts
Promise<number> extends Promise<infer U> ? U : never // number
number extends Promise<infer U> ? U : nuver // never
```
