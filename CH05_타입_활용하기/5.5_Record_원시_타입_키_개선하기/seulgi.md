# 5.Record 원시 타입 키 개선하기

객체 선언 시 키가 어떤 값이 명확하지 않다면 Record의 키를 string이나 number 값은 원시 타입으로 명시하곤 한다.
이때 타입스크립트는 키가 유효하지 않더라도 타입상으로는 문제가 없기 때문에 오류를 표시하지 않는다.

Record를 명시적으로 사용하는 방안에 대해 알아보자.

## 1) 무한한 키를 집합으로 가지는 Record

다양한 음식 분류를 키로 사용하는 음식 배열이 담긴 객체를 만들었다고 보자.
key의 타입이 string일 경우 아래와 같은 문제가 있다.

```ts
type Category = string;

interface Food {
  name: string;
  // ...
}

const foodByCategory: Record<Category, Food[]> = {
  한식: [{ name: "제육 덮밥" }, { name: "뚝배기 불고기" }],
  일식: [{ name: "초밥" }, { name: "텐동" }],
};

foodByCategory["양식"]; // Food[]로 추론

foodByCategory["양식"].map((food) => console.log(food.name)); // 오류가 발생하지 않는다.
```

그러나, foodByCategory["양식"]은 런타임에서 undefined가 되어 오류를 반환한다.

```ts
foodByCategory["양식"].map((food) => console.log(food.name)); // Uncaught TypeError:
// Cannot read properties of undefined (reading 'map')

// 이때 자바스크립트의 옵셔널 체이닝 등을 사용해 런타임 에러를 방지할 수 있다.
foodByCategory["양식"]?.map((food) => console.log(food.name));
```

- 어떤 값이 undefined인지 매번 판단해야하는 번거로움이 있을 수 있다.
- 실수로 undefined일 수 있는 값을 인지하지 못하고 작성 시 런타임 에러가 발생할 수 있다.

> 이럴 때는 개발 중에 유효하지 않은 키가 사용되었는 지 또는 undefined일 수 있는 값이 있는 지 등 사전에 타입스크립트 기능을 활용해 파악하는 것이 중요하다.

> **옵셔널 체이닝(optional chaining)?** > <br />객체의 속성을 찾을 때 중간에 null 또는 undefined가 있어도 오류 없이 안전하게 접근하는 방법이다. <br />
> ?. 문법으로 표현되며 옵셔널 체이닝을 사용할 때 중간에 null 또는 undefined인 속성이 있는지 검사한다. <br />
> 속성이 존재하면 해당 값을 반환하고, 존재하지 않으면 undefined를 반환한다.

<br />
<br />

## 2) 유닛 타입으로 변경하기

키가 유한한 집합이라면 유닛 타입(다른 타입 쪼개기가 아닌 오직 하나의 적확한 값을 가지는 타입)을 사용할 수 있다.
(키가 무한해야 하는 상황에는 적합하지 않음)

```ts
type Category = "한식" | "중식";

interface Food {
  name: string;
  // ...
}

const foodByCategory: Record<Category, Food[]> = {
  한식: [{ name: "제육 덮밥" }, { name: "뚝배기 불고기" }],
  일식: [{ name: "초밥" }, { name: "텐동" }],
};

foodByCategory["양식"]; // 존재하지 않는 타입이라는 에러 발생!
```

<br />
<br />

## 3) Partial을 활용해 정확한 타입 표현하기

키가 무한한 상황에서는 Partial을 활용해 해당 값이 undefined일 수 있는 상태임을 표현할 수 있다.

```ts
type PartialRecord<K extends string, T> = Partial<Record<K, T>>;

type Category = string;

interface Food {
  name: string;
  // ...
}

// ...

const foodByCategory: PartialRecord<Category, Food[]> = {
  한식: [{ name: "제육 덮밥" }, { name: "뚝배기 불고기" }],
  일식: [{ name: "초밥" }, { name: "텐동" }],
};

foodByCategory["양식"]; // Food[] 또는 undefined 타입으로 추론
foodByCategory["양식"].map((food) => console.log(food.name)); // Object is possibly 'undefined'
foodByCategory["양식"]?.map((food) => console.log(food.name)); // OK
```

- 타입스크립트는 Food[] 또는 undefined 타입으로 추론해 개발자에게 이 값은 undefined값일 수 있으니 해당 값에 대한 처리가 필요하다고 표시해준다.
- 개발자는 안내를 보고 옵셔널 체이닝 사용이나 조건문을 사용해 런타임 오류를 줄일 수 있다.
