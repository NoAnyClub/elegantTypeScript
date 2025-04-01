# 타입 좁히기 - 식별할 수 있는 유니온(Discriminated Unions)

태그된 유니온(tagged union) 혹은 식별할 수 있는 유니온 타입(Discriminated Unions)은 타입 좁히기에 널리 사용되는 방식이다.

## 1) 에러 정의하기

유효성 에러가 발생하면 사용자에게 다양한 방식으로 에러를 보여주는 예시로 살펴보자.

우아한 형제들에서는 <u>"텍스트 에러", "토스트 에러", "얼럿 에러"로 분류</u>한다. 이들 모두 유효성 에러로 에러 코드와 메시지를 가지고 있고 <u>에러 노출 방식에 따라 추가로 필요한 정보가 있을 수 있다고 가정하자.</u>

```ts
type TextError = {
  code: string;
  message: string;
};

type ToastError = {
  code: string;
  message: string;
  toastShowDuration: number;
};

type AlertError = {
  code: string;
  message: string;
  onConfirm: () => void;
};

type ErrorType = TextError | ToastError | AlertError;

// 에러 타입의 유니온 타입을 원소로 하는 배열로 다양한 에러 객체를 관리 가능
const errorArr: ErrorType[] = [
  { code: "100", message: "텍스트 에러" },
  { code: "200", message: "토스트 에러", toastShowDuration: 3000 },
  { code: "300", message: "얼럿 에러", onConfirm: () => {} },
];
```

### 위의 배열에 에러 타입별로 정의한 필드를 가지는 에러 객체가 포함되길 원한다면 어떨까?

ToastError의 toastShowDuration 필드와 AlertError의 onConfirm 필드를 모두 가지는 객체에 대해서 에러를 받아야 한다.

```ts
const errorArr: ErrorType[] = [
  { code: "100", message: "텍스트 에러" },
  { code: "200", message: "토스트 에러", toastShowDuration: 3000 },
  { code: "300", message: "얼럿 에러", onConfirm: () => {} },
  {
    code: "300",
    message: "잘못된 에러",
    toastShowDuration: 3000,
    onConfirm: () => {},
  },
];
```

> 해당 코드 작성 시 자바스크립트는 덕타이핑 언어이기 때문에 별도의 타입 에러를 뱉지 않는다.

## 2) 식별할 수 있는 유니온

위의 예시처럼 타입 에러가 발생하지 않을 경우 개발 과정에서 무수한 에러 객체가 생겨날 위험성이 커진다.

이럴 때 에러 타입을 구분할 방법이 필요한데 **각 타입이 비슷한 구조를 가지지만
<br /> 서로 호환되지 않도록 만들어줘야 한다.**

> "식별할 수 있는 유니온을 활용하는 것"이 그 방법!

### 식별할 수 있는 유니온이란?

타입 간의 구조 호환을 막기 위해 타입마다 구분할 수 있는 판별자(= 태그)를 달아주어 포함 관계를 제거하는 것이다.

### 판별자의 개념으로 errorType 필드를 새로 정의해보자!

```ts
type TextError = {
  type: "TEXT";
  code: string;
  message: string;
};

type ToastError = {
  type: "TOAST";
  code: string;
  message: string;
  toastShowDuration: number;
};

type AlertError = {
  type: "ALERT";
  code: string;
  message: string;
  onConfirm: () => void;
}; // Object literal may only specify known properties, and 'toastShowDuration' does not exist in type 'AlertError'.
```

- 각 필드마다 다른 값을 가지도록 해 판별자를 달아준다.
- 이 판별자에 의해 포함 관계를 벗어나게 된다.

> 정확하지 않은 에러 객체에 대해 타입 에러가 발생하는 것을 확인할 수 있다.

<br />
<br />

## 3) 식별할 수 있는 유니온의 판별자 선정

식별할 수 있는 유니온을 사용할 때 주의할 점이 있다.

