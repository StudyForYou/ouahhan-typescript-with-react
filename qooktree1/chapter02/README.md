# 타입이란?

> 타입스크립트는 TS, 자바스크립트는 JS로 표기하겠습니다

## 📝 자료형으로서의 타입

컴퓨터의 메모리 공간은 한정적이기 때문에 값의 크기를 명시하면 컴퓨터가 값을 효율적이고 안전하게 저장할 수 있게 한다. (ex. 숫자는 8byte, boolean는 1byte 등)

최신 ECMAScript 표준을 따르는 JS는 다음과 같은 7가지 데이터 타입(자료형)을 정의한다.

- `undefined`, `null`, `Boolean`, `String`, `Symbol`, `Numeric(Number와 BigInt)`, `Object`

## 📝 집합으로서의 타입

타입은 값이 가질 수 있는 유효한 범위의 집합

타입 시스템은 코드에서 사용되는 유효한 값의 범위를 제한해서 런타임에서 발생할 수 있는 유효하지 않은 값에 대한 에러를 방지해준다.

```ts
const num: number = 123;
const str: string = "abc";

function func(n: number) {
  // ...
}

func(str); // Argument of type 'string' is not assignable to parameter of type 'number'
```

- func() 함수의 인자로 들어갈 수 있는 값을 number 타입의 집합으로 제한

## 📝 정적 타입과 동적 타입

타입을 결정하는 시점에 따라 타입을 `정적 타입`과 `동적 타입`으로 분류가 가능

### ✏️ 정적 타입 시스템

- 모든 변수의 타입이 컴파일타임에 결정됨
- C, Java, TypeScript
- 컴파일타임에 타입 에러를 발견할 수 있기 때문에 프로그램의 안정성을 높임

### ✏️ 동적 타입 시스템

- 변수 타입이 런타임에서 결정됨
- Python, JavaScript
- 직접 타입을 정의해줄 필요 X
- 개발 과정에서는 에러 없이 작성 가능하지만 안정성이 떨어진다.

```ts
const multiplyByThree = (number) => number * 3;
multiplyByThree(10); // 30
multiplyByThree("F"); // NaN
```

- 함수의 입력되는 변수가 string 타입이 들어가 예상치 못한 결과를 반환한다(NaN)
- 함수가 실행되기 전까지는 모른다!

<details>
  <summary>💡 컴파일타임과 런타임</summary>

- 컴파일 타임: 컴퓨터가 소스코드를 이해할 수 있도록 기계어로 변환되는 시점
- 런타임: 변환된 파일이 메모리에 적재되어 실행되는 시점

</details>

## 📝 강타입과 약타입

### ✏️ 암묵적 타입 변환

- 개발자가 의도적으로 타입을 명시하거나 바꾸지 않았는데도 컴파일러 or 엔진 등에 의해서 런타임에 자동으로 변경되는 것

#### [ 상황 ] 서로 다른 타입을 갖는 값끼리 연산을 시도하는 상황

- 강타입: 컴파일러 or 인터프리터에서 에러 발생(Python, Ruby, TypeScript)
- 약타입: 컴파일러 or 인터프리터가 내부적으로 판단하여 타입을 변환하여 연산을 수행한 후 값을 도출(Java, C++, JavaScript)

<details>
  <summary>🔍️ 예제</summary>

```java
// Java
class Main {
  public static void main(Stirng[] args) {
    System.out.println('2' - 1);  // 49 -> '2'는 ASCII 값으로 50
  }
}
```

```python
# Python
print('2' - 1);  # TypeError: unsupported operand type(s) for -: 'str' and 'int
```

```js
// JavaScript
console.log("2" - 1); // 1
```

```ts
// TypeScript
console.log("2" - 1); // '2' error 타입 에러
```

암묵적 변환은 개발자가 명시적으로 타입을 변환하지 않아도 되는 편리함이 있지만 `작성자의 의도와 다르게 동작`할 수 있기 때문에 오류가 발생할 가능성이 높아진다.

- 예시로 Java와 JavaScript에서의 결과값이 다르다는 것을 볼 수 있다.

</details>

