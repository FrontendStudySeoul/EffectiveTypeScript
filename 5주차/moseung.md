## 아이템 32. 유니온의 인터페이스보다는 인터페이스의 유니온 사용하기

유니온 타입의 인터페이스 사용을 고려중이라면, 인터페이스의 유니온을 쓰는게 더 알맞는게 아닌지 겈토해야한다.

```tsx
interface Layer {
    layout: FillLayout | LineLayout | PointLayout
    paint: FillPaint | LinePaint | PointPaint
  }
  //layout은 모양이 그려지는 방법
  //paint는 속성의 스타일
```
근데 이런식으로 코드를 작성하면 FillLayout이면서 LinePaint인 혼종이 탄생하게 됩니다.

```tsx
interface FillLayer{
    layout : FillLayout;
    paint: FillPaint;
  }

  interface LineLayer{
    layout : LineLayout;
    paint: LinePaint;
  }

  interface PointLayer{
    layout : PointLayout;
    paint: PointPaint;
  }

  type Layer = FillLayer | PointLayer | LineLayer
```
<img width="750" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/c219a5c4-43d2-40d9-8531-dac2d84207ea">

이런식으로 정의하게된다면 위에서처럼 layoutr과 point가 다른 혼종은 나올 수 없게 된다. 실제 사용 예시로 가보자.

```tsx
interface Layer {
    type : "fill" | "line" | "point"
    layout: FillLayout | LineLayout | PointLayout
    paint: FillPaint | LinePaint | PointPaint
    //❌ 위처럼 혼종인 결과가 나옴
  }
```
```tsx
interface FillLayer{
    type : "fill";
    layout : FillLayout;
    paint: FillPaint;
  }

  interface LineLayer{
    type : "line";
    layout : LineLayout;
    paint: LinePaint;
  }

  interface PointLayer{
    type : "point";
    layout : PointLayout;
    paint: PointPaint;
  }

  type Layer = FillLayer | PointLayer | LineLayer
  
  function drawLayer(layer: Layers) {
    if (layer.type === 'fill') {
      const { paint } = layer
      const { layout } = layer
      //비즈니스로직
    } else if (layer.type === 'line') {
      const { paint } = layer
      const { layout } = layer
            //비즈니스로직
    } else if (layer.type === 'point') {
      const { paint } = layer
      const { layout } = layer
            //비즈니스로직
    }
  }
```
위의 코드처럼 type을 추가해 해당 레이어를 그리는 함수를 만들 수 있고 우리가 원하는 기능을 타입스크립트로 추천을 받아서 사용했다.

```tsx

  interface Person {
    name:string;
    //다음은 둘 다 있거나 둘 다 없을 수 있습니다.
    placeOfBirth?:string;
    dateOfBirth?:Date;
  }
```
타입정보를 담고 있는 주석은 문제가 될 소지가 매우 많다.<br>
<img width="302" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/94a87703-96e0-444b-b5d0-8197702f43b1">
<br>
그래서 불필요한 주석은 없애고 아래처럼 코드로 명확하게 해주는것이 좋다.
```tsx
interface Person {
    name:string;
    birth:{
      place:string;
      date:Date;
    }
  }
  
   function eulogize(person: Person) {
    if (person.birth) {
      console.log(
        `이 아이가 태어난 곳은 ${person.birth.place}이고 태어난 날은 ${person.birth.date}입니다.`
      )
    }
  }
// 함수를 사용해서 해당 birth를 체크해서 만들 수 있다.
```

유니온의 인터페이스는 오류가 발생하거나 가독성이 좋지 않으므로 인터페이스의 유니온을 사용하는 편이 좋다.
