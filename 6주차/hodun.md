## 아이템 37. 공식 명칭에는 상표를 붙이기

```ts
interface Vector2D {
  x: number;
  y: number;
}

function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y + p.y);
}

calculateNorm({x: 3, y: 4}); // 정상, 5
const vec3D = {x: 3, y: 4, z: 1};
calculateNorm(vec3D); // 정상 5
```
- 구조적 타이핑 관점에서 문제가 없는 코드
- 하지만 **수학적으로 2차원 벡터**만 사용해야 함

### 상표 (brand)
- 3차원 벡터를 허용하지 않으려면 상표를 붙이기
- 상표를 붙여서 `Vector2D`라고 알려줌
```ts
interface Vector2D {
 _brand: '2D';
 x: number;
 y: number;
}
```
<img width="585" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/65100540/b7732e84-159b-4136-ae78-6ba8a56e573e">

<br />

**이진탐색**
```ts
function binarySearch<T>(xs: T[], x: T): boolean {
  let low = 0, high = xs.length - 1;
  while(high >= low) {
    const mid = low + Math.floor(high - low) / 2);
    const v = xs[mid];
    if(v === x) return true;
    [low, high] = x > v ? [mid + 1, high] : [low, mid - 1];
  }
  return false;
}
```
- 이진탐색은 정렬되어 있는 상태에서 유효함
- 타입 시스템에서 목록이 정렬되어 있다는 것을 표현하기는 어려움
- 상표 사용하기

```ts
type SortedList<T> = T[] & {_brand: 'sorted'};

function isSorted<T>(xs:T[]): xs is SortedList<T> { ... }

function binarySearch<T>(xs: SortedList<T>, x: T): boolean { ... }
```
- isSorted 함수에서 list 전체를 루프 도는것이 효율적인 방법은 아니지만, 안전정은 확보함

**number**

```ts
type Meters = number & {_brand: 'meters'};
type Seconds = number & {_brand: 'seconds');

const meters = (m: number) => m as Meters;
const seconds = (s: number) => s as Seconds;

const oneKm = meters(1000);
const oneMin = seconds(60);
```
- number 타입에 상표를 붙여도 산술 연산 후에는 상표가 없어짐
```ts
const tenKm = oneKm * 10; // number
const v = oneKm / oneMin; // number
```
- 코드에 여러 단위가 혼합되어 있는 경우 숫자 단위를 문서화하는 좋은 방법이 될 수 있음
