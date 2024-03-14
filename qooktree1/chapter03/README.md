# 타입스크립트만의 독자적 타입 시스템

## 📝 any 타입

- 어떠한 값을 할당하더라도 오류가 발생하지 않음(never는 제외)
- 지양해야 할 패턴. But, 어쩔 수 없이 사용해야 하는 경우가 있다.

### 개발 단계에서 임시로 값을 지정해야 할 때

- 타입을 세세하게 명시하는 데 소요되는 시간 절약 가능
- any 타입으로 지정하고 나서 다른 타입으로 바꾸는 과정이 누락되면 문제가 발생하니 주의하자

### 어떤 값을 받아올지 or 넘겨줄지 정할 수 없을 때

- API 요청/응답 처리, 콜백 함수 전달, 외부 라이브러리 등을 사용할 때는 어떤 인자를 주고받을지 특정하기 힘듦
<details>
    <summary>🔍️ 예제</summary>

```ts
type FeedbackModalParams = {
  show: boolean;
  content: string;
  cancelButtonText?: string;
  confirmButtonText?: string;
  beforeOnClose?: () => void;
  action?: any;
};
```

- 주고받을 값이 명확하지 않을 때 열린 타입(any)을 선언해야 할 수 있음

</details>

### 값을 예측할 수 없을 때 암묵적으로 사용

- 외부 라이브러리나 웹 API의 요청에 따라 다양한 값을 반환하는 API가 존재할 수 있음

<details>
    <summary>🔍️ 예제</summary>

```ts
async function load() {
  const res = await fetch("https://api.com");
  const data = await res.json(); // response.json()의 return type은 Promise<any>로 정의되어 있음
  return data;
}
```

</details>

## 📝 unknown type

- any 타입과 유사하게 모든 타입의 값이 할당될 수 있음
- any를 제외한 다른 타입으로 선언된 변수에는 unknown type 값을 할당 불가능

### 의문 - any type도 있는데 unknown type은 왜 필요할까?

- any 타입을 사용 후 특정 타입으로 수정해야 하는 과정을 누락하면 런타임에 예상치 못한 버그가 발생할 가능성이 있기 때문에 위험함
- 따라서, any타입과 유사하지만 타입 검사를 강제하고 타입이 식별된 후에 사용할 수 있는 unknown type이 등장함

<details>
  <summary>💬 우형 이야기 - any와 unknown</summary>

#### Q. any를 사용한다면 어떤 상황에서 사용할까요?

- 객체의 구조를 정확히 알 수 없는 상황 -> But, 리팩토링하면서 다시 any 타입을 없애는 중
- 표 컴포넌트같이 어떤 값을 받을지 모르는 상황에서 unknown을 사용하면, 가공할 때 타입 캐스팅을 모두 해야 하는 상황이 생겨 any를 사용
- JS에서 TS 마이그레이션 할 때 사용

#### Q. unknown은 어떨 때 사용할 수 있을까요?

- 강제 타입 캐스팅을 통해 타입을 전환할 때 사용
  - `const env = process.env as unknown as ProcessEnv`
- 에러 핸들링할 때 사용
  </details>

## 📝 void type

- `void 타입은 undefined가 아니다!`
  - undefined or null 값을 할당 가능
- 함수가 어떤 값을 반환하지 않는 경우에 void를 지정

## 📝 never type

- 값을 반환할 수 없는 타입
- 모든 타입의 하위 타입
- never 자신을 제외한 어떤 타입도 never 타입에 할당 불가능

### JS에서 값을 반환할 수 없는 경우

#### 에러를 던지는 경우

- throw keyword를 사용하면 에러를 발생시킬 수 있음 -> 값을 반환하는 것으로 간주 X
- 특성 함수가 실행 중 마지막에 `에러를 던지는 작업을 수행하면 반환 타입은 never`
  <details>
    <summary>🔍️ 예제</summary>

  ```ts
  function generateError(res: Response): never {
    throw new Error(res.getMessage());
  }
  ```

  </details>

#### 무한히 함수가 실행되는 경우

- 무한 루프는 함수가 종료되지 않음을 의미 -> 값을 반환하지 못함

## 📝 Array type

-

## 📝 enum type

# 타입 조합

## 📝 Intersection type

## 📝 Union type

## 📝 Index Signatures

## 📝 Indexed Access Types

## 📝 Mapped Types

## 📝 Template Literal Types

## 📝 제네릭

# 제네릭 사용법

## 📝 함수의 제네릭

## 📝 호출 시그니처의 제네릭

## 📝 제네릭 클래스

## 📝 제한된 제네릭

## 📝 확장된 제네릭

## 📝 제네릭 예시
