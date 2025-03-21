# 자바스크립트의 한계

## 1) 동적 타입 언어

자바스크립트는 동적 타입 언어라 코드가 실행되는 런타임에 변숫값이 할당될 때 해당 값의 타입에 따라 변수 타입이 결정된다.

<br />
<br />

## 2) 동적 타이핑 시스템의 한계

```js
const sumNumber = (a, b) => {
  return a + b;
};

sumNumber(100); // NaN;
sumNumber("a", "b"); // "ab"
```

위의 예시처럼 숫자를 계산하려는 함수로 의도했지만 의도와 다르게 동작할 수 있는 문제가 있다. 동적 타입 언어라는 특성으로 함수를 호출할 때 사용되는 인수 값에 따라 a,b 타입이 결정되기 때문이다.

자바스크립트 엔진에서는 주석, 함수 이름, 개발자의 의도는 고려 대상이 아니다.

<br />
<br />

## 3) 한계 극복을 위한 해결 방안

자바스크립트 인터페이스의 필요성을 느끼고, JSDoc, propTypes, 다트 같은 해결 방안이 등장했다.

### JSDoc

- 모듈, 네임스페이스, 클래스, 메서드, 매개변수 등에 대한 API 문서 생성 도구다.
- 주석에 @ts-check를 추가하면 타입 및 에러 확인이 가능하다.
- 자바스크립트 소스코드에 타입 힌트를 제공하는 HTML 문서를 생성할 수 있다.
- 주석 사용이기 때문에 강제성을 부여하긴 어렵다.

### propTypes

- 리액트에서 props 타입 검사용으로 사용하는 속성이다.
- prop에 유효한 값이 전달되었는지 확인 가능한데, 전체 타입 검사는 어렵다.
- 리액트라는 특정 라이브러리만 사용 가능하다.

### 다트

- 구글에서 자바스크립트 대체를 위한 새로운 언어이다.
- 타이핑이 가능하나, 새로운 언어라는 단점이 있다.

<br />
<br />

## 4) 타입스크립트의 등장

마이크로소프트는 자바스크립트의 슈퍼셋 언어인 타입스립트를 공개했고 아래와 같은 장점을 지녀 많은 환영을 받았다.

> [!NOTE]
> **슈퍼셋(Superset)?** <br /> 기존 언어에 새로운 기능과 문법을 추가해서 보완하거나 향상하는 것. 슈퍼셋 언어는 기존 언어와 호환되며 일반적으로 컴파일러 등으로 기존 언어 코드로 변환되어 실행된다.

### (1) 안정성 보장

- 타입스크립트는 정적 타이핑을 제공해 컴파일 단계에서 타입 검사를 해줘 자바스크립트의 빈번한 타입 에러를 줄여준다.
- 런타임 에러를 사전에 방지해 안정성을 보장해준다.

### (2) 개발 생산성 향상

- VSCode 등의 IDE에서 타입 자동 완성 기능을 제공한다.
- 변수와 함수 타입을 추론하고, 리액트 사용 시 어떤 prop을 넘겨야 하는지 매번 확인하지 않아도 되는 등 개발 생산성이 향상된다.

> [!NOTE]
> **IDE (Integrated Development Environment, 통합 개발 환경)란?** <br /> 코드를 작성하고, 실행하고, 디버깅하는 모든 작업을 하나의 프로그램에서 할 수 있도록 도와주는 개발 도구. 쉽게 말해서 코딩에 필요한 모든 기능이 한곳에 모여 있는 소프트웨어이다.

### (3) 협업에 유리

- 인터페이스, 제네릭 등으로 인터페이스가 기술되면 코드를 쉽게 이해할 수 있다.
- 자동 완성 기능이나 기술된 인터페이스로 코드를 쉽게 파악 가능하다.

> [!NOTE]
> **타입스크립트 인터페이스?** <br /> 객체 구조를 정의한다. 특정 객체가 가져야 하는 속성과 메서드 집합을 인터페이스로 정의해 객체가 그 구조를 따르게 한다.

### (4) 자바스크립트에 점진적으로 적용 가능

- 슈퍼셋 언어이기 때문에 점진적 도입이 가능하다.
