# 타입 확장하기ㆍ 좁히기

## 📝 타입 확장하기

- 기존 타입을 사용해서 새로운 타입을 정의하는 것
- 기본적으로 `extends`, `교차 타입`, `유니온 타입`을 사용하여 타입을 확장

### ✏️ 타입 확장의 장점

1. `코드 중복을 제거`할 수 있다.
2. 어떤 타입 확장해서 만들었는지를 쉽게 파악할 수 있는 `명시적인 코드 작성`이 가능하다.
3. 기본 타입을 기반으로 다양한 타입을 생성하는 `확장성`을 가진다.

   ```ts
   // 기본 Base 타입
   interface BaseMenuItem {
     itemName: string | null;
     itemImageUrl: string | null;
     itemDiscountAmount: number;
     stock: number | null;
   }

   interface BaseCartItem extends BaseMenuItem {
     quantity: number;
   }

   interface EditableCartItem extends BaseCartItem {
     isSoldOut: boolean;
   }

   interface EventCartItem extends BaseCartItem {
     orderable: boolean;
   }
   ```

### ✏️ 유니온 타입

- 유니온 타입으로 선언된 값은 유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성에만 접근할 수 있다.

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
    // ❌ Property 'distance' does not exist on type 'CookingStep | DeliveryStep'.
    // ❌ Property 'distance' does not exist on type 'CookingStep'.
  }
  ```

### ✏️ 교차 타입

- 기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입을 만드는 것

  ```ts
  interface DeliveryTip {
    tip: string;
  }

  interface StarRating {
    rate: number;
  }

  type Filter = DeliveryTip & StarRating;

  const filter: Filter = {
    tip: "1000원 이하",
    rate: 4,
  };
  ```

- 교차 타입을 사용할 때 타입이 서로 호환되지 않는 경우

  ```ts
  type IdType = string | number;
  type Numeric = number | boolean;

  type Universal = IdType & Numeric;
  ```

  - Universal 타입을 다음 4가지로 생각해 볼 수 있다.
    1. string 이면서 number 인 경우
    2. string 이면서 boolean 인 경우
    3. number 이면서 number 인 경우
    4. number이면서 boolean 인 경우
  - 두 타입을 모두 만족해야 하기 때문에 Universal의 타입은 number가 된다.

### ✏️ extends와 교차 타입

- extends 키워드를 사용한 타입이 교차 타입과 100% 상응하지 않는다!

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

  type DeliveryTip2 = { tip: number };
  type Filter2 = DeliveryTip2 & { tip: string }; // tip은 never 속성 타입
  ```

- interface와 extends를 이용하는 경우 속성 간의 타입 호환이 되지 않는 경우 에러를 일으키지만, type과 교차 타입을 이용하는 경우에는 속성 간의 타입이 호환되지 않는 경우 never 타입으로 설정된다.
- type 키워드는 교차 타입으로 선언되었을 때 새롭게 추가되는 속성에 대해 미리 알 수 없기 때문에 선언시 에러가 발생하지 않는다.

### ✏️ 배달의민족 메뉴 시스템에 타입 확장 적용하기

