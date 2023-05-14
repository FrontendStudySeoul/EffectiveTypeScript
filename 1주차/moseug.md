## [아이템1] 타입스크립트와 자바스크립트의 관계 이해하기
타입스크립트는 자바스크립트의 상위 집합(superset)이다.
타입스크립트는 타입이 정의된 자바스크립트의 상위집합이다.

.js 파일에 있는 코드는 이미 타입스크립트이다.
즉 main.js 를 main.ts 로 변경해도 동일하다.
이러한 특성은 자바스크립트에서 타입스크립트로 마이그레이션 하는데 엄청난 이점이 된다.

모든 자바스크립트 프로그램은 타입스크립트이다 -> 참
모든 타입스크립트 프로그램은 자바스크립트이다 -> 거짓



타입 시스템은 런타임 오류를 방지한다
타입 시스템의 목표 중 하나는 런타임에 오류를 발생시킬 코드를 미리 찾아내는 것이다.
타입스크립트가 정적 타입 시스템이기 때문에 가능하다.

```js run
let city = "seoul";
console.log(city.touppercase()); // touppercase 속성이 string 형식에 없습니다.

```
타입 체커를 통과한 타입스크립트

평소 작성하는 타입스크립트 코드가 "타입 체커를 통과한 타입스크립트 프로그램"에 속하게 된다.

타입스크립트의 타입 시스템은 자바스크립트의 런타임 동작을 모델링한다.

```js run
const x = 2 + '3'; // 정상. string
하지만 타입스크립트는 추가적인 타입 체크를 하는 경우도 있다. 다음은 자바스크립트 런타임에선 오류가 발생하지 않지만, 타입 체커는 문제점을 표시한다.

const a = null + 7; // 에러. + 연산자를 ... 형식에 적용할 수 없습니다.

```
<img width="591" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/750bde75-9f03-423d-82b9-f6227076180f">

타입 체커를 통과 하더라도 런타임 에러가 발생한다.
const name = ['hee', 'jinu'];
console.log(name[2].toUpperCase()); // TypeError : Cannot read property 'toUpperCase' of undefined
타입스크립트는 앞의 배열이 범위 내에서 사용될 것이라고 가정했지만 실제로는 범위를 벗어나서 런타임 오류가 발생했다.

타입스크립트가 이해하는 값의 타입과 실제 값에 차이가 존재하기 때문에 발생하는 오류들이다.
타입 시스템이 정적 타입의 정확성을 보장해줄 것 같지만 그렇지 않기 때문이다.

## [아이템2] 타입스크립트 설정 이해하기
cli 로 타입 체커 설정
tsc --noImplicitAny program.ts
cli는 타입스크립트를 어떻게 사용할 계획인지 공유할 수 없어서 지양하는 것이 좋다.

tsconfig.json 설정 파일
```js run
{
 "compilerOptions": {
 	"noImplicitAny": true
 }
}
```
설정 파일은 tsc --init 으로 간단히 생성된다.

noImplicitAny
변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어한다.

noImplicitAny 가 해제되어 있을 경우엔 암시적 any가 타입으로 사용된다.
any는 타입 체커를 무력하게 만드니 매우 주의해서 사용해야한다.

noImplicitAny 를 설정하면 타입을 반드시 명시해야 하므로 다음과 같이 add 함수를 정의해야한다.

function add(a: number, b:number){
 return a + b; 
}

strictNullChecks
null과 undefined가 모든 타입에서 허용되는지 확인한다.

strictNullChecks 해제
const x: number = null; // 정상
strictNullChecks 설정
const x: number = null; // null 형식은 number 형식에 할당할 수 없습니다.
의도적으로 null이나 undefined 를 타입으로 명시하여 위의 오류를 고칠 수 있다.

const x: number | null = null;
undefined 는 객체가 아닙니다. 와 같은 런타임 오류를 방지하기 위해서는 strictNullChecks 를 사용해야 한다.

타입 단언 연산자 A!.
A 가 null, undefined가 아니라고 단언한다.

strictNullChecks를 설정한 경우엔 단언 연산자를 사용해서 값이 null, undefined가 아니라고 단언해주어야 한다.

if (el) el.textContent = 'hello'; // null은 제외됨
el!.textContent = 'hello'; // el이 null이 아님을 단언함
## [아이템3] 코드 생성과 타입이 관계 없음을 이해하기
타입스크립트 컴파일러는 두 가지 역할을 수행한다.

