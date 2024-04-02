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

## ✏️ 훅으로 호출하기

# 📝 API 에러 핸들링

## ✏️ 타입 가드 활용하기

## ✏️ 에러 서브클래싱하기

## ✏️ 인터셉터를 활용한 에러 처리

## ✏️ 에러 바운더리를 활용한 에러 처리

## ✏️ 상태 관리 라이브러리에서의 에러 처리

## ✏️ react-query를 활용한 에러 처리

## ✏️ 그 밖의 에러 처리

# 📝 API 모킹

## ✏️ JSON 파일 불러오기

## ✏️ NextApiHandler 활용하기

## ✏️ API 요청 핸들러에 분기 추가하기

## ✏️ axios-mock-adapter로 모킹하기

## ✏️ 목업 사용 여부 제어하기
