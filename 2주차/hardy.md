## item 9: 타입 단언보다 타입 선언 사용하기

타입 단언은 타입 체커가 오류를 무시합니다. 그래서 정말 타입에 확신이 있는 경우에만 사용해야 하지만 상황에 따라 타입 단언이 고민되는 상황들이 생길 수 있습니다. 

**Case 1:** 

```tsx
const fn = (v: string | number ): string | number => v
const result = fn(' str ') // Type: string | number 
result.trim() // error
```

위와 같이 리턴 타입이 유니온 타입일 경우 타입이 정해지지 않아 값에 대한 처리가 힘들어집니다. 이를 타입 단언으로 해결할 수 있습니다.

```tsx
const result = fn(' str ') as string // 타입 단언
result.trim()
```

**Case 2:**

```tsx
interface  List {
  title: string,
  data: Array<number> | Array<string>
}
const list: List = {
  title: '',
  data: [1,2,3]
}
const listData = list.data.map((v) => v).filter(v => v > 10) // v는 any
```

이 문제도 타입 단언으로 해결이 가능합니다.

```tsx
// listData: number[]
const listData = list.data.map(v => v as number).filter(v => v > 10)
```

애매한 상황들을 타입 단언을 통해 쉽게 해결할 수 있었습니다. 하지만 타입 단언은 올바른 해결 방법이 아닙니다. 정확하게 타입 단언을 사용했다면 문제가 되지 않겠지만 잘못된 타입 단언으로 인해 문제가 생길 수 있습니다.

```tsx
// case 1
const result = fn(' str ') as number
result.trim() // error

// case 2
// [] filter에서 올바른 평가가 되지 않음. 에러도 뜨지 않음 
const listData = list.data.map(v => v as string).filter(v => v > 10)
```

case 1의 경우엔 컴파일 에러가 표시되어 문제를 인지할 수 있겠지만 case 2의 경우에는 에러가 생기지 않고 의도하지 않은 결과가 만들어지는 최악의 상황이 되었습니다. 

타입 단언을 지양하면 좋겠지만 가끔 타입 단언이 필요한 경우가 있습니다. 이럴 때 정확한 타입 단언에 신경쓰기 위해 타입 단언의 위험성을 인지하고 있어야 합니다.

위의 케이스와 같은 대부분의 문제들은 타입 단언을 하지 않고 해결 가능합니다.

```tsx
// case 1
const fn = <T extends string | number >(v: T): T => v
const result = fn<string>(' str ')
result.trim()

// case 2
interface  List<T> {
  title: string,
  data: Array<T>
}
const list: List<number> = {
  title: '',
  data: [1,2,3]
}
const listData = list.data.map((v) => v).filter(v => v > 10)
```

## item 11: 잉여 속성 체크의 한계 인지하기

잉여 속성 체크는 spread operator도 체크를 하지 않습니다.

```tsx
const obj = {
  type: 0,
  title: "",
};

const tt: { title: string; description: string } = { ...obj, description: "" };
```

비슷하게 rest paramiter에서도 체크되지 않습니다.

```tsx
interface Props {
  title: string;
  description: string;
}
const Component = ({ title, description }: Props) => {
  return (
    <>
      {title}
      {description}
    </>
  );
};

const Page = () => {
  const props = { title: "", description: "", data: "", arr: ["1"] };
  return (
    <>
      <Component {...props} />
    </>
  );
};
```

이런 상황도 인지하고 있음 좋을 것 같습니다.

## item 13: 타입과 인터페이스의 차이점 알기

**Class**

저번 부터 클래스도 타입으로 이용 가능하다는 이야기가 계속 나오지만 뚜렷한 설명이 없어서 간단하게 정리해봤습니다.

```tsx
interface PersonData {
  name: string;
  age: number;
  gender: "man" | "woman";
  birth_date: Date;
}

class Person {
  name: string;
  age: number;
  gender: "man" | "woman";
  birthDate: string;
  adult: boolean;

  constructor(data: PersonData) {
    this.name = data.name;
    this.age = data.age;
    this.gender = data.gender;
		// class 내에서 값을 가공할 수 있습니다.
    this.birthDate = this.getBirthDateToString(data.birth_date);
    this.adult = this.checkAdult(data.age);
  }
	
	// 데이터에 관련된 로직을 class에 정의해 사용할 수 있습니다.
  get isMan() {
    return this.gender === "man";
  }

  private getBirthDateToString(date: Date) {
    // ...
    return "yyyy-mm-dd";
  }

  private checkAdult(age: number) {
    return age > 19;
  }
}

const data: PersonData = {
  name: "",
  age: 20,
  gender: "man",
};

// Type: Person
// 타입이자 값으로서 활용할 수 있습니다.
const person = new Person(data);

person.name
person.adult
person.isMan
```

## item 14: 타입 연산과 제네릭 사용으로 반복 줄이기

**extends와 & 연산자로 타입 중복 제거**

