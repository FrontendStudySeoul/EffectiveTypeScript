## item 29: 사용할 때는 너그럽게, 생성할 때는 엄격하게

**포스텔의 법칙**

포스텔의 법칙(Postel's law)은 네트워킹에 있어서 중요한 디자인 원칙 중 하나입니다. 그것은 "로버스트니스 원칙"(Principle of Robustness)이라고도 불리며, 1981년 RFC 791 문서에서 처음 언급되었습니다. 이 원칙은 네트워크 엔지니어와 소프트웨어 개발자 모두에게 영향을 미칩니다.

포스텔의 법칙은 "자신의 출력은 엄격하게 (strict) 검증하되, 다른 사람의 입력은 관대하게 (liberal) 받아들이라"는 것입니다. 이는 인터넷 프로토콜을 설계하고 구현하는 방식에 대한 기본 지침으로, 이로 인해 다양한 프로토콜과 시스템이 호환성을 유지하면서 효과적으로 동작할 수 있게 됩니다.

이 원칙에 따라, 개발자들은 자신의 시스템이 규약을 정확히 따르도록 만들면서, 동시에 다른 시스템들로부터의 일부 완벽하지 않은 입력도 처리할 수 있도록 유연성을 유지합니다. 이런 유연성이 있기에 소프트웨어와 하드웨어 시스템들이 서로 다른 구현방식과 버전에도 불구하고 상호작용하고, 그로 인해 네트워크 전체가 더욱 견고해집니다.

---

<aside>
💡 함수의 매개변수는 타입의 범위가 넓어도 되지만, 결과를 반환할 때는 일반적으로 타입의 범위가 더 구체적이여야 합니다.
</aside>
<br/>
<br/>
<br/>


```tsx
declare function setCamera(camera: CameraOptions): void;
declare function viewprotForBounds(bounds: LngLatBounds): CameraOptions;

interface CameraOptions {
	center?: LngLat;
	zoom?: number;
        bearing?: number;
	pitch?: number;
}
type LngLat = 
	{ lng: number; lat: number } | 
	{ lon: number; lat: number } | 
	[number., number]
type LngLatBounds = 
	{ northeast: LngLat ; southwest: LngLat } | 
	[LngLat, LngLat ] | 
	[number, number, number, number]
```

CameraOptions의 타입은 일부 값은 건드리지 않으면서 동시에 다른 값을 설정할 수 있어야 하므로 모두 옵셔널 타입입니다. LngLat의 경우엔 매개 변수로 다양한 값이 들어갈 수 있어 유니온 타입으로 자유롭게 만들어진 타입입니다.

```tsx
function focusOnFeature(f: Feature) {
	const bounds = ...
	const camera = viewprostForBounds(bounds);
	setCamera(camera);
  const {center: {lat,lng}, zoom} = camera;
	// 'lat', 'lng' 속성이 없습니다.
	zoom; // 타입이 number | undefined 
}
```

실제로 사용하게 되면 center는 유니온 타입이라 lat과 lng을 찾을 수 없는 문제가 발생하고 zoom은 옵셔널 타입이라 undefined일 가능성이 생겼습니다.근본적인 문제는 viewportForBounds의 타입 선인이 사용될 때뿐만 아니라 만들어질 때에도 너무 자유롭다는 것입니다. 이로 인해 viewprostForBounds을 사용하기 어렵게 만듭니다.

매개변수 타입의 범위가 넓으면 사용하기 편리하겠지만, 반환 타입의 범위가 넓으면 불편해집니다. 즉, 사용하기 편리한 API일수록 반환 타입이 엄격합니다.

이런 경우의 유니온 타입은 타입의 각 요소별로 코드를 분기하는 것이 좋습니다. 요소별 분기를 위한 한 가지 방법은 좌표를 위한 기본 형식을 구분하는 것입니다.

```tsx
interface LngLat { lng: number; lat: number; };
type LngLatLike = LngLat | { lon: number; lat: number; } | [number, number];

interface Camera {
	center: LngLat;
	zoom: number;
  bearing: number;
	pitch: number;
}
interface CameraOptions extends Omit<Partical<Camera>, 'center'> {
	center?: LngLatLike;
}
type LngLatBounds = 
	{ northeast: LngLatLike ; southwest: LngLatLike  } | 
	[LngLatLike , LngLatLike ] | 
	[number, number, number, number]

declare function setCamera(camera: CameraOptions): void;
declare function viewprotForBounds(bounds: LngLatBounds): Camera;

function focusOnFeature(f: Feature) {
	const bounds = ...
	const camera = viewprostForBounds(bounds);
	setCamera(camera);
  const {center: {lat,lng}, zoom} = camera; // 정상
	zoom; // 타입이 number
}
```

타입을 반환 타입과 매개변수 타입으로 나눔으로서 앞서 생긴 문제가 해결됐습니다.
