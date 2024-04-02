### 1-1.
```ts
type ElementType<T> = T extends (infer U)[] ? U : never
type ElementType<T> = T extends Array<infer U> ? U : never
```

### 1-2.
```ts
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any
```

### 2.
#### 1. Exclude<T, U>

**[정의]**
T 타입에서 U에 할당할 수 있는 모든 속성을 제외한 타입
`Exclude<T, U> = T extends U ? never : T`

**[활용 예시]**
```ts
// 어떤 유니온 타입에서 특정한 타입들을 제외시킬 때
type Fruit = 'cherry' | 'banana' | 'strawberry' | 'lemon';

type RedFruit = Exclude<Fruit, 'banana' | 'lemon'>; // 'cherry' | 'strawberry'

// 객체의 키값을 모아 새로운 타입으로 정의할 때 일부 키를 제외하고 싶을 때
interface Player {
  id: number
  name: string
  team: string
  earnings: number
}

type PublicPlayerKey = Exclude<keyof Player, 'earnings' | 'id'> // 'name' | 'team'
```

#### 2. Extract<T, U>

**[정의]**
T 타입에서 U에 할당할 수 있는 모든 속성을 추출하여 타입을 구성
`Extract<T, U> = T extends U ? T : never`

**[활용 예시]**
```ts
// 문자열 또는 숫자로 이루어진 유니온 타입이 있을 때, 이 중에서 문자열만을 추출하고 싶을 때
type UnionType = 'a' | 'b' | 1 | 2;
type StringOnly = Extract<UnionType, string>; // 'a' | 'b'
```

#### 3. Uppercase / Lowercase / Capitalize / Uncapitalize

**[정의]**
문자열의 각 문자를 변환하여 리터럴 타입으로 만듬
템플릿 문자열 리터럴에서의 문자열 조작을 돕기 위한, 타입 시스템 내에서 문자열 조작에 사용할 수 있는 타입 집합이다.
`Uppercase<StringType>`
`Lowercase<StringType>`
`Capitalize<StringType>`
`Uncapitalize<StringType>`

**[활용 예시]**
```ts
type Greeting = "Hello, world";
type ShoutyGreeting = Uppercase<Greeting>; // type ShoutyGreeting = "HELLO, WORLD"

type ASCIICacheKey<Str extends string> = `ID-${Uppercase<Str>}`;
type MainID = ASCIICacheKey<"my_app">; // type MainID = "ID-MY_APP"
```

#### 4. Unpacked<T>

**[정의]**
제네릭 T를 아래 활용 예시에 적은 조건대로 순차적으로 확인하여 unpacked 시킨 타입을 반환

**[활용 예시]**
```ts
type Unpacked<T> = T extends (infer U)[]
  ? U // 1. T가 어떤 값의 배열 (infer U)[] 이면 그 배열의 타입을 반환
  : T extends (...args: any[]) => infer U
    ? U // 2. 배열이 아니고 함수 타입이면, 함수 반환 타입을 반환
    : T extends Promise<infer U>
      ? U // 3. 배열, 함수도 아니고 프로미스 타입이면, 프로미스의 값을 반환
      : T // 4. 위의 모든 조건이 만족하지않으면 T 자기 자신을 반환

type T0 = Unpacked<string> // string
type T1 = Unpacked<string[]> // string
type T2 = Unpacked<() => string> // string
type T3 = Unpacked<Promise<string>> // string
type T4 = Unpacked<Promise<string>[]> // Promise<string>
type T5 = Unpacked<Unpacked<Promise<string>[]>> // string
```

#### 5. StringPropertyNames<T>

**[정의]**
타입에서 값이 'string' 타입인 프로퍼티를 추출

**[활용 예시]**
```ts
type StringPropertyNames<T> = {
   [K in keyof T]: T[K] extends string ? K : never; // 조건부 타입에서 string 인 것만 반환
}[keyof T]; // 그리고 속성의 밸류를 타입으로 반환

interface Person {
   name: string;
   age: number;
   nation: string;
}

type T1 = StringPropertyNames<Person>; // "name" | "nation"

type StringProperties<T> = Pick<T, StringPropertyNames<T>>;

type T2 = StringProperties<Person>;
/*
type T2 = {
  name: string;
  nation: string;
}
*/
```

참고) https://inpa.tistory.com/entry/TS-%F0%9F%93%98-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%9C%A0%ED%8B%B8%EB%A6%AC%ED%8B%B0-%ED%83%80%EC%9E%85-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC

### 3. typeof와 keyof의 순서가 반대로 쓰여있음
