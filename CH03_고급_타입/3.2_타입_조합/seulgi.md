# 타입 조합

타입 검사 심화 버전을 알아보자!

<br />
<br />

## 1) 교차 타입(intersection)

교차 타입을 사용하면 여러 가지 타입을 결합해 하나의 단일 타입으로 만들 수 있다.<br />
기존에 존재하는 다른 타입을 합쳐 모든 멤버를 가지는 새로운 타입을 생성하는 것이다.

- 교차 타입은 &을 사용해서 표기한다.
- 결과물로 탄생한 단일 타입에는 타입 별칭을 붙일 수 있다.
- 여러 개의 타입을 교차시킬 수도 있다.

```ts
type Me = {
  name: string;
  age: number;
  location: string;
  isWork: boolean;
};

type MyInfo = Me & { address: string };
```

> Me 타입의 변수를 선언하고 값을 할당하면 Me타입의 모든 멤버와 address까지 가지게 된다.

<br />
<br />

## 2) 유니온 타입(union)

유니온 타입은 타입 A 또는 타입 B 중 하나가 될 수 있는 타입이며 A | B 같이 표기한다.
주로 특정 변수가 가질 수 있는 타입을 전부 나열하는 용도로 사용된다.

교차 타입과 마찬가지로 2개 이상의 타입을 이어 붙일 수 있고,
타입 별칭을 통해 중복을 줄일 수 있다.

```ts
type Me = {
  name: string;
  age: number;
  location: string;
  isWork: boolean;
};

type You = {
  name: string;
  age: number;
  location: string;
  areMarried: boolean;
};

type Info = Me | You;

const printInfo = (info: Info) => {
  console.log(info.location);
  console.log(info.areMarried); // Error!
};
```

> 해당 함수 내부에서 areMarried 속성을 참조하려고 하면 에러가 발생하는데, 이는 areMarried가 한 속성에만 존재하기 때문이다.

### 맨 앞에 & 혹은 |를 붙이는 경우를 알아보자!

교차 타입과 유니온 타입은 여러 줄에 걸쳐 표기할 수 있는데,<br />
이럴 때 각 줄의 맨 앞에 & 혹은 |를 붙여서 표기하면 된다.

<br />
<br />

## 3) 인덱스 시그니처(Index Signatures)

인덱스 시그니처는 특정 타입의 속성 이름을 알 수 없지만 속성값의 타입을 알고 있을 때 사용하는 문법이다.

인터페이스 내부에 `[Key: K]: T` 꼴로 타입을 명시한다. 이는 해당 타입의 속성 키는 모두 K타입이어야 하고 속성값은 모두 T타입을 가져야한다는 뜻이다.

```ts
interface IndexSignature1 {
  [key: string]: number;
}

interface IndexSignature2 {
  [key: string]: number;
  length: number;
  isValid: boolean; // Error!
  name: string; // Error!
}
```

> 인덱스 시그니처를 선언할 때 다른 속성을 추가로 명시해줄 수 있는데 이때 추가로 명시된 속성은 인덱스 시그니처에 포함되는 타입이어야 한다.

<br />
<br />

## 4) 인덱스드 엑세스 타입(Indexed Access Types)

인덱스드 엑세스 타입은 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용된다.<br />
인덱스에 사용되는 타입 또한 그 자체로 타입이기 때문에 유니온 타입, keyof, 타입 별칭 등의 표현을 사용할 수 있다.

```ts
type Example = {
  a: number;
  b: string;
  c: boolean;
};

type IndexedAccess = Example["a"]; // 인덱스드 엑세스 타입
type IndexedAccess1 = Example[keyof Example]; // number | string | boolean
```

배열의 요소 타입을 조회하기 위해 인덱스드 엑세스 타입을 사용하는 경우가 있다.

1. number로 인덱싱해 배열 요소를 얻는다.
2. typeof 연산자를 붙여준다.
3. 해당 배열 요소의 타입을 가져올 수 있다.

```ts
const People = [
  { name: "a", age: 1 },
  { name: "b", age: 2 },
  { name: "c", age: 3 },
];

type PeopleOf<T> = (typeof People)[number];

// type People = { name: string; age: number }
type PeopleType = PeopleOf<People>;
```

