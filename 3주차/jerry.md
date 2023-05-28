## 아이템 16. number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

### 타입스크립트에서의 객체의 키 타입

- 배열 또한 객체이므로 키는 숫자가 아니라 문자열이다.
- 객체의 키 값은 런타임에 모두 문자열로 변환된다.
- 타입스크립트는 숫자 키를 허용하고, 문자열 키와 다른 것으로 인식한다.
- 런타임에는 문자열 키로 인식하지만 타입 체크 시점에서 오류를 잡을 수 있어 유용하다.

```typescript
// 키 값이 문자열 타입으로 정의됨을 알 수 있다.
interface Array<T> {
  // ...
  [n: number]: T;
}
```

- 단, 배열을 순회하는 코드에 대해서 인덱스 값은 string 타입을 허용한다.

```typescript
const xs = [1, 2, 3];
const x0 = xs[0]; // OK
const x1 = xs["1"]; // 에러 발생

// Object.keys와 같은 구문은 문자열로 반환된다.
const keys = Object.keys(xs); // Type is string[]
for (const key in xs) {
  // key는 string, xs는 number[], x는 number
  const x = xs[key]; // Type is number
}
```

- 결론: 객체의 키 값을 숫자로 사용하고 싶다면 인덱스 시그니처에 number를 사용하기보다 Array, 튜플, ArrayLike 타입을 사용하는 것이 좋다.

<br />

### ArrayLike 타입

- 어떤 길이를 가지는 배열과 비슷한 형태의 튜플을 사용하고 싶다면 ArrayLike 타입을 사용한다.

```typescript
// ArrayLike 타입의 정의
interface ArrayLike<T> {
  readonly length: number;
  readonly [n: number]: T;
}
```

<br />

## 아이템 17. 변경 관련된 오류 방지를 위해 readonly 사용하기

### readonly 특징

- 배열의 요소를 읽을 수 있지만, 쓸 수는 없습니다. 즉, 변경할 수 없다.
- length를 읽을 수 있지만 바꿀 수 없다.
- readonly는 얕은 복사로 비교한다.
- Readonly 제너릭을 사용할 수도 있다.
- 인덱스 시그니처에도 readonly를 사용할 수 있으며 객체의 속성이 변경되는 것을 방지할 수 있다.

<br />

### const와 readonly의 차이점

- const
  - 변수에 다른 값을 할당할 수 없다.
  - 변수에 사용된다.
  - 만약 const로 배열을 생성한 경우, 배열을 변경하는 push, pop과 같은 메서드를 사용할 수 있다.
- readonly
  - 프로퍼티에 재할당을 막는 접근 제어자이다.
  - 값을 읽기 전용으로 만들 때 사용된다.
  - 반대로 배열을 변경하는 메서드를 사용할 수 없다.

<br />

## 아이템 18. 매핑된 타입을 사용하여 값을 동기화하기

> 값을 동기화하는 방법은 keyof, 모든 속성을 하나씩 확인하는 방법, 매핑된 타입을 이용하는 방법이 있습니다.

### keyof를 이용한 방법 -> 추천 X

- keyof를 이용해 정의된 타입의 키를 순회하는 방법

```typescript
interface ScatterProps {
  // The data
  s: number[];

  // Display
  range: [number, number];
  color: string;

  // Events
  onClick: (index: number) => void;
}

// shouldUpdate가 true를 반환할 때만 렌더링하도록 최적화함
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== "onClick") return true;
    }
  }
  return false;
}
```

<br />

### 타입의 모든 속성을 직접 작성해 비교하는 방법 -> 추천 X

- 새로운 프로퍼티가 생기면 해당 프로퍼티를 매번 직접 추가해야 하는 문제가 발생함

```typescript
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  return (
    oldProps.s !== newProps.s ||
    oldProps.range !== newProps.range ||
    oldProps.color !== newProps.color
    // (no check for onClick)
  );
}
```

<br />

### 매핑된 타입과 객체 -> 추천 O

> 매핑된 타입을 이용해 관련된 값과 타입을 동기화할 수 있다.

- 타입 체커가 타입을 비교하도록 하는 것이 좋다.
- 매핑된 타입을 가진 객체를 생성해 객체 속성의 정보를 제공하는 방법이다.
- 매핑된 타입은 **한 객체가 또 다른 객체와 정확히 같은 속성을 가지게 할 때** 이상적이다.

