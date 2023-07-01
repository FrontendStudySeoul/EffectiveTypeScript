## 아이템 52. 테스팅 타입의 함정에 주의하기

> 타입을 테스트해보자.

### 반환값에 대한 타입 체크가 없으면 완전한 테스트가 아니다.

```typescript
declare function map<U, V>(array: U[], fn: (u: U) => V): V[];

map(["2017", "2018", "2019"], (v) => Number(v)); //
map("10", (v) => Number(v));
```

만약 map의 첫 번째 매개변수에 단일 값을 넣었다면 매개변수 타입에 대한 오류는 잡을 수 있지만 반환값에 대한 체크가 없어 완전한 테스트라고 할 수 없다.

위 코드는 실행'에서 오류가 발생하지 않는지만 체크한다.
즉, 실행의 결과 타입도 체크하는 것이 더 좋은 테스트 코드이다.

<br />

### 헬퍼 함수 정의

```typescript
function assertType<T>(x: T) {}

assertType<number[]>(map(["john", "paul"], (name) => name.length));
```

<br />

1. 덕 타이핑

```typescript
const beatles = ["john", "paul", "george", "ringo"];
assertType<{ name: string }[]>(
  map(beatles, (name) => ({
    name,
    inYellowSubmarine: name === "ringo",
  }))
); // OK
```

`{ name: string, inYellowSubmarine: boolean }` 객체의 배열을 반환하지만 inYellowSubmarine 속성 부분이 체크되지 않는 문제가 발생한다.

2. 매개변수

```tsx
const add = (a: number, b: number) => a + b;
assertType<(a: number, b: number) => number>(add); // OK

const double = (x: number) => 2 * x;
assertType<(a: number, b: number) => number>(double); // OK!?
```

타입스크립트의 함수는 매개변수가 더 적은 함수 타입에 할당 가능하기 때문에 이상한 결과가 나타난다.

<br />

### 반환 타입 체크하기

매개변수 타입과 반환 타입을 분리해서 테스트할 수 있다.

```typescript
// null!의 의미: 초기값은 null이지만 null이 아니라고 가정
const double = (x: number) => 2 * x;
let p: Parameters<typeof double> = null!; // [number] 타입
assertType<[number, number]>(p); // [number, number] != [number]이므로 에러 발생

let r: ReturnType<typeof double> = null!; // number 타입
assertType<number>(r); // OK

<br />;
```

### 결론: [dtslint](https://github.com/microsoft/dtslint)

`// $ExpectType` 단언문을 통해 `_.pluck`의 반환 타입을 체크할 수 있다.

```tsx
var stooges = [
  { name: "moe", age: 40 },
  { name: "larry", age: 50 },
  { name: "curly", age: 60 },
];
_.pluck(stooges, "name"); // $ExpectType string[]
```

[코드 출처](https://medium.com/hackernoon/testing-types-an-introduction-to-dtslint-b178f9b18ac8)
