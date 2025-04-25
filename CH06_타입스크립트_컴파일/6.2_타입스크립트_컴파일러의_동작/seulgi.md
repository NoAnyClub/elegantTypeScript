# 2. 타입스크립트 컴파일러의 동작

## 1) 코드 검사기로서의 타입스크립트 컴파일러

타입스크립트는 컴파일타임에 문법 에러와 타입 관련 에러를 모두 검출한다.

```js
const developer = {
  work() {
    console.log("working");
  },
};

developer.work();
developer.sleep(); // TypeError;
```

> 해당 코드를 자바스크립트로 작성하는 시점에는 에러가 발생하지 않으나, 실제 실행 시 에러가 난다.

```ts
const developer = {
  work() {
    console.log("working");
  },
};

developer.work();
developer.sleep(); // Property "sleep" does not exist on type "{work(): void}"
```

> 타입스크립트는 코드를 실행 전에 에러를 사전에 발견해 알려준다. <br />
> 즉, 런타임에 발생할 수 있는 문법 오류 등의 에러뿐 아니라 타입 에러도 잡을 수 있다.

타입스크립트 컴파일러는 tsc binder를 사용하여 타입 검사를 하며, 컴파일타임에 타입 오류를 발견한다.<br />
타입 검사를 거쳐 코드를 안전하게 만든 이후에는 타입스크립트 AST를 자바스크립트 코드로 변환한다.

<br />
<br />

## 2) 코드 변환가로서의 타입스크립트 컴파일러

타입스크립트 코드를 각자의 런타임 환경에서 동작할 수 있도록<br />
구버전의 자바스크립트로 트랜스파일한다.<br />
이것이 <u>타입스크립트 컴파일러의 두 번째 역할</u>이다.

- 타입스크립트 컴파일러의 target 옵션을 사용하여 특정 버전의 자바스크립트 소스코드로 컴파일할 수 있다.
- 타입스크립트 컴파일러는 타입 검사를 수행한 후 코드 변환을 시작한다.
- 이때 타입 오류가 있더라도 일단 컴파일을 진행한다.

> 타입스크립트 코드가 자바스크립트 코드로 변환되는 과정은<br />
> 타입 검사와 독립적으로 동작하기 때문이다.<br />

> 타입스크립트 코드 타이핑이 잘못되서 발생하는 에러도 런타임 에러로 처리된다. 자바스크립트 실행 과정에서 런타임 에러로 처리된다.

### 다만 문제가 있다..!

타입스크립트 소스코드에 타입 에러가 있더라도
자바스크립트로 컴파일되어 타입 정보가 모두 제거된 후에는
타입이 아무런 효력을 발휘하지 못한다.

```ts
interface Square {
  width: number;
}

interface Rectangle extends Square {
  height: number;
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  // 에러! Rectangle이 타입이기 때문에 런타임은 해당 코드를 이해하지 못함
  if (shape instanceof Rectangle) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

> 타입스크립트 코드가 자바스크립트로 <u>컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문이 제거되어 버리기 때문에 런타임에서는 타입을 사용할 수 없다.</u>

### 타입스크립트 컴파일러의 역할?

1. 최신 버전의 타입스크립트 / 자바스크립트 코드를 구버전의 자바스크립트로 트랜스파일한다.
2. 코드의 타입 오류를 검사한다.

### 바벨 (Babel)?

ECMAScript 2015 이후의 코드를 현재 또는 오래된 브라우저와 호환되는 버전으로 변환해주는 자바스크립트 컴파일러이다.

<br />
<br />
