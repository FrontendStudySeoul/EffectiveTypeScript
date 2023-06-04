## 아이템 24. 일관성있는 별칭 사용하기

```tsx
interface Coordinate {
  x: number;
  y: number;
}

interface BoundingBox {
  x: [number, number]
  y: [number, number]
}

interface Polygon {
  exterior: Coordinate[];
  holes: Coordinate[][];
  bbox?: BoundingBox;
}

function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  if(polygon.bbox) {
    if(pt.x < ploygon.bbox.x[0] || pt.x > ploygon.bbox.x[1] ||
	    pt.y < polygon.bbox.y[0] || pt.y > polygon.bbox.y[1] {
        return false;
    }
  }
}
```

<br />

### 반복되는 `polygon.bbox` 줄이기

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox;
  if(polygon.bbox) {
    if(pt.x < box.x[0] || pt.x > box.x[1] || pt.y < box.y[0] || pt.y > box.y[1] {
      return false;
    }
  }
}
```

⚠️ `box`는 `undefined`일 수 있음

```tsx
const box = poloygon.bbox;
```

- 해당 시점에서 `polygon.bbox`의 타입이 `BoundBox | undefined`
- 따라서 box의 타입은 `BoundBox | undefined`

<br />

```tsx
if(polygon.bbox) {
  if(pt.x < box.x[0] || pt.x > box.x[1] ||
    pt.y < box.y[0] || pt.y > box.y[1] {
      return false;
  }
}
```

- 분기문에서 `poloygon.bbox`의 타입은 `BoundingBox`로 좁혀졌지만 box의 타입은 여전히 `BoundBox | undefined`

<br />

따라서 별칭은 일관성 있게 사용해야 한다.

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const box = polygon.bbox;
  if(box) {
    if(pt.x < box.x[0] || pt.x > box.x[1] || pt.y < box.y[0] || pt.y > box.y[1] {
        return false;
    }
  }
}
```

- 이젠 분기문내 box의 타입은 `BoundingBox`로 좁혀짐

<br />

### 개선하기

`box`와 `bbox`는 같은 값인데 다른 식별자를 사용해 코드를 읽는 사람에게 혼란을 야기할 수 있다.

객체 비구조화 할당을 이용해 일관된 이름을 사용하자.

```tsx
function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
  const { bbox } = polygon.bbox;
  if(bbox) {
    const { x, y } = bbox
    if(pt.x < x[0] || pt.x > x[1] || pt.y < y[0] || pt.y > y[1] {
      return false;
    }
  }
}
```

<br />

### 주의

```tsx
const { bbox } = poloygon;

if(!bbox) {
  calculatePolygonBbox(polygon); // polygon의 bbox가 채워집니다.
}
```

![Untitled](https://file.notion.so/f/s/fc05fbdb-903f-4c6b-8d81-4568962bfffc/Untitled.png?id=2e961884-ad8a-47f0-99de-3630ed669210&table=block&spaceId=460aeeee-d1c6-4253-8817-e4cfb135299e&expirationTimestamp=1685936206514&signature=nS-ziP5eQR39KunT6wp-SsDV4tVPUMmwx8NiN9n-6bE&downloadName=Untitled.png)

`bbox`는 `polygon`의 `bbox`가 채워지기 이전의 `bbox`를 참조하고 있다.

![Untitled](https://file.notion.so/f/s/52fdfc44-02cc-439f-95fc-e99006e053dc/Untitled.png?id=0b276cb7-1993-4620-970c-b31d8f7d23d4&table=block&spaceId=460aeeee-d1c6-4253-8817-e4cfb135299e&expirationTimestamp=1685936217795&signature=C4sb8lF-Ox92adVXGvxi85uBrrHglY-VUokkXgH6sDo&downloadName=Untitled.png)

하지만 분기문내에서 `polygon`의 `bbox`가 채워지게 되고 **서로 다른 값을 참조**하게 된다.

<br />

```tsx
function fn(p: Polygon) { /* ... */ };

polygon.bbox // 타입이 BoundingBox | undefined
if(polygon.bbox) {
  polygon.bbox // 타입이 BoundingBox
  fn(polygon.bbox); 
  polygon.bbox // 타입이 BoundingBox
}
```

- `fn(polygon)`은 `polygon.bbox`를 제거할 가능성이 있다.
- 함수호출 이후 `polygon.bbox`의 타입은 `BoundBox | undefined`가 되어야 안전함

<br />

```tsx
const {bbox} = polygon 
if(bbox) {
  bbox 
  fn(polygon); 
  bbox
}
```

- `bbox`는 분기문에 들어가기전 `polygon`의 `bbox`를 참조하고 있다.
- 함수 호출이후 `bbox`의 타입은 여전히 `BoundBox`
- 타입 정제를 믿을 수 있다.

<br />

### 타입스크립트 처음했을 때 많이 했던 실수

```tsx
function Component() {
  const ref = useRef<HTMLDivElement>();
  ...
  const fn = () => {
    const element = ref.current;
    if(!ref.current) return;
    doSomething(element);
  }
	...
}
```

### 정리

- 별칭은 타입을 좁히는 것을 방해함 → 일관되게 사용
- 비구조화 할당을 사용해 일관된 이름 사용
- 속성보다 지역 변수를 사용해 타입 정제 신뢰성 높이기
