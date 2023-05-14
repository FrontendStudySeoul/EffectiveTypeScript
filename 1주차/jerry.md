
## 아이템 1. 타입스크립트와 자바스크립트의 관계 이해하기

- 타입스크립트는 자바스크립트의 상위 집합(superset)이다.
- 타입스크립트는 자바스크립트의 런타임 동작을 모델링하는 타입 시스템을 가지고 있다.
- 타입 시스템은 런타임에 오류를 발생시킬 코드를 미리 찾아낸다.

<br />

## 아이템 2. 타입스크립트 설정 이해하기

- noImplicitAny : 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어 -> 모든 값에 타입을 정의해야 함
- strictNullChecks : null과 undefined가 모든 타입에서 허용되는지 확인하는 설정

  ```typescript
  // strictNullChecks: false
  const x: number = null; // 정상

  // strictNullChecks: true
  const x: number = null; // 에러 'null'형식을 'number'형식에 할당할 수 없습니다.
  ```

- strict : 모든 체크를 설정하고 싶은 경우 사용하며 대부분의 오류를 잡아낸다.

<br />

## 아이템 3. 코드 생성과 타입이 관계없음을 이해하기

### 타입스크립트 컴파일러의 역할

> 아래의 두 가지가 완벽히 독립적으로 이루어진다.

1. 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일링함.
2. 코드의 타입 오류를 체크함.

<br />

### 타입 오류가 있는 코드도 컴파일이 가능하다.

- 컴파일과 타입 체크는 독립적으로 이루어지므로 타입 오류가 있는 코드도 컴파일이 가능하다.
- 이때, '컴파일에 문제가 있다'는 틀린 말이며 타입의 오류가 있던 없던 컴파일은 가능하므로 엄밀히 말하면 '타입 체크에 문제가 있다'가 더 정확한 말이다.

<br />

### 런타임에는 타입 체크가 불가능하다.

타입스크립트는 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 모두 제거된다.

```typescript
// typescript
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    // ~~~~~~~~~ 'Rectangle' only refers to a type,
    //           but is being used as a value here
    return shape.width * shape.height;
    //         ~~~~~~ Property 'height' does not exist
    //                on type 'Shape'
  } else {
    return shape.width * shape.width;
  }
}
```

