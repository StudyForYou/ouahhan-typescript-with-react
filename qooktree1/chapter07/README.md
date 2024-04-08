<h1>비동기 호출</h1>

# 📝 API 요청

## ✏️ fetch로 API 요청하기

- fetch를 사용해 외부 DB에 접근하여 사용자가 장바구니에 추가한 정보를 호출하는 코드를 구성해보자.

```tsx
import React, { useEffect, useState } from "react";

const CartBadge: React.FC = () => {
  const [cartCount, setCartCount] = useState(0);
  useEffect(() => {
    fetch("https://api.baemin.com/cart")
      .then((res) => res.json())
      .then(({ cartItem }) => {
        setCartCount(cartItem.length);
      });
  }, []);
  return <>{/* cartCount 상태를 이용하여 컴포넌트 렌더링 */}</>;
};
```

- 백엔드에서 기능 변경을 해야 해서 API URL을 수정해야 한다고 가정해보자.
- 새로운 API 요청 정책이 추가될 때마다 계속해서 비동기 호출 코드를 수정해야 하는 번거러움이 있다.

## ✏️ 서비스 레이어로 분리하기

- 여러 API 요청 정책이 추가되어 코드가 변경될 수 있다는 점을 감안한다면 비동기 호출 코드는 **컴포넌트 영역에서 분리되어 서비스레이어에서 처리**되어야 한다.
- 앞의 코드 기준으로는 fetch함수를 호출하는 부분이 서비스 레이어로 이동하고 컴포넌트는 서비스 레이어의 비동기 함수를 호출하여 그 결과를 받아와 렌더링 하는 흐름이 된다.

- 하지만 단순히 fetch 함수를 분리한다고 API요청 정책이 추가되는 것을 해결하기는 어렵다.

  ```jsx
  async function fetchCart() {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => confroller.abort(), 5000);
    const response = await fetch("https://api.baemin.com/cart", {
      signal: controller.signal,
    });
    clearTimeout(timeoutId);
    return response;
  }
  ```

  - Query parameter나 커스텀 헤더 추가 또는 쿠키를 읽어 토큰을 집어넣는 등 다양한 API 정책이 추가될 수 있는데 이를 모두 구현하는 것은 번거로운 일이다.
  - `AbortController`: 비동기 작업을 중단 할 수 있는 Web API이다.

## ✏️ Axios 활용하기

- `fetch`는 내장 라이브러리이기 때문에 설치할 필요가 없지만 기능들을 직접 구현해서 사용해야 한다.
- 이러한 번거러움 때문에 fetch 함수를 쓰는 대신 `Axios`를 사용하고 있다.

```tsx
const apiRequester: AxiosIntance = axios.create({
  baseURL: "https://api.baemin.com",
  timeout: 5000, // JS의 AbortController를 Axios 라이브러리에서는 이런식으로 사용한다.
});
const fetchCart = (): AxiosPromise<FetchCartResponse> =>
  apiRequester.get<FetchCartResponse>("cart");
const postCart = (
  postCartRequest: PostCartRequest
): AxiosPromise<PostCartResponse> =>
  apiRequester.post<PostCartResponse>("cart", postCartRequest);
```

## ✏️ Axios 인터셉터 사용하기

- 인터셉터 기능을 사용하여 requester에 따라 비동기 호출 내용을 추가해서 처리 가능하다.

