### fetch로 API요청하기
---
아래와 같은 장바구니를 조회해서 볼수 있는 코드가 있다
```ts
const CartBadge = () => {
  const [cartCount, setCartCount] = useState(0);

  useEffect(() => {
    fetch('https://api.baemin.com/cart')
      .then((response) => response.json())
      .then(({ cartItem }) => setCartCount(cartItem.length));
  });

  return (
    // ...
  );
};
```
이렇게 코드를 작성했는데, 백엔드에서 API주소를 수정해야 한다고 한다.. <br/>
이렇게 이미 컴포넌트 내부에 깊숙이 자리잡은 비동기 호출 코드는 변경에 취약함. 또한 API를 요청할 경우 타임아웃 설정이 필요하거나, 커스텀 헤더가 필요한 경우가 있음.

이처럼 API요청 정책이 계속 추가될때마다 계속해서 비동기 호출 코드를 수정해야 하는 번거로움이 생김.

이러한 점을 감안한다면, 비동기 호출 코드는 따로 모듈로 분리해서(서비스 레이어로 분리해서) 작성되어야 함. <br />
그러나 레이어를 나누는 것만으로 위와같이 수많은 변화에 대응하기는 충분치 않음.

<br />

### Axios 활용하기
---
fetch는 네이티브 라이브러리이기때문에 별도의 설치없이 사용할 수 있다는 장점이 있지만, 그만큼 모든것을 직접 구현해서 사용해야 한다는 단점이 존재함. <br/>
따라서 `axios`라이브러리를 많이 사용함.

***만약 `axios`를 사용한다면***
- **여러개의 API Entry(base URL)을 사용해야 하는 경우** <br />
  ```ts
  const defaultRequester: AxiosInstance = axios.create(defaultConfig);
  const orderRequester: AxiosInstance = axios.create({
    baseUrl: ORDER_API_BASE_URL,
    ...defaultConfig
  });
  const caerRequester: AxiosInstance = axios.create({
    baseUrl: CART_API_BASE_URL,
    ...defaultConfig
  });
  ```
  요런식으로 각각의 서로 다른 baseURL을 가진 axios인스턴스를 활용할 수 있음.

- ***커스텀 헤더를 일괄적으로 추가하거나, 에러 핸들링을 한군데서 처리하고 싶을 경우*** <br />
이러한 경우에는 axios의 `interceptor`를 활용할 수 있음.
  ```ts
  const setCustomHeader = (request: AxiosRequest) => {
    // 커스텀 헤더 셋팅...
  }

  const handleErrorResponse = (response: AxiosResponse) => {
    // 응답 에러 핸들링..
  }

  const defaultRequester: AxiosInstance = axios.create(defaultConfig);

  // request interceptor
  defaultRequester.interceptors.request.use(setCustomHeader);

  // response interceptor
  defaultRequester.interceptors.response.use(handleErrorResponse);
  ```
  - `request interceptor`를 설정해주면 해당 인스턴스로 요청하는 모든 비동기 호출이 해당 setCustomHeader를 거친 뒤 호출을 하게 됨. <br />
  - `response interceptor`를 설정해주면 해당 인스턴스의 요청에 대한 모든 응답이 콜백함수를 거치게 됨(위 예시에서는 `handleErrorResponse`)

<br />

### 요청에 따라 다른 인터셉터 만들기
---
위에서 본것처럼 각각의 axios 인스턴스를 만들어서 활용할수도 있지만, 요청 옵션에 따라서 다른 인터셉터를 만들기 위해서 `빌더 패턴`을 활용한 `API builder`같은 클래스 형태로 구성하기도 함.
> [!NOTE]
> **빌더 패턴** <br />
> 객체 생성을 더 편리하고, 가독성있게 만들기 위한 디자인 패턴으로, <br/>
> 주로 복잡한 객체의 생성을 단순화하고, **객체 생성 과정을 분리하여 객체를 조립하는 방법**을 제공함.

