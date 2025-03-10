# 원시 타입

자바스크립트에서는 값은 타입을 가지지만 변수는 별도의 타입을 가지지 않기 때무네 어떤 타입 값이라도 자유롭게 할당할 수 있다. 타입스크립트는 이 변수에 타입을 지정할 수 있는 타입 시스템 체계를 구축한다.

### 원시 값과 원시 래퍼 객체?

자바스크립트 내장 타입을 파스칼 표기법으로 표기했다.
반면, 타입스크립트에서는 타입을 소문자로 표기하는데 자바스크립트는 컴파일 시점에 타입스크립트의 타입 시스템이 적용되지 않아 타입스크립트와의 구분을 위해 소문자로 표기하지 않았다.

타입을 파스칼 표기법으로 표기하면 자바스크립트에서 이것을 원시 래퍼 객체로 부른다.<br />
`null`과 `undefined`를 제외한 모든 원시 값은 해당 원시 값을 래핑한 객체를 가진다.

<br />
<br />

## 1) boolean

```ts
const isEmpty: boolean = true;

function isTextError(errorCode: ErroCodeType): boolean {
  if (errorCode) {
    return true;
  }

  return false;
}
```

- true, false 값만 할당할 수 있는 타입
- 위의 함수처럼 비교식의 결과 또한 boolean 타입을 가질 수 있다.
- 형 변환을 통해 true / false로 취급되는 `Truthy` / `Falsy` 값은 boolean 타입에 해당하지 않는다.

<br />
<br />

## 2) undefined

```ts
let value: string;
console.log(value); // undefined(값이 아직 할당되지 않음)
```

- 정의되지 않았다는 의미의 타입
- 오직 undefined만 할당할 수 있다.
- 초기화되지 않은 값을 의미하며 변수 선언만 하고 값을 할당하지 않을 때나 옵셔널로 지정되어 있을 때 undefined를 할당할 수 있다.

> 즉, 초기화되어 있지 않거나, 존재하지 않음을 나타낸다.

<br />
<br />

## 3) null

- null만 할당이 가능하다.
- 자바스크립트에서 빈 값을 할당할 때 사용하는 등 명시적 / 의도적으로 값이 아직 비어있을 수 있음을 보여준다.
- undefined와 비슷하지만 엄연히 따로 존재하는 원시 값이기 때문에 서로의 타입에 할당할 수 없다.

### null과 undefined의 차이를 보여주는 예시를 살펴보자.

```ts
type Person1 = {
  name: string;
  job?: string;
};

type Person2 = {
  name: string;
  job: string | null;
};
```

- Person1은 job이라는 속성이 있을 수도 없을 수도 있음을 나타낸다.(job이라는 속성 유무를 통해 무직인지 아닌지를 나타냄)
- Person2는 job이라는 속성을 사람마다 갖고 있지만 값이 비어있을 수도 있다는 것을 나타낸다. (명시적인 null을 할당해 무직 상태를 나타냄)

<br />
<br />

## 4) number

- 숫자에 해당하는 모든 원시 값을 할당할 수 있다.
- 자바스크립트의 숫자는 정수, 부동소수점수를 구분하지 않기 때문에 모두 number 타입에 할당 가능하다.
- NaN이나 Infinity도 포함된다.

<br />
<br />

## 5) bigInt

```ts
const bigNumber1: bigInt = BigInt(999999999999);
```

- ES2020에 새롭게 도입된 데이터 타입
- 타입스크립트 3.2버전부터 사용할 수 있다.
- 자바스크립트에서는 가장 큰 수인 Number.MAX_SAFE_INTEGER($2^{53}$ - 1)을 넘어가는 값을 처리할 수 없었는데 bigInt를 사용하면 된다.
- number타입과 상호 작용은 불가능하다.

<br />
<br />

## 6) string

- 문자열을 할당할 수 있는 타입
- 공백, 작은따옴표, 큰따옴표 문자열 외에도 백틱으로 감싼 문자열 내부에 변숫값을 포함할 수 있는 템플릿 리터럴(template literal) 문법도 포함된다.

<br />
<br />

## 7) symbol

```ts
const MOVIE_TITLE = Symbol("title");
const MUSIC_TITLE = Symbol("title");

console.log(MOVIE_TITLE === MUSIC_TITLE); // false

let SYMBOL: unique symbol = Symbol(); // Error!
```

- ES2015에 도입된 타입으로 Symbol() 함수를 사용하면 어떤 값과도 중복되지 않는 유일한 값을 생성한다.
- 타입스크립트에서는 symbol 타입과 const 선언에서만 사용할 수 있는 unique symbol 타입이라는 symbol의 하위 타입도 있다.

<br />
<br />

## 8) 마무리

null이나 undefined는 tsconfig 옵션이나 사용자 취향에 따라 다르게 사용될 여지가 있다.

타입스크립트의 모든 타입은 기본적으로 null과 undefined를 포함하고 있다.
tsconfig의 strictNullChecks 옵션을 활성화했을 때는 사용자가 명시적으로 해당 타입에 null이나 undefined를 포함해야만 null과 undefined를 사용할 수 있다.

보통은 null과 undefined를 `타입 가드`로 걸러내거나 `!연산자`를 사용해 타입을 단언하기도 한다.
일반적으로는 타입 가드를 사용하는 것이 더 안전하다고 여겨지기도 한다.
