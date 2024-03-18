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

- 유니온 타입으로 선언된 값은 유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성에만 접근할 수 있다.

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

## 📝 타입 좁히기 - 식별할 수 있는 유니온(Discriminated Unions)

## 📝 Exhaustiveness Checking으로 정확한 타입 분기 유지하기