```javascript
// javascript
function calculateArea(shape) {
  if (shape instanceof Rectangle) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

컴파일 과정에서 타입과 관련된 구문은 모두 제거되므로 if문에서 선언되지 않은 Rectangle를 참조하므로 reference 에러가 발생한다.
타입스크립트에서는 Rectangle가 타입으로서 사용되지만 컴파일된 자바스크립트 파일에서는 값으로서 사용되어 에러가 발생하는 것이다.
즉, 타입은 런타임에 접근 불가하며 값은 런타임에 접근 가능하기 때문이다.

이를 런타임에 타입 정보를 유지하는 방법으로 해결할 수 있다.

- 애초에 instanceof 뒤에 타입이 아닌 값을 넣어 shape의 타입을 구분하는 방법
- 런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 태그 기법
- 타입(Square, Rectangle)을 클래스로 만들어 타입과 값을 둘 다 사용하는 방법

```typescript
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;
function calculateArea(shape: Shape) {
  if ("height" in shape) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

in 연산자를 이용해 shape이라는 값으로 shape의 타입을 구분하고 있기 때문에 에러가 발생하지 않는다.

<br />

### 타입스크립트 타입으로는 함수를 오버로드할 수 없다.

- 타입스크립트에서는 타입과 런타임 동작이 무관하기 때문에 함수 오버로딩은 불가능하다.
- 타입스크립트에서 함수 오버로딩을 지원하기는 하나 '타입 수준'에서만 동작한다.
- 또한 하나의 함수에 대해 여러 선언문을 작성할 수 있으나 구현체는 하나여야 한다.

```typescript
// 타입 정보로서 함수 오버로드 가능
function add(a: number, b: number): number;
function add(a: string, b: string): string;

// 구현체는 반드시 하나여야 한다.
function add(a, b) {
  return a + b;
}

const one = add(1, 0); // 타입이 number
const ten = add("1", "0"); // 타입이 string
```

<br />

## 아이템 4. 구조적 타이핑에 익숙해지기

```typescript
interface Vector2D {
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}

interface NamedVector {
  name: string;
  x: number;
  y: number;
}

const v: NamedVector = { x: 3, y: 4, name: "Zee" };
calculateLength(v); // OK, result is 5
```

변수 v는 Vector2D 타입이 아니라 NamedVector 타입임에도 불구하고 calculateLength 함수 실행이 정상적으로 이루어진다.
그 이유는 NamedVector 내 Vector2D 타입 속성인 x와 y 속성이 있어 NamedVector 구조가 Vector2D와 호환되기 때문에 함수를 에러없이 호출할 수 있는 것이다.

이렇게 타입스크립트는 정확하고 봉인된 타입만을 받는 것을 넘어 열린 타입들도 받는 특성으로 타입 확장이 가능해진다.

<br />

## 아이템 5. any 타입 지양하기

- any 타입은 타입 안정성이 없다.
- any는 함수 시그니처를 무시한다. -> 매개변수와 리턴값의 타입 무시

<br />

## 아이템 6. 편집기를 사용해 타입 시스템 탐색하기

- 편집기를 사용하면 어떻게 타입 시스템이 동작하는지, 타입스크립트가 어떻게 타입을 추론하는지 알 수 있다.
- 타입스크립트가 어떻게 모델링하는지 알고 싶다면 타입 선언 파일을 찾아보자.

<br />

## 아이템 7. 타입이 값들의 집합이라고 생각하기

### never 타입

- 가장 작은 집합의 타입
- 선언된 변수의 범위가 공집합이기 때문에 아무런 값도 할당할 수 없다.
- never 타입은 도달하면 안되는 곳에 보통 할당하며 예를 들어 에러 객체가 있다.

<br />

### 리터럴 타입

- 유닛(unit) 타입이라고도 불린다.
- 한가지 값만 포함되는 타입

```typescript
type A = "A";
type Ten = 10;

// const는 재할당이 불가능한 변수이므로 선언한 값 그대로를 리터럴 타입으로 갖는다.
const Ten = 10; // 타입은 10이다.
let Ten = 10; // 타입은 number이다.
```

<br />

### 유니온 타입

- `|` 연산자를 이용해 여러 값들을 묶어 타입으로 선언할 수 있다.
- 유니온 타입은 값 집합들의 합집합을 일컫는다.

<br />

### 인터섹션 타입

- `&` 연산자를 이용해 두 타입을 결합할 수 있다.

```typescript
interface Person {
  name: string;
}

interface Lifespan {
  birth: Date;
  death?: Date;
}

type PersonSpan = Person & Lifespan;
```

- 또는 `extends` 키워드를 이용해 PersonSpan 타입을 선언할 수 있다.
- `extends`는 '~에 할당 가능한'과 비슷하게 '~의 부분 집합'이라는 뜻이다.

```typescript
interface Person {
  name: string;
}

interface PersonSpan extends Person {
  birth: Date;
  death?: Date;
}
```

<br />

### 서브 타입

- 'A는 B를 상속', 'A는 B에 할당 가능', 'A는 B의 서브타입'은 'A는 B의 부분 집합'과 같은 의미이다.
- `extends`는 제너릭 타입에서 한정자로도 쓰인다.

  아래의 예시에서 K 제너릭 타입은 string 타입을 상속한다. 즉, 문자열만 받을 수 있다.
  string 타입을 상속한다는 의미를 집합의 관점으로 보면 K는 string 리터럴 타입, string 리터럴 타입의 유니온, string 자신을 포함한다.

  ```typescript
  function getKey<K extends string>(val: any, key: K) {
    // ...
  }

  getKey({}, "x"); // OK, 'x' extends string => string 리터럴 타입
  getKey({}, Math.random() < 0.5 ? "a" : "b"); // OK, 'a'|'b' extends string => string 리터럴 타입의 유니온
  getKey({}, document.title); // OK, string extends string => string
  getKey({}, 12); // string 타입 부분 집합에 해당하지 않는 number 타입이므로 에러가 발생한다.
  ```

<br />

### keyof

keyof는 해당 타입의 키 값을 유니온 타입으로 반환한다.

```typescript
interface Point {
  x: number;
  y: number;
}

type PointKeys = keyof Point; // Type is "x" | "y"

function sortBy<K extends keyof T, T>(vals: T[], key: K): T[] {
  // ...
}

const pts: Point[] = [
  { x: 1, y: 1 },
  { x: 2, y: 0 },
];

// T는 Point가 되고 K는 T의 key 값으로 이루어진 유니온 타입을 칭한다.
sortBy(pts, "x"); // OK, 'x' extends 'x'|'y' (aka keyof T)
sortBy(pts, "y"); // OK, 'y' extends 'x'|'y'
sortBy(pts, Math.random() < 0.5 ? "x" : "y"); // OK, 'x'|'y' extends 'x'|'y'
sortBy(pts, "z"); // 'z'는 "x" | "y"에 속하지 않으므로 에러 발생
```

<br />

### 배열과 튜플의 관계

list 타입은 `number[]` 타입이지만, list 요소가 튜플에 선언한 요소의 개수보다 적을 가능성이 있으므로 에러가 발생한다.
즉, `number[]`는 `[number, number]`의 부분집합이 아니므로 할당할 수 없는 것이다.
하지만 이 반대의 경우는 정상적으로 동작한다.

```typescript
const list = [1, 2]; // Type is number[]
const tuple: [number, number] = list; // 에러 발생
```

<br />

### Exclude

일부 타입을 제외할 수 있지만, 타입스크립트 타입일 때만 정상적으로 동작한다.
`Exclude`는 유틸리티 타입으로 선언되며 콤마를 기준으로 좌측의 타입에서 우측의 유니온 타입을 제거한 나머지를 타입으로 갖는다.

```typescript
type T = Exclude<string | Date, string | number>; // 타입은 Date

// 0과 같이 값이 아닌 타입을 작성해야 Exclude가 유효하다.
type NonZeroNums = Exclude<number, 0>; // 타입은 여전히 number
type NonZeroNums = Exclude<number, number>; // 타입은 never
```

<br />

## 아이템 8. 타입 공간과 값 공간의 심벌 구분하기

- 상황에 따라 값으로 쓰이거나 타입으로 쓰일 수 있지만 이 경우 오류를 발생시킬 수 있다.
- `class`나 `enum` 같은 키워드는 타입과 값 두가지로 사용될 수 있다.

```typescript
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });

function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape.radius; // 에러
  }
  /*
    이 경우 instanceof 뒤에 쓰인 Cylinder는 값으로 연산되므로 에러를 발생한다.
    다시 말해, instanceof는 런타임 연산자이므로 값에 대해 연산을 하며, Cylinder는 인터페이스 타입이 아닌 함수를 참조하게 되어 에러가 발생한다.
  */
}
```

- 타입 선언`(:)`과 단언문`(as)` 다음에 나오는 심벌은 타입이며
- = 다음에 나오는 모든 것들은 값이다.

<br />

### typeof를 이용해 타입의 공간과 값 공간 구별하기

- 타입의 관점에서는 `typeof`는 값을 읽어 타입스크립트 타입을 반환한다.
- 값의 관점에서는 `typeof`는 자바스크립트 런타임의 `typeof` 연산자가 되므로 값 공간의 `typeof`는 대상 심벌의 런타임 타입을 가리키는 문자열을 반환하며 타입스크립트의 타입과는 다르다.

```typescript
// 타입
type T1 = typeof p; // 타입은 Person
type T2 = typeof Cylinder; // 타입은 (radius: number, height: number) => { radius: number; height: number; }

// 값
const v1 = typeof p; // 값은 object
const v2 = typeof Cylinder; // 값은 function
```

<br />

### 속성 접근자 []

- `obj['field']`와 `obj.field`는 값이 동일하지만
- 타입의 경우 다를 수 있으므로 타입의 속성을 얻고자 한다면 반드시 `[]`로 접근해야 한다.

```typescript
const first: Person["first"] = p["first"];
const first: Person["first"] = p.first;
```

- `[]` 내부에는 유니온 타입과 기본형 타입을 포함한 어떠한 타입이든 사용할 수 있다.

```typescript
type PersonEl = Person["first" | "last"]; // 타입은 string
type Tuple = [string, number, Date];
type TupleEl = Tuple[number]; // 타입은 string | number | Date
// Tuple의 인덱스에는 0, 1, 2 즉, 숫자만 허용되며 대괄호 내부에 number를 작성하면 가능한 타입의 유니온 타입을 반환한다.
// Tuple[number] => Tuple[0] | Tuple[1] | Tuple[2]
```

<br />

### 구조 분해 할당

- 타입 공간과 값 공간을 명확히 구분하자.
- 구조 분해 할당의 원리로 `:` 뒤에 쓰인 값은 변수명에 불과하다.
- 즉, `:` 뒤에 쓰인 Person과 string이 값으로서 해석되었으므로 에러가 발생했다.

```typescript
interface Person {
  first: string;
  last: string;
}

function email({ person: Person, subject: string, body: string }) {}
// string이라는 변수명이 중복되어 에러 발생
```

```typescript
interface Person {
  first: string;
  last: string;
}

function email({
  person,
  subject,
  body,
}: {
  person: Person;
  subject: string;
  body: string;
}) {} // OK
```

<br />
