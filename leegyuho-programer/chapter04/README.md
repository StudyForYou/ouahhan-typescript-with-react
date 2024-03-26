> # 4.1 타입 확장하기

- 기존 타입을 확장하여 새로운 타입을 정의하는 것
- ex) interface, type을 사용하여 타입 정의
- ex) extends, 교차 타입, 유니온 타입을 사용하여 타입 확장

### 타입 확장의 장점

- `코드 중복을 줄일 수 있음`
- `명시적인 코드 작성`
- `확장성`

```ts
interface BaseMenuItem {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
}

interface BaseCartItem extends BaseMenuItem {
  quantity: number;
}
```

```ts
type BaseMenuItem = {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
};

type BaseCartItem = {
  quantity: number;
} & BaseMenuItem;
```

### 유니온 타입

- 2개 이상의 타입을 조합하여사용하는 방법

```ts
interface CookingStep {
  orderId: string;
  price: number;
}

interface DeliveryStep {
  orderId: string;
  time: number;
  distance: string;
}

function getDeliveryDistance(step: CookingStep | DeliveryStep) {
  return step.distance;
  // Property 'discount' does not exist on type 'CookingStep | DeliveryStep'
  // Property 'discount' does not exist on type 'CookingStep'
  // distance 속성을 찾을 수 없기 때문에 에러 발생
}
```

### 교차 타입

- 기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입을 만드는 것

```ts
interface CookingStep {
  orderId: string;
  time: number;
  price: number;
}

interface DeliveryStep {
  orderId: string;
  time: number;
  distance: string;
}

type BaedalProgress = CookingStep & DeliveryStep;

function logBaedalInfo(progress: BaedalProgress) {
  console.log(`주문 금액: ${progress.price}`);
  console.log(`배달 거리: ${progress.distance}`);
}
```

### extends와 교차 타입

```ts
interface BaseMenuItem {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
}

interface BaseCartItem extends BaseMenuItem {
  quantity: number;
}
```

- type으로 선언한 이유: 유니온 타입과 교차 타입을 사용한 새로운 타입은 오직 type 키워드로만 선언할 수 있음

```ts
type BaseMenuItem = {
  itemName: string | null;
  itemImageUrl: string | null;
  itemDiscountAmount: number;
  stock: number | null;
};

type BaseCartItem = {
  quantity: number;
} & BaseMenuItem;

const baseCartItem: BaseCartItem = {
  itemName: '지은이네 떡볶이',
  itemImageUrl: 'https://www.woowahan.com/images/jieun-tteokbokkio.png',
  itemDiscountAmount: 2000,
  stock: 100,
  quantity: 2,
};
```

- extends 키워드를 사용한 타입이 교차 타입과 100% 상응하지 않음

```ts
interface DeliveryTip {
  tip: number;
}

interface Filter extends DeliveryTip {
  tip: string;
  // Interface 'Filter' incorrectly extends interface 'DeliveryTip'
  // Types of property 'tip' are incompatible
  // Type 'string' is not assignable to type 'number'
}

type DeliveryTip = {
  tip: number;
};

type Filter = DeliveryTip & {
  tip: string;
};

// 이렇게 하면 에러가 발생하지 않음
// tip 속성의 타입은 never가 됨 (서로 호환되지 않는 타입이 선언되어서)
```

### 배달의민족 메뉴 시스템에 타입 확장 적용하기

- 주어진 타입에 무분별하게 속성을 추가하여 사용하는 것보다 `타입을 확장해서 사용`하는 것이 좋음
  - 적절한 네이밍을 사용해 타입의 의도 명확히 표현 가능
  - 코드 작성 단계에서 예기치 못한 버그 예방 가능

