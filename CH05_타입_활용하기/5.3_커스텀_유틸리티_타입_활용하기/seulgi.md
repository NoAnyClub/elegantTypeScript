# 3.커스텀 유틸리티 타입 활용하기

## 1) 유틸리티 함수를 활용해 styled-components의 중복 선언 피하기

컴포넌트의 스타일 관련 props는 styled-components에 전달되며,
styled-components에도 해당 타입을 정확하게 작성해줘야 한다.

styled-component로 만든 컴포넌트에 넘겨주는 타입은 props에서 받은 타입과 동일할 때가 대부분이다.
이럴 경우, **Pick, Omit과 같은 유틸리티 타입을 활용해 코드를 간결히 작성**할 수 있다.

```tsx
// HrComponent.tsx
type Props = {
  height?: string;
  color: keyof typeof colors;
  isFull: boolean;
  className: string;
};

const HR: VFC<Props> = ({ height, color, isFull, className }) => {
  // ...

  return (
    <HrComponent
      height={height}
      color={color}
      isFull={isFull}
      className={className}
    />
  );
};

// style.ts
import { Props } from "...";

type StyledProps = Pick<Props, "height" | "color" | "isFull">;

const HrComponent = styled.hr<StyledProps>`
  height: ${(height) => height || "10px"};
  margin: 0;
  background-color: ${(color) => colors[color || "gray7"]};
  border: none;

  ${(isFull) =>
    isFull &&
    css`
      margin: 0 -15px;
    `}
`;
```

- Hr 컴포넌트 Props의 height, color, isFull 속성은 styled-components 컴포넌트인 HrComponent에 바로 연결되며 타입도 같다.
- StyledProps를 따로 정의하려면 Props와 똑같은 타입임에도 새로 작성해야해 중복 코드가 생긴다.

> 컴포넌트가 커질수록 또 styled-components로 만든 컴포넌트가 늘어날수록 중복되는 타입이 많아져 관리 포인트가 늘어난다.

### 이런 문제를 Pick, Omit 타입을 사용해 styled-components 컴포넌트 타입을 작성해본다면?

```ts
type StyledProps = Pick<Props, "height" | "color" | "isFull">;
```

- Pick 유틸리티 타입을 활용해 props에서 필요한 부분만 선택해 styled-components 타입을 정의하면 중복된 코드를 작성하지 않아도 된다.
- 이외에도 상속받는 컴포넌트 혹은 부모 컴포넌트에서 자식 컴포넌트로 넘겨주는 props 등에도 Pick, Omit 같은 유틸리티 타입을 활용할 수 있다.

<br />
<br />

## 3) NonNullable 타입 검사 함수를 사용하여 간편하게 타입 가드하기

타입 가드는 타입스크립트에서 많이 사용된다.

특히 null을 가질 수 있는 값의 null 처리는 자주 사용되는 타입 가드 패턴의 하나이다.
if문을 일반적으로 타입 가드로 적용하기도 하지만, is + NonNullable타입으로 타입 검사를 위한 유틸 함수를 만들어 사용할 수 있다.

### NonNullable 타입이란?

타입스크립트에서 제공하는 유틸리티 타입으로 제네릭으로 받는 T가 null 또는 undefined일 때 never 또는 T를 반환하는 타입이다. NonNullable을 사용하면 null이나 undefined가 아닌 경우를 제외할 수 있다.

```ts
type NonNullable<T> = T extends null | undefined ? never : T;
```

### null, undefined를 검사해주는 NonNullable 함수

NonNullable 타입을 사용해 null, undefined를 검사해주는 타입 가드 함수를 만들어 쓸 수 있다.

```ts
function NonNullable<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined;
}
```

> Nullable 함수를 사용하는 쪽에서 true가 반환된다면 넘겨준 인자는 null이나 undefined가 아닌 타입으로 타입 가드가 된다.

### Promise.all을 사용할 때 NonNullable 적용하기

Promise.all을 사용할 때 NonNullable를 적용한 예시를 살펴보자.

```ts
// AdCampaignAPI 클래스: 광고 캠페인 관련 API를 제공
class AdCampaignAPI {
  // operating 메서드: 상점 번호(shopNo)를 인자로 받아 해당 상점의 광고 캠페인 정보를 비동기적으로 가져옴
  static async operating(shopNo: number): Promise<AdCampaign> {
    try {
      // 주어진 상점 번호에 해당하는 광고 캠페인 정보를 가져오기 위해 HTTP 요청을 보냄
      return await fetch(`/ad/shopNumber=${shopNo}`);
    } catch (error) {
      // 오류 발생 시 null을 반환
      return null;
    }
  }
}

// shopList 배열: 상점 정보
const shopList = [
  { shopNo: 100, category: "chicken" },
  { shopNo: 101, category: "pizza" },
  { shopNo: 102, category: "noodle" },
];

// shopAdCampaignList 변수: 각 상점의 광고 캠페인 정보를 비동기적으로 가져와 저장
const shopAdCampaignList = await Promise.all(
  shopList.map((shop) => AdCampaignAPI.operating(shop.shopNo))
);
```

> 위의 코드는 원하는 것처럼 Array<AdCampaign[]>타입으로 추론되는 것이 아니라 null이 될 수 있는 상태인 Array<AdCampaign[] | null>로 추론된다.

```ts
const shopList = [
  { shopNo: 100, category: "chicken" },
  { shopNo: 101, category: "pizza" },
  { shopNo: 102, category: "noodle" },
];

const shopAdCampaignList = await Promise.all(
  shopList.map((shop) => AdCampaignAPI.operating(shop.shopNo))
);

const shopAds = shopAdCampaignList.filter(NonNullable);
```

> 해당 코드처럼 shopAdCampaignList를 필터링하면 Array<AdCampaign[]>타입으로 추론 가능하다.
