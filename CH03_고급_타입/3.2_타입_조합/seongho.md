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

<br />

### 6. 템플릿 리터럴 타입(Template Literal Types)
`템플릿 리터럴 타입`은 js의 템플릿 리터럴 문자열(``)을 사용해서 문자열 리터럴 타입을 선언할 수 있는 문법임. <br />
위에서 본 예시에 템플릿 리터럴 타입도 같이 포함되어져 있음. 조금 더 간단한 예시로 다시 봐보자
<img src="../../assets/CH03/template_literal.jpeg" />

이처럼 `템플릿 리터럴`을 사용해서 변수자리에 `문자열 리터럴 유니온 타입`을 넣으면, 각 유니온 타입의 멤버들이 차례대로 해당 변수로 들어가서 재가공 됨.

<br />

### 7. 제네릭(Generic)
제네릭은 보통 C나 Java와 같은 정적 언어에서 타입간의 재사용성을 높이기 위해서 사용하는 문법임. ts도 `정적 타입`을 가지는 언어이기때문에 `제네릭` 문법을 지원하고 있음.<br />

ts의 제네릭을 조금 더 풀어서 보면 `함수`, `타입`, `클래스` 등에서 내부적으로 사용할 타입을 미리 정해두지 않고, 타입 변수를 사용해서 해당 위치를 비워둔 뒤 실제로 해당 값을 사용할 때 외부에서 `타입 변수` 자리를 사용해 타입을 지정해서 사용하는 방식임.<br />
`타입 변수`는 주로 `<T>`와 같이 꺾쇠 괄호 내부에 정의하고, 매개변수를 넣는것과 유사하게 원하는 타입을 넣어주면 됨.<br />

```ts
type ArrayType<T> = T[];

const stringArray: ArrayType<string> = ['hello', 'world'];
```

> [!TIP]
> **제네릭의 컨벤션** <br />
> 보통 타입 변수명으로 `T(Type)`, `E(Element)`, `K(Key)`, `V(Value)`등 한글자로 된 이름을 많이 사용함.

> [!CAUTION]
> **`any`와 `제네릭`은 달라요** <br />
> 둘의 명확한 차이점은 배열을 예로 들어보면 편한데, `any`타입의 배열에는 배열 요소들이 전부 같지 않을 수 있음 <br />
> 하지만 제네릭은 `any`처럼 아무 타입이나 무분멸하게 받는게 아니라, 배열의 생성 시점에 원하는 타입으로 특정하는 것임.
> ```ts
> type AnyArray = any[];
> type ArrayType<T> = T[];
>
> const list1: AnyArray = [1, 'hello', true, () => {}]; // 아무거나 가능
> const list2: ArrayType<number> = [1, 2, 3, 4];
> const list3: ArrayType<string> = [1, 'hello', true, () => {}]; // > Error!
> ```

<br />

제네릭 함수를 호출 할 때 ***반드시*** 꺽쇠괄호 안에 타입을 명시해줘야 되는 것은 아님. 타입을 명시하는 부분을 생략하면 컴파일러가 `인자`를 보고 타입을 추론 해줌.
<img src="../../assets/CH03/generic_type_inference.jpeg" />
-> 인자에 string타입의 값이 들어와서 컴파일러가 제네릭 함수에 들어온 인자값을 보고 타입을 스스로 추론했음.

만약 특정 요소의 타입을 알 수 없는 경우에는 `타입 기본값`도 할당해 줄 수 있음.
<br />
<br />
<img src="../../assets/CH03/generic_default.png" width='400px' /> <br />

***제네릭을 사용할때 항상 유의헤야 하는 점은 제네릭에는 어떤 타입이든 올 수 있다는 점임.*** <br />
이걸 왜 유의해야 하냐? 이는 곧, <u>**특정 타입에만 존재하는 프로퍼티나 메서드를 참조하려고 하면 안된다는 뜻**</u>임!! <br />
예를 들자면
```ts
function genericFunc<T>(params: T): number {
  return params.length; // Error : Property 'length' does not exist on type 'T'.
}
```
이런거임.<br />

`length`는 배열에만 존재하는 프로퍼티고, T에는 어떤 타입이 올지 모르기 때문. 이 문제는 기본 타입을 할당해줘도 해결이 안됨.
```ts
function genericFunc<T = number[]>(params: T): number {
  return params.length; // Error : Property 'length' does not exist on type 'T'.
}
```
<br />

만약, 어떤 타입이 올지 모르지만 `length` 프로퍼티는 반드시 갖는 프로퍼티만 오게 하고 싶다면, 제약을 걸어서 만족시켜줄 수 있음.
```ts
interface HasLengthProperty {
  length: number;
}

function genericFunc<T extends HasLengthProperty>(params: T): number {
  return params.length;
}

function foo(a: number, b: string, ...params: unknown[]) {
  genericFunc(arguments); // arguments는 유사 배열이므로 length 프로퍼티를 가지기 때문에 가능.
  genericFunc(params); // 배열도 가능. 배열도 js에서는 객체니까
  genericFunc({
    length: 1,
    name: 'seongho'
  }); // length프로퍼티를 갖고 있으니까 이것도 가능
}
```

> [!CAUTION]
> **tsx파일에서 화살표 함수에 제네릭을 사용하면 에러가 발생함** <br />
> 제네릭의 꺾쇠 괄호와 태그의 꺾쇠괄호를 구분할 수 없기 때문임. <br />
> 아래 이미지를 참고하자. <br />
> <br />
> <img src="../../assets/CH03/arrow_function_generic.png">