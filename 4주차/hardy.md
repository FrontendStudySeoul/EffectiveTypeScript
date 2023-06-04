### Item 27: 함수형 기법과 라이브러리로 타입 흐름 유도하기

자바스크립트의 부족한 점들을 많은 라이브러리들이 표준 라이브러리의 역할을 대신하기 위해 노력해 왔습니다. DOM을 다루기 쉽지 않았던 때에는 jQuery가 쉽게 DOM을 다룰 수 있게 도와줬고 map, filter 등이 없던 시절엔 lodash가 대신 유틸리티 함수를 제공했습니다. 시간이 지나 부족한 점들을 보완하고 map, filter 등등 표준으로 만들어졌습니다. 

반복문에 관한 유틸리티 함수는 타입스크립트와 조합하여 활용하기 좋습니다. 타입 정보가 그대로 유지되면서 타입 흐름이 계속 전달되도록 하기 때문입니다. 반면 직접 루프를 구현하면 타입 체크에 대한 관리도 직접 해야합니다. 

```tsx
interface BasketballPlayer {
    name: string;
    team: string;
    salary: string;
}
declare const rosters: {[team: string]: BasketballPlayer[]};
```

이런 구조가 있다고 했을 때 반복문을 사용해 목록을 만들려면 아래와 같이 작성할 수 있습니다.

```tsx
let allPlayers = []; // any[]
for (const players of Object.values(rosters)) {
    allPlayers = allPlayers.concat(players); // any[] 형식이 포함됩니다.
}
```

코드는 동작하겠지만 타입 체크가 정상적이지 않다고 에러를 표시해줍니다. 이 에러를 고치려면 allPlayers에 타입 구문을 추가해주어야합니다.

```tsx
let allPlayers: BasketballPlayer[] = [];
for (const players of Object.values(rosters)) {
    allPlayers = allPlayers.concat(players);
}
```

그러나 앞서 말했듯 유틸리티 함수를 사용하면 보다 편하게 해결할 수 있습니다. 

```jsx
const allPlayers = Object.values(rosters).flat();
```

flat 메서드는 다차원 배열([[],[],[]])을 flat하게 만들어줍니다.

```tsx
type FlatArray<Arr, Depth extends number> = {
    "done": Arr,
    "recur": Arr extends ReadonlyArray<infer InnerArr>
        ? FlatArray<InnerArr, [-1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20][Depth]>
        : Arr
}[Depth extends -1 ? "done" : "recur"];

flat<A, D extends number = 1>(
    this: A,
    depth?: D
): FlatArray<A, D>[]
```

직접 루프를 사용해 구현할 경우 타입 구문을 통해 타입을 인지시켜야 했지만 유틸리티 함수의 경우 타입에 대한 처리가 이미 되어 있기 때문에 타입 흐름이 끊기지 않게 됩니다. 

이런 유틸리티 함수를 체인 형식으로 사용하면 타입 흐름이 매끄럽게 유지할 수 있습니다.

```tsx
interface A {
  a: string[];
}
interface B {
  b: string[];
}

const arr: Array<A | B> = [{ a: ["a", "b", "c"] }, { b: ["a", "b", "c"] }];
const hasAlphabet = (target: string, alphabet: string) => {
  return arr
    .map(item => (target in item ? item[target] : []))
    .flat()
    .some(item => item === alphabet);
};
const hasC = hasAlphabet("a", "c");
const hasB = hasAlphabet("a", "b");

// filter(item => 'a' in item)는 (A|B)[] 타입이 리턴돼서 안됨 ㅠ..
```