```ts
// 1. 하나의 타입에 여러 속성을 추가할 때
interface Menu {
  name: string;
  image: string;
  gif?: string;
  text?: string;
}

// 2. 타입을 확장하는 방법
interface Menu2 {
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

- 다양한 상태를 수용하기 위해 optional로 선언하면, 정작 타입이 꼭 필요한 곳에서 데이터가 없어 오류를 발생시킬 수 있다.
- 적절한 네이밍을 사용해서 타입의 의도를 명확히 표현 가능하고, 코드 작성 단계에서 예기치 못한 버그도 예방 가능하다.

## 📝 타입 좁히기 - 타입 가드

- **타입 좁히기**: 변수 or 표현식의 타입 범위를 더 작은 범위로 좁혀나가는 과정

### ✏️ 타입 가드에 따라 분기 처리하기

#### 분기 처리

- 조건문과 타입 가드를 활용하여 변수나 표현식의 타입 범위를 좁혀 다양한 상황에 따라 다른 동작을 수행하는 것

#### 타입 가드

- 런타임에 조건문을 사용하여 타입을 검사하고 타입 범위를 좁혀주는 기능
- 타입 정보는 컴파일 시 모두 제거되기 되어 런타임에 존재하기 때문에 타입을 사용하여 조건을 만들 수 없다.
- 즉, 컴파일해도 타입 정보가 사라지지 않는 방법을 사용해야 한다.

#### **타입 가드 종류**

1. 자바스크립트 연산자를 사용한 타입 가드(typeof ,instanceof, in)
2. 사용자 정의 타입 가드(is 연산자)

### ✏️ 원시 타입을 추론할 때: **typeof 연산자 활용하기**

- typeof A === B 조건으로 분기 처리하면, 해당 분기 내에서는 A의 타입이 B로 추론된다.
- 다만 typeof는 JS 타입 시스템만 대응할 수 있다.
- null과 배열 타입 등이 object 타입으로 판별되는 등 복합한 타입을 검증하기에는 한계가 존재
- 따라서, `원시 타입을 좁히는 용도로만 사용할 것을 권장한다.`
- typeof 연산자를 사용하여 검사할 수 있는 타입 목록

  - string, number, boolean, undefined, object, function, bigint, symbol

  ```ts
  const replaceHyphen: (date: string | Date) => string | Date = (date) => {
    if (typeof date === "string") {
      // 이 분기에서는 date의 타입이 string으로 추론된다.
      return date.replace(/-/g, "/");
    }

    return date;
  };
  ```

### ✏️ 인스턴스화된 객체 타입을 판별할 때: instanceof 연산자 활용하기

- A instanceof B 형태로 사용하며 A에는 타입을 검사할 대상 변수, B에는 특정 객체의 생성자가 들어간다.
- instanceof는 A의 프로토타입 체인에 생성자 B가 존재하는지를 검사해서 존재한다면 true, 그렇지 않다면 false를 반환한다.
- A의 프로토타입 속성 변화에 따라 instanceof 연산자의 결과가 달라질 수 있다는 점은 유의해야 한다.

  ```ts
  const onKeyDown = (e: React.KeyboardEvent) => {
    if (e.target instanceof HTMLInputElement && e.key === "Enter") {
      // 이 분기에서는 e.target의 타입이 HTMLInputElement이다.
      e.target.blur();
    }
  };
  ```

### ✏️ 객체의 속성이 있는지 없는지에 따른 구분: in 연산자 활용하기

- in 연산자는 객체에 속성이 있는지 확인한 다음 `true` or `false` 를 반환한다.
- **A in B** -> A라는 속성이 B 객체에 존재하는 지를 검사
- JS의 in 연산자는 런타임의 값만을 검사하지만 TS에서는 객체 타입에 속성이 존재하는지를 검사한다.

  ```ts
  // JavaScript
  const obj = { a: 1 };
  console.log("a" in obj); // true
  console.log("b" in obj); // false

  //TypeScript
  interface MyObj {
    a: number;
  }

  const obj: MyObj = { a: 1 };
  console.log("a" in obj); // true
  console.log("b" in obj); // 컴파일 에러 발생 - 정적 타입 정보를 기반으로 프로퍼티 존재 여부를 검사
  ```

### ✏️ is 연산자로 사용자 정의 타입 가드 만들어 활용하기

- 반환 타입이 타입 명제인 함수를 정의하여 사용 가능하다.
- JS 객체를 사용할 때에는 `instanceof`나 `typeof`와 같은 연산자를

#### 타입 명제: 함수의 반환 타입에 대한 타입 가드를 수행하기 위해 사용되는 특별한 형태의 함수

```ts
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet();

if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

## 📝 타입 좁히기 - 식별할 수 있는 유니온(Discriminated Unions)

- 태그된 유니온으로도 불림(타입 좁히기에 널리 사용되는 방식)

### ✏️ 에러 정의하기

```ts
type TextError = {
  errorCode: string;
  errorMessage: string;
};
type ToastError = {
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number;
};
type AlertError = {
  errorCode: string;
  errorMessage: string;
  onConfirm: () => void;
};
type ErrorFeedbackType = TextError | ToastError | AlertError;

const errorArr: ErrorFeedbackType[] = {
  // 정상 작동
  {
    errorCode: '1',
    errorMessage: "에러",
    toastShowDuration: 3000,
    onConfirm: () => {},
  }
};
```

