# 1. 조건부 타입

프로그래밍에서는 다양한 상황을 다루기 위해 조건문을 많이 활용한다. 타입도 마찬가지로 조건에 따라 다른 타입을 반환해야 할 때가 있다.

### 타입스크립트에서는 조건부 타입을 사용해 조건에 따라 출력 타입을 다르게 도출할 수 있다!

```ts
Condition ? A : B;
```

- A는 Condition이 true일 때 도출되는 타입
- B는 false일 때 도출되는 타입

> 조건부 타입을 활용하면 중복 타입 코드를 제거할 수 있고, 상황에 따라 적절한 타입을 얻을 수 있어 더 정확한 타입 추론이 가능하다.

<br />
<br />

## 1) extends와 제네릭을 활용한 조건부 타입

`extends` 키워드는 타입스크립트에서 다양한 상황에서 활용된다.

- 타입을 확장할 때
- 타입을 조건부로 설정할 때
- 제네릭 타입에서는 한정자 역할로 사용

### 조건부 타입에서 extends를 사용할 때

```ts
T extends U ? X : Y
```

- 자바스크립트 삼항 연산자를 extends와 함께 쓴다.
- 타입 T를 U에 할당할 수 있으면 X타입
- 할당할 수 없으면 Y타입으로 결정

### 결제 수단과 관련된 타입 예시를 살펴보자.

```ts
interface Bank {
  code: string;
  companyName: string;
  name: string;
  fullName: string;
}

interface Card {
  code: string;
  companyName: string;
  name: string;
  addCardType?: string;
}

type PayMethod<T> = T extends "card" ? Card : Bank;
type CardPayMethodType = PayMethod<"card">;
type BankPayMethodType = PayMethod<"bank">;
```

- PayMethod 타입은 제네릭 타입으로 extends를 사용한 조건부 타입이다.
- 제네릭 매개변수에 "card"가 들어오면 Card타입, 그 외의 값은 Bank 타입으로 결정된다.
- PayMethod를 활용해 CardPayPayMethodType, BankPayMethodType을 도출할 수 있다.

<br />
<br />

## 2) 조건부 타입을 사용하지 않았을 때의 문제점

조건부 타입을 사용하기 전에 어떤 이슈가 있었는지 react query 활용 예시를 통해 알아보자.
<br />계좌 / 카드 / 앱 카드 등 3가지 결제 수단 정보를 가져오는 API가 있으며 API의 엔드포인트는 다음과 같다.

```
계좌 : www.site.com/.../bank
카드 : www.site.com/.../card
앱 카드 : www.site.com/.../appcard
```

3가지 API 엔드포인트가 비슷하기에 서버 응답을 처리하는 공통 함수를 생성하고, 해당 함수에 타입을 전달하여 타입별로 처리 로직을 구현할 것이다.

```ts
interface PayMethodBaseFromRes {
  code: string;
  name: string;
}

interface Bank extends PayMethodBaseFromRes {
  fullName: string;
}

interface Card extends PayMethodBaseFromRes {
  appCardType?: string;
}

type PayMethodInfo<T extends Bank | Card> = T & PayMethodInterface;
type PayMethodInterface = {
  companyName: string;
  // ...
};

type PayMethodType = PayMethodInfo<Card> | PayMethodInfo<Bank>;

export const useGetRegisterList = (
  type: "card" | "appcard" | "bank"
): UseQueryResult<PayMethodType[]> => {
  const url = `/codes/${type === "appcard" ? "card" : type}`;

  const fetcher = fetcherFactory<PayMethodType[]>({
    onSuccess: (res) => {
      const usablePayMethodList =
        res?.filter(
          (pocket: PayMethodInfo<Card> | PayMethodInfo<Bank>) =>
            pocket?.useType === "USE"
        ) ?? [];

      return usablePocketList;
    },
  });

  const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);

  return result;
};
```

- useQuery를 사용해 구현한 커스텀 훅 "useGetRegisteredList" 함수는 useQuery 반환 값을 돌려준다.
- useCommonQuery<T>는 useQuery를 한 번 래핑해서 사용하고 있는 함수로 useQuery의 반환 data를 T타입으로 반환한다.
- fetcherFactory는 axios를 래핑하는 함수로 서버에서 데이터를 받아온 후 onSuccess 콜백 함수를 거친 결괏값을 반환한다.

> useGetRegisteredList 함수는 타입으로 "card", "appcard", "bank"를 받아서 해당 결제 수단의 결제 수단 정보 리스트를 반환하는 함수다.

### 위의 코드는 왜 잘못되었을까?

원래는 useGetRegisteredList 함수가 반환하는 Data 타입은 PayMethodInfo라고 유추할 수 있다. 하지만 useGetRegisteredList 함수가 반환하는 Data 타입은 PayMethodType이기 때문에 사용하는 쪽에서 PayMethodInfo일 수 있다.

> useGetRegisteredList 함수는 타입을 구분해서 넣는 사용자 의도와 다르게 정확한 타입을 반환하지 못하는 함수가 되었다. <br /> 인자로 넣는 타입에 알맞은 타입을 반환하고 싶지만, 타입 설정이 유니온으로만 되어있기 때문에 타입스크립트는 해당 타입에 맞는 Data 타입을 추론할 수 없다.

이처럼 인자에 따라 반환되는 타입을 다르게 설정하고 싶다면 extends를 사용한 조건부 타입을 활용하면 된다.

<br />
<br />

## 3) extends 조건부 타입을 활용하여 개선하기

하나의 함수에서 한 번에 API를 관리해야 하는 상황이면 **조건부 타입을 활용하면 하나의 API 함수에서 타입에 따른 정확한 반환 타입을 추론**하게 만들 수 있다.

### extends의 활용사례를 정리해보자!

1. 제네릭과 extends를 함께 사용해 제네릭으로 받는 타입을 제한했다. 따라서 개발자는 잘못된 값을 넘길 수 없기 때문에 휴먼 에러를 방지할 수 있다.
2. extends를 활용해 조건부 타입을 설정했다. 조건부 타입을 사용해서 반환 값을 사용자가 원하는 값으로 구체화할 수 있었다. 이에 따라 불필요한 타입 가드, 타입 단언 등을 방지할 수 있다.

<br />
<br />

## 4) infer를 활용해 타입 추론하기

- extends를 사용할 때 infer 키워드를 사용할 수 있다.
- infer는 "추론하다"라는 의미를 지니고 있다.
- 타입스크립트에서는 단어 의미처럼 타입을 추론하는 역할을 한다.
- 삼항 연산자를 사용한 조건문의 형태를 가지는데, extends로 조건을 서술하고 infer로 타입을 추론하는 방식을 취한다.

### 예시로 infer를 알아보자

```ts
type UnpackPromise<T> = T extends Promise<infer K>[] ? K : any;

const promises = [Promise.resolve("Chu"), Promise.resolve(11)];

type Expected = UnpackPromise<typeof promises>; // string | number
```

- UnpackPromise 타입은 제네릭으로 T를 받아 T가 Promise로 래핑된 경우면 K를 반환하고, 아니면 any를 반환한다.
- Promise<infer K>는 Promise 반환 값을 추론해 해당 값의 타입을 K로 한다.

> extends, infer, 제네릭을 활용하면 타입을 조건에 따라 더 세밀하게 사용할 수 있게 된다.
