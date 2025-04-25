# 1. API 요청

## 1) fetch로 API 요청하기

장바구니 조회 기능을 만들기 위해 외부 데이터베이스에 접근해
사용자가 장바구니에 추가한 정보를 호출하는 코드를 작성한다고 해보자.

아래는 fetch 함수를 이용해 사용자가 담은 장바구니 물품 개수를 배지로 만든 예시이다.

```tsx
const CartBadge = () => {
    const [cartCount, setCartCount] = useState(0);

    useEffect(() => {
        fetch("").then.((response) => response.json()).then(({cartItem} => {
            setCartCount(cartItem.length);
        }))
    },[]);

    return (
        // 렌더링 로직
    );
}
```

위의 경우처럼 구현해 다른 컴포넌트에서도 사용할 경우에<br />

백엔드에서 기능 변경을 해야해 API URL을 수정해야 한다면,<br />
이미 컴포넌트 내부 깊숙이 자리 잡은 비동기 호출 코드는 이런 변경 요구에 취약하다.<br />
<br />
그 외에도 커스텀 헤더, 타임아웃 설정과 같은 정책이 추가될 때마다 수정의 번거로움이 생긴다.

<br />
<br />

## 2) 서비스 레이어로 분리하기

여러 API 변경으로 코드가 변경될 수 있다는 걸 감안해,<br />
비동기 호출 코드는 컴포넌트 영역에서 분리되 서비스 레이어에서 처리되어야 한다.

단순히, fetch 함수 호출 부분을 서비스 레이어로,<br />
컴포넌트는 서비스 레이어 비동기 함수를 호출해 그 결과를 받아<br />
렌더링하는 흐름만으로는 API 요청 정책이 추가되는 것을 해결하기 어렵다.

예를 들어, fetch함수에서 타임아웃을 설정하기 위해서는 다음 같이 구현해야 한다.

```tsx
async function fetchCart() {
  const controller = new AbortController();

  const timeoutId = setTimeout(() => controller.abort(), 5000);

  const response = await fetch("url", {
    signal: controller.signal,
  });

  clearTimeout(timeoutId);

  return response;
}
```

<br />
<br />

## 3) Axios 활용하기

fetch는 내장 라이브러리라 설치 필요는 없지만, 많은 기능을 사용하려면 직접 구현해야 한다.
이런 번거로움 때문에 fetch 함수를 직접 쓰는 대신 Axios 라이브러리를 사용한다.

```tsx
const apiRequester: AxiosInstance = axios.create({
  baseURL: "url",
  timeout: 5000,
});

const fetchCart = (): AxiosPromise<FetchCartResponse> =>
  apiRequester.get<FetchCartResponse>("cart");

const postCart = (
  postCartRequest: PostCartRequest
): AxiosPromise<FetchCartResponse> =>
  apiRequester.post<PostCartResponse>("cart", postCartRequest);
```

### API Entry가 2개 이상일 경우

```tsx
const apiRequester: AxiosInstance = axios.create(defaultConfig);

const orderApiRequester: AxiosInstance = axios.create({
  baseURL: "url",
  ...defaultConfig,
});
const orderCartApiRequester: AxiosInstance = axios.create({
  baseURL: "url",
  ...defaultConfig,
});
```

각 서버의 기본 URL을 호출하도록 orderApiRequester, orderCartApiRequester 같이<br />
2개 이상의 API 요청을 처리하는 인스턴스를 따로 구성해야 한다.

이후, 다른 URL로 서비스 코드를 호출할 때는 각각의 apiRequester를 사용하면 된다.

<br />
<br />

## 4) Axios 인터셉터 사용하기

서로 다른 역할을 담당하는 다른 서버이기 때문에
requester별로 다른 헤더를 설정해줘야 하는 로직이 필요할 수 있다.

이때, 인터셉터 기능을 사용하여 requester에 따라 비동기 호출 내용을 추가해서 처리할 수 있다.<br />
또한 API 에러를 처리할 때 하나의 에러 객체로 묶어서 처리할 수도 있다.

