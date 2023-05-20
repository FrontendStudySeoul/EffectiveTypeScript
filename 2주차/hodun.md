### 아이템9. 타입 단언보다는 타입 선언을 사용하기

```ts
interface Person {
  name: string;
}

const alice: Person = { name: "Alice" }; // 타입은 Person
const bob = { name: "Bob" } as Person; // 타입은 Person
```

타입 선언: `: Person`
타입 단언: `as Person`

타입 단언시 타입스크립트가 추론한 타입이 있더라도 해당 타입으로 간주함

- 타입 단언 없는 경우

  ```ts
  const bob = { name: "Bob" };
  ```

  <img height=200 src="https://private-user-images.githubusercontent.com/65100540/239682959-8c6c9264-e36f-4f2d-91d4-b4eed41843fa.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJrZXkiOiJrZXkxIiwiZXhwIjoxNjg0NTc5OTM5LCJuYmYiOjE2ODQ1Nzk2MzksInBhdGgiOiIvNjUxMDA1NDAvMjM5NjgyOTU5LThjNmM5MjY0LWUzNmYtNGYyZC05MWQ0LWI0ZWVkNDE4NDNmYS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBSVdOSllBWDRDU1ZFSDUzQSUyRjIwMjMwNTIwJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIzMDUyMFQxMDQ3MTlaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT0wMTViZGY0ODUyMzE4ZWFkNTZiOWMzOWZkYzY0MDFiNDVhYTFhOWYwMmJiNTcwYmU5NmYzMzI2MzZkMTM1OGY4JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.X8mKDhrMzV3T1hvmFcylL93dWeXc8xsW0ObFvEz27JE" />

- 타입 단언 있는 경우

  ```ts
  const bob = { name: "Bob" } as Person;
  ```

    <img height=200 src="https://private-user-images.githubusercontent.com/65100540/239682961-e9231307-d353-4fac-8ef8-dbdcef2ec050.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJrZXkiOiJrZXkxIiwiZXhwIjoxNjg0NTc5OTM5LCJuYmYiOjE2ODQ1Nzk2MzksInBhdGgiOiIvNjUxMDA1NDAvMjM5NjgyOTYxLWU5MjMxMzA3LWQzNTMtNGZhYy04ZWY4LWRiZGNlZjJlYzA1MC5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBSVdOSllBWDRDU1ZFSDUzQSUyRjIwMjMwNTIwJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIzMDUyMFQxMDQ3MTlaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1kYjY2OGM0Yzg3ZGJmYmFlYzkwYWQ3OGI2ZmUxYWNjNTVlNTk0OGYxZTAwNjdjNjA3MzRiMDVkOTE4YTEwYzE5JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.bsP6SpP7LiAHq90sBZV7zS-yJvTs0WW2M3Ly0rm0zOU" />

**궁금한점**

```ts
const foo = {} as string; // ✅
const foo = { name: "hi" } as string; // ❌
```

- `{}`, `{name: "hi"}`둘 다 객체 리터럴인데 string 타입 단언 결과가 다름
- **뇌피셜**

  - `{}`은 구조적 타이핑 관점에서 무엇이든 될 수 있음
  - 따라서 `as string`이 가능하지 않을까?

    ```ts
    const foo = {} as "hello";
    const foo = {} as 100;
    const foo = { slice: () => "" } as string;
    ```

### 아이템10. 객체 래퍼 타입 피하기

- 기본형

  - `object`, `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`
  - **불변**이며 **메서드를 가지지 않는다.**

- 자바스크립트에 `String '객체'` 타입 정의

  - 기본형을 **String으로 래핑**

