# 4장. 타입 확장하기, 좁히기

- 타입 확장하기: 기존에 정의한 타입을 이용하여 새로운 타입을 정의
- 타입 좁히기: 어떤 대상에 대한 타입 추론을 더 작은 범위로 좁힘

## 4-1. 타입 확장하기

### 1. 타입 확장의 장점
- 중복 코드 제거, 명시적인 코드 작성, 확장성

### 2. 유니온 타입
```ts
type unionType = A | B;
/**
 * 💡 유니온 타입은 합집합 같은 개념으로, 유니온 타입에는 변수가 더 쉽게 할당될 수 있다.
 * 대신 변수가 유니온 타입의 특정 타입이라고 단정할 수는 없기에 변수에서 속성 값을 꺼내기는 어려워진다.
 */
```

### 3. 교차 타입
```ts
type intersectionType = A & B;
/**
 * 💡 교차 타입은 교집합 같은 개념으로, 교차 타입에는 A, B의 속성을 모두 가진 변수가 할당될 수 있기 때문에 할당이 어렵다.
 * 대신 변수가 A, B 타입에 둘 다 속하기 때문에 변수에서 속성 값을 꺼내기는 쉬워진다.
 */
```

### 4. extends와 교차 타입
- 교집합 개념의 타입을 정의할 때
  - type 키워드는 &을 사용,
  - interface 키워드는 extends를 사용한다.
- 이 때 extends와 &는 일부 차이가 있다.

```ts
// 1. interface와 extends를 사용할 때
interface DeliveryTip {
  tip: number;
}

// 🚨 자식(Filter)에 부모(DeliveryTip)과 호환되지 않는 속성을 선언하면 에러 발생
interface Filter extends DeliveryTip {
  tip: string;
}

// 2. type과 &를 사용할 때
type DeliveryTip = {
  tip: number;
}

// ✅ 에러가 발생하지는 않는다. 이 때 자식은 never 타입이 된다.
type Filter = DeliveryTip & {tip: string};
```

### 👣 활용하기
일반적인 메뉴 타입이 존재하는데, 요구사하아이 추가되어 특정 메뉴의 속성의 변화가 일어났을 때
```ts
interface Menu {
  name: string;
  image: string;
}

interface SpecialMenu extends Menu {
  gif: string;
}

interface PackageMenu extends Menu {
  text: string;
}
```
💡 이렇게 하나의 타입에 여러 속성을 추가하는 것보다, 확장 타입을 새로 선언하면 해당 타입을 보다 안전하게 사용할 수 있다. 앞으로 props에 무작정 속성을 추가하기 보다, 확장 타입으로 기존 타입과 분리할 수 있는지를 계속 생각해보자.

## 4-2. 타입 좁히기 - 타입 가드
### 1. 타입 가드에 따라 분기 처리하기
- 타입스크립트에서의 분기 처리: 변수나 표현식의 타입 범위를 좁혀 사용하는 것
- 이 때 타입을 사용해서 조건을 만들 수는 없다. 컴파일 시 타입 정보는 제거되어 런타임에 존재하지 않기 때문이다.
- 따라서 특정 문맥 안에서 타입스크립트가 해당 변수를 특정 타입으로 추론하도록 유도하면서 런타임에서도 유효한 방법이 필요하다.
- 이 때 타입 가드를 사용하는데 JS의 typeof, instanceof, in 연산자를 사용한 타입 가드와 사용자 정의 타입 가드가 있다.

### 2. 원시 타입을 추론할 때: typeof 연산자 활용하기
- typeof는 JS 문법으로 JS의 타입 시스템만 대응할 수 있다. null이나 배열의 경우 object 타입으로 판별되는 한계가 있다.
- 따라서 typeof는 주로 **원시 타입**을 좁히는 용도로만 사용하자.

- typeof 연산자가 판별할 수 있는 타입
  - string, number, boolean, undefined, object, function, bigint, symbol

### 3. 인스턴스화된 객체 타입을 판별할 때: instanceof 연산자 활용하기
- A instanceof B: A에 타입을 검사할 대상 변수, **B에 특정 객체의 생성자**를 넣어 인스턴스화된 객체 타입을 판별할 수 있다. 
- A의 프로토타입 체인 상에 B가 존재하는지 검사하여 boolean을 리턴한다.

### 4. 객체의 속성이 있는지 없는지에 따른 구분: in 연산자 활용하기
- in 연산자: **객체에 프로퍼티**가 있는지 확인

