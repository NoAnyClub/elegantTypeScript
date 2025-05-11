# API 에러 핸들링

비동기 API 에러를 구체적이고 명시적으로 핸들링하는 방법을 살펴보자!

## 1) 타입 가드 활용하기

Axios 라이브러리에서는 Axios 에러에 대해 isAxiosError라는 타입 가드를 제공한다.

타입 가드를 직접 사용하는 방법도 있지만, 서버 에러임을 명확하게 표시하고<br />
서버에서 내려주는 에러 응답 객체에 대해서도 구체적으로 정의함으로<br />
어떤 속성을 가진 에러 객체인지 파악할 수 있다.

```ts
interface ErrorResponse {
  status: string;
  serverDateTime: string;
  errorCode: string;
  errorMessage: string;
}
```

- ErrorResponse 인터페이스를 사용해 처리해야 할 Axios 에러 형태는 AxiosError<ErrorResponse>로 표현한다.
- 아래와 같이 타입 가드를 명시적으로 작성할 수 있다.

```ts
function isServerError(error: unknown): error is AxiosError<ErrorResponse> {
  return axios.isAxiosError(error);
}
```

> 사용자 정의 타입 가드를 정의할 때는 타입 가드 함수의 반환 타입으로<br />
> parameterName is Type 형태의 타입 명제를 정의해주는 게 좋다.

<br />
<br />

## 2) 에러 서브클래싱하기

실제 요청을 처리할 때 단순한 서버 에러도 발생하지만 다양한 에러가 발생할 수 있다.
이를 더 명시적으로 표시하기 위해 서브클래싱을 활용할 수 있다.

> **서브클래싱**?<br />
> 기존 클래스를 확장해 새로운 클래스를 만드는 과정을 말한다.<br />
> 새로운 클래스는 상위 클래스의 모든 속성과 메서드를 상속받아 사용할 수 있고 추가적인 속성과 메서드를 정의할 수도 있다.

예를 들어, 서버에서 전달된 메세지를 보고 개발자 입장에서는 사용자 로그인 정보가 만료되었는지,<br />
타임아웃이 발생한 건지 혹은 데이터를 잘못 전달한 것인지를 구분할 수 없다.

이때, 서브클래싱을 활용하면 에러가 발생했을 때 코드상에서 어떤 에러인지를 바로 확인할 수 있다.<br />
또한 에러 인스턴스가 무엇인지에 따라 에러 처리 방식을 다르게 구현할 수 있다.

<br />

### 서브클래싱으로 상세 에러 객체를 정의해보자!

```ts
class OrderHttpError extends Error {
  private readonly privateResponse: AxiosResponse<ErrorResponse> | undefined;

  constructor(message?: string, response?: AxiosResponse<ErrorResponse>) {
    super(message);
    this.name = "OrderHttpError";
    this.privateResponse = response;
  }

  get response(): AxiosResponse<ErrorResponse> | undefined {
    return this.privateResponse;
  }
}

class NetworkError extends Error {
  constructor(message = "") {
    super(message);
    this.name = "NetworkError";
  }
}

class UnauthorizedError extends Error {
  constructor(message?: string, response?: AxiosResponse<ErrorResponse>) {
    super(message, response);
    this.name = "UnauthorizedError";
  }
}
```

### 요청 코드로 돌아와 활용해보자!

```ts
const onActionError = (
  error: unknown,
  params?: Omit<AlertPopup, "type" | "message">
) => {
  if (error instanceof UnauthorizedError) {
    onUnauthorizedError(
      error.message,
      errorCallback?.onUnauthorizedErrorCallback
    );
  } else if (error instanceof NetworkError) {
    alert("네트워크 실패", {
      onClose: errorCallback?.onNetworkErrorCallback,
    });
  } else if (error instanceof OrderHttpError) {
    alert(error.message, params);
  } else if (error instanceof Error) {
    alert(error.message, params);
  } else {
    alert(defaultHttpErrorMessage, params);
  }
};

const getOrderHistory = async (page: number): Promise<History> => {
  try {
    const { data } = await fetchOrderHistory({ page });
    const history = await JSON.parse(data);

    return history;
  } catch (error) {
    const customError = errorHandler(error);
    onActionError(customError);
  }
};
```

<br />
<br />

## 3) 인터셉터를 활용한 에러 처리

