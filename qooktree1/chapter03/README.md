# 타입스크립트만의 독자적 타입 시스템

## 📝 any 타입

- 어떠한 값을 할당하더라도 오류가 발생하지 않음(never는 제외)
- 지양해야 할 패턴. But, 어쩔 수 없이 사용해야 하는 경우가 있다.

### 개발 단계에서 임시로 값을 지정해야 할 때

- 타입을 세세하게 명시하는 데 소요되는 시간 절약 가능
- any 타입으로 지정하고 나서 다른 타입으로 바꾸는 과정이 누락되면 문제가 발생하니 주의하자

### 어떤 값을 받아올지 or 넘겨줄지 정할 수 없을 때

- API 요청/응답 처리, 콜백 함수 전달, 외부 라이브러리 등을 사용할 때는 어떤 인자를 주고받을지 특정하기 힘듦

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

### 값을 예측할 수 없을 때 암묵적으로 사용

- 외부 라이브러리나 웹 API의 요청에 따라 다양한 값을 반환하는 API가 존재할 수 있음
  ```ts
  async function load() {
    const res = await fetch("https://api.com");
    const data = await res.json(); // response.json()의 return type은 Promise<any>로 정의되어 있음
    return data;
  }
  ```

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

  ```ts
  function generateError(res: Response): never {
    throw new Error(res.getMessage());
  }
  ```

#### 무한히 함수가 실행되는 경우

- 무한 루프는 함수가 종료되지 않음을 의미 -> 값을 반환하지 못함

## 📝 Array type

- `Object.prototype.toString.call()` 문법을 사용하여 객체의 타입(인스턴스)을 알아낼 수 있음

  ```ts
  Object.prototype.toString.call({}); // [object Object]
  Object.prototype.toString.call(null); // [object Null]
  Object.prototype.toString.call(1); // [object Number]"
  ```

- TS에서는 일반적으로 배열의 크기까지 제한 X

### Tuple type

- 배열 타입의 하위 타입으로 기존 TS의 배열 기능에 길이 제한까지 추가한 타입 시스템

  ```ts
  let tuple: [number] = [1];
  tuple = [1, 2]; // Error
  tuple = [1, "string"]; // Error
  let tuple: [number, string, boolean] = [1, "string", true]; // 여러 Types 혼용 가능
  ```

- useState는 튜플 타입을 반환

  ```ts
  // 첫 번째와 두 번째의 타입이 다름. 두 번째는 setter 함수
  const [num, setNum] = useState(null);
  ```

- 튜플과 배열의 성질을 혼합해서 사용 가능(스프레드 연산자 사용)

  ```ts
  const httpStatusFromPaths: [number, string, ...string[]] = [
    400,
    "Bad Request",
    "/users/:id",
    "/users/:userId",
    "/users/:uuid",
  ];
  ```

- 옵셔널 체이닝 사용가능
  ```ts
  const optionalTuple1: [number, number, number?] = [1, 2];
  const optionalTuple2: [number, number, number?] = [1, 2, 3];
  ```

## 📝 enum type

- 일종의 구조체를 만드는 타입 시스템

- 누락된 멤버는 이전 멤버 값의 숫자를 기준으로 1씩 늘려가며 자동 할당

  ```ts
  enum ProgrammingLanguage {
    TypeScript = 300, // 300
    JavaScript = 400, // 400
    Java, // 401
    Python, // 402
    Go, // 403
  }

  // 객체 속성에 접근하는 방식과 동일
  ProgrammingLanguage.TypeScript; // 400
  ProgrammingLanguage["TypeScript"]; // 400

  // 역방향 접근 가능
  ProgrammingLanguage[400]; // TypeScript
  ```

### enum의 왜 쓸까?

- 주로 문자열 상수를 생성하는 데 사용
- 가독성을 높임

  - enum 타입을 이용하여 타입을 선언하면 해당 enum 타입이 가지는 모든 멤버를 `값`으로 받을 수 있음

  ```ts
  enum ItemStatusType {
    DELIVERY_HOLD = "DELIVERY_HOLD",
    DELIVERY_READY = "DELIVERY_READY",
    DELIVERING = "DELIVERING",
    DELIVERED = "DELIVERED",
  }

  const checkItemAvailable = (itemStatus: ItemStatusType) => {
    switch (itemStatus) {
      case ItemStatusType.DELIVERY_HOLD:
      case ItemStatusType.DELIVERY_READY:
      case ItemStatusType.DELIVERING:
        return false;
      case ItemStatusType.DELIVERED:
      default:
        return true;
    }
  };
  ```