```ts
// 타입 내에서 속성 추가

interface Menu {
  name: string;
  image: string;
  gif?: string;
  text?: string;
}

// 타입 확장 활용

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

> # 4.2 타입 좁히기 - 타입 가드

- 변수 또는 표현식의 `타입 범위를 더 작은 범위로 좁혀나가는 과정`
- 적절한 네이밍으로 타입의 의도를 명확히 표현 가능
- 코드 작성 단계에서 예기치 못한 버그 예방 가능

### 타입 가드에 따라 분기 처리하기

- 타입스크립트에서의 분기 처리: 조건문과 타입 가드를 활용하여 변수나 표현식의 타입 범위를 좁혀 다양한 상황에 따라 다른 동작을 수행하는 것
- 자바스크립트 연산자를 사용한 타입가드
  - typeof, instanceof, in과 같은 연산자를 사용하여 타입을 좁히는 방식
  - 자바스크립트 연산자를 사용하는 이유는 `런타임에 유효한 타입 가드를 만들기 위해`<br> (런타임에 유효하다 == TS뿐만 아니라 JS에서도 사용할 수 있는 문법이어야 한다.)
- 사용자 정의 타입 가드
  - 사용자가 직접 어떤 타입으로 값을 좁힐지를 지정하는 방식

### 원시 타입을 추론할 때: typeof 연산자 활용하기

- ex) typeof A === B 를 조건으로 분기 처리
- 자바스크립트의 동작 방식으로 인해 null과 배열 타입 등이 object 타입으로 판별되는 등 복잡한 타입을 검증하기에는 한계가 있음
- string, number, boolean, undefined, object, function, bigint, symbol

```ts
const replaceHyphen: (date: string | Date) => string | Date = (date) => {
  if (typeof date === “string”) {
  // 이 분기에서는 date의 타입이 string으로 추론된다
  return date.replace(/-/g, “/”);
  }

  return date;
};
```

### 인스턴스화된 객체 타입을 판별할 때: instanceof 연산자 활용하기

- instanceof 연산자는 인스턴스화된 객체 타입을 판별하는 타입 가드로 사용할 수 있음
- A instanceof B 형태로 사용하며 A에는 타입을 검사할 대상 변수, B에는 특정 객체의 생성자가 들어감
- A의 프로토타입 체인에 생성자 B가 존재하면 true 그렇지 않다면 false를 반환

```ts
const onKeyDown = (event: React.KeyboardEvent) => {
  if (event.target instanceof HTMLInputElement && event.key === “Enter”) {
  // 이 분기에서는 event.target의 타입이 HTMLInputElement이며
  // event.key가 ‘Enter’이다
  event.target.blur();
  onCTAButtonClick(event);
  }
};
```

### 객체의 속성이 있는지 없는지에 따른 구분: in 연산자 활용하기

- in 연산자는 객체에 속성이 있는지 확인한 후 true 또는 false를 반환
- A라는 속성이 B 객체에 존재하는지 검사
- `자바스크립트의 in 연산자는 런타임의 값만을 검사`하지만 `타입스크립트에서는 객체 타입에 속성이 존재하는지를 검사`

```ts
interface Person {
  name: string;
  age: number;
}

const person: Person = {
  name: 'John',
  age: 30,
};

if ('name' in person) {
  console.log('person 객체에 name 속성이 존재합니다.');
} else {
  console.log('person 객체에 name 속성이 존재하지 않습니다.');
}

if ('address' in person) {
  console.log('person 객체에 address 속성이 존재합니다.');
} else {
  console.log('person 객체에 address 속성이 존재하지 않습니다.');
}
```

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

export type NoticeDialogProps =
| BasicNoticeDialogProps
| NoticeDialogWithCookieProps;

const NoticeDialog: React.FC<NoticeDialogProps> = (props) => {
  if (“cookieKey” in props) return <NoticeDialogWithCookie {...props} />; // NoticeDialogWithCookieProps 타입
  return <NoticeDialogBase {...props} />; // BasicNoticeDialogProps 타입
};
```

### is 연산자로 사용자 정의 타입 가드 만들어 활용하기

