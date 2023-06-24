## 아이템 42. 모르는 타입의 값에는 any 대신 unknown을 사용하기

unknown에는 함수의 반환값과 관련된 형태, 변수 선언과 관련된 형태, 단언문과 관련된 형태가 있다.

### is
```tsx
function isString(value: any): value is string {
    return typeof value === 'string'
  }

  function isStringNotUseIs(value: any): string | boolean {
    return typeof value === 'string' && value
  }

  function isNumber(value: any): value is number {
    return typeof value === 'number'
  }

  function processValue(value: any) {
    if (isString(value)) {
      // value는 string 타입으로 추론됩니다.
      console.log(value.length)
    } else {
      console.log('Value is not a string.')
    }
  }

  processValue('모승')
  processValue(123)
```
**isString**<br>
<img width="412" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/a5472ed8-50b5-4c9e-87a3-f621a9b88131">
<br>
**isStringNotUseIs**<br>
<img width="433" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/8483bd33-a46d-4d84-834d-9548111d7ebb">


```tsx
//Generic으로 리팩토링
function isType<T>(value: any, typeName: string): value is T {
  return typeof value === typeName;
}

function processValue<T>(value: any) {
  if (isType<T>(value, "string")) {
    // value는 T 타입으로 추론됩니다.
    console.log(value.length);
  } else {
    console.log("Value is not of the specified type.");
  }
}

```

<img width="584" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/845211e9-0afb-4f94-956d-58b9cb566671">
<br>

그래도 value는 any이기 떄문에 length가 사용이 불가능하다 그래서 이를 어떻게 해결할 수 있을까요 ?

```tsx
// 타입 단언으로 해결
function processValueGeneric<T>(value: any) {
    if (isType<T>(value, 'string')) {
      // value는 T 타입으로 추론됩니다.
      const STRING_VALUE = value as string;
      console.log(typeof value)
      console.log(STRING_VALUE.length)
    } else {
      console.log('Value is not of the specified type.')
    }
  }
```

### unknown
```tsx
interface Book {
    name: string
    author: string
  }

  const book = parseYaml(`name:Wuthering Heights 
  author : Emily Bronte`)

  alert(book.title)
  //book의 타입이 any이기 떄문에 타입이 통과되지만 런타임떄 에러를 뱉는다. 왜냐면 title이라는 값이 없기에
```
<br>

```tsx
interface Book {
    name: string
    author: string
  }

  function safeParesYaml(yaml: string): unknown {
    return parseYaml(yaml)
  }

  const book = safeParesYaml(`name:Wuthering Heights 
  author : Emily Bronte`) as Book

  alert(book.name)
  alert(book.title)
  // book형식에 title이 없습니다.
```
<br>

타입 단언문 말고 다른 방법으로도 타입을 보장 받을 수 있다.
```tsx
//instanceof
 function processValues(value: unknown) {
    if (value instanceof Date) {
      console.log(value)
    }
  }
```
<br>

타입가드 함수로도 가능하다 어떻게 할 수 있을까요?
```tsx
function isBook(value: unknown): value is Book {
    return (
      typeof value === 'object' &&
      value !== null &&
      'name' in value &&
      'author' in value
    )
  }

  function processValuess(value: unknown) {
    if (isBook(value)) {
      console.log(value)
    } else {
      console.log('value is not Book')
    }
  }
  processValuess({ name: 'dad', author: 'aff' })
  processValuess(1)
```

<img width="426" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/978c2558-4a4e-4248-adbf-8ec10a1d66ba">





unknown으로 알 수 없는 타입이라고 지정 후 Book으로 타입단언해서 사용하는 것 같은데, any를 쓰는것보다 명확하게 에러의 이유를 알 수 있어서 그런것같다.<br>
그래서 어떤 함수를 만들 때 어떤 값을 return 하는지 알수 없다면 any를 사용하는것 보단 unknown을 사용해서 즉시 해당값의 타입을 좁히도록 할 수 있어야한다.<br>
그리고 any를 사용한 순간 모든 타입이 풀리기때문에 그 타입으로 인해 다른 값을 만들고 전염병처럼 any가 퍼질 확률이 높아서 나중에 어디서 부터 에러가 시작됐는지 가늠이 힘들다.