- 🚨 ToastError와 AlertError의 속성을 다 가지고 있는 객체를 만들어도 에러를 뱉지 않는다

### ✏️ 식별할 수 있는 유니온

- `판별자(태그)`(타입마다 구분할 수 있는 속성)를 사용해보자

  ```ts
  type TextError = {
    errorType: "TEXT";
    errorCode: string;
    errorMessage: string;
  };

  type ToastError = {
    errorType: "TOAST";
    errorCode: string;
    errorMessage: string;
    toastShowDuration: number;
  };

  type AlertError = {
    errorType: "ALERT";
    errorCode: string;
    errorMessage: string;
    onConfirm: () => void;
  };

  type ErrorFeedbackType = TextError | ToastError | AlertError;
  const errorArr: ErrorFeedbackType[] = [
    {
      errorType: "TEXT",
      errorCode: "100",
      errorMessage: "텍스트 에러",
    },
    {
      errorType: "TOAST",
      errorCode: "100",
      errorMessage: "토스트 에러",
      toastShowDuration: 3000,
    },
    {
      errorType: "ALERT",
      errorCode: "100",
      errorMessage: "얼럿 에러",
      onConfirm: () => {},
    },
    {
      errorType: "TOAST",
      errorCode: "100",
      errorMessage: "얼럿 에러",
      toastShowDuration: 3000,
      onConfirm: () => {}, // Error 발생 - Object literal may only specify known properties, and 'onConfirm' does not exist in type 'ToastError'.
    },
  ];
  ```

  - errorType 속성을 판별자로 추가하여 구조적 타이핑의 포함 관계를 벗어나도록 만들었다.

### ✏️ 식별할 수 있는 유니온의 판별자 선정

- 판별자의 기준은?
  - `리터럴 타입`이어야 한다.
  - 판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며, 인스턴스화 할 수 있는 타입(instantiable type)은 포함되지 않아야 한다.

## 📝 Exhaustiveness Checking으로 정확한 타입 분기 유지하기

- **Exhaustiveness Checking**이란 가능한 모든 타입 케이스에 대해 철저하게 타입을 검사하는 것을 말하며 타입 좁히기에 사용되는 패러다임 중 하나

  ```ts
  type ProductPrice = "10000" | "20000" | "5000";

  const exhaustiveCheck = (param: never) => {
    throw new Error("type error");
  };

  const getProductName = (productPrice: ProductPrice): string => {
    if (productPrice === "10000") return "배민상품권 1만 원";
    if (productPrice === "20000") return "배민상품권 2만 원";
    else {
      // 모든 분기처리를 해주지 않고 에러를 발생시키는 코드를 추가하지 않으면 실수할 여지가 있다.
      exhaustiveCheck(productPrice); // Error 발생: Argument of type 'string' is not assignable to parameter of type 'never'
      return "배민상품권";
    }
  };
  ```

- 예상치 못한 런타임 에러를 방지하거나 요구사항이 변경되었을 때 생길 수 있는 위험성을 줄일 수 있다.

## 추가적으로 알게 된 내용

### excess property checking(초과 속성 검사)

```ts
// 1번 상황
type Person = {
  name: string;
  age: number;
};

const me: Person = {
  name: "ramin",
  age: 20,
  hobby: "game", // 에러 발생
};

// 2번 상황
const meObj = {
  name: "ramin",
  age: 20,
  hobby: "game",
};

const me2: Person = meObj; // 정상 작동
```

**초과 속성 검사**: 객체 리터럴을 사용하여 새로운 객체를 생성할 때, TS가 추가로 정의되지 않은 속성을 감지하는 기능

- 즉, `객체 리터럴은 반드시 정의된 타입과 완벽히 일치해야 한다.`
- 하지만, 객체 리터럴이 아닌 경우는 상황이 달라집니다.
- 2번 상황의 경우, 타입을 할당된 객체 리터럴의 형태로 추론하기 때문에 `excess property checking`이 적용되지 않습니다.