이와 달리 요청 옵션에 따라 다른 인터셉터를 만들기 위해 빌더 패턴을 추가하여
APIBuilder 같은 클래스 형태로 구성하기도 한다.

> **빌더 패턴(Builder Pattern)?** <br />
> 객체 생성을 더 편리하고 가독성 있게 만들기 위한 디자인 패턴 중 하나다. <br />
> 주로 복잡한 객체의 생성을 단순화하고, 객체 생성 과정을 분리하여 객체를 조립하는 방법을 제공한다.

다만, 이런 APIBuilder 클래스는 보일러플레이트 코드가 많다는 단점을 갖고 있기 때문에<br />
옵션이 다양한 경우 인터셉터를 설정값에 따라 적용하고,<br />
필요 없는 인터셉터를 선택적으로 사용할 수 있다는 장점도 갖고 있다.

<br />
<br />

## 5) API 응답 타입 지정하기

API 응답 값은 하나의 Response 타입으로 묶일 수 있다.

```ts
interface Response<T> {
  data: T;
  status: string;
  serverDataTime: string;
  errorCord?: string;
  errorMessage?: string;
}

const fetchCart = (): AxiosPromise<FetchCartResponse> =>
  apiRequester.get < Response < FetchCartResponse >> "cart";

const postCart = (
  postCartRequest: PostCartRequest
): AxiosPromise<FetchCartResponse> =>
  apiRequester.post<Response<PostCartResponse>>("cart", postCartRequest);
```

이와 같이 서버에서 오는 응답을 통일할 때 주의할 점은<br />
Response 타입을 apiRequester 내에서 처리한다면<br />
UPDATE, CREATE 같이 응답이 없을 수 있는 API 처리가 까다로워질 수 있다.

따라서, <u>Response 타입은 apiRequester가 모르게 관리</u>되어야 하고,<br />
해당 값에 어떤 응답이 들어있는지 알 수 없거나 값의 형식이 달라지더라도<br />
<u>로직에 영향을 주지 않는 경우에는 unknown 타입을 사용하여 알 수 없는 값임을 표현</u>한다.

```ts
interface response {
  data: {
    cartItems: CartItem[];
    forPass: unknown;
  };
}
```

다만, 이미 설계된 프로덕트에서 쓰고 있는 값이라면
프로트 로직에서 써야 하는 값에 대해서만 타입을 선언한 다음에 사용하는 것이 좋다.

```ts
type ForPass = {
  type: "A" | "B" | "C";
};

const isTargetValue = () => (data.forPass as ForPass).type === "A";
```

<br />
<br />

## 6) 뷰 모델 사용하기

새로운 프로젝트는 서버 스펙이 자주 바뀌기 때문에
뷰 모델을 사용하여 API 변경에 따른 범위를 한정해줘야 한다.

```tsx
interface ListResponse {
  items: ListItem[];
}

const fetchList = async (filter?: ListFetchFilter): Promise<ListResponse> => {
  const { data } = await api
    .params({ filter })
    .get("/api/get-list")
    .call<Response<ListResponse>>();

  return { data };
};

const ListPage = () => {
  const [itemCount, setItemCount] = useState(0);
  const [items, setItems] = useState<ListItem[]>([]);

  useEffect(() => {
    fetchList(filter).then(({items}) => {
        setItemCount(items.length);
        setItems(items);
    })
  },[]);

  return (
    // 렌더링 로직
  );
};
```

흔히 좋은 컴포넌트는 변경될 이유가 하나뿐인 컴퍼넌트라 한다.

API응답의 items 인자를 좀 더 정확한 개념으로 나타내기 위해<br />
각 역할에 맞는 컴포넌트명으로 바꾼다면 해당 컴포넌트도 수정해야 하는데<br />
같은 API를 사용하는 기존 컴포넌트들 모두 수정해야하는 문제가 생긴다.

이 문제를 해결하기 위한 방법으로 뷰 모델을 도입할 수 있다.

API 응답에는 없는 ItemCount와 같은 도메인 개념을 넣을 때도<br />
백엔드나 UI에서 로직을 추가하여 처리할 필요 없이 간편하게 새로운 필드를 뷰 모델에 추가할 수 있다.