최신 타입스크립트/ 자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일(transplie)한다.
코드의 타입 오류를 체크한다.
이 두 가지는 서로 완벽히 독립적이다.

```js run
let x = 'hello'
  x = 3
```
<img width="682" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/3b3aec01-0edf-4d5b-8cb7-7027535e7a89">
<img width="199" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/9bc686b4-fc4a-4318-b03f-9558ff9c23de">

타입스크립트가 자바스크립트로 변환될 때 코드 내의 타입에는 영향을 주지 못한다.
자바스크립트의 실행 시점에서 타입은 영향을 미치지 않는다.
타입 오류가 있는 코드도 컴파일이 가능하다.
컴파일은 타입 체크와 독립적으로 동작하기 때문에, 타입 오류가 있는 코드도 컴파일이 가능하다.

오류가 있을 때 컴파일하지 않으려면 tsconfig.json에 noEmitOnError 를 설정하면 된다.

런타임에는 타입 체크가 불가능하다.
타입스크립트의 타입은 '제거 가능(erasable)'하다.
자바스크립트로 컴파일 되는 과정에서 타입 관련 코드들 (인터페이스, 타입, 타입 구문)은 제거된다.

```js run
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number; 
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape){
	if(shape instanceof Rectangle) {
      	// Rectangle 는 형식만 참조하지만, 여기서 값으로 사용되고 있습니다.
      return shape.width * shape.height; 
		// Shape 형식에 height가 없습니다.
    } else {
       return shape.width * shape.width;
    }
}
```
위는 타입을 값으로 사용한 경우에 발생한 오류이다.
타입은 런타임 이전에, 값은 런타임 때 사용 가능하기 때문이다.

앞의 코드에서 shape의 타입을 명확하게 하려면 런타임에 타입 정보를 유지하는 방법이 필요하다. 다음 3가지 방법으로 런타임때 타입 정보를 유지할 수 있다.

1. 속성값이 있는지 체크
런타임시 타입 정보를 확인하기 위해, height 속성이 존재하는지 체크할 수 있다.

```js run
function calculateArea(shape: Shape){
	if('height' in shape) { // 타입이 Rectangle in 연산자 객체내에 해당 property가 있는지 체크
      return shape.width * shape.height; 
    } else { // 타입이 Square
       return shape.width * shape.width;
    }
}
```
2. tagged union
런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 '태그' 기법이다.

```js run
 interface Square {
    kind: 'square' // tagged union
    width: number
  }
  interface Rectangle {
    kind: 'rectangle'
    width: number
    height: number
  }
  type Shape = Square | Rectangle

  function calculateArea(shape: Shape) {
    if (shape.kind === 'rectangle') {
      return shape.width * shape.height
    }
    return shape.width * shape.width
    // ...
  }

  console.log(calculateArea({ kind: 'square', width: 100 }))
  console.log(calculateArea({ kind: 'rectangle', width: 100, height: 100 }))
```
<img width="276" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/84358adc-aad0-4a52-85c6-68736dec7c48">

런타임에 타입 정보를 손쉽게 유지할 수 있기 때문에 유용한 기법이다.

3. class type
타입(런타임 접근 불가)과 값(런타임 접근 가능)을 둘 다 사용하는 기법이다.
타입을 클래스로 만들면 오류를 해결할 수 있다.

```js run
class Square {
  constructor(public width: number) {};
}
class Rectangle extends Square {
  constructor(public width: number, public height: number) {
  	super(width);
  };
}
type Shape = Square | Rectangle; // 타입으로 사용됨

function calculateArea(shape: Shape){
	if(shape instanceof Rectangle) { // 값으로 사용됨
      return shape.width * shape.height; 
    }
  // ...
}
```
타입 연산은 런타임에 영향을 주지 않는다.
function adNumber(val: number | string): number{
 return val as number; // number 타입으로 단언한다. 
}
as number 는 타입 연산이고 런타임 동작에는 아무런 영향을 미치지 않는다.
따라서 val 이 string 인 경우 number 로 타입 변환을 하지 못한다.

런타임 타입은 선언된 타입과 다를 수 있다.
타입스크립트에서는 함수의 매개변수 타입을 예측하지만, 실제 자바스크립트 런타임에서는 다른 타입의 값으로 호출될 수 있다.

