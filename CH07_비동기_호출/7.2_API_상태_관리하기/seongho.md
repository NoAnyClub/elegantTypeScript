### 상태 관리 라이브러리에서 호출하기
---
실제 API를 호출하는 코드는 컴포넌트 내에서 비동기 함수를 직접 호출하지 않음. <br />
비동기 API를 호출하기 위해서는 API의 성공 및 실패에 따른 상태관리가 되어야 하므로, 상태관리 라이브러리의 액션이나 훅과 같이 재정의된 형태를 사용해야 함

상태 관리 라이브러리의 비동기 함수들은 서비스 코드를 사용해서 비동기 상태를 변화 시킬 수 있는 함수를 제공함.<br />
컴포넌트는 이러한 함수를 사용해서 상태를 구독하며, 상태가 변경될 때 컴포넌트를 다시 렌더링하는 방식으로 동작함.

`Redux`예시를 살펴보자
```ts
import { useEffect } from "react";
import { useDispatch, useSelector } from 'react-redux';

export function useMonitoringHistory() {
    const dispatch = useDispatch();

    const searchState = useSelector((state) => state.monitoringHistory.searchState);

    const getHistoryList = async (
        newState: Partial<MonitoringHistorySearchState>
    ) => {
        const newSearchState = { ...searchState, ...newState };
        dispatch(monitoringHistorySlice.actions.changeSearchState(newSearchState));

        const response = await getHistories(newSearchState); // 비동기 API 호출
        dispatch(monitoringHistorySlice.actions.fetchDate(response));
    }

    return {
        searchState,
        getHistoryList
    };
}
```
해당 코드는 `getHistoryList`만 호출하고, 해당 결과를 받아와서 상태를 업데이트 하는 일반적인 방식으로 사용할 수 있음. 그러나, 해당 코드에서는 `getHistoryList`함수에서는 `dispatch`코드를 제외하더라도 **API호출**과 **상태 관리 코드**를 작성해야 함.

```ts
const API = axios.create();

const setAxiosInterceptor = (store: EnhancedStore) => {
  API.interceptors.request.use(
    (config: AxiosRequestConfig) => {
      const { params, url, method } = config;

      // 전역상태 관리 코드...

      return config;
    },
    (error) => Promise.reject(error)
  );

  API.interceptors.response.use(
    (response: AxiosResponse) => {
      const { method, url } = response.config;

      // 전역상태 관리 코드...

      return response.data.data;
    },
    (error) => {
      // 전역상태 관리 코드...

      return Promise.reject(error);
    }
  );
}
```
요런식으로 API를 호출할 때, 호출한 뒤, 에러가 발생했을 때 각각 전역상태를 세팅해주어야 함.

<br />

### 훅으로 호출하기
---
`react-query`나 `swr`과 같은 훅을 사용한 방법은 전역 상태 관리 라이브러리를 사용한 방식보다 훨씬 간단함. <br />
이러한 훅은 캐시를 사용하여 비동기 함수를 호출하며, 의도치 않은 상태 변경을 방지하는데 도움이 됨.

```ts
// 커스텀 훅
const useGetJobList = () => {
  return useQuery(['getJobList'], async () => {
    const response = await JobService.fetchJobList();

    // View Model을 사용해서 결과 return
    return new JobList(response);
  });
}
```
이렇게 작성한 이후, 일반적인 훅을 호출하는 것 처럼 사용하면 됨. <br />
만약, 항시 최신 상태를 표현하려면 `폴링`이나 `웹소켓` 등의 방식을 사용해야 함.

> [!TIP]
> **폴링(Polling)** <br />
> 클라이언트가 주기적으로 서버에 요청을 보내 데이터를 일정 주기마다 최신 데아터로 업데이트 하는 방식

<br />

전역 상태 관리 라이브러리에서는 비동기로 상태를 변경하는 코드가 추가되면 점점 전역 상태 관리 스토어가 비대해지는 것을 볼 수 있음. <br />
따라서 `redux`나 `mobX`와 같은 라이브러리를 `react-query`로 변경하는 추세임.

그러나 늘 그렇듯이 `react-query`가 정답인것은 아님. 어떤 상태 관리 라이브러리를 선택할지는 상황에 따라 적절한 판단이 필요함.
