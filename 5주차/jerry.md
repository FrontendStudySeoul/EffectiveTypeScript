## 아이템 33. string 타입보다 더 구체적인 타입 사용하기

### 문자열 리터럴 타입의 유니온 사용하기

- string 타입이면서 값이 정해져있는 경우, 유니온 타입으로 정의해 string 타입의 범위를 좁힐 수 있어요.

```typescript
// AdminRole은 아래 3가지 값만 가능한 경우
type AdminRole = string; // (x)
type AdminRole = "manager" | "director" | "staff"; // (o)
```

<br />

### 객체 속성의 키 타입 추출하기

객체 속성의 키 타입은 넓은 범위인 string보다 가능한 키 값들의 유니온으로 작성해 타입을 좁히는 것이 나아요.

```typescript
interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}

// keyof: 객체의 모든 프로퍼티 키 값을 유니온 형태로 반환해요.
type AlbumKeys = keyof Album; // "artist" | "title" | "releaseDate" | "recordingType"
```

<br />

### keyof를 사용해 하드코딩 피하기

```tsx
import { AiOutlineFullscreenExit, AiOutlineArrowLeft } from "react-icons/ai";
import { BiTargetLock, BiPlus, BiMinus } from "react-icons/bi";

const BUTTON_ICONS = {
  "current-location": <BiTargetLock />,
  "zoom-in": <BiPlus />,
  "zoom-out": <BiMinus />,
  "full-screen": <AiOutlineFullscreenExit />,
  "go-back": <AiOutlineArrowLeft />,
};

type ButtonIconType = keyof typeof BUTTON_ICONS;
// "current-location" | "zoom-in" | "zoom-out" | "full-screen" | "go-back"

interface ControlButtonProps {
  ButtonType: ButtonIconType;
  // ...
}

const ControlButton = ({ ButtonType }: ControlButtonProps) => {
  return <div>{BUTTON_ICONS[ButtonType]}</div>;
};
```

<br />

### 제네릭 + 문자열 리터럴 타입의 유니온으로 최대한 타입 좁히기

> Image 공통 컴포넌트를 구현한 코드

이미지 태그에 사용할 속성들을 모두 허용할 수도 있겠지만 필요하지 않은 속성들도 포함하는 문제가 있어요.
이를 문자열 리터럴 타입의 유니온으로 사용할 속성들만 정의해 최대한 타입을 좁힐 수 있어요.

`ImgHTMLAttributes<HTMLImageElement>` 타입 중 사용할 속성들을 AllowedAttributeKeys와 같이 정의할 수 있어요.

```tsx
import { CSSProperties, ImgHTMLAttributes } from "react";

type AllowedAttributeKeys =
  | "src"
  | "alt"
  | "placeholder"
  | "width"
  | "height"
  | "style";

export interface ImageProps
  extends Required<
    Pick<ImgHTMLAttributes<HTMLImageElement>, AllowedAttributeKeys>
  > {
  lazy: boolean;
  threshold?: number;
  block?: boolean;
  mode: CSSProperties["objectFit"];
}

const Image = ({
  lazy,
  threshold = 0.5,
  block = true,
  mode,
  src,
  alt,
  placeholder,
  width,
  height,
  ...props
}: ImageProps) => {
  // ...
};
```

### 문자열 리터럴 타입의 유니온 -> 타입 가드 코드 작성하는 방법

> 출처: (How to create a type guard for string union types in TypeScript)[https://www.qualdesk.com/blog/2021/type-guard-for-string-union-types-typescript/]

```typescript
type ColorKey = "red" | "orange" | "yellow" | "green" | "blue" | "purple";

function isColorKey(colorKey: string): colorKey is ColorKey {
  switch (colorKey) {
    case "red":
    case "orange":
    case "yellow":
    case "green":
    case "blue":
    case "purple":
      return true;
    default:
      return false;
  }
}
```

만약 새로운 색깔이 추가된다면 ColorKey라는 타입과 isColorKey 함수 내부에 작성된 타입 가드에도 수정이 필요하다는 문제점이 발생해요.

최대한 타입에 의존하는 방식으로 개선해볼까요?

```typescript
const COLORS = ["red", "orange", "yellow", "green", "blue", "purple"] as const;

type ColorKey = (typeof COLORS)[number];

function isColorKey(colorKey: string): colorKey is ColorKey {
  return COLORS.includes(colorKey as ColorKey);
}
```

COLORS 상수에 as const를 사용해 타입 단언을 하지 않으면 ColorKey 타입은 string으로 추론돼요.
as const를 사용해서 배열 내 작성한 문자열 자체를 타입으로 갖도록 할 수 있어요.

(type predicates)[https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates]

<br />

### Template Literal Type

> 출처: (Typescript - Union Type, Intersection Type, Etc.)[https://fe-developers.kakaoent.com/2022/221124-typescript-tip/]

Template Literal Type은 기존 타입스크립트의 String Literal Type을 기반으로 새로운 타입을 만들 수 있어요.

예를 들어 `연재 중인 웹툰`, `휴재 중인 소설`과 같이 각 컨텐츠의 상태를 나타내려고 한다면?

```typescript
type ContentType = "webtoon" | "novel" | "book";
type Status = "series" | "completed" | "rest";

// 템플릿 리터럴 타입을 활용하면 사용한 타입의 모든 경우의 수를 계산한 타입들의 유니온을 반환한다.
type Content = `${Status}-${ContentType}`;
// "series-webtoon" | "completed-webtoon" | "rest-webtoon" | "series-novel" ...

type Content = `${Status}${Capitalize<ContentType>}`;
// "completedWebtoon" | "seriesNovel" | "restBook" ...
```

Template Literal Type을 사용해 중복 없이 간결하게 타입을 표현할 수 있어요.

<br />

### Conditional Type과 Template Literal Type

> 출처: (Template Literal Type로 타입 안전하게 코딩하기)[https://toss.tech/article/template-literal-types]

```typescript
// fadeIn, fadeOut에서 fade 접두사를 버리고 In 또는 Out을 가져오고 싶은 상황
type InOrOut<T> = T extends `fade${infer R}` ? R : never;

// type I = "In"
type I = InOrOut<"fadeIn">;
// type O = "Out"
type O = InOrOut<"fadeOut">;
```

제너릭 타입으로 재귀적 타입 선언을 정의할 수 있어요.

```typescript
// 'foo.bar.baz'를 ['foo', 'bar', 'baz']로 나누는 타입을 정의한 코드
type Split<S extends string> = S extends `${infer T}.${infer U}`
  ? [T, ...Split<U>]
  : [S];

// type S = ["foo", "bar", "baz"];
type S = Split<"foo.bar.baz">;
```

### 타입을 좁혀 얻는 이점

1. 타입을 명시적으로 정의해 해당 값을 사용하는 곳에서 타입 정보를 확인할 수 있다.

```typescript
function getAlbumsOfType(recordingType: string) {}
getAlbumOfType(""); // 함수를 호출하는 곳에서 매개변수의 별다른 타입 정보를 얻을 수 없다.

// 개선 코드
type RecordingType = "live" | "studio";

function getAlbumsOfType(recordingType: RecordingType) {}
getAlbumOfType("live"); // 함수를 호출하는 곳에서 매개변수의 타입 정보를 확인할 수 있다.
```

2. 타입 체크를 더 엄격히 할 수 있다. -> 생산성 향상
