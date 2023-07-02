## 아이템 47. 공개 API에 등장하는 모든 타입을 익스포트하기

```ts
interface SecretName {
  first: string;
  last: string;
}

interface SecretSanta {
  name: SecretName;
  gift: string;
}

export function getGift(name: SecretName, gift: string): SecretSanta {
  // ...
}
```
- 해당 라이브러리 사용자는 `SecretName`, `SecretSanta`를 직접 임포트할 수 없음
- 타입들은 함수 시그니처에 등장
  - 추출 가능

```ts
type MySanta = ReturenType<typeof getGift>;
type MyName = Parameters<typeof getGift>[0];
```

### 정리
- 공개 메서드에 등장한 타입은 export 하자.
- 어짜피 사용자가 추출할 수 있으므로 export 하기 쉽게 하는것이 좋다.