- 단순히 문자열을 넣어주는 것보다 enum 타입을 통해 작성했을 때 다음과 같은 장점이 있음
  1. `타입 안정성`: enum 타입에 명시되지 않은 문자열은 인자로 받을 수 없어 타입 안정성이 우수
  2. `명확한 의미 전달과 높은 응집력`: enum 타입은 관련 있는 정보를 모아 놓기에 응집력이 강하며, 데이터들이 무엇을 의미하는지도 명확
  3. `가독성`: 응집도가 높기 때문에 말하고자 하는 바가 명확하여 어떤 상태를 나타내는지 쉽게 이해 가능

### 🚨 enum 타입 사용 시 주의점

1. 숫자로만 이루어졌거나 TS가 자동으로 추론한 열거형은 안전하지 않은 결과를 초래할 수 있음

   - enum 타입이 숫자 값으로만 이루어졌을 때의 문제점은 할당된 값을 넘어서는 범위를 이용하여 역방향으로 접근하더라도 TS는 이를 막지 않음

   ```ts
   enum ProgrammingLanguage {
     TYPESCRIPT = 400,
     JAVASCRIPT, // 401
     JAVA, // 402
     PYTHON, // 403
     RUST, // 404
   }

   // enum ProgrammingLanguage는 현재 400~404까지가 허용범위이지만
   // 범위를 넘어서는 역방향 접근에 대하여 막지 않는다.
   // 때문에 예상치 못한 곳에서 에러를 발생시킬 위험성이 있다.
   ProgrammingLanguage[410]; // undefined
   ```

   #### 해결방법 - `const enum`

   - 역방향으로의 접근을 허용하지 않음(JS의 객체에 접근하는 것과 유사한 동작)

   ```ts
   const enum ProgrammingLanguage {
     TYPESCRIPT = 400,
   }

   // Error - A const enum member can only be accessed using a string literal
   console.log(ProgrammingLanguage[400]);
   ```

   - **하지만**, 숫자 상수로 관리되는 열거형은 선언한 값 이외의 값을 할당하거나 접근할 때 이를 방지하지 못함. 반면 문자열 상수 방식으로 선언한 열거형은 미리 선언하지 않은 멤버로 접근을 방지

   ```ts
   const enum NUMBER {
     ONE = 1,
     TWO = 2,
   }

   const myNumber: NUMBER = 100; // NUMBER enum에서 100을 관리하고 있지 않지만 이는 에러를 발생 X
   //But, TS 5 version 이상부터는 에러를 잡아준다!!!

   const enum STRING_NUMBER {
     ONE = "ONE",
     TWO = "TWO",
   }
   const myStringNumber: STRING_NUMBER = "THREE"; // Error
   ```

2. enum 타입은 컴파일 때 `즉시 실행 함수` 형식으로 바뀜. 문제는 일부 번들러에서 트리쉐이킹 과정 중 즉시 실행 함수로 변환된 값을 사용하지 않는 코드로 인식하지 못하는 경우가 발생할 수 있음. 따라서 `불필요한 코드로 인해 번들 사이즈가 증가하는 결과를 초래`

### [enum, const enum, 트리쉐이킹 관련 블로그](https://engineering.linecorp.com/ko/blog/typescript-enum-tree-shaking)

# 타입 조합

## 📝 교차 타입(Intersection type)

- `&` 사용
- 여러 가지 타입을 결합하여 하나의 단일 타입으로 생성
- `A & B` -> 타입 A와 타입 B를 모두 만족하는 경우

  ```ts
  type ProductItem = {
    id: number;
    name: string;
  };
  type ProductItemWithPrice = ProductItem & { price: number };
  ```

## 📝 유니언 타입(Union type)

- `|` 사용
- `A | B` -> 타입 A 또는 타입 B 중 하나가 될 수 있는 타입(동시 만족 X)
- 멤버를 접근할 때, 유니언 타입에 선언한 타입 중 하나라도 해당 멤버를 가지고 있지 않다면 참조할 수 없음

  ```ts
  type ProductItem = {
    id: number;
    name: string;
    type: string;
    quantity: number;
  };

  type CardItem = {
    id: number;
    name: string;
    type: string;
  };

  type PromotionEventItem = ProductItem | CardItem;

  const printPromotionItem = (item: PromotionEventItem) => {
    console.log(item.name); // ⭕
    console.log(item.quantity); // ❌ CardItem엔 quantity이 없어서 바로 접근 불가
  };
  ```

## 📝 인덱스 시그니처(Index Signatures)

- 미리 알려지지 않은 속성(필드)의 모양을 정의하는 방법으로 `특정 타입의 key는 알 수 없지만, 값의 타입을 알고 있을 때 사용하는 문법`
- `[key: K]: T` -> 속성 키는 모두 K 타입, 속성 값은 모두 T 타입을 가짐
- key에 설정하는 사용할 수 있는 타입은 `string, number, symbol, template literal`로 제한