#### 👣 활용하기
```ts
interface BasicNoticeDialogProps {
  noticeTitle: string;
  noticeBody: string;
}

interface NoticeDialogWithCookieProps extends BasicNoticeDialogProps {
  cookieKey: string;
  noForADay?: boolean;
  neverAgain?: boolean;
}

export type NoticeDialogProps = BasicNoticeDialogProps | NoticeDialogWithCookieProps;

/**
 * props에 cookieKey 프로퍼티가 있는지 여부로 렌더링하는 컴포넌트를 분기 처리하였다.
 * 💡 한 페이지에서 로그인/비로그인 or 일반/어드민 유저처럼
 *  서로 일부의 기능 차이가 있거나 한 쪽이 일부 기능이 추가된 유저를 분리해서 처리할 때 유용할 것으로 보인다.
 */
const NoticeDialog: React.FC<NoticeDialogProps> = (props) => {
  if('cookieKey' in props) return <NoticeDialogWithCookie {...props} />;
  return <NoticeDialogBase {...props} />;
}
```

### 5. 사용자 정의 타입 가드: is 연산자
- 타입 명제(type predicate): 함수의 반환 타입에 의한 타입 가드를 수행하기 위해 리턴 타입을 `매개변수 is 타입`형태로 정의한 함수
- 💡 타입 가드 vs 단언? -> issue

## 4-3. 타입 좁히기 - 식별할 수 있는 유니온(Discriminated Unions)

### 1. 문제 상황
```ts
type TextError = {
  errorCode: string;
  errorMessage: string;
}

type ToastError = {
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number;
}

type AlertError = {
  errorCode: string;
  errorMessage: string;
  onConfirm: () => void;
}

type ErrorFeedbackType = TextError | ToastError | AlertError;
```
위와 같이 ErrorType을 유니온으로 정의한 경우, 유니온의 각 부분 타입에만 해당하는 속성을 여럿(toastShowDuration, onConfirm) 가져도 타입 에러를 잡아낼 수 없다.

### 2. 식별할 수 있는 유니온
```ts
type TextError = {
  errorType: 'TEXT',
  errorCode: string;
  errorMessage: string;
}

type ToastError = {
  errorType: 'TOAST',
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number;
}

type AlertError = {
  errorType: 'ALERT',
  errorCode: string;
  errorMessage: string;
  onConfirm: () => void;
}

type ErrorFeedbackType = TextError | ToastError | AlertError;
```
이 때 유니온의 각 부분 타입에 **판별자로 사용할 수 있는 고유한 속성(errorType)**을 부여하자. 이렇게 되면 유니온의 부분 타입의 속성을 겹쳐서 갖는 객체가 왔을 때 에러를 잡아낼 수 있다.
```ts
  {
    errorType: 'TOAST', 
    errorCode: '100', 
    errorMessage: '얼럿 에러', 
    toastShowDuration: 3000, 
    onConfirm: () => {}
  }
```

### 3. 판별자 선정

- 판별자는 오직 유닛 타입으로 선언되어, 다른 타입으로 쪼개지지 않고 오직 하나의 정확한 값을 가져야 한다. 판별자가 되기 위한 구체적인 조건은 다음과 같다.
  - 리터럴 타입이어야 한다.
  - 판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며, 인스턴스화할 수 있는 타입은 포함되지 않아야 한다.

## 4-4. Exhaustiveness Checking으로 정확한 타입 분기 유지하기

- 모든 케이스에 대해 분기 처리를 해서 높은 유지보수 코드를 만드는 방식
- 모든 케이스에 대해 분기 처리를 해주지 않았을 때 컴파일 타입 에러가 발생하도록 함

### 👣 활용하기

```ts
type ProductPrice = '10000' | '20000' | '5000';

/**
 * exhaustiveCheck를 통해 5000원에 대한 타입 분기 처리를 하지 않았을 때, 컴파일 에러가 발생하도록 함
 */ 
const getProductName = (productPrice: ProductPrice): string => {
  if(ProductPrice === '1000') return '배민상품권 1만원';
  if(ProductPrice === '2000') return '배민상품권 2만원';
  // if(ProductPrice === '5000') return '배민상품권 5천원';
  else {
    exhaustiveCheck(productPrice);
    return '배민상품권';
  } 
}

const exhaustiveCheck  = (param: never) => {
  throw new Error('type error.');
}
```
