> # 5.1 조건부 타입

### 1. extends와 제네릭을 활용한 조건부 타입

- extends 키워드는 `타입을 확장할 때`와 `타입을 조건부로 설정할 때` 그리고 `제네릭 타입에서는 한정자 역할`로 사용됨

```ts
T extends U ? X : Y

// 타입 T를 U에 할당할 수 있으면 X 타입, 아니면 Y 타입으로 결정됨을 의미
```

```ts
interface Bank {
  financialCode: string;
  companyName: string;
  name: string;
  fullName: string; // Card와의 차이
}
interface Card {
  financialCode: string;
  companyName: string;
  name: string;
  appCardType?: string; // Bank와의 차이
}
type PayMethod<T> = T extends 'card' ? Card : Bank; // 제네릭 타입으로 extends를 사용한 조건부 타입
type CardPayMethodType = PayMethod<'card'>; // 제네릭 매개변수 = "card" : Card 타입
type BankPayMethodType = PayMethod<'bank'>; // 제네릭 매개변수 != "card" : Bank 타입
```

### 2. 조건부 타입을 사용하지 않았을 때의 문제점

```ts
interface Dog {
  type: 'dog';
}
interface Cat {
  type: 'cat';
}

type Animal = Dog | Cat;

const pickAnimal = (type: 'dog' | 'cat'): Array<Animal> => {
  const data = [{ type: 'dog' }, { type: 'cat' }];

  return data.filter((v) => v.type === type);
};

const result = pickAnimal('dog');
// 값: [{type: 'dog'}]
// 타입: Animal[]
```

- 위 코드에서 pickAnimal 함수는 type 매개변수를 받아서 해당하는 동물의 배열을 반환한다. 그러나 이 함수는 정확한 타입 정보를 제공하지 않습니다. 아래는 이러한 접근 방식에서 발생할 수 있는 문제점이다.

1. `정확한 타입 정보 부재`: pickAnimal 함수는 Animal 배열을 반환합니다. 하지만 반환된 배열의 요소가 정확히 어떤 동물인지에 대한 정보는 제공하지 않는다. 예를 들어, result 배열의 요소가 { type: 'dog' }임에도 불구하고 해당 요소가 실제로 Dog 타입인지, 또는 Animal 타입인지 명확하지 않다.

2. `타입 안정성 부족`: 반환된 배열의 각 요소가 Dog 또는 Cat 타입일 것으로 예상되지만, 실제로는 단지 Animal 타입으로 추론된다. 따라서 배열의 요소를 사용할 때마다 타입 캐스팅이 필요할 수 있으며, 이는 타입 안정성을 저해할 수 있다.

3. `확장성 문제`: 현재는 Dog와 Cat만 존재하지만, 새로운 동물 타입이 추가될 경우 코드를 수정해야 한다. 이는 유지보수와 확장성에 문제를 일으킬 수 있다.

- 이러한 문제를 해결하고 위험을 최소화하기 위해서는 `조건부 타입을 사용하여 정확한 타입 정보를 제공하고, 타입 안정성을 확보`하는 것이 좋다.

### 3. extends 조건부 타입을 활용하여 개선하기

- extends를 한정자로 활용해서 `제네릭에 넘겨오는 값을 제한`할 수 있음
- 유니온 타입이였던 Animal 타입을 조건부 타입으로 변경해 조건에 맞는 타입을 반환하도록 수정해 문제를 해결

```ts
interface Dog {
  type: 'dog';
}
interface Cat {
  type: 'cat';
}

type Animal<T extends 'dog' | 'cat'> = T extends 'dog' ? Dog : Cat;

const pickAnimal = <T extends 'dog' | 'cat'>(type: T): Array<Animal<T>> => {
  const data = [{ type: 'dog' }, { type: 'cat' }];

  return data.filter((v) => v.type === type);
};

const result = pickAnimal('dog');
// 값: [{type: 'dog'}]
// 타입: Dog[]
```

- extends 활용 예시
  - 제네릭과 extends를 함께 사용하여 `제네릭으로 받는 타입을 제한`<br> -> `에러 방지`
  - extends를 활용하여 `조건부 타입 설정`(반환 값을 사용자가 원하는 값으로 구체화)<br> -> `불필요한 타입 가드, 타입 단언 방지`

### 4. infer를 활용해서 타입 추론하기

- 삼항 연산자를 사용한 조건문의 형태를 가지는데 `extends로 조건을 서술`하고 `infer로 타입을 추론`하는 방식을 취함