API 요청으로 받아온 데이터가 이에 대표적인 예시다.

```js run
interface LightApiResponse {
 lightSwitchValue: boolean; 
}
async function setLight() {
 const response = await fetch('/light');
  const result: LightApiResponse = await response.json(); // 불리언이 아닐 수 있다!
  setLightSwitch(result.lightSwitchValue);
}
```
API를 잘못 파악해서 lightSwitchValue 가 불리언이 아닌 문자열일 수 있다. 하지만 타입스크립트는 이를 찾아내지 못한다.

타입스크립트에서 런타임 타입과 선언된 타입이 맞지 않을 수 있기 때문에, 언제든 선언된 타입이 달라질 수 있다는 것을 명시해야한다.

타입스크립트 타입으로는 함수를 오버로드할 수 없다.
오버로드
동일한 이름에 매개변수만 다른 여러 버전의 함수를 허용함.
타입스크립트는 타입과 런타임의 동작이 무관하기 때문에 함수 오버로딩이 불가능하다.

하나의 함수에 대해 여러 개의 선언문을 작성할 수 있지만 구현체(implementation)은 오직 하나다.

참고 - 타입스크립트 함수 오버로딩

function add(a: number, b:number): number;
function add(a: string, b: string): string;

function add(a, b) { // 단 한개의 구현체
 return a + b; 
}
타입스크립트 타입은 런타임 성능에 영향을 주지 않는다.
타입과 타입 연산자는 자바스크립트 변환 시점에 제거되기 때문에, 런타임의 성능에 아무런 영향을 주지 못한다.

단, 런타임 오버헤드가 없는 대신, 타입스크립트 컴파일러는 빌드타임 오버헤드가 있다.
하지만 컴파일러는 매우 빠르고 증분(incremental) 빌드 시 더욱 빠르다.

타입스크립트가 컴파일하여 생성하는 자바스크립트를 오래된 런타임 환경을 지원하기 위해 호환성을 높이고 성능 오버헤드를 감안할지, 호환성을 포기하고 성능 중심 네이티브 구현체를 선택할지 문제에 마주할 수 있다.
호환성을 높여 헬퍼 코드가 추가되면 오버헤드가 생길것이다.

하지만 이것은 언어 레벨의 문제이고 타입과는 전혀 무관하다.

## [아이템4] 구조적 타이핑
덕 타이핑 (duck typing)
함수의 매개변수 값이 모두 제대로 주어진다면, 그 값이 어떻게 만들어졌는지 신경쓰지 않고 사용한다.
즉 매개변수 값이 요구사항을 만족한다면 타입이 무엇인지 신경쓰지 않는다.
```js run

interface Vector2D {
  x: number;
  y: number;
}
function calculateLength(v: Vector2D){
 return Math.sqrt(v.x * v.x + v.y * v.y); 
}

interface NamedVector {
  name: string;
  x: number;
  y: number;
}
const v: NamedVector = { x: 3, y:4, name: 'zee'}
calculateLength(v); // 정상. 5
```
NamedVector의 구조가 Vector2D와 호환되기 때문에 calculateLength 호출이 가능하다.

```js run
interface Vector2D {
    x: number
    y: number
  }
  function calculateLength(v: Vector2D) {
    console.log(v.name)
    return Math.sqrt(v.x * v.x + v.y * v.y)
  }

  interface NamedVector {
    name: string
    x: number
    y: number
  }
  const v: NamedVector = { x: 3, y: 4, name: 'zee' }
  console.log(calculateLength(v))
  //이 코드는 에러를 뱉는다 아래 처럼 해당 타입이 존재하지 않기때문에
```
<img width="392" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/7a98a29d-7918-4144-b2d8-3de65a8496b5">


덕 타이핑의 문제
```js run
interface Vector3D {
  x: number;
  y: number;
  z: number;
}
function normalize(v: Vector3D){
  const length = calculateLength(v); // Vector2D 가 아닌 Vector3D 타입을 넣어도 오류가 나지 않는다.
  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}//calculateLength의 input의 타입이 Vector2D를 받아야하지만 Vector3D를 받아도 에러를 띄우지 않는다 
```
<img width="445" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/fc509e7f-b2d9-4ed8-833f-892eef0d8d35">