```typescript
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  s: true,
  range: true,
  color: true,
  onClick: false,
};

function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true;
    }
  }
  return false;
}
```

<br />

### 매핑된 타입(Mapped Types)

> 특정 타입을 기반으로 새로운 타입을 선언해야 할 때 유용하다. -> 타입 중복 작성을 줄일 수 있다.

- 인덱스 시그니처란?
  > 인덱스 시그니처를 사용해 유연한 타입 생성하기
  > 동적 프로퍼티 타입을 정의할 수 있다.

```typescript
type DogInfo = {
  [key: string]: string | number;
};

const dog: DogInfo = {
  name: "tommy",
  breed: "beagle",
  age: 2,
};
```

<br />

- in 연산자를 사용해 특정 타입을 기반으로 새로운 타입 생성하기(키를 유니온 타입으로 가진 경우)

```typescript
type UserInfo = "userName" | "age";
type UserInfoMappedType = {
  [P in UserInfo]: boolean;
};

/*
  type UserInfoMappedType = {
      userName: boolean;
      age: boolean;
  }
*/
```

<br />

- 제네릭을 활용한 경우

```typescript
type UserInfo = {
  userName: string;
  age: number;
};

type Info<Type> = {
  [Property in keyof Type]: Type[Property];
};

type UserInfoMappedType = Info<UserInfo>;

/*
  type UserInfoMappedType = {
      userName: string;
      age: number;
  }
 */
```

<br />

- 템플릿 리터럴을 활용해 매핑된 타입의 키를 다시 매핑하기

```typescript
type UserInfo = {
  userName: string;
  age: number;
};

// 만들고 싶은 타입
type UserInfoListeners = {
  onUserNameChange: (newValue: string) => void;
  onAgeChange: (newValue: number) => void;
};

// Capitalize<Property> => 에러 발생 : Type 'keyof Type' is not assignable to type 'string'.(Property는 어떤 타입이든 될 수 있기 때문에 Property가 string 타입인지 확실하지 않음)
// as: 매핑된 타입의 키를 다시 매핑할 수 있다.(TypeScript 4.1 이상)
type Listeners<Type> = {
  [Property in keyof Type as `on${Capitalize<string & Property>}Change`]: (
    newValue: Type[Property]
  ) => void;
};

type UserInfoListeners = Listeners<UserInfo>;

// 다음과 같이 사용할 수도 있어요.
type Listeners<Type> = {
  [Property in keyof Type as `on${Capitalize<string & Property>}Change`]: (
    newValue: Type[Property]
  ) => void;
} & {
  [Property in keyof Type as `on${Capitalize<string & Property>}Click`]: (
    newValue: Type[Property]
  ) => void;
};

/*
  type UserInfoListeners = {
      onUserNameChange: (newValue: string) => void;
      onAgeChange: (newValue: number) => void;
  } & {
      onUserNameClick: (newValue: string) => void;
      onAgeClick: (newValue: number) => void;
  }
*/
```

<br />

- Mapping Modifiers
- 매핑 중에 추가할 수 있는 Modifiers는 readonly와 ?가 존재한다.
- `-`나 `+`로 Modifiers를 추가하거나 제거할 수 있다.(기본값은 `+`)

```typescript
// readonly 제거하기
type LockedAccount = {
  readonly id: string;
  readonly name: string;
};

// readonly Modifier 앞에 -를 붙여 readonly를 제거할 수 있다.
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type UnlockedAccount = CreateMutable<LockedAccount>;

/*
  type UnlockedAccount = {
      id: string;
      name: string;
  }
*/
```

```typescript
// optional 속성 제거하기
type MaybeUser = {
  id: string;
  name?: string;
};

// ? Modifier 앞에 -를 붙여 optional를 제거할 수 있다.
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};

type User = Concrete<MaybeUser>;

/*
  type MaybeUser = {
    id: string;
    name: string;
  }
*/
```

<br />

## 아이템 19. 추론 가능한 타입을 사용해 장황한 코드 방지하기

