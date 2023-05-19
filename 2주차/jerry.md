## 아이템 9. 타입 단언보다는 타입 선언을 사용하기

- 타입 선언`(: Type)`은 값이 해당 타입이어야하고 만약 아니면 에러를 발생시킨다.
- 타입 단언`(as Type)`은 어떤 값이든 그 값을 해당 타입으로 간주한다.
- 단, 타입 단언을 사용할 때 타입의 형태를 비슷해야 한다.
- 타입 단언은 타입 체커에게 오류를 무시하도록 한다.
- 또한 타입 선언은 잉여 속성 체크가 동작하며, 타입 단언에는 동작하지 않는다.

```typescript
interface Person {
  name: string;
}

const alice: Person = { name: "Alice" }; // Type is Person
const bob = { name: "Bob" } as Person; // Type is Person

const alice: Person = {}; // {}라는 값이 Person이라는 타입이 아니므로 에러를 발생시킨다.
const bob = {} as Person; // {}라는 값이 Person 타입은 아니지만 타입 단언에 의해 Person 타입을 갖는다.
const bob = [] as Person; // 에러 발생
```

<br />

### 타입 단언을 사용해야 하는 경우

- DOM 엘리먼트에 대해서는 타입 단언문을 사용하는 것이 좋다.
- `!` 연산자(Non null assertion)을 사용해 null이 아님을 단언한다.

<br />

### 타입 단언의 오해

- 타입 단언문으로 임의의 타입 간의 변환을 할 수는 없다.
- 타입 단언문은 서브 타입에 한해 동작한다.

### 타입 단언 대신 타입 명제(type predicate) 사용하기

```typescript
type Circle = { type: "circle"; radius: number };
type Rect = { type: "rect"; width: number; height: number };
type Shape = Circle | Rect;

const shapes: Shape[] = getShapes();

// 타입 선언으로 타입 좁히기 -> 타입스크립트가 필터링 된 것을 모르기 때문에 에러 발생
const circles: Circle[] = shapes.filter((shape) => shape.type === "circle");

// 단언을 추가한 경우 -> 에러는 발생하지 않는다.
// const circles = myShapes.filter(isCircle) as Circle[];

// 타입 명제를 사용해 filter 호출 후에 타입을 좁힐 수 있도록 한다.
// filter 내부에 작성된 콜백 함수가 true를 반환한다면 그 객체는 Circle 타입임을 명시하는 것이다.
const circles = shapes.filter(
  (shape): shape is Circle => shape.type === "circle"
);
```

<br />

## 아이템 10. 객체 래퍼 타입 피하기

### 객체 래퍼 타입의 주의 사항

- 런타임의 값은 객체가 아니고 기본형이다.
- 기본형 타입은 객체 래퍼에 할당할 수 있기 때문에 타입스크립트는 기본형 타입을 객체 래퍼에 할당하는 선언을 허용한다.
- 즉, string은 String에 할당할 수 있지만 String은 string에 할당할 수 없다.
- 결론 : 타입스크립트 객체 래퍼 타입은 지양하고 기본형 타입을 사용하자.

<br />

## 아이템 11. 잉여 속성 체크의 한계 인지하기

### 잉여 속성 체크란

- 잉여 속성 체크 = 엄격한 객체 리터럴 체크
- 잉여 속성 체크란, '해당 타입의 속성이 있는지'와 '그 외의 속성은 없는지' 확인하는 작업을 말한다.
- 잉여 속성 체크를 이용하면 기본적으로 타입 시스템의 구조적 본질을 해치지 않으면서도 객체 리터럴에 알 수 없는 속성을 허용하지 않게 할 수 있다.
- 타입이 명시된 변수에 객체 리터럴을 할당할 때, 함수 매개변수로 객체 리터럴을 전달할 때 타입스크립트는 잉여 속성을 체크한다.

<br />

### 구조적 타입핑과 잉여 속성 체크, 잉여 속성 체크의 한계

