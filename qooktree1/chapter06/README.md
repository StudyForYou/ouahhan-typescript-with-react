<h1>타입스크립트 컴파일</h1>

# 📝 JS의 런타임과 TS의 컴파일

## ✏️ 런타임과 컴파일타임

- 컴파일 타임: 소스코드가 컴파일 과정을 거쳐 컴퓨터가 인식할 수 있는 기계어 코드(바이트 코드)로 변환되어 실행할 수 있는 프로그램이 되는 과정
- 런타임: 컴파일 과정을 마친 으용 프로그램이 사용자에 의해 실행되는 과정

## ✏️ JS의 런타임

- JS가 실행되는 환경
- 대표적으로 크롬이나 사파리 같은 인터넷 브라우저와 Node.js 등이 있다.
- 주요 구성 요소로 JS 엔진, 웹 API, 콜백 큐, 이벤트 루프, 렌더 큐가 있다.

## ✏️ TS의 컴파일

- TS는 `tsc`라고 불리는 컴파일러를 통해 JS 코드로 변환된다.
- 고수준 언어(TS)가 고수준 언어(JS)로 변환되는 것이기 때문에 `트랜스파일`이라고 부리기도 한다.
- 이러한 변환 과정은 소스코드를 다른 소스코드로 변환하는 것이기에 TS 컴파일러를 `소스 대 소스 컴파일러`라고 지정하기도 한다.
- TS 컴파일러는 소스코드를 해석하여 `AST(최소 구문 트리)`를 만들고 이후 타입확인을 거친 다음에 결과 코드를 생성한다.

### TS 컴파일러가 소스코드를 컴파일하여 프로그램이 실행되기까지의 과정

1. TS 소스코드를 TS AST로 만든다.(tsc)
2. 타입 검사기가 AST를 확인하여 타입을 확인한다.(tsc)
3. 타입스크립트 AST를 JS 소스로 변환한다.(tsc)
4. JS 소스코드를 JS AST로 만든다.(런타임)
5. AST가 바이트 코드로 변환된다.(런타임)
6. 런타임에서 바이트 코드가 평가되어 프로그램이 실행된다.(런타임)
   ### AST 란?
   - 컴파일러가 소스코드를 해석하는 과정에서 생성된 데이터 구조
   - 컴파일러는 lexical 분석과 syntax 분석을 통해 소스코드를 노드 단위의 트리 구조로 구성한다.

- TS는 컴파일타임에 타입을 검사하기 때문에 에러가 발생하면 프로그램이 실행되지 않는다. 이러한 특징 때문에 TS를 컴파일타임에 에러를 발견할 수 있는 `정적 타입 검사기`라고 부른다.

  ```ts
  function add(a: number, b: number) {
    return a + b;
  }
  add(10, "20"); // 에러 발생
  ```

# 📝 TS 컴파일러의 동작

## ✏️ 코드 검사기로서의 TS 컴파일러

- TS는 컴파일타임에 문법 에러와 타입 관련 에러를 모두 검출한다.

  ```ts
  const developer = {
    work() {
      console.log("working...");
    },
  };

  developer.work(); // working...

  // 실행 전 에러 - TypeError: developer.sleep is not a function
  // 실행 후 에러 - Property 'sleep' does not exist on type '{ work(): void; }'
  developer.sleep();
  ```

## ✏️ 코드 변환기로서의 TS 컴파일러

- TS 컴파일러는 타입을 검사한 다음 TS 코드를 각자의 런타임 환경에서 동작할 수 있도록 구버전의 자바스크립트로 `트랜스파일`한다.

```ts
type Fruit = "banana" | "watermelon" | "orange" | "apple" | "kiwi" | "mango";

const fruitBox: Fruit[] = ["banana", "apple", "mango"];

const welcome = (name: string) => {
  console.log(`hi! ${name} :)`);
};
```

