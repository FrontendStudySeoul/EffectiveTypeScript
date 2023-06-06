## 아이템 25. 비동기 코드에는 콜백 대신 async 함수 사용하기

### 비동기 코드

콜백(콜백 지옥 초래) -> 프로미스 -> async, await 키워드 도입

### 콜백보다는 프로미스나 async/await을 사용해야 하는 이유

1. 코드 작성이 쉬움
2. 타입 추론에 유리

```typescript
// async 화살표 함수
const getNumber = async () => 42; // () => Promise<number>

// 프로미스를 직접 생성한 경우
const getNumber = () => Promise.resolve(42); // () => promise<number>
```

```typescript
// 반환 타입을 지정하지 않아도 반환 타입은 Promise<Response>로 추론된다.
async function fetchWithTimeout(url: string, ms: number) {
  // Promise<Response | never> => Promise<Response>
  return Promise.race([fetch(url), timeout(ms)]);
}
```

### 프로미스를 생성하기보다 async/await을 사용해야 하는 이유

1. 간결하고 직관적이다.
2. async 함수는 항상 프로미스를 반환하도록 강제된다.

### async 함수에서 프로미스를 반환하는 경우

async 함수에서 프로미스를 반환하면 또 다른 프로미스로 래핑되지 않는다. 즉, 반환 타입은 Promise<Promise<T>>가 아닌 Promise<T>가 된다.