### ✏️ 타입 시스템

- 타입 검사기가 프로그램에 타입을 할당하는 데 사용하는 규칙 집합
- `어떤 타입을 사용하는지를 컴파일러에 명시적으로 알려줘야하는 부분`과 `자동으로 타입을 추론하는 부분`으로 나뉨
- 즉, 개발자는 직접 타입을 명시하거나, TS가 타입 추론하는 방식 중에서 선택 가능

## 📝 컴파일 방식

### ✏️ 컴파일?

- 사람이 이해할 수 있는 방식으로 작성한 코드를 컴퓨터가 이해할 수 있는 기계어로 바꿔주는 과정
- [EX] Java, C++ -> Binary Code(0, 1로 이루어진 이진 코드)

TS는 JS의 타입 에러를 컴파일 타임에 미리 발견하기 위해 만들어졌기 때문에 `컴파일 시 타입 정보가 없는 순수 JS 코드`를 생성

# TS의 타입시스템

## 📝 타입 애너테이션(annotation) 방식

- 변수나 상수 or 함수의 인자와 반환 값에 타입을 명시적으로 선언해서 어떤 타입 값이 저장될 것인지를 컴파일러에 직접 알려주는 문법

  ```ts
  let isDone: boolean = false;
  let decimal: number = 6;
  let color: string = "blue";
  let list: number[] = [1, 2, 3];
  let x: [string, number]; // tuple
  ```

## 📝 `구조적 타이핑`

### ✏️ 타입 시스템

- 명목적 타입 시스템 & 구조적 타입 시스템으로 나뉨
- 명목적 타입 시스템
  - 타입의 이름을 통해 구분
  - 대표적으로 Java
  - class의 구조가 같더라도 이름이 다르면 다른 타입으로 간주
- 구조적 타입 시스템
  - 객체가 가지고 있는 속성을 바탕으로 타입을 구분
  - 동일한 구조를 가지고 있다면 동일한 타입으로 간주

## 📝 `구조적 서브타이핑`

- 객체가 가지고 있는 속성을 바탕으로 타입을 구분하고 좀 더 타입 간의 관계에 중점을 두는 타입 시스템을 말함

```ts
interface Pet {
  name: string;
}

interface Cat {
  name: string;
  age: number;
}

let pet: Pet;
let cat: Cat = { name: "Zag", age: 2 };

pet = cat;
```

- Cat 타입으로 선언한 cat을 Pet 타입으로 선언한 pet에 할당 가능
- 타입 호환성에 더 많은 유연성을 허용

## 📝 JS를 닮은 TS

명목적 타이핑은 타입의 동일성을 확인하는 과정에서 구조적 타이핑에 비해 조금 더 안전

### ✏️ TS는 왜 구조적 타이핑을 채택했을까?

- 타입스크랩트는 JS를 모델링한 언어
- JS는 `덕 타이핑 기반`
  <details>
    <summary>덕 타이핑</summary>

  - 어떤 타입에 부합하는 변수와 method를 가질 경우 해당 타입에 속하는 것으로 간주하는 방식

  </details>

- 객체 간 속성이 동일하다면 서로 호환되는 구조적 타입 시스템을 제공하여 더욱 편리성을 높임

### ✏️ 덕타이핑과 구조적 타이핑의 차이점은 뭘까?

- 코드로 보면 차이가 없어 보임(두 방식 모두 객체가 가진 속성을 기반으로 타입을 검사)
- 덕 타이핑은 런타임에 검사, 구조적 타이핑은 컴파일타임에 타입체커가 검사
- 덕 타이핑은 주로 동적 타이핑에서 사용됨
- 구조적 타이핑은 주로 정적 타이핑에서 사용됨

## 📝 구조적 타이핑의 결과

구조적 타이핑은 유연성을 챙기긴 했지만, 대신에 정적 타입의 정확성을 100% 보장해주지 않음

