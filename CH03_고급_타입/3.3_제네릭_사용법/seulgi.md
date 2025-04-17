# 제네릭 사용법

## 1) 함수의 제네릭

함수의 매개변수나 반환 값에 다양한 타입을 넣고 싶을 때 제네릭을 쓸 수 있다.

```ts
function ReadOnlyRepository<T>(
  target: ObjectType<T> | EntitySchema<T> | string
): Repository<T> {
  return getConnection("ro").getRepository(target);
}
```

- T 자리에 넣는 타입에 따라 ReadOnlyRepository가 적절하게 사용될 수 있다.
- `<T>`는 현재 "이 함수가 다룰 데이터 모델의 타입"을 의미한다.
- T가 무엇이 될 지는 함수가 호출될 때 결정된다.

### 위의 코드에서 매개변수를 더 알아보자!

```ts
target: ObjectType<T> | EntitySchema<T> | string;
```

- 매개변수는 세 가지 타입 중 하나를 받을 수 있다.

1. ObjectType<T> → 일반적으로 클래스 타입 (User, Post 같은 엔티티 클래스)
2. EntitySchema<T> → 스키마 기반 엔티티 정의
3. string → 엔티티 이름을 문자열로 전달

### 제네릭이 없다면 어떻게 사용해야 했을까?

```ts
function ReadOnlyUserRepository(target: ObjectType<User>): Repository<User> {
  return getConnection("ro").getRepository(target);
}

function ReadOnlyPostRepository(target: ObjectType<Post>): Repository<Post> {
  return getConnection("ro").getRepository(target);
}
```

- User, Post, Comment 등 엔티티마다 새로운 함수를 만들어야 한다.
- 타입이 추가될 때마다 함수를 계속 만들어야 해서 유지보수가 어렵다.

```ts
const userRepo = ReadOnlyRepository<User>(User);
const postRepo = ReadOnlyRepository<Post>(Post);
```

> 위의 코드처럼 제네릭을 통해 유연하게 사용할 수 있다.

<br />
<br />

## 2) 호출 시그니처의 제네릭

호출 시그니처는 <u>타입스크립트 함수 타입 문법으로 함수의 매개변수와 반환 타입을 미리 선언하는 것</u>을 말한다.

호출 시그니처를 사용할 때 제네릭 타입을 **어디에 위치**시키는지에 따라<br />
타입의 범위와 제네릭 타입을 **언제 구체 타입으로 한정할지**를 결정할 수 있다.

```ts
interface useSelectPaginationProps<T> {
  categoryAtom: RecoilState<number>;
  filterAtom: RecoilState<SortType>;
  fetcherFunc: (
    props: CommonListRequest
  ) => Promise<DefaultResponse<ContentListResponse<T>>>;
}
```

- 위의 코드에서 `<T>`를 useSelectPaginationProps의 타입 별칭으로 한정했다.
- `<T>`는 useSelectPaginationProps를 사용할 때 타입을 명시해 제네릭 타입으로 구체 타입을 한정한다.
- 위의 훅을 사용할 때 반환값도 인자에서 쓰는 제네릭 타입과 연관이 있기 때문에 위같이 작성한 케이스다.

```ts
function useSelectPagination<
  T extends CardListContent | CommonProductResponse
>({
  categoryAtom,
  filterAtom,
  fetcherFunc,
}: useSelectPaginationProps<T>): {
  intersectionRef: RefObject<HTMLDivElement>;
  data: T[];
  //... 사용할 타입 정의
} {
  return {
    intersectionRef,
    data: swappedData ?? [],
    //...
  };
}
```

<br />
<br />

## 3) 제네릭 클래스

외부에서 입력된 타입을 클래스 내부에 적용할 수 있는 클래스이다.

```ts
class Box<T> {
  private content: T;

  constructor(content: T) {
    this.content = content;
  }

  getContent(): T {
    return this.content;
  }
}

const stringBox = new Box<string>("Hello");
console.log(stringBox.getContent()); // "Hello"

const numberBox = new Box<number>(1);
console.log(numberBox.getContent()); // 1
```

- 클래스 이름 뒤에 타입 매개변수인 `<T>`를 선언해준다.
- `<T>`는 메서드의 매개변수나 반환 타입으로 사용할 수 있다.
- 제네릭 클래스를 사용하면 클래스 전체에 걸쳐 타입 매개변수가 적용된다.

### 특정 메서드만을 대상으로 제네릭을 적용하려면?

해당 메서드를 제네릭 메서드로 선언하면 된다.

```ts
class A {
  get<T>(value: T): T {
    return value;
  }
}
```

<br />
<br />

## 4) 제한된 제네릭

제한된 제네릭은 타입 매개변수에 대한 제약 조건을 설정하는 기능이다.

### string 타입으로 제약하는 방법을 알아보자

```ts
type AllowedKeys = "name" | "age";

type MyType<T extends string> = Exclude<T, AllowedKeys> extends never
  ? Record<T, string>
  : never;
```

1. AllowedKeys = "name" | "age" → 허용할 키를 지정한다.
2. T extends string → T는 string 타입이어야 한다.
3. Exclude<T, AllowedKeys> → T에서 "name"과 "age"를 제외한다.
4. 결과가 never이면 (T가 전부 "name" | "age"에 속하면`)
   • { name: string, age: string } 같은 객체 타입을 반환한다.
5. 그렇지 않으면 (T에 "name" | "age"가 아닌 값이 포함되면`)
   • never 반환한다.

