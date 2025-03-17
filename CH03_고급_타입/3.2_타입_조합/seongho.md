## 타입 조합
앞에서 다룬 개념을 응용하거나, 내용을 덧붙여서 좀 더 심화한 타입 검사를 수행해보자

<br />

### 1. 교차 타입(Intersection)
`교차 타입`을 사용하면 여러가지 타입을 결합하여 하나의 단일 타입으로 만들 수 있음.<br />
교차 타입은 `&`을 사용해서 표기함.
```ts
interface Person {
  name: string;
  age: number;
};

type PersonWithLocation = Person & { location: string };
```

<br />

### 2. 유니온 타입(Union)
교차 타입의 A & B가 타입 A와 타입 B를 모두 만족하는 경우라면, `유니온 타입`은 타입 A 또는 타입 B중 하나가 될 수 있는 타입을 말함.
```ts
interface Person {
  name: string;
  age: number;
};

interface Duck {
  name: string;
  leg: boolean;
}

type C = Person | Duck;


function foo(bar: C) {
  bar.name;
  bar.leg; // Error : Property 'leg' does not exist on type 'Person'.
}
```
해당 예시를 보면 bar는 `Person`과 `Duck`의 유니온 타입인 `C`를 bar의 타입으로 선언해주었음. <br />
근데, bar는 `Person`일지, `Duck`일지 모르는 상황이기 때문에 두 타입이 공통으로 갖는 name에는 접근이 가능하지만 `Duck`만 갖고있는 `leg` 프로퍼티에는 접근할 수 없음.

<br />

### 3. 인덱스 시그니처(Index Signatures)
`인덱스 시그니처`는 **_특정 타입의 속성 이름은 알 수 없지만, 속성과 속성값의 타입을 알고있을 때_** 사용하는 문법. <br />
인터페이스 내부에 `[key: T]: K` 꼴로 타입을 명시해줌. <br/>
이는 key는 `T`타입이여야 하고, value는 `K`타입이어야 함을 의미함.
```ts
interface IndexSignature {
  [key: '']: number;
}

let obj: IndexSignature;

obj = { name: 1 };
obj = { name: true }; // Error : Type 'boolean' is not assignable to type 'number'.
```
당연하게 `유니온 타입`도 가능함.
```ts
interface IndexSignature {
  [key: string]: number | string;
}

let obj: IndexSignature;

obj = { name: 'seongho', age: 27 }; // ✅

interface IndexSignature2 {
  [key: string | number]: number;
}

let obj2: IndexSignature2;

obj2 = { 1 : 1 }; // ✅

```

> [!CAUTION]
> 인덱스 시그니처의 key의 타입은 `string`, `number`, `symbol`, `템플릿 리터럴`타입만 가능함. <br />
> ```ts
> interface IndexSignature {
>  [key: boolean]: number | string; // Error : An index signature parameter type must be 'string', 'number', 'symbol', or a template literal type.
> }
>
> let obj: IndexSignature;
>
> obj = { name: 'seongho', age: 27 };
> ```

<br />

### 4. 인덱스 엑세스 타입(Index Access Types)
`인덱스 엑세스 타입`은 말 그대로 인덱스로 접근해서 타입을 가져오는 것임. <br />
인덱스에 사용되는 타입도 그 자체로 타입이라서 `유니온`이나 `keyof`, `타입 별칭(type키워드로 선언한 타입)`등 모두 가능함.
```ts
interface IndexAccess {
  a: number;
  b: string;
  c: boolean;
}

type A = IndexAccess['a']; // number;
type B = IndexAccess['a' | 'b']; // number | string;
type C = IndexAccess[keyof IndexAccess]; // number | string | boolean;

type typeAlias = 'b' | 'c'; // 타입 별칭

type D = IndexAccess[typeAlias]; // string | boolean;
```

인덱스 엑세스 타입은 **배열의 요소 타입**을 조회하기 위해 사용되기도 함. <br />
```ts
const arr = [1, 'name', true];
type StringList = string[];

type ItemType = typeof arr[number]; // number | string | boolean
type ItemType2 = StringList[number]; // number

// ======================

const me = [
  { name: 'seongho', age: 27 },
];

type InformationOf<T extends Array<unknown>> = T[number];

const seongho100: InformationOf<typeof me> = {
  name: '오성오',
  age: 100,
}
```

<br />

### 5. 맵드 타입(Mapped Types)
js에서의 map이라고 하면, 배열 A를 기반으로 새로운 배열 B를 만들어 내는 배열 메서드임.<br />
이처럼 ts에서의 `맵드 타입`도 ***다른 타입을 기반으로 한 타입을 선언할 때 사용하는 문법*** 임. 이때 `인덱스 시그니처` 문법을 사용하면 더욱 효과적으로 사용할 수 있음.
```ts
type Example = {
  a: string;
  b: number;
  c: boolean;
}

type Subset<T> = {
  [K in keyof T]?: T[K];
}

const list = [
  { name: 'seongho', age: 27 },
]

const withLoc = { name: 'seongho', age: 27, location: 'gp' };

const a: Subset<Example> = { a: 'string' };
const b: Subset<Example> = { b: 2 };
const c: Subset<typeof list[number]> = withLoc;
```
<br />

특이한 점은 `맵드 타입`에서 타입을 매핑할때는 `readonly`나 `?`를 수식어로 적용할 수 있다는 점인데, 추가하는 것은 물론이고 빼는것도 가능함.
```ts
type Example = {
  a: string;
  b: number;
  c: boolean;
}

type Subset<T> = {
  [K in keyof T]?: T[K];
}

// Subset에서 추가한 optional을 제거
type Require<T> = {
  [K in keyof T]-?: T[K];
}

const a: Subset<Example> = { a: 'string' };
let b: Require<Subset<Example>>;

b = { b: 2 }; // Error!
b = { a: 'a', b: 1, c: true }; // ✅
```

<br />

덧붙여서 `as`키워드를 통해서 key의 이름을 재지정 할 수도 있음.
```ts
type Example = {
  a: string;
  b: number;
  c: boolean;
}

type Keys = keyof Example;

type WithOptionalSeonghoInKey = {
  [K in Keys as `${K}_Seongho`]?: {
    name: string;
    age: number;
  }
}

const obj: WithOptionalSeonghoInKey = {
  a_Seongho: {
    name: '에이성호',
    age: 27,
  },
  b_Seongho: {
    name: '비성호',
    age: 25,
  }
}
```