- 하지만, 구조적 타입핑 관점에서 보면 오류가 발생하지 않아야 한다.
- 임시 변수를 도입하면 잉여 속성 체크를 건너뛸 수 있다.
- 임시 변수인 obj를 r에 할당하면 오류가 발생하지 않는다. 그 이유는 obj의 타입은 `{ numDoors: 1, ceilingHeightFt: 10, elephant: "present" }`로 추론되며 obj 타입은 Room 타입의 부분 집합을 포함하므로 Room에 에러 없이 할당 가능한 것이다.

```typescript
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};

const r: Room = obj; // OK
```

1. 타입 단언문을 이용해 잉여 속성 체크하지 않기

```typescript
const o = { darkmode: true, title: "Ski Free" } as Options; // OK
```

2. 인덱스 시그니처을 이용해 잉여 속성 체크하지 않기

```typescript
interface Options {
  darkMode?: boolean;
  [otherOptions: string]: unknown;
}

const o: Options = { darkmode: true }; // OK
```

<br />

## 아이템 12. 함수 표현식에 타입 적용하기

- 타입스크립트에서는 함수 문장(funtcion 키워드)보다 함수 표현식을 사용하는 것이 좋다.
- 동일한 타입을 가지는 함수를 여러 개 작성할 때는 매개변수 타입과 반환 타입을 반복해 작성하지 말고 **함수 전체의 타입 선언**을 작성하는 것이 좋다.

```typescript
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

<br />

## 아이템 13. 타입과 인터페이스의 차이점 알기

> 대부분의 경우에는 타입을 사용해도 되고 인터페이스를 사용해도 된다.

### type과 interface의 비슷한 점

1. 추가 속성과 함께 할당한다면 오류가 발생한다. => 잉여 속성 체크
2. 인덱스 시그니처는 interface와 type에서 모두 사용할 수 있다.
3. 함수 타입도 interface나 type으로 정의할 수 있다.
4. type과 interface는 모두 제너릭이 가능하다.
5. interface는 타입을 확장할 수 있으며, type은 인터페이스를 확장할 수 있다.
6. 클래스를 implements할 때는 type과 interface 모두 사용할 수 있다.

### type과 interface의 다른 점

1. interface

   - interface는 유니온 같은 타입을 확장하지 못한다.
   - 유니온 타입은 있지만 유니온 interface라는 개념은 없다.

   ```typescript
   // 이 타입은 interface로 표현할 수 없다.
   type NamedVariable = (Input | Output) & { name: string };
   ```

   - interface로도 튜플을 구현할 수는 있지만 type을 이용해 구현하는 것이 더 낫다.
   - interface로 튜플과 비슷하게 구현하면 튜플에서 사용할 수 있는 배열 메서드를 사용할 수 없다.
   - 보강(augment) 기능 : 선언 병합
     - 선언 병합을 위해서는 interface를 사용해야 한다.
     - 최신 버전의 타입스크립트 정의 파일을 보면 선언 병합을 이용해 이전 버전의 타입을 병합하고 있음을 알 수 있다.
     - 그러므로 프로퍼티가 추가되는 것을 원하지 않는다면 interface 대신 type을 이용해야 한다.

<br />

2. type은 interface보다 더 쓰임새가 많다.

- 유니온 사용 가능
- 매핑된 타입 가능
- 조건부 타입 가능
- 튜플 타입을 더 간결하게 표현 가능
  - type으로 튜플 타입을 만들면 바로 배열로 인식하므로 배열 메서드를 에러 없이 사용할 수 있다.

<br />

## 아이템 14. 타입 연산과 제너릭 사용으로 반복 줄이기

### 필드 선언을 복제하는 대신 Pick 제너릭

```typescript
// Pick 제너릭 정의
type Pick<T, K extends keyof T> = { [k in K]: T[k] };