```ts
// 요청을 보내기 전 실행
axios.interceptors.request.use();

// 요청을 받은 후 실행
axios.interceptors.response.use();

// API 엔트리로 요청을 보내기 위한 Axios 인스턴스를 생성하고, 요청 시에 5초 동안 대기하는 timeout 설정
const apiRequester: AxiosInstance = axios.create({
  baseURL: "https://api.baemin.com",
  timeout: 5000,
});

// Axios 요청 config에 특정 header를 추가하는 함수
const setRequestDefaultHandler = (requestConfig: AxiosRequestConfig) => {
  const config = requestConfig;
  config.headers = {
    ...config.headers,
    "Content-Type": "application/json;charset=utf-8",
    user: getUserToken(),
    agent: getAgent(),
  };
  return config;
};

// Axios 주문 요청 config에 특정 헤더 값을 추가하는 함수
const setOrderRequestDefaultHeader = (requestConfig: AxiosRequestConfig) => {
  const config = requestConfig;
  config.headers = {
    ...config.headers,
    "Content-Type": "application/json;charset=utf-8",
    "order-client": getOrderClientToken(),
  };
  return config;
};

// 기본 헤더 값을 설정하는 Axios 요청 인터셉터
apiRequester.interceptors.request.use(setRequestDefaultHeader);

// 주문 API를 위한 Axios 인스턴스를 생성, 기본 URL과 기본 설정인 defaultConfig 사용
const orderApiRequester: AxiosInstance = axios.create({
  baseURL: orderApiBaseUrl,
  ...defaultConfig,
});

// orderApiRequester에 요청 전에 setOrderRequesterDefaultHeader 함수를 호출하여 주문 API 전용 헤더 값을 설정하는 Axios 요청 인터셉터가 등록
orderCartApiRequester.interceptors.request.use(setRequestDefaultHeader);

// 응답을 처리하는데 있어서 httpErrorHandler를 사용하도록 설정
orderApiRequester.interceptors.response.use({
    (response: AxiosResponse) => response,
    httpErrorHandler
})

//주문 카트 API를 위한 Axios 인스턴스를 생성하고, 해당 API의 기본 URL과 기본 설정인 defaultConfig를 사용
const orderCartApiRequester: AxiosInstance = axios.create({
    baseURL:orderCartApiBaseUrl,
    ...defaultConfig,
})

//orderCartApiRequester에 요청 전에 setRequesterDefaultHeader 함수를 호출하여 기본 헤더 값을 설정하는 Axios 요청 인터셉터가 등록
orderCartApiRequester.interceptors.request.use(setRequesterDefaultHeader);
```

- 이와 달리 요청 옵션에 따라 다른 인터셉터를 만들기 위해 `빌더 패턴`을 추가하여 APIBuilder 같은 클래스 형태로 구성하기도 한다.

### 빌더패턴

- 객체 생성을 더 편리하고 가독성 있게 만들기 위한 `디자인 패턴` 중 하나다.
- 주로 복잡한 객체의 생성을 단순화하고, 객체 생성 과정을 분리하여 객체를 조립하는 방법을 제공한다.

### 보일러플레이트(Boilerplate) 코드

- `어떤 기능을 사용할 때 반복적으로 사용되는 기본적인 코드`를 말한다.
- 예를 들어 API를 호출하기 위한 기본적인 설정과 인터셉터 등을 설정하는 부분을 보일러플레이트 코드로 간주할 수 있다.

## ✏️ API 응답 타입 지정하기

- 같은 서버에서 오는 응답의 형태는 대체로 통일되어있기 때문에 API의 응답 값은 하나의 Response 타입으로 묶일 수 있다.

```tsx
interface Response<T> {
  data: T;
  status: string;
  serverDateTime: string;
  errorCode?: string; // FAIL, ERROR
  errorMessage?: string; // FAIL, ERROR
}

// 카트 정보를 가져오기 위한 API 요청
// AxiosPromise를 반환하며, 해당 Promise의 제네릭 타입은 Response<FetchCartResponse>로 정의
// FetchCartResponse는 서버에서 받아온 카트 정보에 대한 타입
const fetchCart = (): AxiosPromise<Response<FetchCartResponse>> => {
  apiRequester.get<Response<FetchCartResponse>>("cart");
};

// 카트에 데이터를 추가하거나 업데이트하기 위한 API 요청
// AxiosPromise를 반환하며, 해당 Promise의 제네릭 타입은 Response<PostCartResponse>로 정의
// PostCartResponse는 서버에서 받아온 카트에 대한 업데이트 결과에 대한 타입
const postCart = (
  postCartRequest: PostCartRequest
): AxiosPromise<Response<PostCartResponse>> => {
  apiRequester.post<Response<PostCartResponse>>("cart", postCartRequest);
};
```

- 이와 같이 서버에서 오는 응답을 통일해줄 때 주의할 점이 있다.
- Response 타입을 apiRequester 내에서 처리하고 싶은 생각이 들 수 있는데, 이렇게 하면 update나 create같이 응답이 없을 수 있는 API를 처리하기 까다로워진다.

  ```tsx
  const updateCart = (
    updateCartRequest
  ): AxiosPromise<Response<FetchCartResponse>> => apiRequester.get("cart");
  ```

  - 따라서 Response 타입은 apiRequester가 모르게 관리되어야 한다.

---

