# 타입 좁히기 - 타입 가드

타입 좁히기는 변수 또는 표현식의 타입 범위를 더 작은 범위로 좁혀나가는 과정이다.
더 정확하고 명시적인 타입 추론을 할 수 있고, 복잡한 타입을 작은 범위로 축소하여 타입 안정성을 높일 수 있다.

## 1) 타입 가드에 따라 분기 처리하기

타입스크립트에서 <u>분기 처리</u>는 **조건문과 타입 가드를 활용**한다.<br /><u>변수나 표현식의 타입 범위를 좁혀</u> 다양한 상황에 따라 다른 동작을 수행하는 것이다.

타입 가드는 **런타임에 조건문을 사용해 타입을 검사해 타입 범위를 좁히는 기능**이다.

### 여러 타입을 할당할 수 있는 스코프에서 분기 처리하려면?

여러 타입을 할당할 수 있는 스코프(타입스크립트에서 스코프는 변수와 함수 등 식별자가 유효한 범위,
<br /> 즉 변수와 함수를 선언하거나 사용할 수 있는 영역)에서 특정 타입을 조건으로 만들어 분기 처리하고 싶을 때가 있다.

> 이 때, 여러 타입을 할당할 수 있다는 것은 변수가 유니온 타입 또는 any 타입 등과 같이 여러 가지 타입을 받을 수 있다는 것을 말한다.

### 함수가 A | B 타입의 매개변수를 받을 때, <br /> 인자 타입이 A 또는 B일 때를 구분해서 로직을 처리하고 싶다면?

if 문을 사용해서 처리한다해도 컴파일 시 타입 정보는 모두 제거되기 때문에 런타임에 존재하지 않는다.

> 컴파일해도 타입 정보가 사라지지 않는 방법을 사용해야 하고, 특정 문맥 안에서 타입스크립트가 해당 변수를 타입 A로 추론하도록 유도하면서 런타임에서도 유효한 방법이 필요하다!

### 이 때 사용하는 것이 "타입 가드"!

1. 자바스크립트 연산자를 사용한다.
2. 사용자 정의 타입 가드로 사용한다.

위의 두 방법으로 타입 가드를 사용할 수 있다.

### 자바스크립트 연산자를 활용한 타입 가드에 대해 알아보자.

- typeof, instanceof, in과 같은 연산자를 사용한다.
- 제어문으로 특정 타입 값을 가질 수밖에 없는 상황을 유도해 자연스럽게 타입을 좁히는 방식이다.

> 런타임에 유효한 타입 가드를 만들기 위해 자바스크립트 연산자를 사용한다.

<br />
<br />

## 2) 원시 타입을 추론할 때: typeof 연산자 활용하기

- typeof 연산자를 활용하면 원시 타입에 대해 추론할 수 있다.
- typeof A === B를 조건으로 분기 처리하면 해당 분기 내에서 A의 타입이 B로 추론된다.
- 다만, typeof는 자바스크립트 타입 시스템만 대응 가능하다.
- null, 배열 타입 등 object 타입이 판별되는 등 복잡한 타입을 검증하기에는 한계가 있다.

> 따라서, typeof 연산자는 주로 원시 타입을 좁히는 용도로만 사용하는 것을 권장한다.

### typeof 연산자를 사용해 검사할 수 있는 타입들

- string
- number
- boolean
- undefined
- object
- function
- bigint
- symbol

```ts
const replaceHyphen: (date: string | Date) => string | Date = (date) => {
  if (typeof date === "string") {
    return date.replace(/-/g, "/");
  }

  return date;
};
```

<br />
<br />

## 3) 인스턴스화된 객체 타입을 판별할 때: instanceof 연산자 활용하기

selected 매개변수가 Date인지 검사 후 Range 타입의 객체를 반환할 수 있도록 분기처리하는 예시를 보자.

```ts
interface Range {
  start: Date;
  end: Date;
}

interface DatePickerProps {
  selectedDates?: Date | Range;
}

const DatePicker = ({ selectedDates }: DatePickerProps) => {
  const [selected, setSelected] = useState(convertToRange(selectedDates));
};

export function convertToRange(selected?: Date | Range): Range | undefined {
  return selected instanceof Date
    ? { start: selected, end: selected }
    : selected;
}
```

- instanceof 연산자는 인스턴스화된 객체 타입을 판별하는 타입 가드로 사용한다.
- A instanceof B 형태로 사용한다.
- A에는 타입을 검사할 대상 변수, B에는 특정 객체의 생성자가 들어간다.

> **instanceof?** <br />
> instanceof는 A의 프로토타입 체인에 생성자 B가 존재하는지를 검사해<br /> 존재한다면 true, 그렇지 않다면 false를 반환한다.

### A의 프로토타입 속성 변화에 따라 instanceof 연산자의 결과가 달라질 수 있다.

```ts
const onKeyDown = (event: React.KeyboardEvent) => {
  if (event.target instanceof HTMLElement && event.key === "Enter") {
    event.target.blur();
    onCTAButtonClick(event);
  }
};
```

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

## 4) 객체의 속성이 있는지 없는지에 따른 구분: in 연산자 활용하기

- in연산자는 객체에 속성이 있는지 확인한 다음에 true 또는 false를 반환한다.
- A in B의 형태로 사용한다.
- A라는 속성이 B 객체에 존재하는지를 검사한다.
- 프로토타입 체인으로 접근할 수 있는 속성이면 전부 true를 반환한다.

### 프로토타입 체인으로 접근할 수 있는 속성이면 전부 true를 반환한다?

