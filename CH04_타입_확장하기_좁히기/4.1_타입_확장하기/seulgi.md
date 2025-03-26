# 타입 확장하기

타입 확장은 기존 타입을 사용해서 새로운 타입을 정의하는 것이다.

interface, type을 통해 타입을 정의하고, extends, 교차 타입, 유니온 타입을 사용해 타입을 확장한다.

## 1) 타입 확장의 장점

타입 확장의 장점은 코드 중복을 줄일 수 있다는 것이다. <br />
중복되는 타입을 반복적으로 선언하는 것보다 기존에 작성한 타입을 바탕으로 타입 확장을 함으로써 불필요한 코드 중복을 줄일 수 있다.

```ts
interface MenuItem {
  name: string | null;
  imageUrl: string | null;
  discount: number;
  stock: number | null;
}

interface CartItem extends MenuItem {
  quantity: number;
}
```

> 위의 코드는 메뉴 타입 기준으로 타입 확장을 해서 장바구니 타입을 정의한 것이다. <br />
> 장바구니 요소는 메뉴 요소가 가지는 모든 타입이 필요하다. 해당 코드는 확장을 통해 중복 코드를 줄여준다. <br />
> 장바구니 요소가 메뉴 요소의 확장되었다는 것이 쉽게 확인되어 좀 더 명시적인 코드를 작성할 수 있게 해준다.

### interface를 type으로 바꿔보자.

```ts
type MenuItem = {
  name: string | null;
  imageUrl: string | null;
  discount: number;
  stock: number | null;
};

type CartItem = MenuItem & {
  quantity: number;
};
```

> 타입 확장은 중복 제거, 명시적 코드 작성 외에도 확장성이란 장점이 있다.

### 요구 사항이 계속 늘어난다면 새로운 타입을 확장해 정의할 수 있다.

수정할 수 있는 장바구니 요소 타입으로 "품절 여부, 수정 옵션 배열 정보"가 추가되었다고 가정하자.

```ts
interface EditableCartItem extends CartItem {
  isSoldOut: boolean;
  optionGroups: SelectableOptionGroup[];
}

interface EventCartItem extends CartItem {
  orderable: boolean;
}
```

> 위의 코드처럼 장바구니와 관련된 요구 사항이 생길 때마다 필요한 타입을 손쉽게 만들 수 있다. <br />
> 기존 장바구니 요소에 대한 요구 사항이 바뀌어도 CartItem 타입만 수정하면 된다.(효울적 👏)

<br />
<br />

## 2) 유니온 타입

유니온 타입은 집합 관점에서 본다면 합집합이다.

```ts
type MyUnion = A | B;
```

- A와 B의 유니온 타입인 MyUnion은 A, B의 합집합이다.
- 집합 A의 모든 원소는 집합 MyUnion의 원소이며 집합 B 모든 원소 역시 MyUnion의 원소라는 뜻이다.
- 다만, **유니온 타입으로 선언된 값은 유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성에만 접근할 수 있다.**

```ts
interface Order {
  orderId: string;
  price: number;
}

interface Delivery {
  orderId: string;
  time: number;
  distance: string;
}

function getDeliveryDistance(step: Order | Delivery) {
  return step.distance;
} // Property 'distance' does not exist on type 'Order | Delivery'. Property 'distance' does not exist on type 'Order'.
```

> 함수 본문에서 step.distance를 호출하고 있는데 distance는 Delivery에만 있는 속성이기 때문에 <br />
> step이 Order일 경우엔 해당 속성을 찾을 수 없어 에러가 발생한다. <br />
> 즉, step이라는 유니온 타입은 두 타입에 해당할 뿐이지 Order이면서 Delivery인 것은 아니다.

### 타입스크립트 타입을 속성의 집합이 아닌 값의 집합으로 생각해야 <br /> 유니온 타입이 합집합이라는 개념을 이해할 수 있다.

```ts
type A = "A" | "B";
type B = "B" | "C";

type AB = A | B; // "A" | "B" | "C";

type Cat = { type: "cat"; meow: () => void };
type Dog = { type: "dog"; bark: () => void };
type Animal = Cat | Dog; // { type: "cat"; meow: () => void } 또는 { type: "dog"; bark: () => void };
```

> 아래 잘못된 예시를 보자!

```ts
type Cat = { type: "cat"; sound: () => void };
type Dog = { type: "dog"; sound: string };
type Animal = Cat | Dog;

const animal: Animal = { type: "cat", sound: () => console.log("meow") };
const animal2: Animal = { type: "cat", sound: "bark!" }; // Type '{ type: "cat"; sound: string; }' is not assignable to type 'Animal'. Types of property 'sound' are incompatible. Type 'string' is not assignable to type '() => void'.
const animal3: Animal = { type: "dog", sound: () => console.log("bark!") }; // Type '{ type: "dog"; sound: () => void; }' is not assignable to type 'Animal'. Types of property 'sound' are incompatible. Type '() => void' is not assignable to type 'string'.
```

- 위의 예시처럼 Cat과 Dog 두 타입에 있는 sound 속성의 타입은 () => void | string처럼 자동으로 변환되지 않는다.
- 유니온 타입에서는 각 속성이 완전히 일치해야 한다.

<br />
<br />

## 3) 교차 타입

교차 타입도 기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입을 만드는 것으로 이해할 수 있다.

```ts
interface Order {
  orderId: string;
  price: number;
}

interface Delivery {
  orderId: string;
  time: number;
  distance: string;
}

type Progress = Order & Delivery;

function getProgressInfo(progress: Progress) {
  console.log(progress.price);
  console.log(progress.distance);
}
```

> 유니온 타입과 다른 점으로는 Progress 타입은 Order과 Delivery 타입을 합쳐 모든 속성을 가진 단일 타입이 된다. 따라서, progress 값은 Order과 Delivery 타입의 속성을 포함하고 있다.

