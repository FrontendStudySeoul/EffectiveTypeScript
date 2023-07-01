## 아이템 48. API 주석에 TSDoc 사용하기

타입스크립트 언어에서 JSDoc 형태를 지원합니다.
<br>
아래는 제가 사용하던 방식입니다.
```tsx
// 승모 조회
// GET /seungmo/something?cantunderstand=

export async function getSeungmoInfo(
  cantunderstand: number
): Promise<SeungmoResponse> {
  try {
    const apiUri = `/seungmo/something?cantunderstand=${cantunderstand}`;
    const response = await axiosClient.get(apiUri);

    return response.data;
  } catch (err: any) {
    throw err;
  }
}
```

<br>
그럼 우리의 리액트는 어떻게 사용하고 있을까요 ?
<img width="761" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/de9318d2-eedc-4712-8420-d4f39c1a4e90">

<br>
**그럼 위의 api 함수를 리팩토링 해봅시다.**
<br>

```tsx
/**
 * 승모의 데이터를 조회합니다.
 * @param cantunderstand 승모의 입사 첫날 데이터
 * @returns GET /seungmo/something?cantunderstand=
 * @see www.yourServerUrl.com
 */

export async function getSeungmoInfo(
  cantunderstand: Date
): Promise<SeungmoResponse> {
  try {
    const apiUri = `/seungmo/something?cantunderstand=${cantunderstand}`
    const response = await axiosClient.get(apiUri)

    return response.data
  } catch (err: any) {
    throw err
  }
}
```

<img width="572" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/3c0abf94-2e00-4af2-a02f-3f015f6b5e0a">

<br>
<br>
<br>
**인터페이스에서도 사용 가능합니다.**

```tsx
 /** 승모의 데이터를 가진 인터페이스
   * @param height 승모의 키
   * @param secret 승모의 타스 스터디 첫 시작일
   * @param hamburgerEatTime 승모의 햄버거 하나 먹는 시간
   */
  interface SeungMo {
    /** 승모의 키(cm) */
    height: number
    /** 승모의 타스 스터디 첫 시작일 */
    secret: Date
    /** 승모의 햄버거 하나 먹는 시간(ms) */
    hamburgerEatTime: number
  }
```
<br>
특히 시간은 ms인지 s인지 잘 알수 없는 경우가 많기때문에 주석으로 표현해주면 좋을 것 같습니다. 그리고 JSDOC과는 다르게 해당 param에 대한 타입은 적어두지 않습니다. 왜냐하면 해당 값은 타입스크립트를 통해 알 수 있기 때문입니다.


<img width="456" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/95a9d1cc-5052-46e4-858e-543cdbaba8f1">
<br>
<img width="375" alt="image" src="https://github.com/FrontendStudySeoul/TypeScript/assets/103626175/9452a924-67f4-4f5d-99e1-45477ee9782a">


### 왜 위처럼 표시해야 될까요 ?
일반적으로 회사에서 사용하는 API들은 정말 많습니다. 저희 회사를 보더라도 안쓰는거 쓰는거 다 합치면 100개는 넘는것 같습니다. 근데 이 모든 API들을 불러오는 함수 이름만 보고 알 수 있을까요 ? 위의 getSeungmoInfo에 들어갈 cantunderstand라는 인수도 처음에 만든 개발자는 understand인자 였을 확률이 높습니다. 하지만 시간이 지남에 따라 혹은 저 api를 보는 첫 사용자는 cantunderstand일 확률이 높습니다.
<br>후에 API 호출 코드를 쓸 신규 입사자나 1년뒤의 나를 위해서라도 아래와 같은 주석은 코드의 가독석을 올리는데 도움을 줄거라 생각합니다.