- 직접 타입 가드 함수를 만들 수 있음
  - `반환 타입이 타입 명제인 함수를 정의`하여 사용 할 수 있음
  - 타입명제: 함수의 반환 타입에 대한 타입 가드를 수행하기 위해 사용되는 특별한 형태의 함수
- ex) A is B -> A는 매개변수 이름, B는 타입 -> 반환 값이 참일 때 A 매개변수의 타입을 B 타입으로 취급

```ts
interface Cat {
  name: string;
  breed: string;
}

interface Dog {
  name: string;
  breed: string;
}

// 사용자 정의 타입 가드 함수
function isCat(animal: Cat | Dog): animal is Cat {
  return (animal as Cat).breed !== undefined;
}

// 사용자 정의 타입 가드를 활용하는 함수
function printPetInfo(animal: Cat | Dog) {
  if (isCat(animal)) {
    console.log(animal.name + ' is a cat of breed ' + animal.breed);
  } else {
    console.log(animal.name + ' is a dog of breed ' + animal.breed);
  }
}

const cat: Cat = { name: 'Whiskers', breed: 'Siamese' };
const dog: Dog = { name: 'Buddy', breed: 'Labrador' };

printPetInfo(cat); // Whiskers is a cat of breed Siamese
printPetInfo(dog); // Buddy is a dog of breed Labrador
```

- 반환 값의 타입이 boolean인 것과 is를 활용한 것의 차이
  - 사용자 정의 타입 가드에서 반환 값이 is를 활용한 경우
    - 해당 함수를 `호출한 곳에서 타입 가드를 통과한 경우에만 해당 타입으로 추론`됨
    - 타입스크립트 컴파일러는 `반환 값이 true인 경우에만 해당 객체를 해당 타입으로 추론`하고 그렇지 않은 경우에는 해당 타입으로 추론되지 않음
    - 타입스크립트 컴파일러가 객체의 타입을 더 정확하게 추론할 수 있음
  - boolean 타입인 경우
    - 해당 함수를 호출한 곳에서는 `항상 해당 타입으로 추론`됨
    - 즉, `반환 값이 true 또는 false인 경우에도 해당 타입으로 추론`됨
    - 타입 가드가 `실패하더라도 해당 객체는 여전히 해당 타입으로 추론될 수 있음`
- 따라서 is를 활용한 사용자 정의 타입 가드는 보다 정확한 타입 추론을 가능하게 하며, 코드의 가독성과 안정성을 높일 수 있음

```ts
// 사용자 정의 타입 가드에서 반환 값이 is를 활용한 경우

interface Cat {
  name: string;
  breed: string;
}

interface Dog {
  name: string;
  breed: string;
}

// 사용자 정의 타입 가드 함수
function isCat(animal: Cat | Dog): animal is Cat {
  return (animal as Cat).breed !== undefined;
}

// 사용자 정의 타입 가드를 활용하는 함수
function printPetInfo(animal: Cat | Dog) {
  if (isCat(animal)) {
    // animal은 여기서 Cat 타입으로 추론됩니다.
    console.log(animal.name + ' is a cat of breed ' + animal.breed);
  } else {
    // animal은 여기서 Dog 타입으로 추론됩니다.
    console.log(animal.name + ' is a dog of breed ' + animal.breed);
  }
}

const cat: Cat = { name: 'Whiskers', breed: 'Siamese' };
const dog: Dog = { name: 'Buddy', breed: 'Labrador' };

printPetInfo(cat); // Whiskers is a cat of breed Siamese
printPetInfo(dog); // Buddy is a dog of breed Labrador
```

