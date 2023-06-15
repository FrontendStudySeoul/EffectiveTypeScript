## 아이템 36. 해당 분야의 용어로 타입 이름 짓기

### 모호하고 의미 없는 이름 피하기

data, info, thing, item, object, entity와 같이 모호하고 의미 없는 이름은 피해야 한다.

<br />

## 아이템 41. any의 진화를 이해하기

> 변수의 타입은 변수를 선언할 때 결정된다.

### any의 진화 과정

- any 타입의 진화는 암시적 any 타입에 어떤 값을 할당할 때만 발생한다.
- 어떤 변수가 암시적 any 상태일 때 값을 읽으려고 하면 오류가 발생한다.

```typescript
const result = []; // 여기서 타입은 any[]

result.push("a");
// 여기서 result 타입은 string[]
result.push(1);
// 여기서 result 타입은 (string | number)[]
```

<br />

### 명시적으로 any를 선언한 경우

> any 타입의 진화는 noImplicitAny가 설정된 상태에서 변수의 타입이 암시적 any인 경우에만 일어난다. 즉, 명시적으로 any를 선언하면 타입이 any로 유지된다.

```typescript
let val: any;

if (Math.random() < 0.5) {
  val = /hello/; // 타입은 any
} else {
  val = 12; // 타입은 any
}
// 여기서 val 타입은 any
```

<br />

### 암시적 any를 진화시키는 것보다 명시적 타입 구문을 사용하자.

- 만약 암시적 any로 추론된 변수를 진화시키지 않거나 타입을 지정하지 않는 경우 오류가 발생한다.
- 암시적 any는 함수 호출을 거쳐도 진화하지 않을 수 있다.

```typescript
function range(start: number, limit: number) {
  const out = []; // any[]
  [1, 2, 3].forEach((v) => {
    out.push(v);
  });
  return out; // any[]
}
```