예시를 간단하게 살펴보자면
```ts
class API {
    readonly method: HTTPMethod;
    readonly url: string;
    headers?: HTTPHeaders;
    withCredential?: boolean;
    // ...

    constructor(method: HTTPMethod, url: string) {
        this.method = method;
        this.url = url;
    }

    call<T>() {
        const requester = axios.create();

        if (this.withCredential) {
            requester.interceptors.request.use(setToken);
            // do something...
        }

        // ...

        return requester.request({ ...this });
    }
}
```
이런식으로 기본 API클래스의 호출부를 구성하고, API를 호출하기 위한 래퍼를 `빌더 패턴`으로 만드는 것.
```ts
class APIBuiler {
    private _instance: API;

    constructor(method: HTTPMethod, url: string, data?: unknown) {
        this._instance = new API(method, url);

        this._instance.baseUrl = DEFAULT_BASE_URL;
        this._instance.headers = {
            'Content-Type': 'application/json; charset: utf-8',
        }
        // ...
    }

    static get = (url: string) => new APIBuiler('GET', url);
    static post = (url: string, data: unknown) => new APIBuiler('POST', url, data);
    static put = (url: string, data: unknown) => new APIBuiler('PUT', url, data);
    static delete = (url: string) => new APIBuiler('DELETE', url);

    baseURL(value: string): APIBuiler {
        this._instance.baseUrl = value;

        return this;
    }

    withCredentials(value: boolean) {
        this._instance.withCredential = true;

        return this;
    }

    // ...

    build() {
        return this._instance;
    }
}
```

위와 같은 패턴으로 제공한 `APIBuilder`는 어떻게 사용할까?
```ts
const getSomething = (name: string, size: number) => {
  const api = APIBuilder.get(API_URL)
  .withCredentails(true)
  .params({ name, size })
  .build;

  const response = api.call();

  return response;
};
```

`빌더 패턴`의 단점으로는 `보일러 플레이트`가 굉장히 많다는 점이다..

<br />

### API 응답 타입 지정하기
---
같은 서버에서 오는 응답은 대부분 하나로 통일되있어서 하나의 Response타입으로 묶어서 사용할 수 있음. <br />
그러나 몇몇 예외 케이스로 응답이 없는 경우가 있을 수도 있고, 다른 응답의 형태로 올 수도 있음. 이러한 점으로 보았을 때 **응답의 타입은 요청(`APIRequester`)과는 독립적으로 관리되어야 함**

또한 API응답이 자주 바뀌는 경우에는 `뷰 모델(View Model)`을 활용할 수 있음.<br />
예를 들어서 아래와 같은 코드가 있다
```ts
const CartBadge = () => {
  const [cartCount, setCartCount] = useState(0);

  useEffect(() => {
    fetch('https://api.baemin.com/cart')
      .then((response) => response.json())
      .then(({ item }) => setCartCount(item.length));
  });

  return (
    // ...
  );
};
```
근데 여기서 만약에 **API의 응답이 `item`에서 `cartItem`으로 바뀐다고 하면** UI가 깨져버리고 말것임..

이러한 문제를 해결하기 위해서 `뷰 모델`을 도입할 수 있음
```ts
interface CartListItemResponse {
  name: string;
}

interface CartListResponse {
  cartItem: CartListItemResponse[];
}

class cartList {
  readonly items: CartListResponse[];

  constructor({ cartItem }: CartListResponse) {
    this.items = cartItem;
  }
}
```
`뷰 모델`을 사용하면 API의 응답이 바뀌어도 UI가 꺠지지 않게 유지할 수 있는 대신에, ***추상화가 깊어지면서 코드가 복잡해지고 레이어를 관리하고 개발하는데 비용이 많이 든다는 단점이 존재*** 함.

따라서 결국 API 응답이 바뀌었을 때는 클라이언트 코드를 수정하는데 들어가는 비용을 줄이면서도, 도메인의 일관성을 지킬 수 있는 절충안을 찾아야 함.

<br />

### 런타임에서 발생하는 오류 찾아내기
---
ts는 `정적 검사 도구` 이므로 런타임에서 API의 응답이 ts에서 정의해둔 타입과 정말로 동일하게 오는지는 확인할 수 없음. <br />

따라서 런타임에 API 응답의 타입 오류를 방지하려면 `zod`나 `Superstruct`와 같은 라이브러리를 통해서 검증할 수 있음.