### Exclude?

Exclude<T, U> 는 제네릭 유틸리티 타입으로 T에서 U를 제거하는 타입이다.

> 타입 매개변수가 특정 타입으로 묶였을 때(bind) 키를 바운드 타입 매개변수(bounded type parameters)라고 부른다. 위의 코드에서는 string을 키의 상한 한계(upper bound)라고 한다.

### 상속 받을 수 있는 타입을 알아보자!

상속받을 수 있는 타입으로는 기본 타입뿐만 아니라 상황에 따라 인터페이스나 클래스도 사용할 수 있고 또한 유니온 타입을 상속해서 선언할 수 있다.

#### (1) 인터페이스를 상속받는 경우

```ts
interface Person {
  name: string;
}

function greet<T extends Person>(person: T): string {
  return `Hello, ${person.name}`;
}

const user = { name: "Chu", age: 32 };

console.log(greet(user)); // "Hello, Chu!"
console.log(greet({ age: 32 })); // Error! Object literal may only specify known properties, and 'age' does not exist in type 'Person'.
```

- T extends Person : T는 Person의 구조를 가져야 한다.
- name 속성이 없을 경우 컴파일 오류가 발생한다.

#### (2) 클래스를 상속받는 경우

```ts
class Chu {
  getName(): string {
    return "seulgi chu";
  }
}

class Person extends Chu {
  getName(): string {
    return "name";
  }
}

function getNames<T extends Person>(person: T): string {
  return person.getName();
}

const person = new Person();
console.log(getNames(person));
console.log(getNames({})); // Error! Argument of type '{}' is not assignable to parameter of type 'Person'. Property 'getName' is missing in type '{}' but required in type 'Person'.
```

- T extends Person : T는 Person을 상속받은 클래스여야 한다.

<br />
<br />

## 5) 확장된 제네릭

제네릭 타입은 여러 타입을 상속받을 수 있으며 타입 매개변수를 여러 개 둘 수 있다.

```ts
<Key extends string>
```

> 위의 코드처럼 제약해버리면 제네릭의 유연성을 잃어버린다.

### 유연성을 잃어버리지 않고, 제약해야 할 때는 타입 매개변수 + 유니온 타입 상속 선언을 활용하자

```ts
<Key extends string | number>
```

- 유니온 타입으로 T가 여러 타입을 받게 할 수는 있다.
- 다만 타입 매개변수가 여러 개일 때는 처리할 수 없을 땐 매개변수를 하나 더 추가해 선언한다.

```ts
type Code = 200 | 400 | 500;

class MyResponse<Ok, Err = string> {
  private readonly data: Ok | Err | null;
  private readonly statusCode: Code | null;

  constructor(data: Ok | Err | null, statusCode: Code | null) {
    this.data = data;
    this.statusCode = statusCode;
  }

  getData(): Ok | Err | null {
    return this.data;
  }

  getStatusCode(): Code | null {
    return this.statusCode;
  }
}

const success = new MyResponse<string>("success", 200);
const server_err = new MyResponse<string>("server err", 500);
const redirect = new MyResponse<null, string>("redirect", 300); // Error! Argument of type '300' is not assignable to parameter of type 'Code | null'.
```

> Code 타입 외의 값(300)을 넣으면 컴파일 오류가 발생한다.

<br />
<br />

## 6) 제네릭 예시

제네릭을 가장 많이 활용할 때는 API 응답 값의 타입을 지정할 때이고, 제네릭을 활용해 적절한 타입 추론과 코드 재사용성을 높이고 있다.

```ts
interface ApiResponse<Data> {
  data: Data;
  statusCode: string;
  statusMessage?: string;
}

const fetchPriceInfo = (): Promise<ApiResponse<PriceInfo>> => {
  const priceUrl = "https://";

  return request({
    method: "GET",
    url: priceUrl,
  });
};

const fetchOrderInfo = (): Promise<ApiResponse<OrderInfo>> => {
  const orderUrl = "https://";

  return request({
    method: "GET",
    url: orderUrl,
  });
};
```

> API 응답 값에 따라 달라지는 data를 제네릭 타입 Data로 선언하고 있다.

<br />
<br />

## 7) 제네릭이 굳이 필요 없는 경우

### 제네릭을 굳이 사용하지 않아도 되는 타입

```ts
type MyType<T> = T;
type OurType = "ME" | "YOU" | "OUR";

interface GeneralType {
  getTypes(): MyType<OurType>;
}
```

- 위의 MyType이 다른 곳에 사용되지 않고, 함수 반환값으로 쓰이고 있다 가정할 경우 굳이 필요하지 않다.
- 제네릭을 사용하지 않고 타입 매개변수를 그대로 선언하는 것과 같은 기능을 하고 있다.

```ts
type OurType = "ME" | "YOU" | "OUR";

interface GeneralType {
  getTypes(): OurType;
}
```

> 위와 같은 코드의 동작과 동일하기에 굳이 제네릭 사용이 필요없다.

### any 사용하기

제네릭에 any를 사용하면 제네릭의 장점과 타입 추론 및 타입 검사를 할 수 있는 이점을 누릴 수 없게 된다.

```ts
type ReturnType<T = any> = {
  // ...
};
```

### 가독성을 고려하지 않은 경우

제네릭을 과하게 사용하면 가독성이 해쳐지기 때문에 코드를 읽고 타입을 이해하기 어렵다.