normalize({x:3, y:4, z:5}); // { x: 0.6, y: 0.8, z: 1 }
calculateLength 가 2D 벡터를 받도록 선언했지만 3D 벡터를 받는데 문제가 없었다.

Vector3D 구조가 Vector2D 와 호환되기 때문이다.

이런 타입은 '열려(open)'있는 타입이라고 한다.

함수에서 매개변수의 속성들이 매개변수 타입에 선언된 속성만 가지는 경우를 봉인된(sealed), 정확한(precise) 타입이라고 부른다. 열린 타입은 이와 반대이다.

즉 타입에 선언된 속성 외의 다른 속성들을 가지더라도 두 타입은 서로 호환된다.

덕타입핑으로 인해 루프를 순회할 때 발생하는 당황스러운 결과
```js run
interface Vector3D {
    x: number;
    y: number;
    z: number;
}

function calculateLengthL1(v: Vector3D){
    let length = 0;
    for (const axis of Object.keys(v)) {
      	// axis: string
        const coord = v[axis];
        // Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'Vector3D'.
        // -> 'string'은 'Vector3D'의 인덱스로 사용할 수 없기에 엘리먼트는 암시적으로 'any' 타입입니다.
        length += Math.abs(coord)
    }
    return length;
}
```
예측
axis 는 Vector3D 타입인 v의 키 중 하나이기 때문에 x, y, z 중 하나여야 하며, Vector3D의 선언에 따르면 이들은 모두 number 이므로 coord 의 타입이 number 가 되어야 할 것으로 예상된다.

사실
타입스크립트가 오류를 정확히 찾아낸 것이 많다. 함수 매개변수 v - Vector3D는 열려있기 때문에 다음과 같이 x, y, z 외의 다른 프로퍼티가 추가된 객체를 넘길 수도 있다.

const vec3D = { x: 3, y: 4, z: 1, address: '서울시' };
calculateLengthL1(vec3D) // 정상. NaN을 반환한다.
결론
이 경우엔 루프를 순회하는 것 보다는 모든 속성을 각각 더하는 구현이 더 낫다. 루프를 순회하면서 타입을 걸러내는 과정이 반드시 필요하기 때문이다.
function calculateLengthL1(v: Vector3D){
    return Math.abs(v.x) + Math.abs(v.y) + Math.abs(v.z)
}
덕타이핑으로 인해 클래스와 관련된 할당문에서 발생하는 당황스러운 결과
class C {
    foo: string;
    constructor(foo: string) {
        this.foo = foo;
    }
}

const c = new C('instance of C');
const d: C = { foo: '객체 리터럴' }; // 정상
d 가 C 타입에 할당되는 당황스러운 결과가 발생했다. 그 이유는 다음과 같다.

d 는 string 타입의 foo 프로퍼티를 가진다.
하나의 매개변수로 호출되는 생성자(Object.prototype)을 가진다.
c는 생성자로 class C 를 가진다.


d는 생성자로 Object 를 가진다.


따라서 구조적으로는 필요한 속성(foo)와 생성자(constructor) 를 가지기 때문에 문제가 되지 않는다.

만약 C의 생성자가 단순 할당이 아닌 연산 로직이 존제한다면 d 의 경우엔 생성자를 실행하지 않으므로 문제가 발생하게 된다.

이런 부분이 클래스의 서브클래스임을 보장하는 C++이나 자바와 같은 언어와는 매우 다른 특징이다.

## [아이템5] any 타입 지양하기
타입 안전성이 없다
number 타입 변수를 as any 를 사용하면 string 타입을 할당할 수 있게 된다.

함수 시그니처(스펙)를 무시한다.
함수의 매개변수 타입을 무시하고 인수로 전달된다.

```js run
function increase(a: number): number {
 return a + 1; 
}
```
let age: any = '25';
increase(age); // 정상
언어 서비스가 제공되지 않는다.
자동완성 기능과 적절한 도움말이 제공되지 않는다.

코드 리팩토링 때 버그를 감춘다.
타입 체커를 통과함에도 런타임에 오류가 발생할 수 있게된다.

타입 설계를 감춘다.
any 타입을 사용하면 타입 설계가 불분명해져서 설계가 잘 되었는지, 어떻게 되었는지 전혀 알 수가 없다. 설계가 명확히 보이도톡 타입을 일일이 작성하는게 좋다.

