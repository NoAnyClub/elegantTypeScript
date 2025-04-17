프로그래밍에서는 다양한 상황을 다루기 위해서 조건문을 많이 활용함 <br />
ts에서도 조건에 따라서 타입을 다르게 주어야 할 경우 `조건부 타입`을 사용할 수 있음. <br />
ts의 조건부 연산자는 js의 `삼항 연산자`와 같은 `condition ? A : B` 구조를 가짐.

<br />

### `extends`와 제네릭을 활용한 조건부 타입
---
`extends`는 ts에서 타입을 확장할 때와 타입을 조건부로 설정할 때 사용되며, 제네릭 타입에서는 ***한정자*** 역할로도 사용됨.

**`T extends U ? X : Y`** <br />
-> 해당 표현은 타입 T를 U에 할당할 수 있다면 X타입, 아니면 Y타입으로 결정됨을 의미함. <br />

예시를 살펴보자
```ts
interface Foo {}

interface Bar {}

type One<T> = T extends 'Foo' ? Foo : Bar;

const foo: One<'Foo'> = {}; // Foo 타입
const bar: One<'Bar'> = {}; // Bar 타입
```
해당 예시에서는 제네릭에 `'Foo'`가 들어오면 `Foo타입`을, 아니면 `Bar타입`으로 결정 됨.

<br />

### 조건부 타입을 사용하지 않았을 때의 문제점
---
조건부 타입을 사용하기 전에는 어떤 이슈가 있었는지 알아보자.
아래는 리액트 쿼리를 활용한 예시임. <br />
(계좌, 카드, 앱카드등 3가지 결제 수단 정보를 가져오는 API가 있으며, API의 엔드포인트는 아래와 같음)
> [!NOTE]
>  - 계좌 정보 : `www.baemin.com/baeminpay/.../bank`<br />
> - 카드 정보 : `www.baemin.com/baeminpay/.../card` <br />
> - 앱카드 정보 : `www.baemin.com/baeminpay/.../appcard`

각 API는 계좌, 카드, 앱카드의 결제 수단 정보를 배열 형태로 반환함. 3가지 API의 엔드포인트가 비슷하기 때문에 서버 응답을 처리하는 공통 함수를 생성하고, 해당 함수에 타입을 전달하여 타입별로 처리 로직을 구현해보자 <br />

```ts
interface PayMethodBaseFromRes {
  financialCode: string;
  name: string;
}

interface Bank extends PayMethodBaseFromRes {
  fullName: string;
}

interface Card extends PayMethodBaseFromRes {
  appCardType: string;
}

// PayMethodInterface: 프론트에서 관리하는 결제 수단 관련 데이터로 UI를 구현하는데 사용되는 타입.
type PayMethodInterface = {
  companyName: string;
  // ...
};

type PayMethodInfo<T extends Bank | Card> = T & PayMethodInterface;
```

이제 리액트 쿼리의 useQuery를 사용한 커스텀 훅인 `useGetRegisteredList`함수를 살펴보자.

```ts
type payMethodType = PayMethodInfo<Card> | PayMethodInfo<Bank>;

// useGetRegisteredList는 useQuery의 반환값을 돌려줌.
export const useGetRegisteredList = (
  type: 'card' | 'bank' | 'appcard'
): UseQueryResult<PayMethodType[]> => {
  const url = `baeminpay/codes/${type === 'appcard' ? 'card' : type}`;

  // fetcherFactory는 axios를 래핑해주는 함수이며, 서버에서 데이터를 받아온 후 onSuccess 콜백함수를 거친 결괏값을 반환함.
  const fetcher = fetchFactory<PayMethodType[]>({
    onSuccess: (res) => {
      const usablePocketList =
        res.filter(
          (payMethod: PayMethodInfo<Card> | PayMethodInfo<Bank>) => payMethod?.useType === 'USE'
        ) ?? [];

      return usablePayMethodList;
    }
  });

  // useCommonQuery<T>는 useQuery를 한번 래핑해서 사용하고 있는 함수로, useQuery의 반환 데이터를 T타입으로 반환함.
  const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);

  return result;
}
```
-> `useGetRegisteredList`는 타입을 구분해서 넣는 사용자의 의도와는 다르게 정확한 타입을 반환하지 못하는 함수가 되었음. <br />
이를 개선하기 위해서는 `extends를 사용한 조건부 타입`을 활용하면 됨.

<br />

### extends 조건부 타입을 활용하여 개선하기
---
`useGetRegisteredList`의 반환 데이터는 인자 타입에 따라서 정해져 있음.<br />
타입으로 `card` 또는 `appcard`를 받는다면 `PayMethodInfo<card>`를 반환하고, `bank`를 받으면, `PayMethodInfo<bank>`를 반환 함.