타입 중복도 코드 중복에서 생기는 문제들이 발생합니다. 수정하려면 중복된 모든 타입을 수정해야하고 놓치는 타입이 생길 수 있습니다. 

```tsx
// extends를 이용
interface Person {
	firstName: string;
	lastName: string;
}

interface PersonWithBirthDate extends Person {
	birth: Date;
}

// & 연산자 이용
type PersonWithBirthDate = Person & { birth: Date };
```

****************************************************인덱싱과 매핑 타입****************************************************

인덱싱과 매핑 타입을 이용해 필요한 타입을 골라와 중복을 제거할 수 있습니다.

```tsx
// 인덱싱
interface State {
	userId: string;
	pageTitle: string;
	recentFiles: string[];
	pageContents: string;
}

interface TopNavState {
	userId: State['userId'];
	pageTitle: State['pagetitle'];
}

// 매핑 타입
interface TopNavState {
	[k in 'userId' | 'pageTitle' | 'recentFiles']: State[k]
}
```

**유틸리티 타입 Pick**

인덱싱과 매핑 타입을 이용해 필요한 타입을 골라올 수도 있지만 유틸리티 타입 Pick을 이용해 더 편하게 중복을 제거할 수 있습니다.

```tsx
type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
```

- **그 외 다양한 유틸리티 타입**
    
    타입스크립트에서 제공하는 유틸리티 타입을 용도에 맞게 잘 활용한다면 편하게 타입을 관리할 수 있습니다.
    
    ```tsx
    interface Data {
      title: string;
      description: string;
      label?: string;
    }
    
    // property를 모두 옵셔널로 변경합니다.
    type Partial<T> = {
      [P in keyof T]?: T[P];
    };
    const partial: Partial<Data> = {};
    
    // property를 모두 필수로 변경합니다
    type Required<T> = {
      [P in keyof T]-?: T[P];
    };
    const required: Required<Data> = { title: "", description: "", label: "" };
    
    // property를 모두 readonly로 변경합니다.
    type Readonly<T> = {
      readonly [P in keyof T]: T[P];
    };
    const readonly: Readonly<Data> = {};
    
    // 원하는 property를 가져옵니다.
    type Pick<T, K extends keyof T> = {
      [P in K]: T[P];
    };
    const pick: Pick<Data, "title"> = { title: "" };
    
    // K 타입의 key, T 타입의 value의 집합을 생성합니다.
    type Record<K extends keyof any, T> = {
      [P in K]: T;
    };
    const record: Record<string, number> = { age: 20 };
    
    // T와 U 교집합만 제거합니다.
    type Exclude<T, U> = T extends U ? never : T;
    type ExcludeType = Exclude<"a" | "b" | "c", "a">; // a
    
    // T, U 교집합만 추출합니다.
    type Extract<T, U> = T extends U ? T : never;
    type ExtractType = Extract<"a" | "b" | "c", "a" | "d">; // a
    
    // 원하는 property를 제거합니다.
    type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
    const omit: Omit<Data, "title"> = { description: "" };
    
    // undefined, null을 제거합니다.
    type NonNullable<T> = T & {};
    type NonNullableType = NonNullable<string | number | undefined | null>; // string | number
    
    // 함수의 인자를 튜플 형식으로 리턴합니다.
    type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;
    const foo = (a: string, b: number) => {};
    type ParamType = Parameters<typeof foo>; // [string, number]
    
    // 클래스 생성자의 인자를 튜플 형식으로 리턴합니다.
    type ConstructorParameters<T extends abstract new (...args: any) => any> = T extends abstract new (
      ...args: infer P
    ) => any
      ? P
      : never;
    
    // 함수의 리턴 타입을 리턴합니다.
    type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
    const foo2 = (): string => "";
    type ReturnTypeEx = ReturnType<typeof foo2>; // string
    
    // T를 this의 타입으로 설정합니다.
    // interface ThisType<T> {}
    type Location = {
      x: number;
      y: number;
    };
    
    type ObjectDescriptor = {
      data: Location;
      moveBy: ((dx: number, dy: number) => void) & ThisType<Location>;
    };
    
    const obj: ObjectDescriptor = {
      data: { x: 0, y: 0 },
      moveBy(dx: number, dy: number) {
        this.data.x += dx;
        this.data.y += dy;
      },
    };
    
    obj.data.x = 10;
    obj.data.y = 20;
    obj.moveBy(5, 5);
    ```
    

**keyof와 typeof**

keyof는 타입스크립트 연산자로 타입의 key를 유니온으로 반환합니다.

```tsx
interface Options {
	width: number;
	height: number;
	color: string;
}

type OptionsKeys = keyof Options
// width | height | color

type OptionsUpdate = {[k in keyof Options]?: Options[k]}
```

typeof는 값의 형태에 해당하는 타입을 정의할 수 있습니다. 

```tsx
const INIT_OPTIONS = {
	width: 640;
	height: 480;
	color: '#FFFFFF';
}

type Options = typeof INIT_OPTIONS
// { width: number; height: number; color: string; }
```