```ts
// boolean 타입인 경우

interface Animal {
  name: string;
}

function isCat(animal: Animal): boolean {
  // 이 함수는 간단한 예시를 위해 동물이 고양이인지 확인하는 것으로 가정합니다.
  // 실제로는 보다 복잡한 로직이 포함될 수 있습니다.
  return animal.name === 'Whiskers';
}

function printAnimalType(animal: Animal) {
  if (isCat(animal)) {
    // animal은 여기서 Animal 타입으로 추론됩니다.
    console.log(animal.name + ' is a cat.');
  } else {
    // animal은 여기서 Animal 타입으로 추론됩니다.
    console.log(animal.name + ' is not a cat.');
  }
}

const cat: Animal = { name: 'Whiskers' };
const dog: Animal = { name: 'Buddy' };

printAnimalType(cat); // Whiskers is a cat.
printAnimalType(dog); // Buddy is not a cat.
```

> # 4.3 타입 좁히기 - 식별할 수 있는 유니온(Discriminated Unions)

- 태그된 유니온으로도 불리는 식별할 수 있는 유니온은 타입 좁히기에 널리 사용되는 방식

### 에러 정의하기

```ts
type TextError = {
  errorCode: string;
  errorMessage: string;
};
type ToastError = {
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number; // 토스트를 띄워줄 시간
};
type AlertError = {
  errorCode: string;
  errorMessage: string;
  onConfirm: () => void; // 얼럿 창의 확인 버튼을 누른 뒤 액션
};
```

```ts
type ErrorFeedbackType = TextError | ToastError | AlertError;
const errorArr: ErrorFeedbackType[] = [
  { errorCode: “100”, errorMessage: “텍스트 에러” },
  { errorCode: “200”, errorMessage: “토스트 에러”, toastShowDuration: 3000 },
  { errorCode: “300”, errorMessage: “얼럿 에러”, onConfirm: () => {} },
];
```

- 각 에러 타입을 위와 같이 정의 하고 에러 타입의 유니온 타입을 원소로 하는 배열을 정의해보면 위와 같이 나옴
- ToastError의 toastShowDuration 필드와 AlertError의 onConfirm 필드를 모두 가지는 개체에 대해서는 타입 에러를 뱉어야 함

```ts
const errorArr: ErrorFeedbackType[] = [
  // ...
  {
  errorCode: “999”,
  errorMessage: “잘못된 에러”,
  toastShowDuration: 3000,
  onConfirm: () => {},
  }, // expected error
];
```

- BUT 자바스크립트는 덕타이핑 언어이기 때문에 별도의 타입 에러를 뱉지 않음
- 앞으로의 개발 과정에서 의미를 알 수 없는 무수한 에러 객체가 생겨날 위험성 커짐

### 식별할 수 있는 유니온

- 타입 간의 구조 호환을 막기 위해 타입마다 구분할 수 있는 판별자를 달아주어 포함 관계를 제거하는 것

```ts
// 판별자 개념으로 errorType이라는 필드 정의
// 각 에러 타입마다 이 필드에 대해 다른 값을 가지도록 하여 판별자를 달아주면 이들은 포함 관계를 벗어남

type TextError = {
  errorType: “TEXT”;
  errorCode: string;
  errorMessage: string;
};
type ToastError = {
  errorType: “TOAST”;
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number;
}
type AlertError = {
  errorType: “ALERT”;
  errorCode: string;
  errorMessage: string;
  onConfirm: () = > void;
};
```

- 이렇게 하면 정확하지 않은 에러 객체에 대해 타입 에러가 발생

```ts
type ErrorFeedbackType = TextError | ToastError | AlertError;

const errorArr: ErrorFeedbackType[] = [
  { errorType: “TEXT”, errorCode: “100”, errorMessage: “텍스트 에러” },
  {
    errorType: “TOAST”,
    errorCode: “200”,
    errorMessage: “토스트 에러”,
    toastShowDuration: 3000,
  },
  {
    errorType: “ALERT”,
    errorCode: “300”,
    errorMessage: “얼럿 에러”,
    onConfirm: () => {},
  },
  {
    errorType: “TEXT”,
    errorCode: “999”,
    errorMessage: “잘못된 에러”,
    toastShowDuration: 3000, // Object literal may only specify known properties, and ‘toastShowDuration’ does not exist in type ‘TextError’
    onConfirm: () => {},
  },
  {
    errorType: “TOAST”,
    errorCode: “210”,
    errorMessage: “토스트 에러”,
    onConfirm: () => {}, // Object literal may only specify known properties, and ‘onConfirm’ does not exist in type ‘ToastError’
  },
  {
    errorType: “ALERT”,
    errorCode: “310”,
    errorMessage: “얼럿 에러”,
    toastShowDuration: 5000, // Object literal may only specify known properties, and ‘toastShowDuration’ does not exist in type ‘AlertError’
  },
];
```

