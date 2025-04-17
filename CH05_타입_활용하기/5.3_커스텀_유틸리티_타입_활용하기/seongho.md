ts로 프로젝트를 진행하다보면 표현하기 힘든 타입을 마주할 때가 있음. 이럴대는 유틸리티 타입을 활용한 커스텀 유틸리티 타입을 제작해서 사용하면 됨. <br />

해당 장에서는 배민이 어떻게 커스텀 유틸리티 타입을 활용하는지 알아보자

### 유틸리티 함수를 활용해서 styled-component의 중복 타입 선언 피하기
---
styled-components에서는 props를 활용해서 컴포넌트에 동적으로 다양하게 스타일을 줄 수 있음.
리액트에서는 props로 받은 값을 스타일드 컴포넌츠에 그대로 넘겨주는 방식이 대부분임. 근데 아래와 같은 코드는
Props와 같은 타입임에도 불구하고, 새로 작성해야 하므로 불가피하게 중복된 코드가 생김.
```ts
type Props = {
    height?: string;
    color: keyof typeof colors;
    isFull: boolean;
    className: string;
};

const HR: VFC<Props> = ({height, color, isFull, className}) => {
    // ...

    return <HrComponent
        height={height}
        color={color}
        isFull={isFull}
        className={className}
    />;
}

// 중복 코드 발생
type StyledProps = {
    height?: string;
    color: keyof typeof colors;
    isFull: boolean;
}


const HrCompoent = styled.hr<StyledProps>`
    height: ${(height) => height || '10px'};
    margin: 0;
    background-color: ${(color) => colors[color || 'gray7']};
    border: none;

    ${(isFull) =>
        isFull &&
        css`
            margin: 0 -15px;
        `}
`;
```
이러면 컴포넌트가 커지고, 늘어날수록 관리해야되는 포인트도 늘어가게 됨. <br />
예를 들어서 `Props`의 값이 수정되면, `StyledProps`에도 같이 수정해주어야 함.(관리포인트가 2개가 됨)

따라서 이런 코드는 `Pick` 유틸리티 함수를 사용하면 중복코드를 줄일 수 있음.
```ts
type StyledProps = Pick<Props, 'height' | 'color' | 'isFull'>;


const HrCompoent = styled.hr<StyledProps>`
    height: ${(height) => height || '10px'};
    margin: 0;
    background-color: ${(color) => colors[color || 'gray7']};
    border: none;

    ${(isFull) =>
        isFull &&
        css`
            margin: 0 -15px;
        `}
`;
```

<br />

### NonNullable 타입 검사 함수를 사용해서 간편하게 타입 가드 하기
타입가드는 타입스크립트에서 많이 사용 됨. 특히, null을 가질 수 있는 값의 null처리는 자주 사용되는 타입 가드의 패턴중에 하나임. <br />
일반적으로 if문을 사용해서 null처리 타입 가드를 적용하지만, is키워드와 NunNullable타입으로 타입 검사를 위한 유틸 함수를 만들어서 사용할수도 있음.

> [!NOTE]
> **NonNullable타입**
> ts에서 제공하는 유틸리타 타입으로 T가 null또는 undefined일 경우, never 또는 T를 반환하는 타입임.
> ```ts
> type NonNullable<T> = T extends null | undefined ? never : T;
> ```

```ts
type NonNullables<T> = T extends null | undefined ? never : T;

function NonNullable<T>(target : T): target is NonNullables<T> {
    return target !== undefined && target !== null;
}
```
-> `NonNullable`함수는 매개변수인 target이 null 또는 undefined라면 false를 return함.
만약 true를 리턴 한다면 타입가드가 발생함.

어떠한 상황에서 쓸일 수 있는지 살펴보자. 예를 들어서 아래와 같은 API를 call하는 함수가 있는데, 에러가 발생하면 null을 return 함.
```ts
interface BrandDetail {};

async function getBrandDetail(brandId: number) {
  try {
    const response = await fetch<BrandDetail>();

    return response;
  } catch {
    return null;
  }
}

const brandInformationList = [
  {brandId: 1},
  {brandId: 2},
  {brandId: 3},
];

const brandList = await Promise.all(brandInformationList.map(({ brandId }) => getBrandDetail(brandId)));
```
이때 brandList의 타입은`(BrandDetail | null)[]` 타입이 됨.
여기서 만약 null을 제외하고 싶다면
```ts
const brandList = brandInformationList.filter(NonNullable); // BrandDetail[]
```
이렇게 null을 예외처리 할 수 있음.