여기서 `PayMethodInfo<Card> | PayMethodInfo<Bank>`였던 `PayMethodType`을 개선해서 다시 작성해보자
```ts
type PayMethodInfo<T extends 'card' | 'appcard' | 'bank'>
  = T extends 'app' | 'appcard'
    ? PayMethodInfo<Card>
    : PayMethodInfo<Bank>;

export const useGetRegisteredList<T extends 'card' | 'bank' | 'appcard'> = (
  type: T
): UseQueryResult<PayMethodType<T>[]> => {
  const url = `baeminpay/codes/${type === 'appcard' ? 'card' : type}`;

  // fetcherFactory는 axios를 래핑해주는 함수이며, 서버에서 데이터를 받아온 후 onSuccess 콜백함수를 거친 결괏값을 반환함.
  const fetcher = fetchFactory<PayMethodType[]>({
    onSuccess: (res) => {
      const usablePocketList =
        res.filter(
          (payMethod: PayMethodInfo<T>) => payMethod?.useType === 'USE'
        ) ?? [];

      return usablePayMethodList;
    }
  });

  // useCommonQuery<T>는 useQuery를 한번 래핑해서 사용하고 있는 함수로, useQuery의 반환 데이터를 T타입으로 반환함.
  const result = useCommonQuery<PayMethodType<T>[]>(url, undefined, fetcher);

  return result;
}
```
-> 이제 조건부 타입을 활용해서 `PayMethodType`이 사용자가 인자에 넣는 타입 값에 맞는 타입만을 반환하도록 구현했음. <br/>
이제 type에 `'card'` 또는 `'appcard'`를 넣으면 `PayMethodInfo<Card>`를, `'bank'`를 넣으면 `PayMethodInfo<Bank>`를 리턴함.

해당 절에서는 타입 확장이 아닌, `extends`의 활용 사례를 살펴보았음. 정리하자면 아래와 같음.
- `제네릭과 extends를 함께 사용해서 제네릭으로 받는 타입을 제한함. 이는 휴먼에러를 방지할 수 있음.`
- `extends를 사용해서 조건부 타입을 설정함. 이는 불필요한 타입가드, 타입 단언을 방지할 수 있음.`

<br />

### infer를 활용해서 타입 추론하기
---
`extends`를 사용할때는 `infer`키워드를 사용할 수 있음.
`infer`는 사전적으로 ***'추론하다'*** 라는 의미를 지니고 있는데 이처럼 ts에서도 타입을 추론하는 역할을 함. <br />
예시를 살펴보자

```ts
type InnerType<T extends Array<unknown>> = T extends Array<infer K> ? K : never;
const numberList = [1,2,3];
type Num = InnerType<typeof numberList>; // number 타입;

type UnpackPromise<T> = T extends Promise<infer K>[] ? K : any;
const promiseList = [Promise.resolve('string'), Promise.resolve(123)];
type Expected = UnpackPromise<typeof promiseList>; // string | number 타입
```

이처럼 `extends`와 `infer`, `제네릭`을 활용하면 타입을 조건에 따라 더 세밀하게 사용할 수 있음. <br />
이번에는 배민에서 사용하는 예시를 살펴보자

```ts
interface RouteBase {
  name: string;
  path: string;
  component: ComponentType;
}

export interface RouteItem {
  name: string;
  path: string;
  component?: ComponentType;
  pages?: routeBase[];
}

export const routes: RouteItem[] = [
  {
    name: '기기내역 관리',
    path: '/device-history',
    component: DeviceHistoryPage,
  },
  {
    name: '헬멧 인증 관리',
    path: '/helmet-certification',
    component: HelmetCertificationPage,
  },
  //...
]
```
RouteBase와 RouteItem은 라이더 어드민에서 라우팅을 위해 사용하는 타입임. routes같이 배열 형태로 사용되며, 권한 API로 반환된 사용자 권한과, name을 비교해서, 인가되지 않은 사용자의 접근을 방지함. <br />
_`RouteItem`의 name은 pages가 있을때는 단순 이름의 역할만 하지만, 그렇지 않을 경우에는 사용자 권한과 비교함._

> [!NOTE]
> **라우팅** <br />
> 웹 애플리케이션에서 사용자가 URL을 통해 다른 페이지로 이동하거나, 다른 경로에 대한 요청을 처리하는 방법을 정의해놓은 것.

<br />

```ts
interface SubMenu {
    name: string;
    path: string;
}

interface MainMenu {
    name: string;
    path?: string;
    subMenus?: SubMenu[];
}

type MenuItem = MainMenu | SubMenu;
const menuList: MenuItem[] = [
    {
        name: '계정 관리',
        subMenus: [
            {
                name: '기기 내역 관리',
                path: '/devide-history'
            },
            {
                name: '헬멧 인증 관리',
                path: '/helmet-certification',
            }
        ],
    },
    {
        name: '운행 관리',
        path: '/operation',
        // ...
    }
];
```