```ts
interface Cube {
  width: number;
  height: number;
  depth: number;
}

function addLines(c: Cube) {
  let total = 0;

  for (const axis of Object.keys(c)) {
    // 🚨 Element implicitly has an 'any' type
    // because expression of type 'string' can't be used to index type 'Cube'.
    // 🚨 No index signature with a parameter of type 'string'
    // was found on type 'Cube'
    const length = c[axis];

    total += length;
  }
}
```

- Cube 인터페이스의 모든 필드는 number 타입을 가지지만, c에 들어올 객체는 Cube의 width, height, depth 외에도 어떤 속성이든 가질 수 있기 때문에 c[axis]의 타입이 string일 수도 있어 에러가 발생
- 이런 한계를 극복하고자 TS에 명목적 타이핑 언어의 특징을 가미한 식별할 수 있는 `유니온` 같은 방법이 생겨남

## 📝 TS의 점진적 타입 확인

### ✏️ 점진적 타입 검사?

- 컴파일 타임에 타입을 검사하면서 필요에 따라 타입 선언 생략을 허용하는 방식
- 타입을 지정한 변수와 표현식은 정적으로 타입 검사, 타입 선언이 생략되면 동적으로 검사를 수행

JS코드를 TS코드로 마이그레이션 할 때, 점진적 타이핑이라는 특징을 유용하게 활용 가능

<details>
  <summary>💡 any 타입</summary>

- TS 내 모든 타입의 종류를 포함하는 가장 상위 타입
- TS 컴파일 옵션인 noImplicitAny 값이 true일때는 에러가 발생
- TS로 코드를 작성할 때는 정확한 타이핑을 위해 tsconfig의 noImplicitAny 옵션을 true로 설정하는 게 좋음

</details>

## 📝 JS 슈퍼셋으로서의 TS

- TS 문법은 모든 JS 문법을 포함

## 📝 값 vs 타입

- 값과 타입은 TS에서 별도의 네임스페이스에 존재
- 값 공간과 타입 공간의 이름은 서로 충돌하지 않으며, 변수를 같은 이름으로 정의 가능
- 값과 타입 공간에 동시에 존재하는 심볼도 있음
  - `Class`와 `enum`
  - 클래스는 타입으로도 사용 가능
  - enum 역시 런타임에 객체로 변환되는 값

<details>
  <summary>💡 트리쉐이킹(tree-shaking)</summary>

- JS, TS 에서 사용하지 않는 코드를 삭제하는 방식
- ES6 이후의 개발 환경에서는 Webpack, Rollup 같은 모듈 번들러를 사용하는데, 번들링 작업을 수행할 때 사용하지 않는 코드는 자동으로 삭제됨

</details>

## 📝 타입을 확인하는 방법

### ✏️ 1. typeof

- 연산하기 전에 피연산자의 데이터 타입을 나타냄(Boolean, null, undefined, Number, BigInt, String, Symbol, Function, object)
- **값에 쓰일 때(JS)**: JS 런타임의 typeof 연산자가 됨
- **타입에서 쓰일 때(TS)**: 값을 읽고 TS 타입을 반환

```ts
interface Person {
  first: string;
  last: string;
}

const person = (Person = { first: "zig", last: "song" });
function email(options: { person: Person; subject: string; body: string }) {}

const v1 = typeof person; // 값은 'object'
const v2 = typeof email; // 값은 'function'
type T1 = typeof person; // 타입은 Person
type T2 = typeof email; // 타입은 (options: {person: Person; subject: string; body: string}) => void
```

### ✏️ 2. instanceof

- 객체가 특정 클래스나 생성자 함수의 인스턴스인지 여부를 확인하는 데 사용됨

```ts
let error: unknown;

if (error instanceof Error) {
  // 이 블록 내에서 error는 Error 타입으로 정제되어 사용된다.
  showAlertModal(error.message); // // 안전하게 Error 클래스의 메소드를 사용할 수 있음
} else {
  // error가 Error 타입이 아닌 경우의 처리
  throw Error(error);
}
```

### ✏️ 3. 타입 단언

- 개발자가 해당 값의 타입을 더 잘 파악할 수 있을 때 사용
- `as` 키워드 사용(강제 형 변환과 유사)