타입 시스템의 신뢰도를 떨어뜨린다.
any 타입으로 인해 런타임에 타입 오류가 발생될 수 있고 이는 신뢰할 수 없는 코드이다.

### 개인적으로 any를 지양해야 되는 이유
- 유지보수에 불리하다.(타입 추론을 받지 못하기에 추후에 이 코드를 보수해야할 코드에서 any가 있다면 해당 코드로 직접 들어가거나 개발자도구 network를 확인해야함) 
- 런타임에 어떤 에러가 생길지 예측 불가능하다.(any로 받아서 사용 가능했는데 코드내에 에러 핸들링이 따로 되자 않는 이상 api에서 undefined가 들어와도 통과되기 떄문)

## [아이템6] 편집기를 사용해 타입 시스템 탐색하기
타입스크립트를 설치하면, 다음 두 가지를 실행할 수 있다.

타입스크립트 컴파일러(tsc)
타입스크립트 코드를 자바스크립트 코드로 변환해주는 도구이다.
 npx tsc {파일명}.ts
위의 명령어로 tsc 를 사용하여 타입스크립트를 자바스크립트 파일로 컴파일 할 수 있다.

npx tsc --strict {파일명}.ts
--strict 옵션으로 엄격한 기준으로 타입 검사를 수행할 수도 있다.

단독으로 실행할 수 있는 타입스크립트 서버(tsserver)
타입스크립트 서버는 타입스크립트 컴파일러와 언어 서비스 를 캡슐화한 실행가능한 노드이다.

타입스크립트 서버는 편집기와 IDE 와 함께 사용하기에 최적화되어있다.

언어 서비스
코드 자동 완성
명세(사양, specification) 검사
검색
리팩터링
...
<img width="525" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/0564fbe8-87b7-421f-ab7c-684d50d44b4b">

언어 서비스는 위의 기능들을 제공한다.
보통은 언어 서비스를 편집기 를 통해 사용하게 된다.
언어 서비스에서 제공하는 기능들은 매우 유용하므로 타입스크립트 서버에서 언어 서비스를 사용할 수 있도록 설정하는 것이 좋다.

그리고 편집기에서는 타입스크립트가 언제 타입 추론을 수행하는지 직접 확인할 수 있기 때문에 타입 시스템에 대한 개념을 쌓기에 매우 좋다.

언어서비스는 라이브러리와 그에 대한 타입 선언을 탐색할 때에 도움이 된다. 편집기는 Go to Definition(정의로 이동) 옵션을 제공해서 라이브러리가 어떻게 모델링되었는지 확인하기 편하다.