- TS 컴파일러의 target 옵션을 사용하여 ES2017버전의 JS 소스코드로 컴파일하면 아래와 같은 코드가 생성된다. ([TS Playground](https://www.typescriptlang.org/play?target=9&filetype=ts#code/Q)에서 확인 가능)

```js
"use strict";
const fruitBox = ["banana", "apple", "mango"];
const welcome = (name) => {
  console.log(`hi! ${name} :)`);
};
```

- 트랜스파일이 완료된 JS 파일에서 타입 정보가 제거된 것을 볼 수 있다.

### 🚨 주의할 점

- JS는 타입정보를 이해하지 못하기때문에 TS 소스코드에 타입 에러가 있더라도 JS로 컴파일되어 타입 정보가 모두 제거된 후에는 타입이 아무런 효력을 발휘하지 못한다.

  ```ts
  const name: string = "zig";
  const age: number = "zig"; // Type ‘string’ is not assignable to type ‘number’
  ```

  - 타입 에러가 발생하지만 자바스크립트로 컴파일할 수는 있다.
  - 컴파일 후의 결과는 아래와 같다.

  ```js
  const name = "zig";
  const age = "zig";
  ```

- 컴파일된 코드가 실행되고 있는 런타임에서는 타입 검사를 할 수 없기 때문에 주의해야 하는 경우도 있다.

```ts
interface Square {
  width: number;
}

interface Rectangle extends Square {
  height: number;
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    // 에러 발생 - ‘Rectangle’ only refers to a type, but is being used as a value here
    // 에러 발생 - Property ‘height’ does not exist on type ‘Shape’
    // 에러 발생 - Property ‘height’ does not exist on type ‘Square’
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

- instance of 체크는 런타임에 실행되지만 Rectangle은 타입이기 때문에 JS 런타임은 해당 코드를 이해하지 못한다.

```ts
// 타입가드를 통해 이 부분은 해결할 수 있다.
interface Square {
  width: number;
}

interface Rectangle extends Square {
  height: number;
}

type Shape = Square | Rectangle;

function isRectangle(shape: Shape): shape is Rectangle {
  return "height" in shape;
}

function calculateArea(shape: Shape) {
  if (isRectangle(shape)) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

# 📝 TS 컴파일러의 구조

- 컴파일러는 하나의 프로그램으로 이를 구현한 소스 파일이 존재한다.
- TS 컴파일러가 동작하는 데 중요한 몇 가지 중요한 구성 요소를 가지고 있다.

## ✏️ 프로그램(Program)

- TS 컴파일러는 tsc 명령어로 실행되며, 컴파일러는 tsconfig.json에 명시된 컴파일 옵션을 기반으로 컴파일을 수행한다.

먼저 `전체적인 컴파일 과정을 관리하는 프로그램 객체(인스턴스)`가 생성된다. 이 프로그램 객체는 컴파일할 타입스크립트 소스 파일과 소스 파일 내에서 임포트된 파일을 불러오는데, 가장 최초로 불러온 파일을 기준으로 컴파일 과정이 시작된다.

## ✏️ 스캐너(Scanner)

- TS 소스코드를 작은 단위로 나누어 의미 있는 토큰으로 변환하는 작업을 수행한다.

## ✏️ 파서(Parser)

- 스캐너가 소스 파일을 토큰으로 나눠주면 파서는 그 토큰 정보를 이용하여 AST를 생성한다.

- AST는 컴파일러가 동작하는 데 핵심 기반이 되는 자료 구조로, 소스코드의 구조를 트리 형태로 표현하다. AST의 최상위 노드는 타입스크립트 소스 파일이며, 최하위 노드는 파일의 끝 지점으로 구성된다.

- `스캐너는 어휘적 분석을 통해 토큰 단위로 소스코드를 나눈다면`, `파서는 이렇게 생성된 토큰 목록을 활용하여 구문적 분석을 수행`한다고 볼 수 있으며 이를 통해 코드의 실질적인 구조를 노드 단위의 트리 형태로 표현하는 것이다.

- 각각의 노드는 코드상의 위치, 구문 종류, 코드 내용과 같은 정보를 담고있다.

## ✏️ 바인더(Binder)

- 체커 단계에서 타입 검사를 할 수 있도록 기반을 마련하는 것이다. 바인더는 타입 검사를 위해 `심볼이라는 데이터 구조를 생성`한다.
- 심볼은 이전 단계의 AST에서 선언된 타입의 노드 정보를 저장한다.
- 결과적으로 바인더는 심볼을 생성하고 해당 심볼과 그에 대응하는 AST 노드를 연결하는 역할을 수행한다.

## ✏️ 체커(Checker)와 이미터(Emitter)

### 체커(Checker)

- AST의 노드를 탐색하면서 심볼 정보를 불러와 주어진 소스 파일에 대해 타입 검사를 진행한다.
- 체커의 타입 검사는 다음 컴파일 단계인 이미터에서 실행된다. checker.ts의 getDiagnostics()함수를 사용해서 타입을 검증하고 타입 에러에 대한 정보를 보여줄 에러 메시지를 저장한다.

### 이미터(Emitter)

- TS 소스를 JS 파일과 타입 선언 파일(d.ts)로 생성한다.
- 이미터는 TS 소스 파일을 변환하는 과정에서 개발자가 설정한 TS 설정 파일을 읽어오고, 체커를 통해 코드에 대한 타입 검증 정보를 가져온다. 그리고 emitter.ts 소스 파일 내부의 emitFiles()함수를 사용하여 타입스크립트 소스 변환을 진행한다.