- 🚨 인덱스 시그니처를 선언함과 동시에 다른 속성도 추가적으로 명시해줄 수 있는데, 이때 추가로 명시된 속성은 인덱스 시그니처에 포함되는 타입이어야 함

  ```ts
  interface IndexSignature {
    [key: string]: number | boolean;
    length: number;
    isValid: boolean;
    name: string; // 인덱스 시그니처에서 value는 number | boolean만 지정했으므로 string 지정이 불가능
  }
  ```

### 🚨 주의점

- 인덱스 시그니처를 선언한 이후부터는 key의 타입만 일치한다면, `존재하지 않는 속성에 접근 가능`. 이는 value에 지정한 타입과 다르게 undefined가 나올수 있다. 때문에 잠재적인 에러의 요인이 될 수 있음

  ```ts
  type IndexSignatureWithType = {
    [key: string]: string | number;
    length: number;
  };
  let x: IndexSignatureWithType = { name: "type index signature", length: 1 };

  // notExist라는 속성은 없지만 TS에서 에러가 발생 X
  // 또한 noData는 인덱스 시그니처에 설정한 value 타입 string | number 타입으로 추론
  const noData = x["notExist"]; // undefined 반환
  ```

## 📝 인덱스드 엑세스 타입(Indexed Access Types)

- 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용
- 인덱스에 사용되는 타입 또한 그 자체로 타입이기 때문에 Union, keyof, 타입 별칭 등의 표현에 사용 가능

  ```ts
  type Example = {
    a: number;
    b: string;
    c: boolean;
  };

  type IndexedAccess1 = Example["a"]; // number
  type IndexedAccess2 = Example["a" | "b"]; // number | string
  type IndexedAccess3 = Example[keyof Example]; // number | string | boolean

  type ExAlias = "b" | "c";
  type IndexedAccess4 = Example[ExAlias]; // string | boolean
  ```

- 배열에서도 사용 가능

  - 배열의 index는 숫자 타입이므로 이를 대신해 number로 인덱싱하여 배열의 요소를 얻은 후 typeof 연산자를 붙여주면 해당 배열 요소의 타입을 가져올 수 있음

  ```ts
  const PromotionList = [
    { type: "product", name: "chicken" },
    { type: "product", name: "pizza" },
    { type: "card", name: "cheer-up" },
  ];

  type PromotionType = (typeof PromotionList)[number]; // // { type: string, name: string }
  ```

## 📝 템플릿 리터럴 타입(Template Literal Types)

- JS의 템플릿 리터럴 문자열을 사용하여 문자열 리터럴 타입을 선언할 수 있는 문법
  ```ts
  type Stage = "init" | "select-image" | "edit-image";
  type StageName = `${Stage}-stage`; // 'init-stage' | 'select-image-stage' | 'edit-image-stage'
  ```
  - 유니언타입 멤버들이 차례대로 해당 변수에 들어가서 -stage가 붙은 문자열 리터럴의 유니온 타입을 결과로 반환

## 📝 제네릭(Generic)

- 다양한 타입 간에 재사용성을 높이기 위해 사용하는 문법
- 꺾쇠괄호(`<>`)를 통해 표현하며, 사용할 때는 함수에 매개변수를 넣는 것과 유사하게 원하는 타입을 넣어주면 된다.
- 보통 타입 변수명으로 `T(Type), E(Element), K(Key), V(Value), P(Parameter)` 등 한 글자로 된 이름을 많이 사용

#### 제네릭의 기본값 - `=`를 사용하여 추가 가능

```ts
// 제네릭에서 배열에만 존재하는 length 속성은 참조 불가
function exampleFunc<T>(arg: T): number {
  return arg.length; // ❌ Property 'length' does not exist on type 'T'
}

interface TypeWithLength {
  length: number;
}

// length 속성을 가진 타입만 받는다라는 제약을 걸어줌으로써 사용 가능
function exampleFucn2<T extends TypeWithLength>(arg: T): number {
  return arg.length; // ⭕
}
```

### 🚨 주의점

- 파일 확장자가 tsx일 때 화살표 함수에 제네릭을 사용하면 에러 발생
- 이유: tsx: TS + JSX 이므로 제네릭의 꺾쇠괄호와 태그의 꺾쇠괄호를 혼동하여 문제가 발생

#### 해결 방법