> 해당 예시는 현재 에러가 난다. (확인 필요)

<br />
<br />

## 5) 맵드 타입(Mapped Types)

자바스크립트의 map은 배열 A를 기반으로 한 타입을 선언할 때 사용하는 문법인데, 인덱스 시그니처 문법을 사용해서 반복적인 타입 선언을 효과적으로 줄일 수 있다.

```ts
type Example = {
  a: number;
  b: string;
  c: boolean;
};

type Subset<T> = {
  [K in keyof T]?: T[K];
};

const aExample: Subset<Example> = { a: 3 };
const bExample: Subset<Example> = { a: 4, c: true };
```

- 맵드 타입에서 매핑할 때는 readonly와 ?를 수식어로 적용할 수 있다.
- 맵드 타입의 특이한 점은 수식어 제거도 가능하다.

```ts
type ExampleType = {
  readonly a: number;
  readonly b: string;
};

type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};
type ResultType = CreateMutable<ExampleType>; // { a: number; b: string }
```

### BottomSheet같은 예시로 알아보자.

바텀 시트마다 각각의 resolver, isOpen 등의 상태를 관리하는 스토어가 필요한데 이 스토어의 타입을 선언해줘야 한다.

BottomSheetMap에 존재하는 모든 키에 대해 일일이 스토어를 만들어 줄 수 있지만 불필요한 반복이 발생하게 된다.

> 이럴 때는 인덱스 시그니처 문법을 사용해서 BottomSheetMap을 기반으로 각 키에 해당하는 스토어를 선언해 불필요한 반복을 줄여준다.

```ts
const BottomSheetMap = {
  RECENT_CONTACTS: RecentContactsBottomSheet,
  CARD_SELECT: CardSelectBottomSheet,
  SORT_FILTER: SortFilterBottomSheet,
  PRODUCT_SELECT: ProductSelectBottomSheet,
  REPLY_CARD_SELECT: ReplyCardSelectBottomSheet,
  RESEND: ResendBottomSheet,
  STICKER: StickerBottomSheet,
  BASE: null,
};

export type BOTTOM_SHEET_ID = keyof typeof BottomSheetMap;

// 불필요한 반복이 발생한다
type BottomSheetStore = {
  RECENT_CONTACTS: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
  CARD_SELECT: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
  SORT_FILTER: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
  // ...
};

// Mapped Types를 통해 효율적으로 타입을 선언할 수 있다
type BottomSheetStore = {
  [index in BOTTOM_SHEET_ID]: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
};
```

### 맵드 + as 키워드로 키를 재지정 가능하다!

```ts
type BottomSheetStore = {
  [index in BOTTOM_SHEET_ID as `${index}_BOTTOM_SHEET`]: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
};
```

> 위의 예시처럼 키 이름을 그대로 쓰거나 모든 키에 특정 문자열을 붙이는 것처럼 새로운 키를 지정할 수 있다.

<br />
<br />

## 6) 템플릿 리터럴 타입(Template Literal Types)

자바스크립트의 템플릿 리터럴 문자열을 사용하여 문자열 리터럴 타입을 선언할 수 있는 문법이다.

```ts
type Me = "chu" | "chuchu" | "seulgi" | "seulgi chu" | "chuseulgi";

type MyName = `${Me}-name`;
```

- Me 타입의 모든 유니온 멤버 뒤에 -name을 붙여 새로운 유니온 타입을 만들었다.
- 템플릿 리터럴을 사용하여 `${Me}-name`와 같이 변수 자리에 문자열 리터럴 유니온 타입인 Me를 넣으면 유니온 타입 멤버들이 차례로 해당 변수로 들어간다.
- 해당 변수로 들어가 -name이 붙은 문자열 리터럴의 유니온 타입을 결과로 반환한다.

<br />
<br />

## 7) 제네릭(Generic)

제네릭은 C, 자바 같은 정적 언어에서 다양한 타입 간에 재사용성을 높이기 위해 사용하는 문법이다.

### 제네릭의 사전적 의미를 알아보자!

