# 12장 타입스크립트 프로젝트 관리

## 12.1 앰비언트 타입 활용하기

### 1. 앰비언트 타입 선언

**.d.ts 확장자 파일**은 타입스크립트의 타입 선언이 가능하지만 값을 표현할 수 없다. 값을 포함하는 일반적인 선언과 구별하기 위해 이를 **앰비언트 타입 선언**이라고 부른다.  
`declare`라는 키워드를 사용하여 어딘가에 자바스크립트 값이 존재한다는 사실을 선언할 수 있다. declare는 타입스크립트 컴파일러에 어떤 것의 존재 여부를 명시해주는 역할을 한다. 단순히 존재 여부만 알려주기 때문에 컴파일 대상이 아니다.

#### 대표적인 앰비언트 타입 선언 활용 사례

타입스크립트는 기본적으로 .ts와 .js 파일만 이해하며 그 외의 다른 파일 형식은 인식하지 못한다.  
예를 들어, png 등 이미지 파일을 모듈로 import할 때 에러가 발생할 수 있다.  
이때 declare 키워드를 사용하여 특정 형식을 모듈로 선언하면 타입스크립트 컴파일러에 미리 정보를 제공함으로써 에러를 수정할 수 있다.

```d
declare module "*.png" {
  const src: string;
  export default src;
}
```

#### 자바스크립트로 작성된 라이브러리

**예시**  
자바스크립트로 작성된 npm 라이브러리  
이 라이브러리는 자바스크립트로 구현되었기 때문에 타입 선언이 존재하지 않음.  
따라서 타입스크립트에서 이 라이브러리를 사용할 수는 있지만 타입 선언이 없으므로 import한 모듈은 모두 any로 추론됨. 만약 tsconfig.json 파일에서 any를 사용하지 못하게 설정했다면 프로젝트 빌드 안 됨.

이때 앰비언트 타입 선언을 사용한다. 자바스크립트 라이브러리 내부 함수와 변수의 타입을 앰비언트 타입으로 선언하면 타입스크립트는 자동으로 .d.ts 확장자를 가진 파일을 검색하여 타입 검사를 진행하게 되므로 문제없이 컴파일됨.  
또한 VSCode와 같은 코드 편집기도 .d.ts 확장자를 가진 파일을 해석하여 코드를 작성할 때 유용한 타입 힌트를 제공함.

#### 타입스크립트로 작성된 라이브러리

타입스크립트로 작성된 라이브러리일지라도 자바스크립트 파일과 .d.ts 파일로 배포되는 것이 일반적

- 자바스크립트 파일과 .d.ts 파일로 배포하면 라이브러리 코드를 따로 컴파일하지 않아도 되기 때문에 컴파일 시간이 크게 줄어듦
- 또한 .d.ts 파일이 있기 때문에 사용자는 .d.ts 파일에 정의된 타입 정보를 활용하여 라이브러리 사용 가능
- 또한 tsconfig.json 파일의 declaration을 true로 설정하면 타입스크립트 컴파일러는 자동으로 .d.ts 파일을 생성함

### 2. 앰비언트 타입 선언 시 주의점

#### 타입스크립트로 만드는 라이브러리에는 불필요

tsconfig.json의 declaration을 true로 설정하면 타입스크립트 컴파일러가 .d.ts 파일을 자동 생성하기 때문에 수동으로 .d.ts 파일을 작성할 필요가 없다.

#### 전역으로 타입을 정의하여 사용할 때 주의해야 할 점

서로 다른 라이브러리에서 동일한 이름의 앰비언트 타입 선언을 한다면 충돌이 발생하여 어떤 타입 선언이 적용될지 알기 어렵고, 의도한 대로 동작하지 않을 수 있다. 또한 앰비언트 타입 선언은 명시적인 import나 export가 없기 때문에 코드의 의존성 관계가 명확하지 않아 나중에 변경할 때 어려움을 겪을 수 있다.

### 3. 앰비언트 타입 선언을 잘못 사용했을 때의 문제점

.ts 파일 내부에 앰비언트 변수를 선언할 때 앰비언트 타입의 의존성 관계가 보이지 않기 때문에 변경에 의한 영향 범위를 파악하기 어렵다. 앰비언트 타입은 명시적인 import나 export 없이 코드 전역에서 사용할 수 있기 때문이다.

#### 예시

