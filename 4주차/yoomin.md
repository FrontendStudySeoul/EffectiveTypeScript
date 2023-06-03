# 아이템 26

타입스크립트는 타입 추론 시 문맥까지 살핀다.

그러나, 이는 원치 않은 결과를 초래할 수도 있다.

## 문자열 리터럴 사용 시 주의점

아래와 같이 Language 타입을 선언하고 함수를 만들었다고 하자.

```tsx
type Language = "JavaScript" | "TypeScript" | "Python";
function setLanguage(language: Language) {
  /* ... */
}
```

다음 코드는 문제 없이 작동한다.

```tsx
setLanguage("JavaScript");
```

그러나, 다음 코드에서는 에러가 발생한다.

```tsx
let language = "JavaScript";
setLanguage(language);
```

이는 'JavaScript'가 string으로 추론되었고, string 형식의 인수가 Language 형식의 매개변수에 할당될 수 없기 때문이다.

**첫 번째 해결 방법은 타입 선언에서 language의 가능한 값을 제한하는 것이다.**

```tsx
let language: Language = "JavaScript";
setLanguage(language);
```

**두 번째 해결 방법은 language를 상수로 만드는 것이다.**

```tsx
const language = "JavaScript";
setLanguage(language);
```

const로 선언하면 language가 'JavaScript'에서 다른 값으로 변경될 수 없기에 string이 아닌 JavaScript라는 문자열 리터럴로 타입이 추론되어 정상적으로 작동한다.

## 튜플 사용 시 주의점

아래와 같은 함수를 만들었다고 하자.

```tsx
function panTo(where: [number, number]) {
  /* ... */
}
```

다음 코드는 문제 없이 작동한다.

```tsx
panTo([10, 20]);
```

그러나, 다음 코드에서는 에러가 발생한다.

```tsx
const loc = [10, 20];
panTo(loc);
```

이는 loc의 타입이 number[]로 추론되었고, number[] 형식의 인수가 [number, number] 형식의 매개변수에 할당될 수 없기 때문이다.

**첫 번째 해결방법은 다음과 같다.**

```tsx
const loc: [number, number] = [10, 20];
panTo(loc);
```

**두 번째 해결방법은 as const를 사용하는 것이다.**

```tsx
const loc = [10, 20] as const;
panTo(loc);
```

그런데, 위 코드에서도 문제가 있다. loc이 readonly [10, 20]으로 과하게 정확하게 추론되고 있는 것이다.

이 때 해결 방법은 panTo 함수에 readonly 구문을 추가하는 것이다.

```tsx
function panTo(where: readonly [number, number]) {
  /* ... */
}
const loc = [10, 20] as const;
panTo(loc);
```

as const는 문맥 손시로가 관련된 문제를 해결할 수 있지만, 타입 정의에 실수가 있을 시 오류가 호출되는 곳에서 발생하는 단점이 있다.

## 객체 사용 시 주의점

다음과 같이 type, interface을 정의하고 함수를 만들었다고 하자.

```tsx
type Language = "JavaScript" | "TypeScript" | "Python";
interface GovernedLanguage {
  language: Language;
  organization: string;
}

function complain(language: GovernedLanguage) {
  /* ... */
}
```

다음 코드는 문제 없이 작동한다.

```tsx
complain({ language: "TypeScript", organization: "Microsoft" });
```

그러나, 다음 코드에서는 에러가 발생한다.

```tsx
const ts = {
  language: "TypeScript",
  organization: "Microsoft",
};
complain(ts);
```

ts의 language가 string 타입으로 추론되기 때문이다.

마찬가지로 해결 방법이 두 가지 있는데, const ts: GovernedLanguage로 선언하거나 as const를 사용하면 된다.

## 콜백 사용 시 주의점

다음 코드를 살펴보자.

```tsx
function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
  fn(Math.random(), Math.random());
}

callWithRandomNumber((a, b) => {
  console.log(a + b);
});
```

a와 b의 타입이 number로 추론된다.

다음과 콜백을 상수로 뽑아냈다고 해보자.

```tsx
const fn = (a, b) => {
  console.log(a + b);
};
callWithRandomNumbers(fn);
```

이 때는 문맥이 소실되고 noImplicitAny 오류가 발생한다.

이런 경우에는 매개변수에 타입 구문을 추가해 해결할 수 있다.

```tsx
const fn = (a: number, b: number) => {
  console.log(a + b);
};
callWithRandomNumbers(fn);
```

함수 전체에 타입 선언을 사용할 수도 있다.
