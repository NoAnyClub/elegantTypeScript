# 4.불변 객체 타입으로 활용하기

상숫값을 관리할 때 객체를 사용하기도 한다.

아래처럼 키 타입을 해당 객체에 존재하는 키값으로 설정하는 것이 아니라 string으로 설정하면<br />
색상 컬러를 가져오는 getColorHex 함수의 반환 값은 any가 된다.

```ts
const colors = {
  red: "#45452",
  green: "#0C952A",
};

const getColorHex = (key: string) => color[key];
```

- 여기서 as const 키워드로 객체를 불변 객체로 선언한 후, keyof 연산자를 사용해<br /> getColorHex 함수 인자로 실제 colors 객체에 존재하는 키값만 받도록 설정할 수 있다.

> keyof, as const로 객체 타입을 구체적으로 설정하면 타입에 맞지 않는 값을 전달할 경우<br /> 타입 에러가 반환되기 때문에 컴파일 단계에서 발생할 수 있는 실수를 방지할 수 있다.

## 1) Atom 컴포넌트에서 theme style 객체 활용하기

Atom 단위의 작은 컴포넌트의 스타일들을 다양하게 사용할 수 있게 구현하려면<br />
대부분 프로젝트에서는 해당 프로젝트의 스타일 값을 관리해주는 theme 객체를 두고 관리한다.

theme 객체로 타입을 구체화하려면 keyof, typeof 연산자가 타입스크립트에서 어떻게 사용되는지 알아야 한다.

### 타입스크립트 keyof 연산자로 객체의 키값을 타입으로 추출하기

타입스크립트에서 keyof 연산자는 객체 타입을 받아 해당 객체의 키값을 string 또는 number의 리터럴 유니온 타입을 반환한다.
객체 타입으로 인덱스 시그니처가 사용되었다면 keyof는 인덱스 시그니처 키 타입을 반환한다.

```ts
interface ColorType {
  red: string;
  green: string;
}

type ColorKeyType = keyof ColorType; // "red" | "green"
```

> ColorType 객체 타입의 keyof ColorType을 사용하면 객체의 키값인 "red" | "green"가 유니온으로 나오게 된다.

### 타입스크립트 typeof 연산자로 값을 타입으로 다루기

타입스크립트의 typeof 연산자는 typeof가 변수 혹은 속성 타입을 추론하는 역할을 하고,<br />
단독으로 사용되기보다 주로 ReturnType 같이 유틸리티 타입이나 keyof 연산자 같이 타입을 받는 연산자와 함께 쓰인다.

```ts
const colors = {
  red: "#45452",
  green: "#0C952A",
};

type ColorsType = typeof colors; // { red: string; green: string }
```

### 객체의 타입을 활용해서 컴포넌트 구현하기

keyof, typeof 연산자를 사용해서 theme 객체 타입을 구체화하고, string으로 타입을 설정했던 Button 컴포넌트를 개선해보자.

```ts
import { FC } from "react";
import styled from "styled-components";

const colors = {
  black: "#000000",
  gray: "#222222",
  white: "#FFFFFF",
  mint: "#2AC1BC",
};

const theme = {
  colors: {
    default: colors.gray,
    ...colors,
  },
  backgroundColor: {
    // ...
  },
};

const colors = {
  default: colors.white,
  gray: colors.gray,
  mint: colors.mint,
  black: colors.black,
};

const fontSize = {
  default: "16px",
  small: "14px",
  large: "18px",
};

type ColorType = keyof typeof colors;
type BackgroundColorType = keyof typeof theme.backgroundColor;
type FontSizeType = keyof typeof fontSize;

interface Props {
  color?: ColorType;
  backgroundColor?: BackgroundColorType;
  fontSize?: FontSizeType;
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void | Promise<void>;
}

const Button: FC<Props> = ({ fontSize, backgroundColor, color, children }) => {
  return (
    <ButtonWrap
      fontSize={fontSize}
      backgroundColor={backgroundColor}
      color={color}
    >
      {children}
    </ButtonWrap>
  );
};

const ButtonWrap = styled.button<Omit<Props, "onClick">>`
  color: ${({ color }) => theme.colors[color ?? "default"]};
  background-color: ${({ backgroundColor }) =>
    theme.backgroundColor[backgroundColor ?? "default"]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize ?? "default"]};
`;
```

> 여러 상숫값을 인자나 props로 받은 다음에 객체의 키값을 추출한 타입을 활용하면 객체에 접근할 때 타입스크립트의 도움을 받아 실수를 방지할 수 있다.