- extends 키워드 사용하여 특정 타입의 하위 타입만 올 수 있음을 알려주기

  ```ts
  // ❌ JSX element 'T' has no corresponding closing tag.
  const arrowExampleFunc = <T>(arg: T): T[] => {
    return new Array(3).fill(arg);
  };

  // 어떤 타입의 객체도 받을 수 있다
  const arrowExampleFunc2 = <T extends {}>(arg: T): T[] => {
    return new Array(3).fill(arg);
  };
  ```

## 📝 맵드 타입(Mapped Types)

- 다른 타입을 기반으로 새로운 타입을 선언할 때 사용하는 문법
- JS의 map method라고 생각하면 이해하기 쉽다.

  ```ts
  type Example = {
    a: number;
    b: string;
  };

  type Subset<T> = {
    // in 연산자를 통해 통해 keyof로 추출한 T타입의 key들을 순차적으로 순회
    [K in keyof T]?: T[K];
  };

  const aExample: Subset<Example> = { a: 100 };
  const bExample: Subset<Example> = { b: "string" };
  const cExample: Subset<Example> = { c: true }; // Object literal may only specify known properties, and 'c' does not exist in type 'Subset<Example>'
  ```

  ### 다양한 수식어

  #### `readonly`: 읽기 전용 속성으로 만들 때

  ```ts
  type ReadOnly<T> = {
    readonly [K in keyof T]: T[K];
  };
  ```

  #### `?`: 옵셔널 매개변수 만들 때

  ```ts
  // T의 모든 속성을 옵션 속성으로 변경
  type Optional<T> = {
    [K in keyof T]: T[K];
  };
  ```

  #### `-`: 특정 속성을 제거할 때 조합하여 사용

  ```ts
  // T의 모든 속성을 필수 속성으로 변경
  type Concrete<T> = {
    [K in keyof T]-?: T[K];
  };
  ```

  #### `as`: key의 이름을 재지정 가능

  ```ts
  const BottomSheetMap = {
    RECENT_CONTACTS: RecentContactBottomSheet,
    CARD_SELECT: CardSelectBottomSheet,
    SORT_FILTER: SortFilterBottomSheet,
    PRODUCT_SELECT: ProductSelectBottomSheet,
  };

  type BOTTOM_SHEET_ID = keyof typeof BottomSheetMap;
  type BottomSheetStore = {
    [index in BOTTOM_SHEET_ID as `${index}_BOTTOM_SHEET`]: {
      resolver?: (payload: any) => void;
      args?: any;
      isOpened: boolean;
    };
  };
  ```

# 제네릭 사용법

## 📝 함수의 제네릭

- 함수의 매개변수나 반환 값에 다양한 타입을 넣고 싶을 때 제네릭을 사용 가능

  ```ts
  function ReadOnlyRepository<T>(
    target: ObjectType<T> | EntitySchema<T> | string
  ): Repository<T> {
    return getConnection("ro").getRepository(target);
  }
  ```

## 📝 호출(타입) 시그니처의 제네릭

- 함수 타입을 지정하는 문법으로 매개변수와 반환 타입을 미리 선언하여 타입을 정의하는 것

  ```ts
  type LogFn = (text: string) => void;
  let log: LogFn = (text) => console.log(text);
  ```

## 📝 제네릭 클래스

- 외부에서 입력된 타입을 클래스 내부에 적용할 수 있는 클래스
- 설명중.......

## 📝 제한된 제네릭

- 타입 매개변수에 대한 제약 조건을 설정하는 기능

  ```ts
  type Fruit = {
    name: string;
    price: number;
  };

  function sellFruit<T extends Fruit>(obj: T, key: keyof T): void {
    console.log(obj[key]);
  }

  // 제한된 제네릭으로 상속받은 Fruit 타입과 유사한 타입의 값을 넘겨받을 경우 컴파일 에러가 발생 X
  sellFruit({ name: "banana", price: 1000 }, "name"); // banana
  sellFruit({ name: "apple", price: 2000, color: "red" }, "name"); // apple

  sellFruit({ name: "kiwi", color: "green" }, "name"); // Error 발생
  ```

  - sellFruit 타입 매개변수 T는 Fruit라는 타입으로 제약 조건이 설정되어 있다. 이처럼 타입 매개변수가 특정 타입에 묶여 있을 때 해당 키를 `바운드 타입 매개변수`라 부른다.
  - 상속된 Fruit는 T의 `상한 한계`라고 부른다.

## 📝 확장된 제네릭

- 제네릭 타입은 여러 타입을 상속받을 수 있으며 타입 매개변수를 여러 개 둘 수 있다.

  ```ts
  // 이런 식으로 타입을 제약해버리면 제네릭의 유연성을 읽어버린다.
  <Key extends string>

  // 타입 매개변수에 유니온 타입을 상속할 수 있다.
  <Key extends string | number>
  ```

## 📝 제네릭 예시

- 설명중.......
