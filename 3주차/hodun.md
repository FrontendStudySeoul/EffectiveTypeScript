## 아이템 16. number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

- 자바스크립트의 객체
    - 키/값 쌍의 모음
    - 키는 보통 문자열 (Symbol도 가능)

- 배열 ⇒ 객체
    - 숫자 인덱스 ⇒ 문자열로 변환되어 사용
    
    ```tsx
    const x = [1, 2, 3];
    // [1, 2, 3]
    
    x[0];
    // 1
    
    x['1'];
    // 2
    ```
    
    - Object.keys를 이용해 배열의 키를 확인
        - 키 → 문자열
        
        ```tsx
        Object.keys(x);
        // ['0', '1', '2']
        ```
        

- **Array**
    
    ```tsx
    interface Array<T> {
    	// ...
      [n: number]: T;
    }
    ```
    
    - `number`로 인덱스 타입을 선언하지만, 런타임에서 인덱스 타입은 `string`
    - 타입 체크 시점에 오류를 잡을 수 있음
    
    ```tsx
    const xs = [1, 2, 3];
    const x0 = xs[0]; // ✅
    const x1 = xs['1']; // ❌
    ```
    
    ```tsx
    function get<T>(array: T[], key: string): T {
    	return array[k]; // ❌
    }
    ```
    
    - 실제로 동작 ❌
    - 타입은 런타임에 제거

