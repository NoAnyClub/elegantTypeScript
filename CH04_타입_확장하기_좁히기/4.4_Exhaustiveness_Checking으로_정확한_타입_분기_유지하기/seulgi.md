# Exhaustiveness Checking으로 정확한 타입 분기 유지하기

### Exhaustiveness?

사전적으로 철저함, 완전함을 의미한다.
<br />Exhaustiveness Checking은 모든 케이스에 대해 철저하게 타입을 검사하는 것이다.
<br />타입 좁히기에 사용되는 패러다임 중 하나이다.

때때로 모든 케이스에 대해 분기 처리를 해야 유지보수 측면에서 안전하다하는 상황에서
<br /> Exhaustiveness Checking을 통해 모든 케이스에 대한 타입 검사를 강제할 수 있다.

## 1) 상품권

선물하기 서비스에 있는 상품권 가격에 따라 상품권 이름을 반환해주는 함수가 있다.

```ts
type ProductPrice = "10000" | "20000";

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === "10000") {
    return "1만원";
  }

  if (productPrice === "20000") {
    return "2만원";
  } else {
    return "기타";
  }
};
```

### 만약 상품권이 늘어나면 어떨까?

```ts
type ProductPrice = "10000" | "20000" | "5000";

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === "10000") {
    return "1만원";
  }

  if (productPrice === "20000") {
    return "2만원";
  }

  if (productPrice === "5000") {
    return "5천원"; // 조건 추가
  } else {
    return "기타";
  }
};
```

- ProductPrice 타입이 업데이트되었을 때 getProductName 함수도 업데이트 되어야 한다.

### 업데이트된 타입에 맞게 함수를 수정하지 않아도 별도 에러가 발생하지 않을 수 있다!?

이럴 때 필요한 것이 모든 타입에 대한 타입 검사를 강제해야 하는 것이다.

```ts
type ProductPrice = "10000" | "20000" | "5000";

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === "10000") {
    return "1만원";
  }

  if (productPrice === "20000") {
    return "2만원";
  } else {
    exhaustiveCheck(productPrice); // Error!
    return "기타";
  }
};

const exhaustiveCheck = (param: never) => {
  throw new Error("type error!");
};
```

- exhaustiveCheck(productPrice)부분에 에러가 나는데 5000이라는 값에 대한 분기 처리를 하지 않아 발생한 것이다.

> 이렇게 모든 케이스에 대한 타입 분기 처리를 해주지 않았을 때, 컴파일 타임 에러가 발생하게 하는 것을 Exhaustiveness Checking이라 한다.

### exhaustiveCheck 함수를 자세히 알아보자.

exhaustiveCheck 함수 매개변수를 never로 선언한 것은 매개변수로 그 어떤 값도 받을 수 없다는 것이다.

만일 값이 들어오면 에러를 내뱉고, 이 함수를 타입 처리 조건문의 마지막 else문에 사용하면
<br />앞의 조건문에서 모든 타입에 대한 분기 처리를 강제 할 수 있다.

> Exhaustiveness Checking을 활용하면 런타임 에러 방지 + 요구사항 변경 시 위험성을 줄일 수 있다.

<br />
<br />