```ts
type UnpackPromise<T> = T extends Promise<infer K>[] ? K : any;

// UnpackPromise 타입은 제네릭으로 T를 받아 T가 Promise로 래핑된 경우라면 K를 반환하고 그렇지 않은 경우 any를 반환
// Promise<infer K> : Promise의 반환 값을 추론해 해당 값의 타입을 K라고 지정

const promises = [Promise.resolve('Mark'), Promise.resolve(38)];
type Expected = UnpackPromise<typeof promises>; // string | number
```

> # 5.2 템플릿 리터럴 타입 활용하기

- 타입스크립트에서는 유니온 타입을 사용하여 변수 타입을 특정 문자열로 지정할 수 있음
- `휴먼 에러 방지` 가능
- 자동 완성 기능을 통해 `개발 생산성을 높일 수 있음`
- 템플릿 리터럴 타입에 삽입된 유니온 조합의 경우의 수가 너무 많지 않게 적절하게 나누어 타입을 정의하는 것이 좋음

```ts
type HeaderTag = 'h1' | 'h2' | 'h3' | 'h4' | 'h5';
type HeadingNumber = 1 | 2 | 3 | 4 | 5;
type HeaderTag = `h${HeadingNumber}`;
```

```ts
// before
type Direction = 'top' | 'topLeft' | 'topRight' | 'bottom' | 'bottomLeft' | 'bottomRight';

// after
type Vertical = 'top' | 'bottom';
type Horizon = 'left' | 'right';

type Direction = Vertical | `${Vertical}${Capitalize<Horizon>}`;
```

> # 5.3 커스텀 유틸리티 타입 활용하기

- 타입스크립트에서 제공하는 유틸리티 타입만으로 표현하는 데 한계를 느낄 때 유틸리티 타입을 활용한 `커스텀 유틸리티 타입` 제작

### 1. 유틸리티 함수를 활용해 styled-components의 중복 타입 선언 피하기

- styled-components를 사용할 때 `타입이 중복`되는 경우가 많다.<br>
  -> Pick, Omit 같은 유틸리티 타입을 활용하여 간단하게 가능

#### 유틸리티 타입을 활용하지 않았을 때 발생하는 불편함

- Props 타입과 styled-components 타입의 중복 선언 및 문제점

```ts
// HrComponent.tsx
export type Props = {
  height?: string;
  color?: keyof typeof colors;
  isFull?: boolean;
  className?: string;
};

export const Hr: VFC<Props> = ({ height, color, isFull, className }) => {
  /* ... */
  return (
    <HrComponent
      height={height}
      color={color}
      isFull={isFull}
      className={className}
    />
  );
};

// style.ts
import { Props } from '../HrComponent.tsx';

type StyledProps = Pick<Props, "height" | "color" | "isFull">;

const HrComponent = styled.hr<StyledProps>`
  height: ${({ height }) = > height || "10px"};
  margin: 0;
  background-color: ${({ color }) = > colors[color || "gray7"]};
  border: none;
  ${({ isFull }) => isFull && css`
    margin: 0 -15px;
  `}
`;
```

- `Pick`
  - 기존 타입에서 선택적으로 속성을 선택하여 새로운 타입을 만들 때 사용
  - 코드를 간결하게 작성하고 재사용성을 높일 수 있음

### 2. PickOne 유틸리티 함수

- 타입스크립트에는 서로 다른 2개 이상의 객체를 유니온 타입으로 받을 때 타입 검사가 제대로 진행되지 않는 이슈가 있음

```ts
type Card = {
  card: string;
};
type Account = {
  account: string;
};
function withdraw(type: Card | Account) {
  /* ... */
}
withdraw({ card: 'hyundai', account: 'hana' });
// Card와 Account 속성을 한 번에 받아도 에러 없음
```

- Card, Account 중 하나의 객체만 받고 싶은 상황에서 Card | Account로 타입을 작성하면 의도대로 타입 검사가 되지 않음
- 타입 에러가 발생하지 않은 이유??
  - 집합 관점으로 볼 때 유니온은 합집합이 되기 때문
  - 이런 문제를 해결하기 위해 식별할 수 있는 유니온 기법 사용

#### 식별할 수 있는 유니온으로 객체 타입을 유니온으로 받기

```ts
type Card = {
  type: 'card';
  card: string;
};

type Account = {
  type: 'account';
  account: string;
};

function withdraw(type: Card | Account) {
  //...
}

withdraw({ type: 'card', card: 'hyundai' });
```

