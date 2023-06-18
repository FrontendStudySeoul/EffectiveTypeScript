## 아이템 35. 데이터가 아닌, API와 명세를 보고 타입 만들기

잘 설계된 타입은 타입스크립트 사용을 유용하게 해주지만, 그렇지 않은 타입은 우리를 힘들게 한다.
<br>
앞서 31장에서 사용했던 boudningBox를 계산하는 함수에서 아래와 같이 쓰였을때 문제 없으나 실제로 해당 geoJson의 타입을 넣게되는 순간 문제가 생긴다.

```tsx
import {Feature} from "geojson" ;

function calculateBoundingBox(f:Feature):BoundingBox|null{
    let box:BoundingBox|null =null;

    const helper = (coords:any[]){

    }
    const {geometry} =f;
    if(geometry){
      helper(geometry.coordinates)
      //geometry에 coordinates가 없음 -> GeometryCollection에서 coordinates 프로퍼티가 없음
    }
    return box
  }
```
그래서 이를 차단하기 위해 GeometryCollection일때 예외를 던지는 방법으로 해결할 수 있다.
```tsx
 const {geometry} =f;
    if(geometry){
      if(geometry.type==="GeometryCollection"){
    throw new Error("GeometryCollection are not supported")
  }
      helper(geometry.coordinates)
      //geometry에 coordinates가 없음 -> GeometryCollection에서 coordinates 프로퍼티가 없음
    }
```

하지만 부분적으로 에러를 띄워주는거보다 모든 타입을 받아서 해결할 수 있도록 만들어주면 좋다
```tsx
const geometryHelper = (g:Geometry)=>{
  if(geometry.type==="GeometryCollection"){
    geometry.geometries.forEach(geometryHelper)
  }else{
    helper(geometry.coordinates)
  }
}
 const {geometry} =f;
    if(geometry){
     geometryHelper(geometry)
    }
```

이와 같은 문제가 생긴이유는 Geojson이라는 데이터에 의존해서 타입을 만들어서 야기된 문제이다.
<br>
GraphQL 처럼 자체적인 타입이 정의된 API에서 잘 동작한다. 쿼리에서 타입을 생성하기 위해 GraphQL 스키마가 필요하고 Apollo를 통해 스키마를 얻는다. 이를 통해 명세로부터 타입을 가져올 수 있고, 타입과 실제 값이 항상 일치하여 좋다!

### 실제 인터페이스로 타입 만들기
#### Swagger
<img width="1388" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/fd425f4b-43e6-4957-9bc1-6c771d617956">
<img width="1408" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/a331523d-72c1-46ec-9d99-71c2b74392d6">

[실제로 살펴보기](https://nginx-nginx-883524lbvylaj6.gksl2.cloudtype.app/api/swagger-ui/#/member-controller/oauthLoginUsingPOST)
#### Postman

<img width="548" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/31b84f72-3ec1-4c96-bf7b-f196fb150c88">
[실제로 살펴보기]([https://nginx-nginx-883524lbvylaj6.gksl2.cloudtype.app/api/swagger-ui/#/member-controller/oauthLoginUsingPOST](https://documenter.getpostman.com/view/14876606/2s935iu6Hm#9b2af9a7-1195-4c5f-9010-2a0421b92cd1)https://documenter.getpostman.com/view/14876606/2s935iu6Hm#9b2af9a7-1195-4c5f-9010-2a0421b92cd1)
