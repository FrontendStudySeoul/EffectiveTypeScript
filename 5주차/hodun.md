# 5주차

## 아이템 34. 부정확한 타입보다는 미완성 타입을 사용하기

> 잘못된 타입은 차라리 타입이 없는 것보다 못할 수 있다.
> 

**예시1**

```tsx
interface Point {
  type: 'Point';
  coordinates: number[][];
}

interface LineString {
  type: 'LineString';
  coordinates: number[][];
}

interface Polygon {
  type: 'Polygon';
  coordinates: number[][];
}

type Geometry = Point | LineString | Polygon
```

```tsx
{
  "type": "Feature",
  "geometry": {
    "type": "Polygon",
    "coordinates": [
      [
        [126.7734180184775, 37.7038713680074],
        [126.79383422613847, 37.726134752272024],
        [126.8367502312495, 37.72745685434032],
        [126.85596876645158, 37.69408701295549],
        [126.82063262578045, 37.67091922887858],
        [126.79403546931206, 37.628218819529636],
        [126.75952016063324, 37.62969347500704],
        [126.73245979378379, 37.647521477939826],
        [126.77746926192918, 37.678688083293544],
        [126.7734180184775, 37.7038713680074]
      ]
    ]
  },
},
```

- 좌표에 사용되는 `number[]`가 추상적

- 튜플 타입으로 변경한다면?

```tsx
type GeoPosition = [number, number];
interface Point {
  type: 'Point';
  coordinates: GeoPosition;
}
```

- 더 나은 코드가 됐을까?
- 만약에 기존에 사용하던 사람들은 위경도 외에도 고도 등의 추가 정보를 표현하고 있을 수 있다.
- 그런 사용자들은 모두 타입 체커를 통과하기 위해 별도로 처리를 해주어야 함. (`as any`, 타입 단언문)

**예시2**

```tsx
12
"red"
["+", 1, 2]
["/", 20, 2]
["case", [">", 20, 10], "red", "blue"]
["rgb", 255, 0, 127]
```

1. 모두 허용
2. 문자열, 숫자, 배열 허용

```tsx
type Expression1 = any;
type Expression2 = number | string | any[];
```

1. 문자열, 숫자, 알려진 함수 이름으로 시작하는 배열 허용

```tsx
type FnName = '+' | '-' | '*' | '/' | '>' | '<' | 'case' | 'rgb';
type CallExpression = [FnName, ...any[]];
type Expression3 = number | string | CallExpression;
```

1. 각 함수가 받는 매개변수의 개수가 정확한지 확인
- 모든 함수 호출을 확인: 재귀적으로 동작
- 인터페이스 직접 만들기

```tsx
type Expression4 = number | string | CallExpression;
type CallExpression = MathCall | CaseCall | RGBCall;

interface MathCall {
  0: '+' | '-' | '*' | '/' | '>' | '<';
  1: Expression4;
  2: Expression4;
  length: 3;
}

interface CaseCall {
  0: 'case';
  1: Expreesion4;
  2: Expression4:
  3: Expression4;
  length: 4 | 6 | 8 // 등등
}

interface RGBCall {
  0: 'rgb';
  1: Expreesion4;
  2: Expression4:
  3: Expression4;
  legnth: 4; 
}
```

```tsx
const tests: Expreesion4[] = [
  10,
  'red',
  true, // ❌
  ["+", 10, 5],
  ["case", [">", 20, 10], "red", "blue", "green"], // ❌
  ["**", 2, 31], // ❌
  ["rgb", 255, 128, 64],
  ["rgb", 255, 128, 64, 73] // ❌
]
```

- 무표한 표현식 전부 오류 발생
- 하지만 오류 메시지가 부정확해짐
    - `+`와 `*`는 더 많은 매개변수를 받을 수 있음
    - `-`는 한개의 매개변수만 필요
    
    ```tsx
    const okExpressions: Expression4[] = [
      ['-', 12], // ❌ 'number'형식은 'string'에 형식에 할당할 수 없습니다.
      ['+', 1, 2, 3], // ❌ 'number'형식은 'string'에 형식에 할당할 수 없습니다.
      ['*', 2, 3, 4], // ❌ 'number'형식은 'string'에 형식에 할당할 수 없습니다.
    ]
    ```
    
1. 각 함수가 받는 매개변수의 타입이 정확한지 확인

### 정리

- 코드를 더 정밀하게 만들려던 시도과 과해지면 오히려 코드가 더 부정확해짐
- 타입으로 부정확함을 바로 잡는 방법보다 테스트 세트를 추가해 확인하는 방법도 존재
- 타입이 구체적으로 정제된다고 하더라도 정확도가 비례하진 않음
