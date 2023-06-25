## item 42: 모르는 타입의 값에는 any 대신 unknown을 사용하기

![image](https://github.com/FrontendStudySeoul/TypeScript/assets/59603529/8dd79b9a-c470-4c01-973d-aec7c1d0bd71)

- any와 unknown은 어떤 타입의 값이더라도 할당이 가능합니다.

```tsx
let a: any;
a = 1;
a = 'str';

let u: unknown;
u = 1;
u = 'str';
```

- any는 어떤 타입에도 할당이 가능합니다.
- unknwon은 any와 unknown을 제외한 타입에 할당할 수 없습니다.

```tsx
let num: number;
num = a // any 타입 할당 가능
num = u // unknown 불가능
```

- any는 타입 체크를 무시하지만 unknown은 타입 체크를 진행합니다.

```tsx
const a: any = {
	title: 'title'
}
a.name // ts 에러 없음, 런타임 에러 발생

const b: unknown = {
	title: 'title'
}
b.name // error: b는 unknown 타입입니다.
```

- unknown 타입인 채로 사용하면 오류가 발생합니다. 적절한 타입으로 강제하거나, 타입을 좁혀서 사용해야 합니다.

```tsx
interface Book {
	title: string
}

let b: unknown

b = {
	title: 'title'
} as Book

b.title
```

```tsx
const b: unknown = {
	title: 'title'
}

if(typeof b === 'object' && b !== null && 'title' in b) {
	b.title
}
```

- 함수의 경우 제네릭으로도 해결 가능합니다
    - 제네릭으로 해결하기 보단 타입 단언과 타입 좁히기로 해결하는 것이 더 좋다고 합니다. 왜인지 책에선 안알려주네요…

```tsx
const foo = (): unknown = {
	return {
		title: 'title'
	}
}
const book = foo()
book.title // error

const foo = <T>(): T= {
	return {
		title: 'title'
	}
}
const book = foo<Book>()
book.title
```

- {}, object로 unknown과 비슷한 효과를 줄 수 있습니다
    - {} 타입은 null과 undefined를 제외한 모든 값을 포함합니다.
    - object 타입은 모든 비기본형 타입으로 이루어져 있습니다.
    - unknown이 생기기 전엔 {}를 일반적으로 사용했지만 unknown이 생긴 시점부턴 사용하지 않습니다. 레거시 코드에서 {} 타입을 만난다면 unknown을 사용하려 했구나 이해할 수 있도록 알아두면 좋을 것 같습니다.

unknown은 어떤 값이든 할당할 수 있지만 이후에 그 값을 사용할 때는 타입 체크를 해서 안전하게 사용할 수 있는 장점이 있는 것 같습니다. any의 타입 체크가 되지 않는 단점을 보완하려고 나온 타입이 아닐까 생각했는데 찾아보니 any처럼 어떤 타입의 값이든 할당할 수 있으며 타입 체킹을 하는 타입이 필요해 만들어졌다고 하네요