Axios 같은 페칭 라이브러리는 인터셉터 기능을 제공한다. 이를 사용하면 HTTP 에러에 일관된 로직을 적용할 수 있다.

```ts
const httpErrorHandler = (
  error: AxiosError<ErrorResponse> | Error
): Promise<Error> => {
  (error) => {
    if (error.response && error.response.status === 401) {
      window.location.href = `${backOfficeAuthHost}/login?targetUrl=${window.location.href}`;
    }

    return Promise.reject(error);
  };

  orderApiRequester.interceptors.response.use(
    (response: AxiosResponse) => response,
    httpErrorHandler
  );
};
```

<br />
<br />

## 4) 에러 바운더리를 활용한 에러 처리

- 에러 바운더리는 리액트 컴포넌트 트리에서 에러가 발생할 때 공통으로 에러를 처리하는 리액트 컴포넌트이다.
- 에러 바운더리는 에러가 발생한 컴포넌트 대신에 에러 처리를 하거나 예상치 못한 에러를 공통 처리할 때 사용할 수 있다.
- 트리 하위에 있는 컴포넌트에서 발생한 에러를 캐치하면, 해당 에러를 가장 가까운 부모 에러 바운더리에서 처리하게 할 수 있다.

```tsx
import React, { ErrorInfo } from "react";
import ErrorPage from "pages/ErrorPage";

interface ErrorBoundaryProps {}
interface ErrorBoundaryState {
  hasError: boolean;
}

class ErrorBoundary extends React.Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(): ErrorBoundaryState {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    this.setState({ hasError: true });
    console.error(error, errorInfo);
  }

  render(): React.ReactNode {
    const { children } = this.props;
    const { hasError } = this.state;
    return hasError ? <ErrorPage /> : children;
  }
}

const App = () => {
  return (
    <ErrorBoundary>
      <OrderHistoryPage />
    </ErrorBoundary>
  );
};
```

> 위처럼 작성하면 OrderHistoryPage 컴포넌트 내에서 처리되지 않은 에러가 있을 때 에러 바운더리에서 에러 페이지를 노출한다. 이외에도 에러 바운더리에 로그를 보내는 코드를 추가하여 예상치 못한 에러의 발생 여부를 추적할 수 있게 된다.

<br />
<br />

## 5) react query를 활용한 에러 처리

```tsx
const JobComponent = () => {
  const { isError, error, isLoading, data } = useFetchJobList();

  if (isError) {
    return <div>{`${error.message}가 발생했습니다.`}</div>;
  }

  if (isLoading) {
    return <div>로딩 중</div>;
  }

  return (
    <>
      {data.map((job) => (
        <JobItem key={job.id} job={job} />
      ))}
    </>
  );
};
```

<br />
<br />

## 6) 그 밖의 에러 처리

비지니스 로직에서의 유효성 검증에 의해 추가된 커스텀 에러는 200 응답과 함께 응답 바디에 별도의 상태 코드를 전달하기도 한다.

예를 들어 장바구니에서 주문을 생성하는 API가 다음과 같은 커스텀 에러를 반환한다고 해보자.

```
httpStatus: 200 {
    "status": "C20005", // 성공인 경우 "SUCCESS"를 응답
    "message": "장바구니에 품절된 메뉴가 있습니다"
}
```

> 이럴 때 만약 커스텀 에러를 사용하고 있는 요청을 일괄적으로 에러로 처리하고 싶다면
> Axios 등의 라이브러리 기능을 활용하면 된다.

특정 호스트에 대한 API requester를 별도로 선언하고<br />
상태 코드 비교 로직을 인터셉터에 추가할 수 있다.

<br />

```ts
export const apiRequester: AxiosInstance = axios.create({
  baseURL: orderApiBaseUrl,
  ...defaultConfig,
});

export const httpSuccessHandler = (response: AxiosResponse) => {
  if (response.data.status !== "SUCCESS") {
    throw new CustomError(response?.data.message, response);
  }

  return response;
};

apiRequester.interceptors.response.use(httpSuccessHandler, httpErrorHandler);

const createOrder = (data: CreateOrderData) => {
  try {
    const response = apiRequester.post("https://...", data);
  } catch (error) {
    // status가 SUCCESS가 아닌 경우 에러로 전달
    errorHandler(error);
  }
};
```

<br />
<br />