- 모든 변수나 함수의 반환 값에 타입을 선언하는 것은 비생산적이다.
- 기본값이 있는 경우 추론 타입을 활용하자.
- 비구조화 할당문은 모든 지역 변수의 타입이 추론되도록 한다.
- 때로는 타입스크립트가 스스로 타입을 판단하기 어려울 수 있으며 이때 명시적 타입 구문을 사용한다.

  - logProduct 함수에서 매개변수 타입을 명시적으로 선언한 이유가 그 예이다.

    ```typescript
    function logProduct(product: Product) {
      // id, name, price 값에 대해 타입을 따로 지정하지 않아도 됨
      const { id, name, price } = product;
      console.log(id, name, price);
    }
    ```

<br />

## 아이템 20. 다른 타입에는 다른 변수 사용하기

- 변수의 값은 바뀔 수 있지만, 타입은 보통 바뀌지 않는다.
  - 아래의 경우, 유니온으로 에러를 해결할 수 있지만 차라리 타입 별 변수를 만드는 것이 낫다.
  - 타입이 다른 값을 다룰 때에는 변수를 무분별하게 재사용하지 않는 것이 좋다.

<br />

## 아이템 21. 타입 넓히기

> 런타임에 모든 변수는 유일한 값을 가지지만 정적 타입 분석 시점에 변수는 가능한 값들의 집합인 타입을 가진다.

### 타입스크립트의 기본 동작 재정의 방법 3가지

1. 명시적 타입 구문

   ```typescript
   const v: { x: 1 | 3 | 5 } = {
     x: 1,
   }; // Type is { x: 1 | 3 | 5; }
   ```

2. 타입 체커에 추가적인 문맥을 제공하기(아이템 26 참고)

   - 예를 들어 함수의 매개변수로 값을 전달하는 방법

3. const 단언문

   - const 단언문은 변수에 쓰이는 let, const와는 다르다.
   - 값 뒤에 as const를 작성하면, 최대한 좁은 타입으로 추론된다.
   - 객체에 as const를 작성하면 객체의 모든 속성이 readonly 속성으로 변환된다.

   ```typescript
   const v1 = {
     x: 1 as const,
     y: 2,
   }; // Type is { x: 1; y: number; }

   const v2 = {
     x: 1,
     y: 2,
   } as const; // Type is { readonly x: 1; readonly y: 2; }

   const a1 = [1, 2, 3]; // Type is number[]
   const a2 = [1, 2, 3] as const; // Type is readonly [1, 2, 3]
   ```

<br />

## 아이템 22. 타입 좁히기

- 타입스크립트는 조건문에서 타입을 좁히는데 능숙하다.
- instanceof를 사용해 타입을 좁힐 수 있다.
- 속성 체크로 타입을 좁힐 수 있다.
- Array.isArray와 같은 내장 함수로도 타입을 좁힐 수 있다.
- 명시적 태그를 이용해 타입을 좁힐 수 있다.
- 사용자 정의 타입 가드를 이용해 타입을 좁힐 수 있다.

  - 함수에 사용자 정의 타입 가드를 사용해 리턴하는 값의 타입을 좁힐 수 있다.
  - 반환 타입이 el is HTMLInputElement는 함수의 반환이 true인 경우, 타입 체커에게 매개변수의 타입을 좁힐 수 있다고 알린다.
  - 예시 1

    ```typescript
    function isInputElement(el: HTMLElement): el is HTMLInputElement {
      return "value" in el;
    }

    function getElementContent(el: HTMLElement) {
      if (isInputElement(el)) {
        el; // Type is HTMLInputElement
        return el.value;
      }
      el; // Type is HTMLElement
      return el.textContent;
    }
    ```

  - 예시 2

    - filter 함수를 이용해도 undefined 값을 걸러낼 수 없다.

    ```typescript
    const jackson5 = ["Jackie", "Tito", "Jermaine"];
    const members = ["Janet", "Michael"]
      .map((who) => jackson5.find((n) => n === who))
      .filter((who) => who !== undefined);
    // Type is (string | undefined)[]
    ```

    - 사용자 정의 타입 가드를 이용하면 x의 undefined 타입을 제외할 수 있다.
    - isDefined 함수에서 x에 undefined 값도 들어올 수 있는데, 반환값의 타입을 T로 제한해 반환 타입을 좁힐 수 있다.

    ```typescript
    const jackson5 = ["Jackie", "Tito", "Jermaine"];
    function isDefined<T>(x: T | undefined): x is T {
      return x !== undefined;
    }
    const members = ["Janet", "Michael"]
      .map((who) => jackson5.find((n) => n === who))
      .filter((x): x is string => x !== undefined);
    // Type is string[]
    ```