```ts
const obj = { a: 1 };

console.log("a" in obj); // true (자기 자신의 속성)
console.log("toString" in obj); // true (Object.prototype에 존재하는 속성)
console.log("b" in obj); // false (존재하지 않는 속성)
```

> 객체 자체뿐만 아니라 프로토타입 체인까지 확인하는 연산자가 바로 in 연산자!

- B 객체에 존재하는 속성에 undefined를 할당한다고 해서 false를 반환하지 않는다.
- delete 연산자를 사용해 객체 내부에서 해당 속성을 제거해야만 false를 반환한다.

```ts
interface BasicDialogProps {
  Title: string;
  Body: string;
}

interface DialogWithCookieProps extends DialogProps {
  cookieKey: string;
  noForADay?: boolean;
}

export type DialogProps = BasicDialogProps | DialogWithCookieProps;
```

위의 Dialog 컴포넌트는 BasicDialogProps | DialogWithCookieProps를 유니온 타입으로 가지는 DialogProps를 props로 받는다.

> Dialog 컴포넌트가 props로 받는 객체 타입이 둘 중에 어떤 타입이냐에 따라 렌더링하는 컴포넌트가 달라지게 하려면 props의 타입에 따라 렌더링하는 컴포넌트를 분기 처리하면 된다.

### 위의 예제에서 두 객체 타입을 어떻게 분기 처리할까?

- 자바스크립트 in 연산자는 런타임의 값만 검사한다.
- 타입스크립트에서는 객체 타입에 속성이 존재하는지를 검사한다.

```ts
const Dialog: React.FC<DialogProps> = (props) => {
  if ("cookieKey" in props) {
    return <DialogWithCookie {...props} />;
  }

  return <Dialog {...props} />;
};
```

- 위의 예제는 두 객체 타입을 cookieKey라는 속성을 가졌는지 확인하는 조건을 만든다.
- if문 스코프에서 타입스크립트는 props 객체를 cookieKey 속성을 갖는 객체 타입인 DialogWithCookieProps로 해석한다.
- 얼리 리턴으로, if문 스코프 밖에 위치하는 return문의 props 객체는 BasicDialogProps 타입으로 해석한다.

> 여러 객체 타입을 유니온 타입으로 가지고 있을 때 in 연산자를 사용해 속성의 유무에 따라 조건 분기를 할 수 있다.

<br />
<br />

## 5) is연산자로 사용자 정의 타입 가드 만들어 활용하기

직접 타입 가드 함수를 만들 수 있다. 해당 방식의 타입 가드는 반환 타입이 타입 명제(type predicates)인 함수를 정의해 사용할 수 있다.

- 타입 명제는 A is B 형식으로 작성한다.
- A는 "매개변수 이름", B는 "타입"이다. ("매개변수" is "타입")
- 참/거짓의 진릿값을 반환하면서 반환 타입을 타입 명제로 지정하게 되면 반환 값이 참일 때 A 매개변수의 타입을 B타입으로 취급한다.
- 타입 명제는 함수의 반환 타입에 대한 타입 가드를 수행하기 위해 사용되는 특별한 형태의 함수다.

### predicates?

수학적으로 주어진 조건을 만족하는지 아닌지를 테스트하는 함수나 표현식을 의미한다.
프로그래밍 컨텍스트에서는 보통 어떤 값을 입력으로 받아 Boolean 값을 반환하는 함수를 가리키는데 대부분 통용되는 용어다.

```ts
const isDestinationCode = (x: string): x is DestinationCode =>
  destinationCodeList.includes(x);
```

1. `isDestinationCode` 함수는 string 타입의 x 매개변수가 `DestinationCode` 배열의 원소 중 하나인지를 검사해 `boolean`을 반환하는 함수이다.
2. 함수의 반환 값을 `boolean`이 아닌 `x is DestinationCode`로 타이핑한다.
3. 해당 타이핑으로 타입스크립트에게 이 함수가 사용되는 곳의 타입을 추론할 때 해당 조건을 타입 가드로 사용하도록 알려준다.

### 반환 값의 타입이 boolean인 것과 비교해보자

```ts
const getAvailableDestinationNameList = async ():Promise<DestinationName[]> => {
const data = await AxiosRequest<string[]>('get', '.../destinations');
    const destinationNames = DestinationName[] = [];

    data?.forEach((str) => {
        if (isDestinationCode(str)) {
            destinationNames.push(DestinationNameSet[str]);
        }
    });

    return destinationNames;
}
```

> isDestinationCode(str)의 반환 값에 is를 사용하지 않고 boolean이라고 하면 에러가 발생한다. 위의 if 문 내 isDestinationCodeList의 문자열 원소 인지 체크하고 맞으면 destinationNames에 push한다.

### isDestinationCode의 반환 값 타이핑을 x is DestinationCode가 아닌<br /> boolean으로 했다면 타입스크립트는 어떻게 추론할까?

개발자는 if문 내부에서 str타입이 DestinationCode라는 걸 알 수 있는데 Array.inclueds를 해석할 수 있기 때문이다.

하지만, 타입스크립트는 isDestinationCode 함수 내부에 있는 includes 함수를 해석해 타입 추론을 할 수 없다.

> 타입스크립트는 if문 스코프의 str타입을 좁히지 못하고 string으로만 추론한다. destinationNames의 타입은 DestinationName[]이기 때문에 string타입의 str을 push할 수 없다는 에러가 발생한다.

### 타입스크립트에게 반환 값에 대한 타입 정보를 알려주고 싶을 때 is를 사용할 수 있다!

반환 값 타입을 x is DestinationCode로 알려줌으로써 타입스크립트는 if문 스코프의 str 타입을 DestinationCode로 추론할 수 있게 된다.

<br />
<br />
