## Item 20: 다른 타입에는 다른 변수 사용하기

```jsx
// js
let id = '12-34-56';
fetchProduct(id);
id = 123456
fetchProductBySerialNumber(id)
```

자바스크립트에선 이런 코드가 가능하고 문제 없이 돌아갑니다.

```jsx
let id = '12-34-56';
fetchProduct(id);
id = 123456 // 123456 형식은 string 형식에 할당할 수 없습니다.
fetchProductBySerialNumber(id) // string 형식의 인수는 number 형식의 인수에 할당할 수 없습니다.
```

반면 타입스크립트는 타입으로 인해 다른 타입의 값을 할당할 수 없습니다. 간단하게 유니온 타입으로 해결할 수 있습니다.

```jsx
let id: string | number = '12-34-56';
fetchProduct(id);
id = 123456
fetchProductBySerialNumber(id)
```

하지만 유니온 타입은 좋은 해결책이 아닙니다. string에 대한 메소드를 사용할 때 마다  타입 가드를 통해 string 타입인지 확인해야합니다.

사실 이렇게 재할당하는 코드는 요즘 보기 어려운 코드 같습니다. 저런 경우 별도의 변수를 만들어서 사용하는게 올바른 것 같습니다. 

```jsx
const id = '12-34-56';
fetchProduct(id);
const serial = 123456
fetchProductBySerialNumber(id)
```

변수를 재사용하지 않고 별도의 변수로 사용하게 되면 다음과 같은 이점이 있습니다.

- 서로 관련이 없는 두 개의 값을 분리합니다.
- 변수명을 더 구체적으로 지을 수 있습니다.
- 타입 추론을 향상시키며, 타입 구문이 불필요해집니다.
- 타입이 간결해집니다.
- let 대신 const로 변수를 선언하게 됩니다.

## Item 23: 한꺼번에 객체 생성하기

객체를 생성할 때는 속성을 하나씩 추가하기보다는 여러 속성을 포함해서 한꺼번에 생성해야 타입 추론에 유리합니다.

```jsx
const pt = {};
pt.x = 3; // pt에 x 속성이 없습니다.
pt.y = 4; // pt에 y 속성이 없습니다.
```

타입스크립트는 pt의 타입을 {} 값을 기준으로 추론되기 때문에 존재하지 않은 x와 y 속성을 추가할 수 없습니다. 

```jsx
interface Point { x: number; y: number; }
const pt: Point = {}; // pt에 x, y 속성이 없습니다.
pt.x = 3;
pt.y = 4; 
```

타입을 정의해서 타입을 선언해줘도 빈 객체에는 x, y가 없기 때문에 에러가 발생합니다. 이런 경우엔 한번에 객체를 만드는 것이 좋습니다. 

```jsx
const pt = {
	x: 3,
	y: 4
}
```

작은 객체들을 조합해서 큰 객체를 만들어야 하는 경우에도 한꺼번에 객체를 만드는 것이 좋습니다.

```jsx
const pt = { x: 3, y: 4 }
const id = { name: 'Pythagoras' }
const namedPoint = {};
Object.assign(namePoint, pt, id)
namedPoint.name // {} 형식에 name이 없습니다.
```

이런 경우 spread 연산자를 사용해 쉽게 큰 객체를 만들 수 있습니다.

```jsx
const namePoint = {...pt, ...id};
namedPoint.name 
```

만약 조건부 속성을 추가하려면 속성을 추가하지 않는 null 또는 {}으로 객체 전개를 사용하면 됩니다.

```jsx
const firstLast = {first: 'Harry', last: 'Truman'}
const president = {...firstLast, ...(hasMiddle ? {middle: 'S'} : {})}

president: {
	middle?: string;
	first: string;
	last: string;
}
```