### 식별할 수 있는 유니온의 판별자 선정

- 식별할 수 있는 유니온의 판별자는 `유닛 타입으로 선언되어야 정상적으로 동작`함
- 유닛 타입: 다른 타입으로 쪼개지지 않고 오직 하나의 정확한 값을 가지는 타입
  - null, undefined, true, 1 등 정확한 값을 나타내는 타입이 해당
  - void, string, number 등은 다양한 타입을 할당할 수 있기 때문에 유닛 타입으로 적용 X
- 유니온의 판별자로 사용할 수 있는 타입
  - 리터럴 타입이어야 함
  - 판별자로 선정한 값에 `적어도 하나 이상의 유닛 타입이 포함`되어야 하며, `인스턴스화할 수 있는 타입은 포함되지 않아야 함`

> # 4.4 Exhaustiveness Checking으로 정확한 타입 분기 유지하기

- `모든 케이스에 대해 철저하게 타입을 검사하는 것`을 말함
- `컴파일 타임` 에러가 발생하게 하는 것
- `예상치 못한 런타임 에러를 방지`하거나 요구사항이 변경되었을 때 생길 수 있는 `위험성을 줄일 수 있음`

### 상품권

- 상품권 가격에 따라 상품권 이름을 반환해주는 함수를 작성하면 다음과 같음

```ts
type ProductPrice = “10000” | “20000”;

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === “10000”) return “배민상품권 1만 원”;
  if (productPrice === “20000”) return “배민상품권 2만 원”;
  else {
    return “배민상품권”;
  }
};
```

- 새로운 상품권이 생겨서 ProductPrice 타입이 업데이터 되어야 한다고 하면 아래와 같이 업데이트 할 수 있음
- ProductPrice가 업데이트 되었을 때 getProductName 함수도 함께 업데이트 되어야 함
- 그러나 getProductName 함수를 수정하지 않아도 별도 에러가 발생하지 않기 때문에 실수할 여지가 있음

```ts
type ProductPrice = “10000” | “20000” | “5000”;

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === “10000”) return “배민상품권 1만 원”;
  if (productPrice === “20000”) return “배민상품권 2만 원”;
  if (productPrice === “5000”) return “배민상품권 5천 원”; // 조건 추가 필요
  else {
    return “배민상품권”;
  }
};
```

- 모든 타입에 대한 타입 검사를 강제하고 싶다면 아래와 같이 할 수 있음

```ts
type ProductPrice = “10000” | “20000” | “5000”;

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === “10000”) return “배민상품권 1만 원”;
  if (productPrice === “20000”) return “배민상품권 2만 원”;
  // if (productPrice === “5000”) return “배민상품권 5천 원”;
  else {
    exhaustiveCheck(productPrice); // Error: Argument of type ‘string’ is not assignable to parameter of type ‘never’
    return “배민상품권”;
  }
};

const exhaustiveCheck = (param: never) => {
  throw new Error(“type error!”);
};
```

- 5000일 때 분기 처리가 주석 처리되어 있기 때문에 에러가 발생함
- 이 함수는 매개변수를 never 타입으로 선언하여 매개변수로 어떤 값도 바을 수 없으면 만일 값이 들어온다면 에러를 내뱉음
- 타입 처리 조건문의 마지막 else문에 사용하면 앞의 조건문에서 `모든 타입에 대한 분기 처리 강제 가능`