MainMenu와 SubMenu은 메뉴 리스트에서 사용하는 타입으로, 권한 API로 반환된 사용자 권한과, name을 비교해서 사용자가 접근할 수 있는 메뉴만 렌더링 함.<br />
_`MainMenu`의 name은 `SubMenus`가 있을때는 단순 이름의 역할만 하지만, 그렇지 않을 경우에는 사용자 권한과 비교함.(권한으로 간주 됨)_ <br />

menuList에는 `MainMenu`와 `SubMenu`타입이 올 수 있기 때문에 유니언 타입인 MenuItem을 정의해서 사용중임. <br />
따라서 menuList에서 subMenus가 없는 MainMenu의 name과 subMenus에서 쓰이는 name, route name에 **동일한 문자열만 입력해야 한다**는 제약이 존재함.

즉, 권한 문자열이 모두 동일하게 들어 있어야 한다는 것임. <br />
예를 들어, 이런 구조가 있다고 치자
```ts
routes = [
  { name: '기기내역 관리', path: '/device-history' }
];

menuList = [
  {
    name: '계정 관리',
    subMenus: [
      { name: '기기 내역 관리', path: '/device-history' } // 띄어쓰기 다름
    ]
  }
];

userPermissions = ['기기 내역 관리']; // 권한 API로부터 받은 권한 이름
```
이 경우 `routes[].name`은 `"기기내역 관리"`, `subMenus[].name`은 `"기기 내역 관리"` <br />
→ 문자열이 다르기 때문에 비교에 실패할 수 있음

***즉, 권한 이름과 비교되는 대상이 3군데에 흩어져 있어서 이름이 완전히 일치해야만 인가 판단이 정확해짐.***

하지만, 모두 string타입이기 때문에 다른 문자열이 들어가도 컴파일 타임에서 에러가 발생하지 않음. <br />
이를 개선하기 위해서 `PermissionType`같은 타입을 별도로 선언해서 관리하는 방법도 있지만, 권한 검사가 필요없는 subMenus나 pages가 존재하는 name은 별도로 처리해주어야 함.
```ts
type PermissionType = '기기 내역 관리' | '헬멧 인증 관리' | '운행 여부 조회'; //...
```

이때 `infer`와 `불변 객체`를 활용해서 menuList또는 routes의 값을 추출하여 타입으로 정의하는 식으로 개선할 수 있음.
```ts
type UnpackMenuNames<T extends ReadOnlyArray<MenuItem>>
    = T extends ReadOnlyArray<infer U>
        ? U extends MainMenu
            ? U['subMenus'] extends infer V
                ? V extends ReadOnlyArray<SubMenu>
                    ? UnpackMenuNames<V>
                    : U['name']
                : never
            : U extends SubMenu
                ? U['name']
                : never
            : never;
```
-> 타입 추론을 계속 하면서 권한 네이밍을 찾는 커스텀 유틸리티 타입임.

- U가 MainItem이라면 subMeus를 V로 추출함
- subMenus는 옵셔널이기 때문에 V가 존재한다면 UnpackMenuNames에 다시 전달함(재귀)
- 없다면 MainMenu의 names은 권한이므로 `U['name']`을 return.
- U가 MainMenu가 아니라면(SubMenu) `U['name']`은 권한에 해당함.

따라서 아래와 같이 개선이 가능함
```ts
interface ComponentType {}

type ReadOnlyArray<T> = Array<{
    readonly [K in keyof T]: T[K];
}>;

interface SubMenu {
    name: string;
    path: string;
}

interface MainMenu {
    name: string;
    path?: string;
    subMenus?: ReadOnlyArray<SubMenu>;
}

type MenuItem = MainMenu | SubMenu;

const menuList: ReadOnlyArray<MenuItem> = [
    {
        name: '계정 관리',
        subMenus: [
            {
                name: '기기 내역 관리',
                path: '/devide-history'
            },
            {
                name: '헬멧 인증 관리',
                path: '/helmet-certification',
            }
        ],
    },
    {
        name: '운행 관리',
        path: '/operation',
    }
] as const;

interface RouteBase {
    name: PermissionNames;
    path: string;
    component: ComponentType;
}

type UnpackMenuNames<T extends ReadOnlyArray<MenuItem>>
    = T extends ReadOnlyArray<infer U>
        ? U extends MainMenu
            ? U['subMenus'] extends infer V
                ? V extends ReadOnlyArray<SubMenu>
                    ? UnpackMenuNames<V>
                    : U['name']
                : never
            : U extends SubMenu
                ? U['name']
                : never
            : never;

type PermissionNames = UnpackMenuNames<typeof menuList>; // menuList를 기반으로 권한 이름들 추출

type RouteItem =
    | {
        name: string;
        path: string;
        component?: ComponentType;
        pages: RouteBase[];
    }
    | {
        name: PermissionNames; // 추출한 권한 이름들 할당
        path: string;
        component?: ComponentType;
    };
```