```ts
const loadedText: unknown;

const validateInputText = (text: string) => {
  if (text.length < 10) return '최소 10글자 이상 입력해야 합니다.');
  return '정상 입력된 값입니다.';
}

validateInputText(loadedText as string);  // 단언하지 않으면 컴파일 단계에서 오류 발생
```

### ✏️ 4. 타입 가드

- 특정 조건을 검사해서 타입을 정제하고 타입 안정성을 높이는 패턴

```ts
interface Foo {
  type: "foo"; // 리터럴 타입
  foo: number;
}
interface Bar {
  type: "bar"; // 리터럴 타입
  bar: number;
}

function doStuff(arg: Foo | Bar) {
  if (arg.type === "foo") {
    console.log(arg.foo); // Good!
    console.log(arg.bar); // Error!
  } else {
    // 백퍼 Bar겠군.
    console.log(arg.foo); // Error!
    console.log(arg.bar); // Good!
  }
}
```

# 원시 타입

> `원시 타입과 원시 래퍼 객체`

> TS에서는 `원시 값`과 `원시 래퍼 객체`를 구분하여 사용함. TS에서는 원시 값에 대응하는 타입을 소문자로 표기하며, 파스칼 표기법을 사용하면 해당 원시 값을 래핑하는 객체 타입을 의미

## 📝 boolean

- true와 false 값만 할당할 수 있는 boolean 타입

## 📝 undefined

- 정의되지 않았다는 의미의 타입, 오직 undefined 값만 할당 가능

## 📝 null

- 오직 null만 할당 가능 -> undefined와 혼용 불가!

```ts
type Person1 = {
  name: string;
  job?: string;
};
type Person2 = {
  name: string;
  job: string | null;
};
```

- Person1은 job이라는 속성이 있을 수도 or 없을 수도 있음을 나타냄
- Person2는 사람마다 갖고 있지만 값이 비어있을 수도 있다는 것을 나타냄

## 📝 number

- 숫자, NaN이나, Infinity도 포함됨
  - `NaN`: Not A Number의 줄임말로, 문자열을 숫자로 변환할 수 없는데 변환하려고 할 때 반환됨
  - `Infinity`: 무한대를 나타내는 숫자형 값

## 📝 bigInt

- ES2020에서 새롭게 도입된 데이터 타입으로 TS 3.2 version부터 사용 가능
- JS에서 가장 큰 수인 `Number.MAX_SAFE_INTEGER(2**53 - 1)`를 넘어가는 값의 처리를 하지 못함
- bigInt를 사용하면 처리 가능

```ts
const bigNumber1: bigInt = BigInt(999999999999999);
```

- `number 타입과 bigint 타입은 엄연히 서로 다른 타입이므로 상호 작용 불가`

## 📝 string

