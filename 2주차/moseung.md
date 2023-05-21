## [아이템9] 타입 단언보다는 타입 선언을 사용하기
타입스크립트에 타입을 부여하는 방법은 2가지가 있다.

```js run
 interface Person {
    name: string
  }

  const bob = {
    name: 'alice',
    occupation: 'developer',
  } as Person
// 왜 위에 코드가 문제가 없을까요 ?
```
<img width="664" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/a4657855-88ed-4bc0-973e-ac03bf3c29b0">

예시 코드를 하나 가져와 봤어요.
```js run
const files = e.target.files as FileList

```



타입 선언과 타입 내로잉을 통해서 확실한 코드를 작성할 수 있습니다.
```js run
if (e.target.files !== null) {
    const files: FileList = e.target.files
    const getSize = () => {
      for (let i = 0; i < files.length; i++) {
        const convertedSize = getByteSize(files[i].size)
        if (convertedSize > MAX_FILE_SIZE) {
          callbackFn()
          isOver5MB = true
          break
        }
      }
    }
    ;[].forEach.call(e.target.files, getSize)
    return isOver5MB
  }
```
<img width="481" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/9524324f-78f0-4b11-a486-b74c718e6c3f">

책에서 말한것처럼 자바스크립트의 Dom이벤트를 다룰때 타입 단언을 쓰는 경우가 많은것 같아요 특히 e.target.value같은것들

```js run
const el = document.getElementById('foo')!
//위와 같은 타입 단언대신 !를 사용할 수도 있다
```


## [아이템10] 객체 래퍼 타입 피하기
[프로토타입이란](https://github.com/Wooteco-JS-study/Modern_Javascript_Teco/blob/main/2%EC%A3%BC%EC%B0%A8/cruelladevil.md)<br>
[프로토타입 훑어보기](https://github.com/Ulsan-JS-Study/modern-javascript-deepdive-study/blob/main/1%EC%A3%BC%EC%B0%A8/heeseop.md)
결론 앞글자 소문자 잘쓰자.

## [아이템11] 잉여 속성 체크의 한계 인지하기
```js run
interface Person {
  name: string;
}

const bob: Person = {
  name: 'Alice',
  occupation: 'Developer',
};
```

위처럼 occupation이 없어도 잉여 속성 체크를 사용해 에러를 내뱉는다.
<br>
#### 9장에 타입단언과 이어져서 타입단언으로 잉여 속성 체크를 무시할 수 있다.
#### --noImplicitAny 옵션을 통해 잉여속성 체크를 강화할 수 있습니다. 
## [아이템12] 함수 표현식에 타입 적용하기
솔직히 매개변수와 리턴타입을 넣는거 보다는 함수 타입을 적으라고 하는데 공통적인 함수가 있지 않은 이상 저는 잘 모르겟어요.

```js run
type AddNumberFn = (a:number,b:number) => number;
const addNumber:AddNumberFn = (a,b)=>{
return a + b ;
}
```

```js run
// FunctionType.ts
type AddNumberFn = (a:number,b:number) => number;

// getAddNumber
const addNumber:AddNumberFn = (a,b)=>{
return a + b ;
}
이럴 경우에는?
```

<img width="678" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/50b26b93-28f1-4820-b4db-b32ac8cc4d6f">

챗 지피티도 그렇게 생각한다고 합니다.
## [아이템13] 타입과 인터페이스의 차이점 알기
#### 타입에서만 union과 인터섹션이 사용 가능하다(|,&)
#### inferface에서는 확장이 가능하다
#### 팀 내에서 적절한 convention을 두고 사용하면 된다
#### props는 무조건 interface로 통일하기, 서버 통신 api에 타입도 interface(객체와 똑같은 형태처럼 생겼기 때문)로 통일하기 등등

## [아이템14] 타입 연산과 제너릭 사용으로 반복 줄이기
```js run
export interface IRecordDataType {
  recordId: number
  categoryId: number
  categoryName: string
  title: string
  content: string
  writer: string
  colorName: string
  iconName: string
  createdAt: string
  imageUrls: string[]
}

export interface RecordCategory {
  recordId: number
  title: string
  iconName: string
  colorName: string
}

export interface CategoryCard {
  colorName: string
  commentCount: number
  iconName: string
  recordId: number
  title: string
}
// 어떻게 바꿀 수 있을까요 ?
```

```js run
export interface RecordData extends CardInfo {
  recordId: number;
  categoryId: number;
  categoryName: string;
  content: string;
  writer: string;
  createdAt: string;
  imageUrls: string[];
}

export interface RecordCategory extends CardInfo {
  recordId: number;
}

export interface CategoryCard extends CardInfo {
  commentCount: number;
  recordId: number;
}

export interface CardInfo {
  title: string;
  icon: string;
  color: string;
}
```
근데 위처럼 줄이는게 무조건 가독성 좋을까요 ? 실제로 코드를 쓸때 타입힌트 받을땐 모르지만 실제로 이 페이지에 왔을때 해당 타입이 뭘 extends했는지 확인해야한다는 단점이 있어서 전에는 그냥 다 풀어썼습니다.

## [아이템15] 동적 데이터에 인덱스 시그니처 사용하기
```js run
type Vec3D = Record<'x' | 'y' | 'z', number>
  const a: Vec3D = { x: 1, y: 2, z: 3 }
  console.log(a)
```

#### 어쩔수 없다면 Record나 interface를 쓰는게 좋다.
## [아이템16] number 인덱스 시그너첩다는 Array, 튜플, ArrayLike를 사용하기
유사배열
![image](https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/7b5e3492-0bc2-49c9-a7c7-ef31e7e32baa)

#### 인덱스 시그니처에 number보다는 array나 arrayLike 사용하는게 좋음