편집기에서 타입스크립트 언어 서비스를 적극 활용해야 한다.
편집기를 사용하면 어떻게 타입 시스템이 동작하는지, 타입을 어떻게 추론하는지 개념을 잡을 수 있다.
타입스크립트가 동작을 어떻게 모델링하는지 알기 위해 타입 선언 파일을 찾아본다.
## [아이템7] 타입은 값들의 집합이다
타입은 "할당 가능한 값들의 집합 또는 타입의 범위"이다.
[원시타입](https://velog.io/@endmoseung/Typescript-1)

타입의 종류
never
가장 작은 타입. 아무 값도 포함하지 않는 공집합이다.

유닛(unit)타입 / 리터럴(literal) 타입
never 다음으로 작은 타입. 한 가지 값만 포함한다.

type A = 'A';
유니온(union)타입
타입을 두 개 이상 묶은 타입이다.

type AB = 'A' | 'B';
```js run
{
  "nickname": "moseung"|"seungmo",
  "type":"admin"|"general",
  "registerSession": "string"
}
//이런 식으로 있을떄 union 타입 많이 쓰는것 같아요
```
타입 체커
집합의 관점에서 타입 체커는 하나의 집합이 다른 집합의 부분 집합인지 검사하는 역할을 가졌다.

const ab: AB = Math.random() < 0.5 ? 'A' : 'B'; // 정상
const back: AB = 1; // '1' 형식은 'AB' 형식에 할당할 수 없습니다. AB는 "A","B"만 할당 가능
할당
T1이 T2에 할당 가능하다. = T1이 T2의 부분집합이다.

상속
T1이 T2를 상속한다. = T1이 T2의 부분집합이다.

```js run

type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12

const ab: AB = Math.random() < 0.5 ? 'A' : 'B'; // 정상, {"A", "B"}는 {"A", "B"} 의 부분집합

const ab12: AB12 = ab; // 정상, {"A", "B"} 는 {"A", "B", 12} 의 부분집합

declare let twelve: AB12;
const back: AB = twelve;
<img width="673" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/89bde078-8538-43c7-ae21-b0b5eb26eb59">
// AB12 타입의 값은 AB 타입의 변수에 할당 불가능하다. 부분집합이 아니다.
```
구조적 타이핑 vs 잉여 속성 체크
구조적 타이핑 규칙을 따른다면, 어떤 값이 다른 속성도 가질 수 있음을 의미한다. 심지어 함수의 매개변수에서도 다른 속성을 가질 수 있다.

잉여 속성 체크: 특정 상황에서 추가 속성을 허용하지 않는 타입 체크이다.

속성에 대한 인터섹션(intersection, 교집합)
```js run
interface Person {
    name: string;
}

interface Lifespan {
    birth: Date;
    death?: Date;
}

type PersonSpan = Person & Lifespan;
const a: PersonSpan = { name: 'a', birth: new Date() }
  console.log(a)
```

<img width="279" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/008eac2c-121f-46bf-918c-951b21033ef6">

하지만 타입 연산자는 인터페이스 속성이 아닌, 값의 집합(타입의 범위) 에 적용된다.
따라서 구조적 타이핑에 의해 추가적인 속성을 가지는 값도 여전히 그 타입에 속하게 된다.

keyof
type A = {
    a: string
}
type B = {
    b: string;
}

// keyof (A&B) 와 동일하다.
// keyof 연산자는 객체 타입에서 객체의 키 값들을 숫자나 문자열 리터럴 유니언을 생성합니다. 아래 타입 P는 “x” | “y”와 동일한 타입입니다.
type AnB = (keyof A) | (keyof B); // "a" | "b" -> 각 속성을 모두 포함

// keyof (A|B) 와 동일하다.
type AuB = (keyof A) & (keyof B); // never
앞에서 설명한 유니온에 대한 keyof 과, 인터섹션에 대한 keyof를 정리하면 다음과 같다.

keyof (A|B) === (keyof A) & (keyof B)
keyof (A&B) === (keyof A) | (keyof B) -> 각 속성을 모두 포함
유니온 |
유니온은 합집합이다.

타입의 합집합은 더 넓은 값의 범위를 가진다는 의미이다.

type C = A | B ;
C는 A도 포함하고 B도 포함한다. 즉 A도 C를 만족하며 B도 C를 만족한다.

만약 A한테만 name 프로퍼티가 있다면 C 타입을 매개변수로 받아 name 프로퍼티에 접근하려 할 때 B 에는 name이 없다는 오류가 발생할 것이다.

즉 C타입이 A타입보다 범위가 넓어졌기 때문에 발생한 오류다.

인터섹션 &
인터섹션은 교집합이다.

타입의 교집합은 값의 범위가 더욱 좁아진다는 의미이다.

interface A {
  name: string;
}
interface B {
  age: number;
}

type C = A & B;

const c: C = {
  name: 'hee',
  age: 25
}
### Javascript의 And 연산자와 Or연산자 
```js run
const a = '승모' || '모승'
  console.log(a)
  const b = '승모' && '모승'
  console.log(b)
  // or연산자는 값을 처음부터 비교하면서 비교할떄의 값이 true이면 해당 값을 띄워주고 
  // and연산자는 값을 비교해서 마지막 값을 띄워준다 아래 사진 참고
```
<img width="275" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/493d3a86-7441-4c4b-9d61-d4492e670c00">


타입 C는 A도 만족해야하고 B도 만족해야한다. 즉 C 의 값의 범위는 훨씬 좁아졌다.

만약 age가 없거나 name이 없다면 타입C 를 만족시키지 못한다.

서브타입 extends 키워드
서브타입은 어떤 집합의 부분집합이라는 뜻이다. 즉 값의 범위가 더욱 제한적이다.

extends 키워드로 클래스를 상속받은 서브클래스는 수퍼클래스보다 훨씬 좁은 범위의 값을 가지게 된다. extends 키워드는 집합의 부분집합을 만든다.

interface Vector1D { x: number; }
interface Vector2D extends Vector1D { y: number; }
interface Vector3D extends Vector2D { z: number; }
Vector2D는 Vector1D 의 서브타입이다.

따라서 Vector2D 타입의 값은 Vector1D 에 포함되며 해당 inferface의 타입을 모두 상속해서 쓸 수 있다.(Class의 상속과 같은 개념)

덕타입핑에 의해서 타입에 선언된 프로퍼티나 값의 범위만 만족시키면, 그 외의 다른 값들은 상관없이 동일한 타입이라고 보기 때문이다.



제네릭 타입에서 extends
function getKey<K extends string>(val: any, ket: K) {
    // ...
}
extends 키워드는 제네릭 타입에서 한정자로 쓰인다.
K extends string 에서 K는 string의 부분집합이다.

string을 상속한다는 의미는 "객체 상속 관점"에서 생각하면 어렵다.
하지만 "집합의 관점"으로 생각하면 쉽게 이해할 수 있다.
즉, K는 string의 부분 집합 범위를 가지는 어떤 타입이 된다.

getKey({}, 'x'); // 정상, 'x'는 string 을 상속
getKey({}, Math.random() < 0.5 ? 'a' : 'b'); // 정상, 'a' | 'b' 는 string을 상속
getKey({}, 12); 
    // Argument of type 'number' is not assignable to parameter of type 'string'.

                              
[아이템8] 타입 공간과 값 공간 구별하기
타입스크립트에서 이름이 같더라도 속한 공간에 따라 다른 것(타입 또는 값)을 나타낼 수 있다.

즉 같은 이름의 타입과 식별자가 있으면 둘이 속한 공간이 다르게 된다.

interface A { // 타입
  b: number;
}

const A = 1; // 변수
위 예제는 오류가 발생하지 않는다.
타입 A와 값 A는 서로 아무런 관련이 없다.

```js run
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });

function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    // instanceof는 타입이 아닌 함수를 참조한다.
    shape.radius; // '{}' 형식에 'radius' 속성이 없습니다.
  }
}
```
위의 예제도 마찬가지이다. shape instanceof Cylinder 에서 Cylinder 는 함수로 참조된다.

type vs const
type T1 = 'string literal';
const v1 = 'string literal';
위에서 type 으로 선언한 T1 은 타입, const로 선언한 v1 는 값이다.
자바스크립트 코드로 컴파일 하면 다음과 같다.

const v1 = 'string literal';
즉 타입은 컴파일 하는 과정에서 제거된다.

### typeof 연산자(얘는 원래 자스에 있던것)
타입에서 쓰일 때 : 타입스크립트의 타입
값에서 쓰일 때 : 자바스크립트 런타임 typeof 연산자
```js run
  interface Person {
 name: string; 
}
const p: Person = { name: 'hee' };

type T = typeof p; // 타입은 Person
const v1 = typeof p; // 값은 'object'
클래스에서 typeof 를 사용한 경우는 다음과 같다.
타입으로 쓰일 때 : 인스턴스 타입이 아닌, 생성자 함수
값으로 쓰일 때 : 'function'
class Cylinder {
    radius=1;
    height=1;
}

const v = typeof Cylinder; // 값이 function
type T = typeof Cylinder; // 타입이 class Cylinder, 즉 생성자 함수

const c = new fn(); // 타입이 Cylinder, 즉 인슽
만약 클래스의 인스턴스를 타입으로 사용하고 싶다면 다음과 같이 InstanceType를 작성하면 된다.

type C = InstanceType<typeof Cylinder>; // 타입이 Cylinder
  ```
속성 접근자 []
속성 접근자 []은 값 또는 타입으로 사용할 때 동일하게 동작한다.

하지만 . 접근자는 타입의 속성을 얻을 때 사용할 수 없다.

**따라서 타입의 속성을 얻을 때에는 반드시 [] 접근자를 사용해야 한다.**

type Person = {
    first: string;
    age: number;
}
const pigme = {
    first: 'hee',
    age: 20,
}

const first: Person['first'] = pigme['first']; // 정상, string
const first2: Person.first = pigme['first']; // 오류, 
// Cannot access 'Person.first' because 'Person' is a type, but not a namespace. Did you mean to retrieve the type of the property 'first' in 'Person' with 'Person["first"]'?
타입과 값 공간 사이에서 다른 의미를 가지는 코드 패턴