![Untitled](https://file.notion.so/f/s/b5d9474d-9d0c-4977-bef1-50f42458acd6/Untitled.png?id=fbc5a641-d7f1-4aa9-bdfd-6b556906d584&table=block&spaceId=460aeeee-d1c6-4253-8817-e4cfb135299e&expirationTimestamp=1685327882378&signature=FUjiR130pS87V_Ar-PA-rhmDTQlptxXOTNzYRwvLZpo&downloadName=Untitled.png)

### for…of

iterable한 객체에 대해 반복하고, 각 value에 대해 실행되는 반복문 생성

```tsx
const iterable = ['a', 'b', 'c'];

for(const value of iterable) {
  console.log(value); 
}

// 'a'
// 'b'
// 'c'
```

### for…in

문자열로 키가 지정된 모든 iterable한 속성에 대해 반복문 실행

⚠️ 인덱스의 순서가 중요한 Array에서 반복을 위해 사용할 수 없음

- 특정 순서에 따라 인덱스를 반환하는것을 보장할 수 없음
- → 정수가 아닌 이름을 가진 속성

**그럼 언제 사용하나?**

- 배열의 반복을 위한건 이미 `for…of`, `Array.prototype.forEach`가 존재
- 쉽게 객체의 속성을 확인할 수 있음 → 디버깅에 용이
- 특정 값을 가진 키가 존재하는지 확인

```tsx
const obj = {a: 1, b: 2, c: 3};

for (const prop in obj) {
  console.log(`obj.${prop} = ${obj[prop]}`);
}

// Output:
// "obj.a = 1"
// "obj.b = 2"
// "obj.c = 3"
```

- for…in
    - key가 string인데 에러가 발생하지 않음
    - 배열을 순회하는 코드 스타일에 대한 실용적인 허용

- 배열 순회
    - 인덱스를 크게 신경쓰지 않음 (value가 중요)
        - for…of
            
            ```tsx
            for(const x of xs) {
              x; // typeof x => number
            }
            ```
            
    - 인덱스 타입이 중요
        - forEach
            
            ```tsx
            xs.forEach((x, i) => {
              x; // typeof x => number
              i; // typeof i => number
            })
            ```
            
    - 루프 중간에 멈춰야한다면
        - for loop
            
            ```tsx
            for(let i = 0; i < xs.length; i++) {
              const x = xs[];
              if(x < 0) break;
            }
            ```
            

- 인덱스를 number로 사용할 이유는 많지 않음
    - 숫자를 사용해 인덱스를 지정해야 한다면 Array이나 튜플을 사용

- **ArrayLike**: 어떤 길이를 가지는 배열과 비슷한 형태의 튜플
    
    ```tsx
    interface ArrayLike<T> {
      readonly length: number
      readonly [n: number]: T
    }
    ```
    
    ![Untitled](https://file.notion.so/f/s/0615ed10-1980-4c87-a06d-d9015d6848f1/Untitled.png?id=12e66c1f-e6ee-46cb-afed-581c6fb225d4&table=block&spaceId=460aeeee-d1c6-4253-8817-e4cfb135299e&expirationTimestamp=1685327849834&signature=xcQ6KJ6h2UMeOurojATn_rIAfKQZnMSc6J1saIZ-YFw&downloadName=Untitled.png)
    
    - **createNormalizedTupleType**
        
        ```tsx
        // Treat everything else as an array type and create a rest element.
        addElement(isArrayLikeType(type) && getIndexTypeOfType(type, numberType) || errorType, ElementFlags.Rest, target.labeledElementDeclarations?.[i]);
        ```
        
        - isArrayLikeType인 경우 index의 type이 number가 아니라면 error를 보여주는듯?

## 아이템 17. 변경 관련된 오류 방지를 위해 readonly 사용하기

- 함수가 매개변수를 수정하지 않는다면 readonly로 선언하는것이 좋음
    - 인터페이스를 명확하게 하고, 매개변수가 변경되는것을 방지
- readonly를 사용하면 변경하면서 발생하는 오류 방지, 변경 발생 코드 쉽게 찾을 수 있음
- readonly는 얕게 동작함
    
    ```tsx
    interface Outer {
    	inner: { x: number }
    }
    
    const o: Readonly<Outer> = { inner: { x: 0 }};
    o.inner = { x: 1 }; // ❌
    o.inner.x = 1 // ✅
    ```
    
    - readonly는 inner에만 적용됨

## 아이템 18. 매핑된 타입을 사용하여 값을 동기화하기

- 매핑된 타입을 사용해서 관련된 값과 타입을 동기화하도록 함
- 인터페이스에 새로운 속성을 추가할 때, 선택을 강제하도록 매핑된 타입 고려

```tsx
[k in keyof ScatterProps]
```

## 아이템 20. 다른 타입에는 다른 변수 사용하기

```tsx
let id: string | number = '12-34-56';
fetchProduct(id);

id = 123456;
fetchProductBySerialNumber(id);
```

- 첫번째 id, 두번째 id 서로 관련이 없음
- 변수를 무분별하게 재사용하면 타입 체커와 사람 모두에게 혼란 야기

## 아이템 21. 타입 넓히기

```tsx
const mixed = ['x', 1];
```

- mixed의 타입이 될 수 있는 후보
    - `(’x’ | 1)[]`
    - `[’x’, 1]`
    - `[string, number]`
    - `readonly [string, number]`
    - `(string | number)[]`
    - `readonly (string | number)[]`
    - `[any, any]`
    - `any[]`

```tsx
const v = {
	x: 1,
}
v.x = 3;
v.x = '3';
v.y = 4;
v.name = 'Pythagoras';
```

- 가장 구체적인 경우: `{readonly x: 1}`
- 추상적인 경우: `{x: number}`
- 타입 스크립트의 넓히기 알고리즘
    - let으로 할당된것처럼 다룸
    - v 타입 → `{x: number}`
    - ❗객체를 한번에 만들어야 함

- 타입 추론의 강도 제어
    1. 명시적 타입 구문 제공
        
        ```tsx
        const v: {x: 1|3|5} = { x: 1 }
        ```
        
    2. 타입 체커에 추가적인 문맥을 제공
    3. `const` 단언문 사용
        
        ```tsx
        const v1 = {
        	x: 1,
        	y: 2,
        }
        // {x: number; y: number;}
        
        const v2 = {
        	x: 1 as const,
        	y: 2,
        }
        // {x: 1; y: number;}
        
        const v3 = {
        	x: 1,
        	y: 2,
        } as const
        // {readonly x: 1; readonly: 2;}
        ```
        
        `as const`를 붙이면 최대한 좁은 타입으로 추론
        

## 아이템 22. 타입 좁히기

```tsx
function isInputElement(el: HTMLElement): el is HTMLInputElement {
	return 'value' in el;
}

function getElementContent(el: HTMLElement) {
	if(isInputElement(el)){
		return el.value; // el => HTMLInputElement
	}
	return el.textContent // el => HTMLElement
}
```

```tsx
function isDefined<T>(x: T | undefined): x is T {
	return x !== undefined;
}

const members = ['Janet', 'Michael'].map(
	who => jackson5.find(n => n === who)
).filter(isDefined);
```

- 태그된/구별된 유니온과 사용자 정의 타입 가드를 사용해 타입 좁히기

### Ref

[for...of - JavaScript | MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/for...of)

[for...in - JavaScript | MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/for...in)