type TopNavState = Pick<State, "userId" | "pageTitle" | "recentFiles">;
```

<br />

### 선택적 필드, Partial 제너릭

1. 매핑된 타입과 keyof의 사용해 중복 코드 줄이기

   - 매핑된 타입(`[k in keyof Options]`)은 순회하며 Options 내 k값에 해당하는 속성이 있는지 찾는다.
   - `?`는 선택적 필드를 만든다.

   ```typescript
   type OptionsUpdate = { [k in keyof Options]?: Options[k] };
   ```

2. Partial 제너릭

   - 특정 타입의 부분 집합 타입을 정의할 수 있다.

   ```typescript
   type Partial<T> = { [P in keyof T]?: T[P] | undefined };
   ```

<br />

### 함수나 메서드의 반환 값에 대한 명명된 타입, ReturnType 제너릭

```typescript
// 반환 값의 타입은 { name: string; age: number }으로 추론된다.
function getUserInfo(userId: string) {
  //...
  return {
    name,
    age,
  };
}

type UserInfo = ReturnType<typeof getUserInfo>;
```

<br />

### 제너릭 타입과 extends

- 제너릭 타입에 extends를 이용해 매개변수를 제한할 수 있다.

```typescript
// extends로 T에 대한 타입을 Name 타입으로 제한한 것이다.
type DancingDuo<T extends Name> = [T, T];
```

### Record와 satisfies

```typescript
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  bleu: [0, 0, 255], // blue 오타
};

// 'bleu' 오타를 잡는 방법
type Colors = "red" | "green" | "blue";

const palette: Record<Colors, [number, number, number] | string> = {
  red: [255, 0, 0],
  green: "#00ff00",
  bleu: [0, 0, 255], // 오타를 잡는다.
};

// 하지만 green이 string 타입이라는 것을 추론하지 못한다.
// green => string | [number, number, number]

// satisfies를 사용하면 red, green, bleu 값이 배열, 문자열, 배열로 추론된다.
// 또한 bleu 오타도 잡는다.
const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  bleu: [0, 0, 255], // blue 오타
} satisfies Record<Colors, [number, number, number] | string>;

/*
const palette: {
  red: [number, number, number];
  green: string;
  blue: [number, number, number];
}
*/
```

<br />

## 아이템 15. 동적 데이터에 인덱스 시그니처 사용하기

### 인덱스 시그니처

- 타입에 '인덱스 시그니처'를 명시해 유연하게 매핑할 수 있다.
- 데이터의 키 값의 이름을 미리 알지 못하는 경우, 즉 동적 데이터를 표현할 때 사용한다.
- 런타임 때까지 객체의 속성을 알 수 없는 경우에만 인덱스 시그니처를 사용해야 한다.
- 만약, 타입에 가능한 필드가 제한되어 있는 경우 인덱스 시그니처로 모델링하지 말아야 한다.
- 가능한 인덱스 시그니처보다 인터페이스, Record, 매핑된 타입과 같은 정확한 타입을 사용하는 것이 좋다.

```typescript
type Rocket = { [property: string]: string };
const rocket: Rocket = {
  name: "Falcon 9",
  variant: "v1.0",
  thrust: "4,940 kN",
}; // OK
```

<br />

### 인덱스 시그니처의 문제

- 잘못된 키를 포함해 모든 키를 허용한다는 점
  - name 대신 Name으로 작성해도 Rocket 타입이 된다.
  - 자동 완성 기능이 동작하지 않는다.
- `{}`도 유효한 Rocket 타입이 된다.
- 키마다 다른 타입을 가질 수 없다는 문제
- 결론: 인덱스 시그니처는 부정확하다는 문제가 있다.

<br />

### 데이터의 개수를 알지 못하는 경우

- 미리 데이터의 개수를 알지 못하는 경우에는 선택적 필드 또는 유니온 타입으로 모델링한다.
- 이 경우 인덱스 시그니처로 모델링할 시 타입이 너무 광범위해진다.
- 이를 다음 두 가지 방법으로 해결할 수 있다.

1. Record 제너릭 사용

- Record는 키 타입에 유연성을 제공하는 제너릭 타입이다.
- K는 키 값의 타입으로 가능한 타입들을 부분 집합으로 갖는다.

```typescript
type Vec3D = Record<"x" | "y" | "z", number>;
```

2. 매핑된 타입을 사용

```typescript
// 위 Vec3D와 같다.
type Vec3D = { [k in "x" | "y" | "z"]: number };

// 조건부 타입(? 연산자)
type ABC = { [k in "a" | "b" | "c"]: k extends "b" ? string : number };
```

<br />