declare 키워드를 사용한 앰비언트 타입 선언은 .d.ts 파일이 아닌, .ts, .tsx 파일 내에서도 할 수 있다.

```tsx
// src/index.tsx
import React from "react";
import ReactDOM from "react-dom";
import App from "App";

declare global {
  interface Window {
    Example: string;
  }
}

const SomeComponent = () => {
  return <div>앰비언트 타입 선언은 .tsx 파일에서도 가능</div>;
};
```

위와 같이 선언된 앰비언트 타입은 아래와 같은 src/test.tsx 파일에서도 import 없이 사용할 수 있다.

```tsx
// src/test.tsx
window.Example; // 앰비언트 타입 선언으로 인해 타입스크립트 에러가 발생하지 않음
```

이렇게 앰비언트 변수 선언은 어느 곳에나 영향을 줄 수 있기 때문에 일반 타입 선언과 섞이게 되면 앰비언트 선언이 어떤 파일에 포함되어 있는지 파악하기 어려워진다.  
만약 작은 컴포넌트에 앰비언트 변수 선언이 포함되어 있다면 모든 파일의 타입에 영향을 주기 때문에 어떤 파일에서 앰비언트 타입이 선언되었는지 찾기 어려워진다.

_따라서 .d.ts 확장자 파일 내에 앰비언트 타입 선언을 해야 한다._

### 4. 앰비언트 타입 활용하기

#### 타입을 정의하여 임포트 없이 전역으로 공유

.d.ts 파일에서의 앰비언트 타입선언은 전역 변수와 같은 역할을 한다. 앰비언트 타입으로 유틸리티 타입을 선언하면 모든 코드에서 임포트하지 않아도 해당 타입을 사용할 수 있다.

```
// src/index.d.ts
type Optional<T extends object, K extends keyof T = keyof T> = Omit<T, K> &
  Partial<Pick<T, K>>;

// src/components.ts
type Props = { name: string; age: number; visible: boolean };
type OptionalProps = Optional<Props>;
// Expect: { name?: string; age?: number; visible: boolean; }
```

#### declare type 활용하기

보편적으로 많이 사용하는 커스텀 유틸리티 타입을 declare type으로 선언하여 전역에서 사용할 수 있다.

```ts
declare type Nullable<T> = T | null;

const name: Nullable<string> = "woowa";
```

#### declare module 활용하기

theme 타입(CSS-in-JS)

```tsx
const fontSizes = {
  xl: "30px";
  // ...
}

const colors = {
  gray_100: "#222222";
  gray_200: "#444444";
  // ...
}

const depths = {
  origin: 0,
  foreground: 10,
  dialog: 100,
  // ...
}

const theme = {
  fontSizes,
  colors,
  depths
};

declare module "styled-components" {
  type Theme = typeof theme;

  export interface DefaultTheme extends Theme {}
}
```

로컬 이미지 또는 SVG 파일 등을 모듈로 인식하여 사용하기

```ts
declare module "*.gif" {
  const src: string;

  export default src;
}
```

#### declare namespace 활용하기

Node.js 에서 .env 파일을 사용할 때, declare namespace를 활용하여 process.env로 설정값을 손쉽게 불러오고 환경변수의 자동 완성 기능을 쓸 수 있다.

```ts
declare namespace NodeJS {
  interface ProcessEnv {
    readonly API_URL: string;
    readonly API_INTERNAL_URL: string;
    // ...
  }
}
```

process.env를 통해 접근하는 변수 또한 타입을 지정할 수 있기 때문에 as 단언을 사용하지 않아도 된다.

#### 예시

```tsx
function log(str: string) {
  console.log(str);
}
```

1. namespace를 활용하여 process.env 타입을 보강해주지 않은 경우

```ts
// .env
API_URL = "localhost:8000";

log(process.env.API_URL as string);
```

2. namespace를 활용하여 process.env 타입을 보강한 경우

```ts
// .env
API_URL = "localhost:8000";

declare namespace NodeJS {
  interface ProcessEnv {
    readonly API_URL: string;
  }
}

log(process.env.API_URL);
```

#### declare global 활용하기

declare global 키워드는 전역 변수를 선언할 때 사용한다.
예시
전역 변수인 Window 객체의 스코프에서 사용되는 모듈이나 변수를 추가할 수 있다.

```
declare global {
  interface Window {
    newProperty: string;
  }
}
```

```

```