식별할 수 있는 유니온의 판별자는 유닛 타입(unit type)으로 선언되어야 정상적으로 동작한다.

### 유닛 타입이란?

다른 타입으로 쪼개지지 않고 오직 하나의 정확한 값을 가지는 타입이다.

- null, undefined, 리터럴 타입을 비롯해 true, 1 등 정확한 값을 나타내는 타입이다.
- 반면, 다양한 타입을 할당할 수 있는 void, string, number와 같은 타입은 유닛 타입이 아니다.

> **식별할 수 있는 유니온의 판별자로 사용할 수 있는 타입은?** <br />(1) 리터럴 타입이어야 한다, <br />(2) 판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며, 인스턴스화할 수 있는 타입은 포함되지 않아야 한다.

### 인스턴스화할 수 있는 타입은 뭘까?

TypeScript에서 **“인스턴스화할 수 있는 타입”** 이란 new 키워드를 사용하여 객체(인스턴스)를 생성할 수 있는 타입을 의미한다.

### 인스턴스화할 수 있는 타입 목록

| 타입 종류                    | 인스턴스화 가능 여부 | 예시                                     |
| ---------------------------- | -------------------- | ---------------------------------------- |
| **클래스**                   | ✅ 가능              | `class Person {}`                        |
| **생성자 함수**              | ✅ 가능              | `function Animal() {}`                   |
| **생성자 인터페이스**        | ✅ 가능              | `interface A { new(): B }`               |
| **함수 타입 (`new()` 포함)** | ✅ 가능              | `type A = new (...args) => B`            |
| **내장 객체**                | ✅ 가능              | `Date, RegExp, Map, Set, Promise, Error` |
| **원시 타입**                | ❌ 불가능            | `string, number, boolean, symbol`        |
| **일반 인터페이스**          | ❌ 불가능            | `interface A { name: string; }`          |
| **객체 리터럴 타입**         | ❌ 불가능            | `type A = { name: string }`              |
| **유니온 타입**              | ❌ 불가능            | `type A = string \| number`              |
| **추상 클래스**              | ❌ 불가능            | `abstract class A {}`                    |

> “인스턴스화할 수 있는 타입”이란 new 키워드를 사용하여 객체를 생성할 수 있는 타입을 의미한다. 클래스, 생성자 함수, new() 시그니처가 있는 인터페이스, 생성자 함수 타입, 일부 내장 객체가 이에 해당한다.

<br />
<br />

### 판별자의 타입에 따라 타입이 달라지는 예시를 알아보자.

```ts
interface A {
  value: "a"; // unit type
  answer: 1;
}

interface B {
  value: string; // not unit type
  answer: 2;
}

interface C {
  value: Error; // instantiable type
  answer: 3;
}

type Unions = A | B | C;

function handle(param: Unions) {
  param.answer; // 1 | 2 | 3

  // "a"가 리터럴 타입으로 좁혀지나 string 타입에 포함도 되기 때문에 param은 A | B로 좁혀진다.
  if (param.value === "a") {
    param.answer; // 1 | 2 return;
  }

  // 유닛 타입이 아니거나 인스턴스화할 수 있는 타입일 경우 타입이 좁혀지지 않는다.
  if (typeof param.value === "string") {
    param.answer; // 1 | 2 | 3 return;
  }

  if (param.value instanceof Error) {
    param.answer; // 1 | 2 | 3 return;
  }

  // 판별자가 answer일 때
  param.value; // string | Error

  // 판별자가 유닛 타입이므로 타입이 좁혀진다.
  if (param.answer === 1) {
    param.value; // "a"
  }
}
```

- 해당 코드에서는 판별자가 value일 떄 판별자로 선정한 값 중 "a"만 유일하게 유닛 타입이다.
- **이떄만 유닛 타입을 포함하고 있어 타입이 좁혀진다!**
- 판별자가 answer일 때를 보면 판별자가 모두 유닛 타입이라 타입이 정상적으로 좁혀진다.

<br />
<br />
