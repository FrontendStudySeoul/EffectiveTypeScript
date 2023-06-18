## any 다루기
### any 타입은 가능한 한 좁은 범위에서만 사용하기

``` ts
// ❌ any 스코프: 선언 ~ 함수의 마지막까지
function f1() {
  const x: any = returnFoo();
  test(x);
}

// ✅ any 스코프: 호출 이후에 any
function f2() {
  const x = returnFoo();
  test(x as any);
}
```

- f1 함수가 any 타입인 x를 반환하게 되는 경우 문제가 커진다.

``` ts
function g() {
  const foo = f1(); // any
  foo.fooMethod()l; // 함수 호출이 체크되지 않는다.
}
```

- 반환 타입을 추론할 수 있는 경우에도 함수의 반환 타입을 명시하는 것이 좋다.
- 함수의 반환 타입을 명시하면 any 타입이 함수 바깥으로 영향을 미치는 것을 방지할 수 있다.
- 강제로 타입 오류를 제거하고 싶다면 any 타입 대신 @ts-ignore를 사용하는 방법도 있다.

### 객체와 관련한 any 사용법
``` ts
// ❌ 객체 전체를 any로 단언하기
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value;
  }
} as any;

// ✅ 최소한의 범위에만 any 적용하기
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value as any;
  }
}
```

- 객체 전체를 as any로 단언하게 되는 경우, a와 b 타입 체크가 불가능하다.
- 두번째 코드처럼 필요한 곳에만 any를 사용하면 a와 b의 타입 체크가 가능하다.