제네릭의 사전적 의미는 **특징이 없거나 일반적인 것**인데 타입스크립트의 제네릭도 이와 비슷한 맥락으로 <u>일반화된 데이터 타입</u>이라고 할 수 있다.

- 함수, 타입, 클래스 등에서 내부적으로 사용할 타입을 정해두지 않고 타입 변수를 사용한다.
- 변수로 해당 위치를 비워 둔 다음에, 실제로 그 값을 사용할 때 외부에서 타입 변수 자리에 타입을 지정해 사용하는 방식이다.

> 이렇게 하면 함수, 타입, 클래스 등 여러 타입에 대해 하나하나 따로 정의하지 않아도 되기 때문에 재사용성이 크게 향상된다.

### 제네릭 사용 방법을 알아보자!

- `<T>`와 같이 꺾쇠괄호 내부에 정의된다.
- 사용할 때는 함수에 매개변수를 넣는 것과 유사하게 원하는 타입을 넣어주면 된다.
- 타입 변수명으로 T(Type), E(Element), K(Key), V(Value) 등 한 글자로 된 이름을 많이 사용한다.

```ts
type Example<T> = T[];

const arr: Example<string> = ["1", "2"];
```

### any와는 어떤 점이 다를까?

any 타입의 배열에서는 배열 요소들의 타입이 전부 같지 않을 수 있다. 쉽게 말해 타입 정보를 잃어버린 것과 같다.

반면 제네릭은 any처럼 아무 타입을 무분별하게 받는 게 아니라 배열 생성 시점에 원하는 타입으로 특정할 수 있는 것이다.

> 제네릭을 사용하면 배열 요소가 전부 동일한 타입으로 보장될 수 있다.

### 다른 특징은 뭐가 있을까?

- 제네릭 함수를 호출할 때 반드시 꺾쇠괄호(<>) 안에 타입을 명시해야 하는 것은 아니다.
- 타입을 명시하는 부분을 생략하면 컴파일러가 인수를 보고 타입을 추론한다.
- 타입 추론이 가능한 경우는 타입 명시를 생략할 수 있다.

```ts
function example<T>(arg: T): T[] {
  return new Array(2).fill(arg);
}

example("hi"); // T는 string으로 추론
```

- 특정 요소 타입을 알 수 없을 때는 제네릭 타입에 기본값을 추가할 수 있다.

```ts
interface SubmitEvent<T = HTMLElement> extends SyntheticEvent<T> {
  submitter: T;
}
```

### 특정 타입에서만 존재하는 멤버를 참조하려 하면 안된다!

```ts
function example<T>(arg: T): number {
  return arg.length; // Error!
}
```

배열에만 존재하는 length 속성을 제네릭에서 참조하려고 하면 당연히 에러가 난다.

컴파일러는 어떤 타입이 제네릭에 전달될지 알 수 없기 때문에 모든 타입이 length 속성을 사용할 수 없다 알려준다.

```ts
interface Example {
  length: number;
}

function example<T extends Example>(arg: T): number {
  return arg.length;
}
```

> 이럴 때는 제네릭 꺾쇠괄호 내부에 "length 속성을 가진 타입만 받는다"라는 제약을 걸어준다.

### 제네릭을 사용할 때 주의해야 할 점이 있다?

파일 확장자가 tsx일 때 화살표 함수에 제네릭을 사용하면 에러가 발생한다.

tsx는 타입스크립트 + JSX이므로 제네릭의 꺾쇠괄호와 태그의 꺾쇠괄호를 혼동하여 문제가 생긴다.

### 이런 상황을 어떻게 피할 수 있을까?

제네릭 부분에 extends 키워드를 사용해 컴파일러에게 특정 타입의 하위 타입만 올 수 있음을 확실히 알려주면 된다.

```ts
// 에러 발생 : JSX element "T" has no corresponding closing tag.
const exampleFunc = <T>(arg: T): T[] => {
  return new Array(3).fill(arg);
};

// 에러 발생 X
const exampleFunc2 = <T extends {}>(arg: T): T[] => {
  return new Array(3).fill(arg);
};
```

보통 제네릭을 사용할 때는 function 키워드로 선언하는 경우가 많다.