- 일일이 type을 다 넣어줘야 하는 불편함
- 실수로 수정하지 않은 부분이 생기면 문제 발생
- 이러한 상황을 방지하기 위해 PickOne이라는 유틸리티 타입 구현

#### PickOne 커스텀 유틸리티 타입 구현하기

```ts
type PickOne<T> = {
  [P in keyof T]: Record<P, T[P]> & Partial<Record<Exclude<keyof T, P>, undefined>>;
}[keyof T];
```

- 선택한 하나의 타입을 제외한 다른 타입을 옵셔널한 undefined 값으로 지정

```ts
// before
type Card = {
  card: string;
};
type Account = {
  account: string;
};
function withdraw(type: Card | Account) {
  /* ... */
}
withdraw({ card: "hyundai", account: "hana" });
// Card와 Account 속성을 한 번에 받아도 에러 없음

// after
function withdraw(type: PickOne<Card & Account>) {
  ...
}

withdraw({ card: "hyundai", account: "hana" }); // type error
```

- PickOne을 활용하여 코드를 수정하면 에러가 발생하는 것을 볼 수 있음

### 3. NonNullable 타입 검사 함수를 사용하여 간편하게 타입 가드하기

- null을 가질 수 있는 값의 null 처리는 자주 사용되는 타입 가드 패턴
- 일반적으로 if문을 사용해서 null 처리 타입 가드를 적용
- is 키워드와 NonNullable 타입으로 탕비 검사를 위한 유틸 함수르 만들어 사용 가능

#### NonNullable 타입이란

```ts
type NonNullable<T> = T extends null | undefined ? never : T;
```

- 제네릭으로 받는 T가 null 또는 undefined일 때 never 또는 T를 반환하는 타입

#### null, undefined를 검사해주는 NonNullable 함수

```ts
function NonNullable<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined;
}
```

- value가 null 또는 undefined라면 false를 반환
- is 키워드가 사용되었기 때문에 NonNullable 함수를 사용하는 쪽에서 true가 반환된다면 넘겨준 인자는 null이나 undefined가 아닌 타입으로 타입 가드(타입이 좁혀진다)가 됨

#### Promise.all을 사용할 때 NonNullable 적용하기

```ts
const shopList = [
  { shopNo: 100, category: 'chicken' },
  { shopNo: 101, category: 'pizza' },
  { shopNo: 102, category: 'noodle' },
];

class AdCampaignAPI {
  static async operating(shopNo: number): Promise<AdCampaign[]> {
    try {
      return await fetch(`/ad/shopNumber=${shopNo}`);
    } catch (error) {
      return null;
    }
  }
}

const shopAdCampaignList = await Promise.all(shopList.map((shop) => AdCampaignAPI.operating(shop.shopNo)));
```

- try catch문을 사용하여 에러가 발생할 때 null을 반환<br>
  -> AdCampaignAPI.operating 함수에서 null을 반환할 수 있기 때문에 shopAdCampaignList 타입은 Array<AdCampaign[] | null>로 추론됨<br>
  -> 순회할 때마다 고차 함수 내 콜백 함수에서 if문을 사용한 타입 가드를 반복하게 되는 문제가 생김

```ts
const shopAds = shopAdCampaignList.filter((shop) => !!shop);
// shopAds의 타입 : Array<AdCampaign[] | null>
```

- 이렇게 해도 null이 필터링 되지 않음<br>
  -> Promise.all이 반환하는 배열에서 null 값이 유효한 결과로 간주되기 때문<br>
  -> Promise.all은 모든 프로미스가 완료될 때까지 기다린 후, 각 프로미스의 결과를 배열로 반환하는데 이때, 프로미스가 null을 반환하더라도 이는 정상적인 결과 값으로 처리되어 배열에 포함됨

```ts
// showAdCampaignList가 null이 될 수 있는 경우를 방어하기 위해 NonNullable 사용
const shopAds = shopAdCampaignList.filter(NonNullable);
// shopAds는 필터링을 통해 null이나 undefined가 아닌 값을 가진 배열이 됨
// shopAds의 타입 : Array<AdCampaign[]>

const shopAds = shopAdCampaignList.filter(Boolean);
// 배열에서 Falsy 값들을 필터링할 수 있음
```

- NonNullable를 이용해 null을 필터링해야만 Array<AdCampaign[]>로 추론하게 만들 수 있음