- 문자열을 할당할 수 있는 타입
- 공백, 작은따옴표('), 큰따옴표("), 백틱(`) - 템플릿 리터럴 등이 있음

## 📝 symbol

ES2015에서 도입된 데이터 타입으로 `Symbol()` 함수를 사용하면 어떤 값과도 중복되지 않는 유일한 값을 생성 가능

```ts
const MOVIE_TITLE = Symbol("title");
const MUSIC_TITLE = Symbol("title");
console.log(MOVIE_TITLE === MUSIC_TITLE); // false
let SYMBOL: unique symbol = Symbol(); // A variable whose type is a 'unique' symbol' type must be 'const'
```

TS에는 `symbol 타입`과 const 선언에서만 사용할 수 있는 `unique symbol 타입`이라는 symbol의 하위 타입도 있음

# 객체 타입

원시 타입에 속하지 않는 값은 모두 객체 타입으로 분류 가능

## 📝 object

`object 타입은 가급적 사용하지 말도록 권장됨`

- 이유: any 타입과 유사하게 객체에 해당하는 모든 타입 값을 유동적으로 할당할 수 있어 정적 타이핑의 의미가 퇴색됨

다만 any와는 다르게 원시 타입에 해당하는 값은 object 타입에 속하지 않음

## 📝 {}

- 중괄호 안에 객체의 속성 타입을 지정해주는 식으로 사용
- 타이핑되는 객체가 중괄호 안에서 선언된 구조와 일치해야 한다는 것을 말함

  ```ts
  const cardData = {title: string; description: string} = {
    title: "즐종시", description: "즐거운 종 시간"
  }

  ```

- 빈 객체 타입을 지정하기 위해서는 `{}`보다는 `Record<string, never>` 처럼 사용하는 것이 바람직함
  - [참고자료](https://db2dev.tistory.com/entry/TS-%EB%B9%88-%EA%B0%9D%EC%B2%B4%EC%97%90-%EB%8C%80%ED%95%9C-%EC%98%AC%EB%B0%94%EB%A5%B8-%ED%83%80%EC%9E%85)
- 소문자로 된 TS 타입 체계를 사용하는 게 일반적

## 📝 array

- TS에서는 배열을 `array`라는 별도 타입을 다룸
- 하나의 타입만 가질 수 있음
- 원소의 개수는 타입에 영향을 주지 않음
- `Array 키워드`로 선언하거나 `대괄호([])`를 사용해서 선언하는 방식이 있다.
- 튜플 타입도 대괄호로 선언하여 배열 타입과 구분해서 사용하자!

```ts
// Type[] 과 Array<Type> 의 차이점 - 가독성, 여러 타입 사용시, readonly
const x1: ComputedRef<Array<number>>; // 중첩시 혼란 야기 가능
const y1: ComputedRef<number[]>; // 간결함

const x2: Array<string | number> = [1, "a"]; // Array<Type>: 여러 타입 허용 가능
const y2: string[] = ["a", "b"]; // Type[]: 여러 타입 허용 불가
const z2: [string, number] = [1, "a"]; // 이런 식으로 사용 가능하지만 이건 튜플이다

const x3: ReadonlyArray<number>; // ReadonlyArray 활용 가능
const y3: readonly number[]; // readonly 추가
```

- [참고자료](https://dev.to/rahulrajrd/array-vs-type-vs-type-in-typescript-5g1h)

## 📝 type과 interface 키워드

- 객체 타입을 type or interface 키워드를 사용해 선언하면 반복적으로 사용 가능하고 중복 없이 해당 타입을 쓸 수 있다.

<details>
  <summary>💬 우형에서는 type과 interface 키워드를 어떻게 사용할까??</summary>

### ✏️ Q. type과 interface 둘 중 주로 어떤 것을 사용하나요?

- `결론: 정말 팀마다 다른 성격을 띄고 있다.`

- 전역적으로 사용할 때는 interface, 작은 범위 내에서 한정적으로 사용할 때 type 사용
- 정적으로 결정되는 것들은 type, 확장될 수 있는 basis 정의 or object 구성은 interface

### ✏️ Q. 팀 내에서 type이나 interface 만을 써야 하는 상황은?

**Type**

- 유니온 타입이나 교차 타입 등 type 정의에서만 쓸 수 있는 기능을 활용할 때 사용
- computed value를 써야 했을 때 사용

  - computed value란, `표현식(expression)을 이용해 객체의 key 값을 정의하는 ES6 문법`

  ```ts
  // 인터페이스는 사용 불가능
  type UserColor = "red" | "green" | "blue";

  type ColorPalette = {
    [key in UserColor]: string;
  };
  // type ColorPalette = {
  //  red: string;
  //  green: string;
  //  blue: string;
  // }
  ```

**Interface**

- 객체 지향적으로 코드를 짤 때, 상속하는 경우(extends, implements 사용)
- 다른 컴포넌트들끼리 같은 속성을 공유하는 기준 interface를 정의하고 확장할 때 사용

</details>

## 📝 function

JS에서는 함수도 일종의 객체로 간주하지만 `typeof` 연산자로 함수 타입을 출력해보면 `function`으로 분류된다

#### 함수 시그니처(Call Signature)

- 함수 타입을 정의할 때 사용하는 문법
- 함수의 매개변수와 반환 값의 타입을 명시하는 역할

```ts
type add = (a: number, b: number) => number;
```

- TS에서 함수 자체의 타입을 명시할 때는 `화살표 함수 방식으로만 정의 가능`
