# 이펙티브 타입스크립트

## 아이템4. 구조적 타이핑에 익숙해지기

### 덕 타이핑

> 만약 어떤 새가 오리처럼 걷고, 헤엄치고, 꽥꽥거리는 소리를 낸다면 나는 그 새를 오리라고 부를 것이다.

어떤 함수의 매겨변수 값이 모두 제대로 주어진다면, 그 값이 어떻게 만들어졌는지 신경 쓰지 않고 사용한다.

```tsx
interface Vertor2D {
  x: number;
  y: number;
}

interface NamedVector {
  name: string;
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}
```

```tsx
interface Vector3D {
  x: number;
  y: number;
  z: number;
}

function normalize(v: Vector3D) {
  const length = calculateLength(v);

  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}
```

`sealed`, `precise`: 매개 변수의 타입에 선언된 속성만 가질 것이라고 기대

`open`: 타입에 선언된 속성 외에 임의의 속성을 추가하더라도 오류가 발생하지 않음

- 고양이 타입에 크기 속성이 추가되어 `‘작은 고양이’` 가 되어도 고양이라는 사실은 변하지 않음

타입 스크립트에서 타입은 `open` 이다.

## 아이템6. 편집기를 사용하여 타입 시스템 탐색하기

### 타입스크립트 컴파일러

**1 SourceCode to Data**

```ts
const msg: string = "Hello, world";
welcome(msg);

function welcome(str: string) {
  console.log(str);
}
```

위의 코드를 3개의 문으로 나눌 수 있다.

1. 변수 `msg`를 선언하는 문
2. 함수 `welcome`을 호출하는 문
3. 함수 `welcome`을 선언하는 문

![](https://velog.velcdn.com/images/dohun31/post/aac194c5-9298-4df9-a27f-1afab4d32996/image.gif)
각 문들을 chunck 단위로 나누고 추상구문트리로 표현한다.
![](https://velog.velcdn.com/images/dohun31/post/63be7b5b-0ded-49ca-a49d-1a5a623f7f3d/image.png)

**2. Type Checking**

추상 구문 트리를 읽어 타입 검사에 필요한 Symbol을 만들고, Scope를 정의하고, 타입 체크에 필요한 메타 데이터들을 수집한다.

이후 타입 체크 실행 (타입 체크, 타입 비교, 타입 추론)

**3. Creating File**
Javascript 파일을 만들어 준다.

**요약**

1. SourceCode to Data

   - Scanner: `Text` -> `Tokens`
   - Parser: `Tokens` -> `Tree`

2. Type Checking

   - Binder: `Syntax` -> `Symbol`
   - Checker: `Type Check`

3. Creating File
   - Transformer: `Syntax Tree(TS)` -> `Syntax Tree(JS)`
   - Emitter: `Syntax Tree` -> `File`

<br/>

```ts
function getElement(elOrId: string | HTMLElement | null): HTMLElement {
  if (typeof elOrId === "object") {
    return elOrId;
    // ~~~~~~~~~~ 'HTMLElement | null' 형식은 'HTMLElement' 형식에 할당할 수 없습니다.
  } else if (elOrId === null) {
  } else {
    ...
  }
}
```

타입스크립트는 스코프 내의 타입 변화를 추적한다.

**flow condition**: `typeof elOrId === "object"`
**flow container**: 플로우를 추적하는 스코프

현재 3개의 **flow container**가 생겼다.

첫번째 if 분기 내부의 `elOrId`의 타입을 알기 위해 bottom-up 방식으로 추적한다.
제일 먼저 만나는 **flow condition**을 통해서 `elOrId`의 타입이 ` HTMLElement | null`인것을 알게 되었고, 해당 **flow container** 내부에서 `elOrId`의 타입은 ` HTMLElement | null`이 된다.

## 아이템8. 타입 공간과 값 공간의 심벌 구분하기

```tsx
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });
```

```tsx
function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape.radius;
    // ~~~~~~~~ `{}` 형식에 `radius` 속성이 없습니다.
  }
}
```

`intanceof` : 자바스크립트의 **런타임** 연산자, 값에 대해 연산

- `type Cylinder`가 아니라 `function Cylinder`를 참조

```tsx
type T1 = typeof p; // Person
type T2 = typeof email; // {p: Person, subject: string, body: string} => Response
```

타입의 관점에서 `typeof`는 **타입스크립트의 타입**을 반환

```tsx
const v1 = typeof p; // 'object'
const v2 = typeof email; // 'function'
```

값의 관점에서 `typeof`는 **런타임의 typeof** 연산자

- 런타임 타입: `string`, `number`, `boolean`, `undefined`, `object`, `function`

```tsx
function email({
	person: Person,
	// ~~~~~~~~ 바인딩 요소 'Person'에 암시적으로 'any' 형식이 있습니다.
	subject: string,
	// ~~~~~~~~ 'string' 식별자가 중복되었습니다.
	//          바인딩 요소 'string'에 암시적으로 'any' 형식이 있습니다.
```

해당 방식으로 구조 분해 할당을 한다면 오류 발생

값의 관점에서 Person, string이 해석되었기 때문