- ⚠️ `string`은 `String`에 할당가능, `String`은 `string`에 할당 불가능

  ```ts
  const foo = { slice: () => "" } as string;
  ```

  ![](https://private-user-images.githubusercontent.com/65100540/239682957-1084b292-e2de-44e8-8569-497c7436b25f.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJrZXkiOiJrZXkxIiwiZXhwIjoxNjg0NTc5OTM5LCJuYmYiOjE2ODQ1Nzk2MzksInBhdGgiOiIvNjUxMDA1NDAvMjM5NjgyOTU3LTEwODRiMjkyLWUyZGUtNDRlOC04NTY5LTQ5N2M3NDM2YjI1Zi5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBSVdOSllBWDRDU1ZFSDUzQSUyRjIwMjMwNTIwJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIzMDUyMFQxMDQ3MTlaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT02YWVjNDAwOGQ4ZDZmMGE5ZjE1Y2UyMDViNzliN2QxNWE1NmI2MzdmMTQ5MGZkZDBhZmE1NWY2YzcwMjkwYzA5JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCJ9.ByPGtCcR_8J1n3t_ti_9nc_mSlJ7QoAhEFBYNArZp3I)

  - 저 시점에서 `String`으로 래핑되어 있음

### 아이템11. 잉여 속성 체크의 한계 인지하기

```ts
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}

const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};
```

- 에러: `Room`타입에 존재하지 않는 `elephant`가 존재
  - 구조적 타이핑 관점에서 가능한 코드

```ts
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: "present",
};

const r: Room = obj;
```

- obj가 Room 타입의 부분 집합을 포함, Room에 할당 가능

- **`잉여 속성 체크`**
  - 객체 리터럴에만 적용
  - 약한 타입에도 동작
    - `값` 타입과, `선언`타입에 공통 속성이 있는지 확인하는 별도의 과정

### 아이템12. 함수 표현식에 타입 적용하기

```ts
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

### 아이템13. 타입과 인터페이스의 차이점 알기

<table>
    <thead>
        <tr>
            <th></th>
            <th>타입</th>
            <th>인터페이스</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>인덱스 시그니처</td>
            <td><center>✅</center></td>
            <td><center>✅</center></td>
        </tr>
        <tr>
            <td>함수 타입 정의</td>
            <td><center>✅</center></td>
            <td><center>✅</center></td>
        </tr>
        <tr>
            <td>제너릭</td>
            <td><center>✅</center></td>
            <td><center>✅</center></td>
        </tr>
        <tr>
            <td>타입 확장</td>
            <td><center>✅ ( & )</center></td>
            <td><center>✅ ( extends )</center></td>
        </tr>
        <tr>
            <td>클래스 구현 (implements)</td>
            <td><center>✅</center></td>
            <td><center>✅</center></td>
        </tr>
        <tr>
            <td>유니온</td>
            <td><center>✅</center></td>
            <td><center>❌</center></td>
        </tr>
        <tr>
            <td>튜플, 배열</td>
            <td><center>✅</center></td>
            <td><center>⚠️ (메서드 직접 구현해야 함)
            </center></td>
        </tr>
        <tr>
            <td>타입 보강</td>
            <td><center>❌</center></td>
            <td><center>✅</center></td>
        </tr>
    </tbody>
</table>

### 아이템14. 타입 연산과 제너릭 사용으로 반복 줄이기

```ts
interface State {
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

interface TopNavState {
  userId: State["userId"];
  pageTitle: State["pageTitle"];
  recentFiles: State["recentFiles"];
}
```

- 활용

  ```ts
  interface FlexProps {
  flexDirection: CSSProperties['flexDirection'];
  justifyContent: CSSProperties['justifyContent'];
  ...
  }
  ```

- **Pick**

  ```ts
  type Pick<T, K extends keyof T> = {
    [k in K]: T[k];
  };
  ```

  - 책 읽기 전에는 `T[k]`를 보고 **`프로퍼티에 접근했는데 어떻게 타입을 반환하지?`** 라는 의문
  - `아이템8`에서 설명한것 처럼 타입 맥락에서 사용됐기 때문에 **값이 아닌 타입**

    ```ts
    const first: Person["first"] = p["first"];
    ```

### 아이템15. 동적 데이터에 인덱스 시그니처 사용하기

1. **Record** 사용하기
   ```ts
   type Vec3D = Record<"x" | "y" | "z", number>;
   ```
2. 매핑된 타입 사용하기
   ```ts
   type Vec3D = { [k in "x" | "y" | "z"]: number };
   type ABC = { [k in "a" | "b" | "c"]: k extends "b" ? string : number };
   ```