- API 요청 및 응답 값 중에서는 하나의 API 서버에서 다른 API 서버로 넘겨주기만 하는 값도 존재할 수 있다.
- 해당 값에 어떤 응답이 들어있는지 알 수 없거나 값의 형식이 달라져도 로직에 영향을 주지 않는 경우에는 `unknown` 타입을 사용하자.

  ```ts
  interface Response {
    data: {
      cartItems: CartItem[];
      forPass: unknown;
    };
  }
  ```

## ✏️ 뷰 모델(View Model) 사용하기

- 이건 도저히 모르겠네요 :(

## ✏️ Superstruct를 사용해 런타임에서 응답 타입 검증하기

### Superstruct 라이브러리

- 인터페이스 정의와 JS 데이터의 유효성 검사를 쉽게 할 수 있다.
- `런타임에서의 데이터 유효성 검사를 제공`한다.
- 다른 유효성 검사 라이브러리로는 `Zod`, `Yup`, `Joi` 등이 있다.

```ts
import {
  assert,
  is,
  validate,
  object,
  number,
  string,
  array,
} from "superstruct";

const Article = object({
  id: number(),
  title: string(),
  tags: array(string()),
  author: object({
    id: number(),
  }),
});

const data = {
  id: 34,
  title: "Hello World",
  tags: ["news", "features"],
  author: {
    id: 1,
  },
};

// assert는 유효하지 않을 경우 에러를 던진다.
assert(data.Article);

// is는 유효성 검사 결과에 따라 true or false 즉, boolean 값을 return한다.
is(data Article);

// validate는 [error, data] 형식의 튜플을 return한다. 유효하지 않을 때는 에러 값이 반환되고 유효한 경우에는 첫 번째 요소로 undefined, 두 번째 요소로 data value가 return된다.
vaildate(data, Article);
```

## ✏️ 실제 API 응답 시의 Superstruct 활용 사례

```ts
interface ListItem {
  id: string;
  content: string;
}

interface ListResponse {
  items: ListItem[];
}

const fetchList = async (filter?: ListFetchFilter): Promise<ListResponse> => {
  const { data } = await api
    .params({ ...filter })
    .get("/apis.get-list-summaries")
    .call<Response<ListResponse>>();

  // 런타임 검증을 하려면 밑의 isListItem 코드를 활용하면 된다.
  // isListItem(data.items);
  return { data };
};
```

- TS로 작성한 코드는 명시한 타입대로 응답이 올 거라고 기대하고 있지만 실제 서버 응답 형식은 다를 수 있다.
- TS는 실제 서버 응답의 형식과 명시한 타입이 일치하는지를 확인할 수 없다.

```ts
import { assert } from "superstruct";

function isListItem(listItems: ListItem[]) {
  // isListItem은 ListItem의 배열 목록을 받아와 데이터가 ListItem 타입과 동일한지 확인하고 다를 경우에는 에러를 던진다.
  listItems.forEach((listItem) => assert(listItem, ListItem));
}
```

- 이제 fetchList 함수에 isListItem(검증 함수)를 추가하면 런타임 유효성 검사를 진행할 수 있다.

# 📝 API 상태 관리하기

## ✏️ 상태 관리 라이브러리에서 호출하기

- 상태 관리 라이브러리의 비동기 함수들은 서비스 코드(action, dispatch)를 사용해서 비동기 상태를 변화시킬 수 있는 함수를 제공한다.
- 컴포넌트는 이러한 함수를 사용하여 상태를 구독하며, 상태가 변경될 때마다 컴포넌트를 다시 렌더링하는 방식으로 동작한다.

### 🚨 문제점

- 모든 상태 관리 라이브러리에서 비동기 처리 함수를 호출하기 위해 액션이 추가될 때마다 관련된 스토어나 상태가 계속 늘어난다.
- `전역 상태 관리자가 모든 비동기 상태에 접근하고 변경할 수 있다는 것`
- 만약 2개 이상의 컴포넌트가 구독하고 있는 비동기 상태가 있다면 쓸데없는 비동기 통신이 발생하거나 의도치 않은 상태 변경이 발생 할 수 있다.

## ✏️ 훅으로 호출하기

- react-query나 useSwr 같은 훅을 사용한 방법은 의도치 않은 상태 변경을 방지하는 데 도움이 된다. -상태관리 라이브러리에서는 비동기로 상태를 변경하는 코드가 추가되면 전역 상태 관리 스토어가 비대해지기 때문에 상태를 변경하는 액션이 증가하는 것뿐만 아니라 전역 상태 자체가 복잡해진다.
- 이러한 이유 때문에 react-query로 변경하려는 시도가 이루어지고 있다.

# 📝 API 에러 핸들링

## ✏️ 타입 가드 활용하기

- Axios 라이브러리에서는 Axios 에러에 대해 isAxiosError라는 타입 가드를 제공하고 있다.
- ErrorResponse 인터페이스를 사용하여 처리해야 할 Axios 에러 형태는 `AxiosError<ErrorResponse>`로 표현할 수 있으며 다음과 같이 타입 가드를 명시적으로 작성할 수 있다.

  ```ts
  // 공통 에러객체에 대한 타입
  interface ErrorResponse {
    status: string;
    serverDateTime: string;
    errorCode: string;
    errorMessage: string;
  }

  // 사용자 정의 타입 가드 작성
  function isServerError(error: unknown): error is AxiosError<ErrorResponse> {
    return axios.isAxiosError(error);
  }

  const onClickdeleteHistoryButton = async (id: string) => {
    try {
      await axios.post("https://...", { id });
      alert("주문 내역이 삭제되었습니다.");
    } catch (error: unknown) {
      // 서버 에러일 때의 처리임을 명시적으로 알 수 있다.
      if (isServerError(e) && e.response && e.response.data.errorMessage) {
        serErrorMessage(e.response.data.errorMessage);
        return;
      }

      setErrorMessage(
        "일시적인 에러가 발생했습니다. 잠시 후 다시 시도해주세요"
      );
    }
  };
  ```

## ✏️ 에러 서브클래싱하기

- 실제 요청을 처리할 때 단순한 서버 에러도 발생하지만 인증 정보 에러, 네트워크 에러, 타임아웃 에러 같은 에러가 발생하기도 한다.
- 이를 더욱 명시적으로 표시하기 위해 `서브클래싱`을 활용할 수 있다.

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

  class UnauthorizedHttpError extends Error {
    constructor(message: string, response?: AxiosResponse<ErrorResponse>) {
      super(message, response);
      this.name = "UnauthorizedError";
    }
  }
  ```

## ✏️ 인터셉터를 활용한 에러 처리

```ts
const httpErrorHandler = (
  error: AxiosError<ErrorResponse> | Error
): Promise<Error> => {
  let promiseError: Promise<Error>;

  if (axios.isAxiosError(error)) {
    if (Object.is(error.code, "ECONNABORTED")) {
      promiseError = Promise.reject(new TimeoutError());
    } else if (Object.is(error.message, "Network Error")) {
      promiseError = Promise.reject(new NetworkError(""));
    } else {
      const { response } = error as AxiosError<ErrorResponse>;

      switch (response?.status) {
        case HttpStatusCode.UNAUTHORIZED:
          promiseError = Promise.reject(
            new UnauthorizedError(response?.data.message, response)
          );
          break;
        default:
          promiseError = Promise.reject(
            new OrderHttpError(response?.data.message, response)
          );
      }
    }
  } else {
    promiseError = Promise.reject(error);
  }

  return promiseError;
};

// 인터셉터 내부에서 사용
orderApiRequester.interceptors.response.use(
  (response: AxiosResponse) => response,
  httpErrorHandler
);
```

## ✏️ 에러 바운더리를 활용한 에러 처리

- `react-error-boundary` 라이브러리를 통해 `ErrorBoundary` 사용 가능
- react query와 혼합해서 사용할 수 있다.

### 참고 사항

- [React 공식문서 - useTransition 예제](https://react.dev/reference/react/useTransition#displaying-an-error-to-users-with-error-boundary)
- [React query와 사용한 예제](https://yiyb-blog.vercel.app/posts/error-boundary-with-react-query)

## ✏️ 상태 관리 라이브러리에서의 에러 처리

- Redux 에러 처리

```ts
//api 호출에 관한 api call reducer
const apiCallSlice = createSlice({
  name: "apiCall",
  initialState,
  reducers: {
    setApiCall: (state, { payload: { status, urlInfo } }) => {
      /* api state를 채우는 로직*/
    },
    setApiCallError: (state, { payload }: PayloadAction<any>) => {
      state.error = payload;
    },
  },
});

const API = axios.create();

const setaxiosInterceptor = (store: EnhancedStroe) => {
  //중복 코드 생략

  //onSuccess시 처리를 인터셉터로 처리
  API.interceptors.response.use(
    (response: AxiosResponse) => {
      const { method, url } = reponse.config;

      sotre.dispatch(
        setApiCall({
          status: ApiCallstauts.None,
          urlInfo: { url, method },
        })
      );

      return response?.data?.data || response?.data;
    },
    (error: AxiosError) => {
      //에러 상태를 관리하지 않고 처리할 수 있는 경우 바로 처리
      if (error.response?.status === 401) {
        window.location.href = error.response.headers.location;
        return;
      } else if (error.response?.stateu === 403) {
        window.location.href = error.response.headers.location;
        return;
      } else {
        message.error("error");
      }

      const {
        config: { url, method },
      } = error;

      store.dispatch(
        setApiCall({
          status: ApiCallStatus.None,
          urlInfo: { url, method },
        })
      );

      return Promise.reject(error);
    }
  );
};

const fetchmenu = createAsyncThunk(
    FETCH_MENU_REQUESTm
    async({shopId, menuId}:FetchMenu) => {
        try{
            const data = await api.fetchMenu(shopId, menuId)
            return data
        }catch(error) {
            //에러를 상태로 처리
            setApiCallError({error})
        }
    }
)
```

- 에러 상태를 관리하지 않고 처리할 수 있다면 바로 처리(ex. 401, 403)하고, 그렇지 않다면 reject로 넘겨준다.
- 이후 액션을 정의하면서 setApiCallError를 사용하여 에러를 상태로 처리한다.

## ✏️ react-query를 활용한 에러 처리

- react-query나 swr과 같은 데이터 fetching 라이브러리를 사용하면 요청에 대한 상태를 반환해주기 때문에 요청 상태를 확인하기 쉽다.

  ```tsx
  const JobComponent: React.FC = () => {
    const { isError, error, isLoading, data } = useFetchJobList();

    if (isError) {
      return <div>{error.message}</div>;
    }

    if (isLoading) {
      return <div>로딩중입니다...</div>;
    }

    return <>{data}</>;
  };
  ```

## ✏️ 그 밖의 에러 처리

- 200번 대 성공 응답에 대한 에러 처리가 필요한 경우를 예시로 들어보자.

```ts
const successHandler = (response: CreateOrderResponse) =>{
    if(response.status ==== 'SUCCESS') {
        return;
    }

    throw new CustomError(response.status, response.message)
}

const createOrder=(data:CreateOrderData) => {
    try{
        const response = apiRequester.post('https://...',data)

        successHandler(response)
    }catch(error) {
        errorHandler(error)
    }
}
```

- API가 많을 때는 매번 `if(reponse === 'SUCCESS')`구문을 추가해줘야 함
- 커스텀 에러를 사용하고 있는 요청을 일괄적으로 에러로 처리하고 싶은 경우 Axios 등의 라이브러리 기능을 활용하면 됨

  ```tsx
  const apiRequester: AxiosInstance = axios.create({
    baseURL: orderApiBaseUrl,
    ...defaultConfig,
  });

  export const httpSuccessHandler = (response: AxiosResponse) => {
    if (response.data.status !== "SUCCESS") {
      throw new CustomError(response.data.message, response);
    }

    return response;
  };

  apiRequester.interceptors.response.use(httpSuccessHandler, httpErrorHandler);

  const createOrder = (data: CreateOrderData) => {
    try {
      const response = apiRequester.post("PostUrl", data);
      httpSuccessHandler(response);
    } catch (e) {
      httpErrorHandler(e);
    }
  };
  ```

# 📝 API 모킹

- 프론트엔드 개발이 서버 개발보다 먼저 이루어지거나 서버와 프론트엔드 개발이 동시에 이루어지는 경우가 많다.
- 서버가 별도의 가짜 서버를 제공한다고 하더라도 프론트엔드 개발 과정에서 발생할 수 있는 모든 예외 사항을 처리하는 것은 쉽지 않다.
- 이럴 때 `모킹(Mocking)`이라는 방법을 활용할 수 있다.

#### 모킹이란?

- 가짜 모듈을 활용하는 것

## ✏️ JSON 파일 불러오기

- 간단한 조회만 필요한 경우 `*.json` 파일 생성 및 JS 파일 안에 `JSON` 형식의 정보를 저장하고 export해주는 방식을 사용하면 된다.

  ```ts
  // mock/service.ts
  const SERVICES: Service[] = {
    {
      id: 0,
      name: "배달의 민족",
    },
    {
      id: 1,
      name: "만화경"
    }
  }

  export default SERVICES;

  // api
  const getServices = ApiRequester.get("/mock/service.ts");
  ```

````

## ✏️ NextApiHandler 활용하기

- Next.js를 사용하고 있다면 `NextApiHandler`를 활용할 수 있다.
- 응답하고자 하는 값을 정의하고 핸들러 안에서 요청에 대한 응답을 정의한다.

  ```ts
  // api/mock/brand
  import { NextApiHandler } from "next";

  const BRANDS: Brand[] = [
    {
      id: 1,
      label: "배민스토어",
    },
    {
      id: 2,
      label: "비마트",
    },
  ];

  const handler: NextApiHandler = (req, res) => {
    // request 유효성 검증 - 중간 과정에 응답 처리 로직 가능

    res.json(BRANDS); // 요청에 대한 응답 정의
  };
  export default handler;
  ```

## ✏️ API 요청 핸들러에 분기 추가하기

- 요청 경로를 수정하지 않고 평소에 개발할 때 필요한 경우에만 실제 요청을 보내고 그 외에는 목업을 사용하여 개발하고 싶다면
  API 요청을 훅 또는 별도 함수로 선언해준 다음 조건에 따라 목업 함수를 내보내거나 실제 요청 함수를 내보낼 수 있다.

  ```ts
  const fetchBrands = () => {
    // if(분기문)을 사용하여 목업 데이터와 실제 서버 데이터를 분기 처리
    if (useMock) return mockFetchBrands(); // 목업 데이터를 fetch하는 함수
    return requester.get("/brands"); // 실제 서버에서 API 호출
  };
  ```

## ✏️ axios-mock-adapter로 모킹하기

- 서비스 함수에 분기문이 추가되는 점을 바라지 않는다면 라이브러리를 사용하면 된다.
- GET뿐만 아니라 POST, PUT, DELETE 등 다른 HTTP 메서드에 대한 목업을 작성할 수 있게 된다.
- networkError, timeoutError 등을 메서드로 제공하기 때문에 다음처럼 임의로 에러를 발생시킬 수도 있다.

  ```ts
  // axios 및 axios-mock-adapter 가져오기
  import axios from "axios";
  import MockAdapter from "axios-mock-adapter";

  // Axios Mock Adapter 인스턴스 생성
  const mock = new MockAdapter(axios);

  // Mock 데이터 정의
  const mockData = [
    { id: 1, name: "Mock Brand 1" },
    { id: 2, name: "Mock Brand 2" },
  ];

  export const fetchBrandListMock = () => {
    // 특정 엔드포인트에 대한 GET 요청을 가로채고 목업 응답 반환
    mock.onGet("/brands").reply(200, mockData);
  };

  export const fetchBrandListWithNetworkErrorMock = () => {
    mock.onGet("/brands").networkError();
  };
  ```

## ✏️ 목업 사용 여부 제어하기

- 로컬에서 목업을 사용하고 dev나 운영환경에서는 사용하지 않으려면 플래그를 사용하여 목업을 사용하는 상황을 구분할 수 있다.

```ts
const useMock = process.env.REACT_APP_MOCK === 'true';

const mockFn = ({ status = 200, time = 100, use = true }: MockResult) =>
  use &&
  mock.onGet('').reply(() =>
    new Promise((resolve) =>
      setTimeout(() => {
        resolve([
          status,
          status === 200 ? fetchBrandSuccessResponse : undefined,
        ]);
      }, time)
    );
  );

  if (useMock){
    mockFn({status:200,time:100,use:true})
  }
```

- 다음처럼 플래그에 따라 mockFn을 제어할 수 있는데 매개변수를 넘겨 특정 mock 함수만 동작하게 하거나 동작하지 않게 할 수 있다.
- 스크립트 실행 시 구분 짓고자 한다면 package.json에 관련 스크립트를 추가해줄 수 있다.

  ```json
  // package.json
  {
    ...,
    "scripts": {
      ...
      "start:mock": "REACT_APP_MOCK=true npm run start",
      "start": "REACT_APP_MOCK=false npm run start",
      ...
    },
    ...
  }
  ```

- axios-mock-adapter는 API를 중간에 가로채는 것으로 실제 API요청을 주고 받지 않는다.
- 따라서 API 요청의 흐름을 파악하기 위해서는 react-query-devtools 혹은 redux test tool과 같은 도구의 힘을 빌려야한다.
````
