# 타입 활용하기

## 📝 조건부 타입

- 기존 타입을 사용해서 새로운 타입을 정의하는 것
- 기본적으로 `extends`, `교차 타입`, `유니온 타입`을 사용하여 타입을 확장

### ✏️ extends와 제네릭을 활용한 조건부 타입

- `T extends U ? X : Y`
- 타입 T를 U에 할당할 수 있으면 X 타입, 없으면 Y 타입으로 결정됨

  ```ts
  interface Bank {
    financialCode: string;
    fullName: string;
  }
  interface Card {
    financialCode: string;
    appCardType?: string;
  }
  type PayMethod<T> = T extends "card" ? Card : Bank;
  type CardPayMethodType = PayMethod<"card">;
  type BankPayMethodType = PayMethod<"bank">;
  ```

  - PayMethod의 매개변수가 "card"일 시 Card 타입, 아닐 시 Bank 타입

### ✏️ 조건부 타입을 사용하지 않았을 때의 문제점

#### React-Query + TypeScript 프로젝트 예시 코드

- 계좌(bank), 카드(card), 앱카드(appcard) 3가지의 결제 수단이 있다.
- 주어진 결제 수단 타입을 서버 응답을 처리하는 공통 함수(`useGetRegisteredList`)에 전달
- 각 API를 통해 결제 수단 정보를 배열로 받아와 최종적으로 필터링된 배열(`result`)을 반환하는 코드

  ```ts
  type PayMethodType = PayMethodInfo<Card> | PayMethodInfo<Bank>;

  export const useGetRegisteredList = (
    type: "card" | "appcard" | "bank"
  ): UseQueryResult<PayMethodType[]> => {
    const url = `baeminpay/codes/${type === "appcard" ? "card" : type}`;
    const fetcher = fetcherFactory<PayMethodType[]>({
      onSuccess: (res) => {
        const usablePocketList =
          res?.filter(
            (pocket: PocketInfo<Card> | PocketInfo<Bank>) =>
              pocket?.useType === "USE"
          ) ?? [];
        return usablePocketList;
      },
    });
    const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);
    return result;
  };
  ```

  - 하지만, 타입을 명확히 구분하는 로직이 없다.
  - 사용자가 인자로 "`card`"를 전달했을 때 함수가 반환하는 타입인 PayMethodType이 유니온으로 되어 있기 때문에 구체적으로 추론 불가능(`PayMethodInfo<Card>[]`면 좋겠지만 구체적으로 추론 X)

### ✏️ extends 조건부 타입을 활용하여 개선하기

```ts
type PayMethodType<T extends "card" | "appcard" | "bank"> = T extends
  | "card"
  | "appcard"
  ? Card
  : Bank;
```

- PayMethodType의 제네릭으로 받은 값이 "card" or "appcard"면 `PayMethodInfo<Card>` 타입을, 아니라면 `PayMethodInfo<Bank>` 타입을 반환

```ts
export const useGetRegisteredList = <T extends "card" | "appcard" | "bank">(
  type: T
): UseQueryResult<PayMethodType<T>[]> => {
  /* ... */
  const result = useCommonQuery<PayMethodType<T>[]>(url, undefined, fetcher);
  return result;
};
```

#### `활용 예시 정리`

1. 제네릭과 extends를 함께 사용해 제네릭으로 받는 타입을 제한

   - 잘못된 값을 넘길 수 없기 때무에 휴먼 에러 방지

2. extends를 활용해 조건부 타입을 설정

   - 반환 값을 사용자가 원하는 값으로 구체화 가능
   - 불필요한 타입 가드, 타입 단언 등을 방지
   - 불필요한 타입 가드와 불필요한 타입 단언을 하지 않아도 된다!

### ✏️ infer를 활용해서 타입 추론하기

- extends를 사용할 때 `infer` 키워드를 사용 가능

```ts
type UnpackPromise<T> = T extends Promise<infer K>[] ? K : any;
const promises = [Promise.resolve("Mark"), Promise.resolve(38)];
type Expected = UnpackPromise<typeof promises>; // string | number
```

## 📝 템플릿 리터럴 타입 활용하기

- JS의 템플릿 리터럴 문법을 사용해 특정 문자열에 대한 타입을 선언할 수 있는 기능
- 타입스크립트 4.1부터 지원하고 있는 기능이다.

  ```ts
  type Vertical = "top" | "bottom";
  type Horizon = "left" | "right";

  type Direction = Vertical | `${Vertical}${Capitalize<Horizon>}`;
  // "top" | "topLeft" | "topRight" | "bottom" | "bottomLeft" | "bottomRight"
  ```

  - 더욱 읽기 쉬운 코드 작성 가능
  - 코드를 재사용하고 수정하는 데 용이한 타입 선언 가능

### 🚨 주의할 점

- TS 컴파일러가 유니온을 추론하는 데 시간이 오래 걸리면 비효율적이기 때문에 TS가 타입을 추론하지 않고 에러를 내뱉을 때가 있다.
- 템플릿 리터럴 타입에 삽입된 `유니온 조합의 경우의 수가 너무 많지 않게 적절하게 나누어 타입을 관리`하는 것이 좋다.

  ```ts
  type Digit = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9;
  type Chunk = `${Digit}${Digit}${Digit}${Digit}`;
  type PhoneNumberType = `010-${Chunk}-${Chunk}`; // Error - Expression produces a union type that is too complex to represent.
  ```

  - 위의 예제일 경우 PhoneNumberType은 10000 \*\* 2의 경우의 수를 가지고 있는 유니온 타입이 되기 때문에 TS에서 에러가 발생할 수 있다.

