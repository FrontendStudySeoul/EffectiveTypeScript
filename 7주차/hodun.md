## 아이템 45. devDependencies에 typescript와 @types 추가하기

### npm의 의존성
1. dependencies
   - 현재 프로젝트를 실행하는데 **`필수`** 적인 라이브러리
   - 다른 사용자가 해당 프로젝트 설치 시 **함께 설치**
2. devDependencies
   - 개발, 테스트에 사용되지만 **`런타임엔 필요없는`** 라이브러리
   - 다른 사용자가 해당 프로젝트 설치 시 **제외**
3. peerDependencies
   - 런타임에 필요하지만 의존성을 **`직접 관리하지 않는`** 라이브러리
   - 다른 사용자가 해당 프로젝트 설치 시 **직접 버전 선택**

<br />

- 일반적으로 `dependencies`, `devDependencies` 사용

### 타입스크립트 프로젝트에서 관리해야하는 의존성
> 타입스크립트는 런타임에 존재하지 않기 때문에 일반적으로 devDependencies에 존재
1. 타입스크립트 자체 의존성 고려
   - 팀원 모두 동일한 버전을 설치한다는 보장이 없음
   - 시스템 레벨로 설치하는것이 아니라 devDependencies에 넣는것이 좋음
   - 항상 정확한 버전의 타입스크립트 설치 가능
2. 타입 의존성(`@types`) 고려
   - 원본 라이브러리 자체가 dependencies에 있더라도 @types는 devDependencies에 존재
