## 타입 넓히기[item 21]
개발자 관점에서는 **추론될 때는 타입이 넓혀져서 타입추론이 되는구나~** 라고 생각하면될 것 같다
개발자 관점에서는 **추론될 때는 타입이 넓혀져서 타입추론이 되는구나~** 라고 생각하면될 것 같다.
변수를 초기화할 때 타입이 명시되어 있지 않으면 타입스크립트의 타입체커는 타입을 명시해야 하는데 이 때 지정된 값을 가지고 할당 가능한 값들의 집합을 유추해야 한다. 이 과정을 ‘타입 넓히기’라고 한다.

예제 1 
3D 벡터를 다루는 라이브러리 코드를 살펴보자

```js run
interface Vector3 {
  x: number;
  y: number;
  z: number;
}
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}
let x = 'x'; // type is 'string'
let vec = { x: 10, y: 20, z: 30 };
getComponent(vec, x); //❌ error : 'string'형식의 인수는 'x'|'y'|'z' 형식의 매개변수에 할당할 수 없습니다.

```
<img width="714" alt="스크린샷 2023-05-28 오후 2 59 32" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/b25cf829-e4a1-49e3-9d02-a6023586f6f9">

그럼 이걸 어떻게 해결 할 수 있을까요 ?

1. as 사용하기

```js run
getComponent(vec, x as "x");

```
<img width="311" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/a0cc1445-4c5a-400b-9323-e164498e0d61">

<br>근데 저희가 앞에서 배울때 as는 지양하자고 했기에 저희는 최대한 타입을 이용해서 해결해야합니다.

2. const 사용하기
```js run
let x = 'x'; //type is string
//but
const x = 'x'; //type is x
```
그래서 상수를 이용하면 정확한 타입을 추론받을 수 있기에 const 사용을 지향합시다:)
<img width="555" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/4c44eac3-32fe-49cc-820f-1382a110e897">
**eslint로 const사용을 강제할수도 있어요(prefer const)**

예제 2
const mixed = ['x', 1];
mixed는 어떤 타입으로 추론될까? 추론될 수 있는 타입 후보는 아래와 같이 굉장히 많다.

('x'|1)[]
['x', 1]
[string, number]
readonly [string, number]
(string|number)[]
readonly (string|number)[]
[any, any]
any[]
타입스크립트는 (string|number)\[\]로 추측한다.

넓히기 제어
넓히기 과정을 제어할 수 있는 방법이 있다.

방법은 앞서 배운것처럼 타입을 지정해주기이다.

2. 명시적으로 타입 지정하기
```js run
const mixed: (1 | 'sdd')[] = [1, 'sdd']
//이 처럼 배열에 직접 값을 할당해줘서 타입을 설정해줄 수 있다.
```
<img width="328" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/44ad58aa-fd35-4acd-b235-cc822090a843">
<img width="364" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/9155d044-93a6-4dc2-ba7e-f413a5629a30">

3. const 단언문 사용하기
as const를 사용하면 타입스크립트는 최대한 좁은 타입으로 추론한다.

<img width="329" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/2df2ebf0-194e-475b-8667-18de0cb7c880">
<br> 이렇게 개별적인 상수에 as const도 사용 가능해요.<br>
<img width="388" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/6a24f41e-acfb-45a0-86f3-1bca419bc4d1">


### as const 정말 필요할까?
근데 사실 let을 굳이 as const를 써서 const로 바꿀 이유는 없다. 그냥 const 키워드로 변수를 선언하면 되니까 말이다.
자 그럼 이 쯤에서 처음 질문으로 돌아가보자. '상수가 아닌 것을 왜 상수로 만들어야 할까?'에서 이 논의는 출발하였다. 그러면 '상수가 아닌 것'을 좀 더 파고 들어봐야한다.

Object의 Value는 상수가 아니다. 다시 말해, 특정 Object에 특정 Key가 존재해야 한다고 지정해줄 수는 있지만, 특정 Key에 와야만 하는 Value를 구체적으로 지정할 수는 없다는 뜻이다. 이는 Object가 본디, 구조화된 데이터 형태로서 가변적인 데이터 값을 저장해야 하기 위해 탄생했기 때문이다.

하지만 Object를 Object 본래 용도가 아닌 enum스럽게 활용하고 싶을 때, 이러한 가변성이 발목을 잡게 된다.
따라서, 이럴 때 쓰는 것이 바로 as const 문법이다.

## 타입 좁히기[item 22]
타입 좁히기 정말 많이써요.
저는 특히 회사코드에서 데이터 받아올때 많이써요.

### 1. 조건문
```js run
 type ServerData = {
    name: string
    age: number
    type: 'admin' | 'ananymous'
  }
  const [serverData, setServerData] = useState<ServerData | null>(null)

  useEffect(() => {
    const data = async () => {
      const response = await getData()
if(response!==undefined){
      setServerData(response)
    }
  }, [])
  return (
    <div>
      <div>hihi</div>
      {serverData && <div>{serverData.name}</div>}
      {serverData && <div>{serverData.name}</div>}
    </div>
  )
```
위처럼 if문으로 하는게 제일 일반적인것 같아요.

### 2. instanceof
```js run

function contains(text: string, search: string | RegExp) {
    if (search instanceof RegExp) {
      search // type is RegExp
      return !!search.exec(text)
    }
    search // type is string
    return text.includes(search)
  }

  console.log(contains('afaf', 'a'))
```

```js run
class Animal {
    name: string

    constructor(name: string) {
      this.name = name
    }

    eat() {
      console.log(`${this.name} is eating.`)
    }
  }

  class Dog extends Animal {
    breed: string

    constructor(name: string, breed: string) {
      super(name)
      this.breed = breed
    }

    bark() {
      console.log(`${this.name} is barking.`)
    }
  }

  class Cat extends Animal {
    color: string

    constructor(name: string, color: string) {
      super(name)
      this.color = color
    }

    meow() {
      console.log(`${this.name} is meowing.`)
    }
  }

 function performAction(animal: Animal) {
    if (animal instanceof Dog) {
      animal.bark() 
    } else if (animal instanceof Cat) {
      animal.meow() 
    } else {
      animal.eat()
    }
  }
  이런식으로도 가능해요.
```

### 3. in 연산자
```js run
interface A {
    a: number
  }
  interface B {
    b: number
  }

  function pickAB(ab: A | B) {
    if ('a' in ab) {
      ab // type is A
    } else {
      ab // type is B
    }
    ab // type is A | B
  }
```