## 📝 커스텀 유틸리티 타입 활용하기

### ✏️ 유틸리티 함수를 활용해 styled-components의 중복 타입 선언 피하기

- `Pick`, `Omit` 과 같은 유틸리티 함수를 활용하여 중복되는 코드를 제거할 수 있다.
- 유틸리티 함수를 사용하면 중복 코드 제거 및 유지보수에 용이

  ```ts
  export type Props = {
    height?: string;
    color?: keyof typeof colors;
    isFull?: boolean;
    className?: string;
    //...
  };

  export const Hr: VFC<Props> = ({ height, color, isFull, className }) => {
    //...
    return (
      <HrComponent
        height={height}
        color={color}
        isFull={isFull}
        classNane={className}
      ></HrComponent>
    );
  };

  //styles.ts
  //Pick 유틸리티 타입 활용: 특정 타입에서 몇 개의 속성을 선택하여 타입을 정의
  type StyledProps = Pick<Props, "height" | "color" | "isFull">;
  const HrComponent = styled.hr<StyledProps>`
    height: ${({ height }) => height || "10px"};
    background-color: ${({ color }) => colors[color || "gray7"]};
    ${({ isFull }) =>
      isFull &&
      css`
        margin: 0 -16px;
      `}
  `;
  ```

  #### Pick 과 Omit

  - `Pick` - 특정 타입에서 몇 개의 속성을 선택하여 타입을 정의
  - `Omit` - 특정 속성만 제거한 타입을 정의

  ```ts
  interface Product {
    id: number;
    name: string;
    price: number;
    brand: string;
    stock: number;
  }

  type shoppingInfo = Pick<Product, "id" | "name">;
  /*
  type shoppingInfo = {
    id: number;
    name: string;
  }
  */
  type shoppingInfo2 = Omit<Product, "id" | "name">;
  /*
  type shoppingInfo2 = {
    price: number;
    brand: string;
    stock: number;
  }
  */
  ```

### ✏️ PickOne 유틸리티 함수

#### 🚨 문제 상황

- TS는 서로 다른 2개 이상의 객체를 유니온 타입으로 받을 때 타입 검사가 제대로 되지 않는 이슈가 있다.

  ```ts
  type Card = {
    card: string;
  };
  type Account = {
    account: string;
  };
  function withdraw(type: Card | Account) {}
  withdraw({ card: "hyundai", account: "hana" }); // Card와 Account 속성을 한 번에 받아도 에러 없음
  ```

  - 유니온은 집합 관점으로 볼 때 합집합이 되기 때문에 card, account 모두 포함되어도 합집합의 범주에 들어가기 때문에 타입에러가 발생하지 않는다.

  ### 해결방법 1. 식별할 수 있는 유니온 사용

  ```ts
  type Card = {
    type: "card"; // 판별자 추가
    card: string;
  };
  type Account = {
    type: "account"; // 판별자 추가
    account: string;
  };
  function withdraw(type: Card | Account) {}
  withdraw({ card: "hyundai", account: "hana" }); // 에러 발생 - Argument of type '{ card: string; account: string; }' is not assignable to parameter of type 'Card | Account'.
  withdraw({ type: "card", card: "hyundai" });
  withdraw({ type: "account", account: "hana" });
  ```

  - 일일이 type을 다 넣어줘야 하는 불편함이 있다.
  - 이미 구현된 상태에서 식별할 수 있는 유니온을 적용하려면 해당 함수를 사용하는 부분을 모두 수정해야하는 번거러움도 있다.

  ### 해결방법 2. PickOne 커스텀 유틸리티 타입 구현하기

  - 하나의 속성이 들어왔을 때 다른 타입을 옵셔널한 undefined 값으로 지정하는 방법
  - -> 사용자가 의도적으로 undefined 값을 넣지 않는 이상, 원치 않은 속성에 값을 넣었을 때 타입 에러가 발생한다.

  ```ts
  type PickOne<T> = {
    [P in keyof T]: Record<P, T[P]> &
      Partial<Record<Exclude<keyof T, P>, undefined>>;
  }[keyof T];
  ```

### ✏️ NonNullable 타입 검사 함수를 사용하여 간편하게 타입 가드하기

#### NonNullable 타입

- TS에서 제공하는 유틸리티 타입으로 제네릭으로 받는 T가 null or undefined일 때 never or T를 반환하는 타입
- null이나 undefined가 아닌 경우를 제외하기 위해 사용

```ts
type NonNullable<T> = T extends null | undefiend ? never : T;
```

#### NonNullable 커스텀 함수

```ts
function NonNullable<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined;
}
```

- 매개변수(value)가 null 또는 undefined일 때 false를 반환하는 함수
- 반환값이 true라면 null과 undefined가 아닌 다른 타입으로 타입 가드된다.

#### Promise.all 사용 시 NonNullable 적용 예시

```ts
const shopList = [
  { shopNo: 100, category: "chicken" },
  { shopNo: 101, category: "pizza" },
  { shopNo: 102, category: "noodle" },
];

// 반환 타입이 Array<AdCampaign[] | null>로 추론된다.
class AdCampaignAPI {
  static async operating(shopNo: number): Promise<AdCampaign[]> {
    try {
      return await fetch(`/ad/shopNumber=${shopNo}`);
    } catch (error) {
      return null;
    }
  }
}

const shopAdCampaignList = await Promise.all(
  shopList.map((shop) => AdCampaignAPI.operating(shop.shopNo))
);
```