### 뷰 모델에도 문제는 있다?!

추상화 레이어 추가는 결국 코드를 복잡하게 만들며 레이어를 관리하고 개발하는 비용이 든다.

```tsx
interface JobListResponse {
  jobItems: JobListItemResponse[];
}

class JobListItem {
  constructor(item: JobListItemResponse) {
    //  JobListItemResponse => JobListItem 객체로 변환
  }
}

class JobList {
  readonly itemCount: number;
  readonly items: JobListItemResponse[];

  constructor({ jobItems }: JobListResponse) {
    this.itemCount = jobItem.length;
    this.items = jobItems.map((item) => new JobListItem(item));
  }
}

const fetchJobList = async (
  filter?: ListFetchFilter
): Promise<JobListResponse> => {
  const { data } = await api
    .params({ filter })
    .get("/api/get-list")
    .call<Response<ListResponse>>();

  return new JobList(data);
};
```

### 도메인의 일관성 + 코드 수정 비용의 절충안을 찾자!

1. 필요한 곳에만 뷰 모델을 부분적으로 만들어서 사용하기
2. 백엔드와 클라이언트 개발자가 충분히 소통 후에 개발해 API 응답 변화 줄이기
3. 뷰 모델에 필드를 추가하기 대신에 getter 등 함수를 추가해 실제 어떤 값이 뷰 모델에 추가한 값인지 알기 쉽게 하기

### AI로 찾아본 각 예시들

1. 필요한 곳에만 뷰 모델(ViewModel)을 부분적으로 만들어서 사용하기

- 전체 도메인 객체를 뷰에 억지로 끼워 맞추기보다는, 필요한 정보만 추려서 가볍게 사용하자.

```ts
// 전체 도메인 모델
interface User {
  id: number;
  name: string;
  email: string;
  birthday: string;
  createdAt: string;
  updatedAt: string;
}

// 뷰 모델 (뷰에서 필요한 정보만 선택)
interface UserProfileViewModel {
  name: string;
  birthday: string;
}

// 사용처에서 가볍게 매핑
const toUserProfileViewModel = (user: User): UserProfileViewModel => ({
  name: user.name,
  birthday: user.birthday,
});
```

<br />

2. API 응답 스펙의 변경을 줄이기 위해 백엔드/프론트 간 소통 강화

- 프론트와 백엔드가 협의해서 “이건 프론트에서 계산할게요” 또는 “백엔드가 계산해서 내려줄게요” 협의한다고 가정하자.

```ts
{
"price": 1000,
"discount": 100
}

// 클라이언트에서 totalPrice 계산
const totalPrice = price - discount;
```

- 백엔드에서 계산 요청하고 응답 스펙을 바꾸지 않음 (기존 유지)
- 프론트에서는 필요한 값을 뷰 모델에서 따로 계산

<br />

3. 뷰 모델에 필드를 직접 추가하지 말고 함수(getter)를 통해 표현

- 어떤 값이 기존 도메인에서 온 값인지, 아니면 UI용 가공값인지 명확히 하자.

```ts
interface Product {
  name: string;
  price: number;
  discount: number;
}

// 뷰 모델 형태로 가공
class ProductViewModel {
  constructor(private product: Product) {}

  get name() {
    return this.product.name;
  }

  get price() {
    return this.product.price;
  }

  get finalPrice() {
    return this.product.price - this.product.discount;
  }

  // UI에서만 사용하는 값
  get isOnSale() {
    return this.product.discount > 0;
  }
}
```

- 이 방식은 실제 데이터는 그대로 두고, 추가된 값들이 어떤 용도인지 명확하게 드러나고,
- finalPrice, isOnSale은 UI 전용이라는 게 getter로 구분되어 이해하기 쉽다.

> 다만, 타입스크립트는 정적 검사 도구라 런타임에 API 응답의 타입 오류를 방지하려면 Superstruct, zod같은 라이브러리를 사용하면 된다.

<br />
<br />
