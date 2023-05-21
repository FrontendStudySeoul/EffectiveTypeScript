## 아이템 9: 타입 단언보다는 타입 선언을 사용하기
> 변수에 타입을 부여하는 방법은 1. 타입 단언 2. 타입 선언이 있다.

### 타입 선언
``` ts
const alice: Person = {name: ‘Alice’ }
```

- 선언된 타입임을 명시


### 타입 단언

``` ts
const alice = {name: ‘Alice’ } as Person
```

- 타입스크립트가 추론한 타입을 무시
- 강제로 타입을 지정하였기 때문에 타입 체커에게
- 임의로 타입 간 변환을 할 수 없다

### 그럼에도 타입 단언을 사용하는 경우
> 컴파일 타임에 알기 어려운 특정 타입을 안전하게 허용한다.

1. DOM 엘리먼트
   - `event.target as HTMLInputElement;`
1. non-null assertioin operator (!)

## 아이템 10: 객체 래퍼 타입 피하기
> 타입을 지정할 때 객체 래퍼로 지정하는 것을 지양하자.

## 아이템 11: 잉여 속성 체크의 한계 인지하기
### 잉여 속성 체크
- 타입이 명시된 변수에 객체 리터럴을 할당할 때 **그 외의 속성이 없는지 확인**한다.
- 엄격한 객체 리터럴 체크
- 할당 가능 검사와는 별도의 과정이다.

### 잉여 속성 체크의 한계
1. 객체 리터럴에만 적용
2. 단언문을 사용할 때는 적용되지 않음

## 아이템 12: 함수 표현식에 타입 적용하기
> 매개변수와 반환 값에 대한 타입을 한꺼번에 적용할 수 있다

``` ts
type DiceRollFn = (sides:number) => number;

const rollDice: DiceRollFn = sides => { }
```

- 불필요한 코드의 반복을 줄일 수 있다.
- 라이브러리는 공통 함수 시그니처를 타입으로 제공
  - 리액트의 `MouseEventHandler` 타입 


## 아이템 13: 타입 vs 인터페이스

- 대부분의 경우 타입을 사용해도 되고 인터페이스를 사용해도 된다.
- 하지만 차이점을 분명하게 알고 타입을 정의해 일관성을 유지하는 것이 좋다.

### 인터페이스
> 소프트웨어의 이상적인 특징은 확장에 개방되어 있기 때문에, 간으하면 항상 타입보다 인터페이스를 사용해야합니다. 
> 
> 만약 인터페이스로 어떤 형태를 표현할 수 없고 유니언이나 튜플 타입을 사용해야 한다면, 일반적으로 타입 별칭을 사용 


- 유니온 인터페이스는 없다
- 선언 병합(declaration merging)
  - 의도치 않게 선언 병합이 될 수 있기 때문에 주의해야한다. 

``` ts
interface IState {
  name: string;
  capital: string;
}

interface IState {
  population: number;
}

const dory: IState = {
  name: 'dory',
  capital: 'park',
  population: 1000
}
```


### 타입 선언
- 배열, 튜플 타입을 더 간결하게 표현할 수 있다.
``` ts
type Pair = [number, number];
```

## 아이템 14: 타입연산과 제네릭 사용으로 반복 줄이기
### 1. 인덱싱
### 2. 매핑된 타입

``` ts
type  TopNavState = {
  [ k in 'userId' | 'pageTitle' | 'recentFiles' ] : State[k];
} 
```

### 3. 제네릭 타입
- 타입을 위한 함수

#### 제네릭 타입에서 매개변수 제한하기
- `extends`를 사용해서 매개변수 타입을 제한할 수 있다.

``` ts
type Pick<T, K extends keyof T> = {
  [k in K]: T[k]
}
```

#### 표준 라이브러리에 정의한 제네릭 타입
> `Pick`, `Partial`, `ReturnType` 

- 반환 값 타입 추론하기
``` ts
type UserInfo = ReturnType<typeof getUserInfo>;
```


## 아이템 15: 동적 데이터에 인덱스 시그니처 사용하기
- 타입스크립트에서 타입에 인덱스 시그니처를 명시하여 유연하게 매핑을 표현할 수 있다.
- `[property: string]: string`

### 인덱스 시그니처의 문제점
1. 잘못된 키를 포함해 모든 키를 허용한다.
2. 특정 키가 필요하지 않다.
3. 키마다 다른 타입을 가질 수 없다.
4. 타입스크립트 자동완성 기능이 지원되지 않는다.

### 인덱스 시그니처를 사용해야하는 경우
- 런타임때까지 객체의 속성을 알 수 없는 경우
  - 안전한 접근을 위해 인덱스 시그니처 값 타입에 undefined을 추가할 수 있다.

### Record, mapped type
> Record, 매핑된 타입을 사용하여 인덱스 시그니처의 문제점을 해결할 수 있다.

- Record
  - `Record<K, T>`: 타입 T의 프로퍼티의 집합 K로 타입을 구성한다.
  - 타입의 프로퍼티들을 다른 타입에 매핑시키는데 사용할 수 있다.

- mapped type
``` ts
// index signature
const PRICE_MAP: { [fruit: string]: number } = { ... }

// mapped type
const PRICE_MAP: { [fruit in Fruit]: number } = { ... }
```
