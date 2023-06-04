## 아이템 23. 한꺼번에 객체 생성하기

타입스크립트는 타입이 일반적으로 변경되지 않습니다. 20장 참조(다른변수에는 다른 타입 지정하기)<br>
그리고 20장에서는 타입이 바뀌는 변수는 일반적으로 피해야한다고 말했습니다.
```tsx
const pt = { x: 3, y: 4 }
  const id = { name: 'moseung2' }
  const namedInput = {}
  pt.adad = '3'
//타입스크립트에서 pt는 {}로 타입이 지정됐기에 아래 두줄은 컴파일 에러를 뱉습니다.
```
<img width="480" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/4f27bc8a-2f55-441e-8c56-b3bae3a1b240">

```tsx
interface Point {
    x: number
    y: number
  }
  const pt: Point = {}
  pt.x = 2
  pt.y = 3
  // pt가 빈 객체이기 때문에 아래처럼 에러를 뱉습니다.
```
<img width="710" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/f24b8c35-3dbe-4e8e-b7aa-31a8ffd60d14">

앞서 배운것처럼 as연산자를 사용하면 해결 가능합니다. 근데 앞서 21장에서 언급됐던것처럼 as를 최대한 지양해서 사용해야합니다.
```tsx
interface Point {
    x: number
    y: number
  }
  const pt: Point = {} as Point
  pt.x = 2
  pt.y = 3
```

이를 해결하는 가장 쉬운 방법은 제목처럼 한꺼번에 객체를 생성하는 것 입니다.
```tsx
const pt = { x: 3, y: 4, adad: '3' }
```
<img width="650" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/7828615c-6d1c-48b0-a835-d228ec5fcb3f">
그럼 앞서 21장에서 있었던것처럼 const로 선언했기때문에 pt라는 객체는 각 프로퍼티에 위 사진처럼 타입을 지정받습니다.

<br />

### 작은 객체들을 조합해 큰 객체 만들기
```tsx
const pt = { x: 3, y: 4 }
  const id = { name: 'moseung2' }
  const namedInput = {}
  Object.assign(namedInput, pt, id)
  console.log(namedInput, pt, id)
  console.log(namedInput.name)
```
<img width="352" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/bb3b5fdc-63eb-474b-8110-02d1dfd662b5">
<br>사진처럼 잘 나오긴하나 해당 namedInput에 property를 접근하려고하면 에러를 뱉습니다.
<br>
<img width="513" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/f45bf491-5e78-4682-b740-c0ebac6d7045">
왜냐면 해당 namedInput의 타입은 {}이기 때문입니다. 그럼 앞서 했던것처럼 namedInput의 타입을 지정해서 선언해줘도 또 위와같은 에러를 뱉을것이고 그럼 저희는 as를 사용해야할것입니다.
<br>
이를 해결하기 위한 좋은 방법이 있습니다.
```tsx
const sumOfObject = { ...pt, ...id }
```
<img width="342" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/ff1c2f86-dd45-4e26-a347-d7decb784d14">
위의 사진처럼 해당 객체를 합치면서 타입까지 적절히 받은것을 볼 수 있습니다.
```tsx
const pt = { x: 3, y: 4 }
  const id = { name: 'moseung2' as const }

  const sumOfObject = { ...pt, ...id }
```
앞서 배웠던것처럼 as const를 사용하면 name의 타입도 moseung2로 추론받을 수 있습니다.그리고 아래 사진처럼 타입추론도 받을 수 있습니다.
<img width="580" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/592f00a2-9171-4dad-8d81-1d82751a82d9">

### 조건부 객체 만들기
객체내에 프로퍼티중 원하는것을 조건부로 만들어야할땐 어떻게 할까요 ?
```tsx
const [hasMiddle, setHasMiddle] = useState<boolean>(false)
  const firstLast = { first: 'kim', last: 'moseung' }
  const moseung2 = { ...firstLast, ...(hasMiddle ? { middle: 's' } : {}) }
  const moseung3 = { ...firstLast, ...(hasMiddle ? { middle: 's' } : null) }
```
위 코드처럼 빈객체나 null을 할당해주면 됩니다.<br>
<img width="384" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/2c3eaa70-6dd6-4b89-a444-96e2c9a21b8b">
<img width="592" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/d1ecdb9c-00c5-4c7c-8f17-f0d8f6f97923">

#### 여러 속성 추가하기
```tsx
const [hasDates, setHasDates] = useState<boolean>(false)
  const nameTitle = { name: 'korea', title: '살기좋은' }
  const korea = {
    ...nameTitle,
    ...(hasDates ? { start: -2589, end: -2566 } : {}),
  }
  useEffect(() => {
    setHasDates(true)
  }, [])
  console.log(korea.start)
  //만약 hasDates가 false라면 korea.start는 undefined를 내뱉습니다.
```


### 실전 적용
그럼 우리가 타입스크립트에서 객체를 직접 선언해야하는 경우는 언제일까요 ?
<br>
<br>
```tsx
interface ServerWant {
    toGive: { name: string; age: number; isDeveloper: boolean }
  }

  interface ServerGive {
    result: { region: string; isGood: boolean }
  }

  const getData = (input: ServerWant): Promise<ServerGive> => {
    const data = fetch(`https://velog.io/@endmoseung?input=${input}`).then(
      (response) => response.json()
    )

    return data
  }

  const [serverData, setServerData] = useState<ServerGive>()

  useEffect(() => {
    const getServerApis = async () => {
      const toGive:ServerGive = {
        toGive: { name: 'a', age: 12, isDeveloper: true },
      }
      const data = await getData()
      return data
    }
    const fetchData = async () => {
      const data = await getServerApis()
      if (data) {
        setServerData(data)
      }
    }

    fetchData()
  }, [])
```
결론: 프론트엔드에서 객체 자료구조를 사용하는곳은 주로 서버에 데이터를 보낼때고 이떄는 인터페이스나 타입으로 지정해주는것이 좋을것 같다. 왜냐하면 해당 인터페이스만 고쳐주면 실제로 적요용하는데에서 확인이 가능하기 떄문에.