### 교차 타입의 개념을 다시 짚고 넘어가자.

```ts
type MyIntersection = A & B;
```

- 교차 타입은 교집합의 개념과 비슷하다.
- MyIntersection 타입의 모든 값은 A 타입 값 + B 타입 값

> 집합의 관점에서 보면 MyIntersection의 모든 원소는 집합 A의 원소이자 집합 B의 원소이다.

### 다른 예시를 살펴보자.

```ts
interface Dog {
  type: "dog";
}
interface Cat {
  sound: () => void;
}
type Animal = Cat & Dog;

const animal: Animal = { type: "dog", sound: () => console.log("bark") };
```

- 교차 타입은 두 타입의 교집합을 의미하는데 Dog, Cat 타입에 공통 속성이 없어도 Animal 타입은 공집합(never)가 아닌 모두 포함한 타입이다.
- 교차 타입 Animal은 Dog의 type 속성과 Cat의 sound 속성을 모두 만족하는 값이 된다.

> 왜냐하면, 타입이 속성이 아닌 값의 집합으로 해석되기 때문이다. 다만 교차 타입을 사용할 때 타입이 서로 호환되지 않는 경우도 있다.

```ts
type MyId = string | number;
type MyNumber = number | boolean;

type MyInfo = MyId & MyNumber;
```

### 위의 MyInfo 타입을 뭘까?

1. string이면서 number
2. string이면서 boolean
3. number이면서 number
4. number이면서 boolean

> MyInfo 타입은 MyId & MyNumber 교차 타입이므로 두 타입을 모두 만족하는 경우만 유지된다. 따라서 3번인 number 타입이 된다.

<br />
<br />

## 4) extends와 교차 타입

extends 키워드를 사용해서 교차 타입을 작성할 수 있다.

```ts
type MenuItem = {
  name: string | null;
  imageUrl: string | null;
  discount: number;
  stock: number | null;
};

type CartItem = MenuItem & {
  quantity: number;
};
```

- 유니온 타입과 교차 타입을 사용한 새로운 타입은 오직 type 키워드로만 선언할 수 있다.

### extends 키워드를 사용한 타입이 교차 타입과 100% 상응하지 않는다?!

```ts
interface A {
  a: string;
}

interface B extends A {
  a: number;
}

// Interface 'B' incorrectly extends interface 'A'.
// Types of property 'a' are incompatible.
// Type 'number' is not assignable to type 'string'.
```

> extends 키워드로 확장한 B타입에 number 타입의 a 속성을 선언하면 a 타입이 호환되지 않는다는 에러가 발생한다.

```ts
type A = {
  a: string;
};

type B = A & {
  a: number;
};
```

- 에러는 발생하지 않지만, a속성의 타입은 never다.
- type 키워드는 교차 타입으로 선언되면 새롭게 추가되는 속성에 대해 미리 알 수가 없다.

> 그래서, 에러는 발생하지 않지만 같은 속성에 대해 서로 호횐되지 않은 타입이라 never 타입이 된다.

<br />
<br />

## 5) 배달의 민족 메뉴 시스템에 타입 확장 적용하기

```ts
// 메뉴에 대한 타입 : 메뉴명, 이미지 정보가 있다.
interface Menu {
  name: string;
  image: string;
}

function MainMenu() {
  const menuList: Menu[] = [{ name: "중식", image: "중식.png" }, ...];

  return (
    <ul>
        {menuList.map((menu) =>
        <li>
        <img src={menu.image} />
        <span>{menu.name}</span>
        </li>)}
    </ul>
  )
}
```

### 특정 메뉴의 중요도를 다르게 주기 위한 요구 사항이 추가되었다면?

1. 특정 메뉴를 누르면 gif 재생되어야 한다.
2. 특정 메뉴 이미지 대신 텍스트만 노출되어야 한다.

```ts
// 방법 1: 타입 내 속성을 추가한다.
interface Menu {
  name: string;
  image: string;
  gif?: string;
  text?: string;
}

// 방법 2: 타입을 확장한다.
interface SpecialMenu extends Menu {
  gif: string;
}
interface TextMenu extends Menu {
  text: string;
}
```

### 방법 1: 하나의 타입에 여러 속성을 추가할 때

```ts
const menuList: Menu[] = [...];
const specialMenuList: Menu[] = [...];
const textMenuList: Menu[] = [...];
```

- 각 메뉴 목록을 모두 Menu[]로 표현할 수 있다.
- 다만, specialMenuList 배열의 원소가 각 속성에 접근할 때 아래와 같은 문제가 생길 수 있다.

```ts
specialMenuList.map((menu) => menu.text); // TypeError!
```

- specialMenuList는 Menu 타입의 원소를 갖기 때문에 text 속성에 접근할 수 있지만, text 속성을 가지지 않아 에러가 난다.

### 방법 2: 타입을 확장하는 방식

각 배열의 타입을 확장할 타입에 맞게 명확히 규정할 수 있다.

```ts
const menuList: Menu[] = [...];
const specialMenuList: SpecialMenu[] = [...];
const textMenuList: TextMenu[] = [...];

specialMenuList.map((menu) => menu.text); // Property "text" does not exist on type "SpecialMenu"
```

- specialMenuList는 text 속성 자체가 없어 바로 에러를 알 수 있다.

> 결과적으로 주어진 타입에 무분별하게 속성을 추가하여 사용하는 것보다 타입을 확장해서 사용하는 것이 좋다.
> 위의 예시처럼 적절한 네이밍으로 사용해 타입 의도를 명확히 할 수 있고, 코드 작성 단계에서 예기치 못한 버그 예방도 가능하다.
