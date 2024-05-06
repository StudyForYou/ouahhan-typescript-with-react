<h1>타입스크립트 프로젝트 관리</h1>

# 📝 앰비언트(ambient) 타입 활용하기

## ✏️ 앰비언트 타입 선언

- TS의 타입 선언은 `.ts` or `.tsx` 확장자 파일 내에서 할 수 있지만 `.d.ts` 확장자 파일에서도 선언이 가능하다.

### 앰비언트 타입

- `.d.ts` 확장자를 가진 파일에서는 타입 선언만 가능(`값 표현 불가`)
- 값을 포함하는 일반적인 선언과 구별하기 위해 `.d.ts` 확장자를 가진 파일에서 하는 타입 선언을 `앰비언트 타입 선언`이라고 부른다.
- `declare`이라는 키워드를 사용하여 어딘가에 JS 값이 존재한다는 사실을 선언할 수 있다.
  - declare는 TS 컴파일러에 어떤 것의 존재 여부를 명시해주는 역할을 하기 때문에 컴파일 대상이 아니다.

### 대표적인 앰비언트 타입 선언 활용 사례

- TS는 기본적으로 `.ts` or `.js` 파일만 이해하며 그 외의 다른 파일 형식은 인식하지 못한다.
- 따라서 알지 못하는 파일 형식(`png`, `jpg`)을 모듈로 가져오려 하면 에러가 발생한다.

  ```ts
  declare module "*.png" {
    const src: string;
    export default src;
  }
  ```

  declare 키워드는 이미 존재하지만 TS가 알지 못하는 부분을 컴파일러에 "이러한 것이 존재해"라고 알려주는 역할을 한다.

### JS로 작성된 라이브러리

- JS로 작성된 라이브러리의 경우 타입 선언이 없으므로 임포트한 모듈은 모두 `any`로 추론될 것이다.
- 이때 JS 라이브러리 내부 함수와 변수의 타입을 앰비언트 타입으로 선언하면 TS는 자동으로 `.d.ts` 확장자를 가진 파일을 검색하여 타입 검사를 진행하게 되므로 문제없이 컴파일된다.

#### 예시 - **@types/react**

- `npm install -D` 명령을 통해 설치한다면 node_modules/@types/react에 `index.d.ts`와 `global.d.ts`가 설치된다.
- 앰비언트 타입 선언은 TS에게 "JS 코드 안에는 이러한 정보들이 있어"라고 알려주는 도구라고 이해하면 된다.

### JS 어딘가에 전역 변수가 정의되어 있음을 TS에 알릴 때

- 타입스크립트로 직접 구현하지 않았지만 실제 자바스크립트 어딘가에 전역 변수가 정의되어 있는 상황을 타입스크립트에 알릴 때 앰비언트 타입 선언을 사용한다.
- 예를 들어 웹뷰를 개발할 때 네이티브 앱과의 통신을 위한 인터페이스를 네이티브 앱이 Window 객체에 추가하는 경우가 많다.
- 이러한 전역 객체인 Window에 변수나 함수를 추가하면 타입스크립트에서 직접 구현하지 않았더라도 실제 런타임 환경에서 해당 변수를 사용할 수 있다.

```ts
declare global {
  interface Window {
    deviceId: string | undefined;
    appVersion: string;
  }
}
```

## ✏️ 앰비언트 타입 선언 시 주의점

### TS로 만드는 라이브러리에는 불필요

tsconfig.json의 declaration을 true로 설정하면 TS 컴파일러가 .d.ts 파일을 자동으로 생성해주기 때문에 수동으로 작성할 필요가 없다.

### 전역으로 타입을 정의하여 사용할 때 주의해야 하는 점

- 서로 다른 라이브러리에서 동일한 이름의 앰비언트 타입 선언을 한다면 충돌이 발생하여 어떤 타입 선언이 적용될지 알기 어려우며, 의도한 대로 동작하지 않을 수 있다.
- 또한 앰비언트 타입 선언은 명시적인 임포트나 익스포트가 없기 때문에 코드의 의존성 관계가 명확하지 않아 나중에 변경할 때 어려움을 겪을 수 있다.

## ✏️ 앰비언트 타입 선언을 잘못 사용했을 때의 문제점

- `.d.ts 확장자 파일 내에서 앰비언트 타입 선언을 하는 것은 일종의 개발자 간의 약속`이다.
- .ts 파일 내의 앰비언트 변수 선언은 추후 유지보수에 악영향을 미친다.

## ✏️ 앰비언트 타입 활용하기

### 타입을 정의하여 임포트 없이 전역으로 공유

```ts
// src/index.d.ts
type Optional<T extends object, K extends keyof T = keyof T> = Omit<T, K> &
  Partial<Pick<T, K>>;

// src/components.ts
type Props = { name: string; age: number; visible: boolean };
type OptinalProps = Optinal<Props>;
```

### declare type 활용하기

- 보편적으로 많이 사용하는 커스텀 유틸리티 타입을 declare type으로 선언하여 전역에서 사용 가능하다.

```ts
declare type Nullable<T> = T | null;
const name: Nullable<string> = "woowa";
```

### declare module 활용하기

CSS-in-JS 라이브러리 중 하나인 styled-components는 기존의 폰트 크기, 색상 등을 객체로 관리한다. 이렇게 정의된 theme의 인터페이스 타입을 확장하여 theme 타입이 자동으로 완성되도록 하는 기능을 지원하고 있다.

```ts
const fontSizes = {
  xl: "30px",
  /* ... */
};

const theme = {
  fontSizes,
  /* ... */
};

declare module "styled-components" {
  type Theme = typeof theme; // 정의된 theme에서 스타일 값을 가져옴
  export interface DefaultTheme extends Theme {} // 기존 인터페이스 타입과 통합하여 theme 타입을 자동 완성
}
```

이외에도 로컬 이미지나 SVG같이 외부로 노출되어 있지 않은 파일을 모듈로 인식하여 사용할 수 있게끔 만들 수 있다.

```ts
declare module "*.gif" {
  const src: string;
  export default src;
}
```

### declare namespace 활용하기

- Node.js 환경에서 .env 파일을 사용할 때, declare namespace를 활용하여 process.env로 설정값을 손쉽게 불러오고 환경변수의 자동 완성 기능을 쓸 수 있다.

```ts
// .env
API_URL = "localhost:8000";

declare namespace NodeJS {
  interface ProcessEnv {
    readonly API_URL: string;
  }
}

console.log(process.env.API_URL);
// namespace를 활용하여 process.env 타입을 보강해주지 않은 경우는 as 단언 사용
// console.log(process.env.API_URL as string);
```

### declare global 활용하기

- 전역 변수인 Window 객체의 스코프에서 사용되는 모듈이나 변수를 추가할 수 있다.

```ts
declare global {
  interface Window {
    newProperty: string;
  }
}
```

예를 들어 위처럼 전역 변수인 Window 객체에 newProperty 속성을 추가한 것과 같은 동작 구현이 가능하다. 대표적으로 네이티브 앱과의 통신을 위한 인터페이스를 Window 객체에 추가할 때 앰비언트 타입 선언을 활용할 수 있다.

아래는 iOS 웹뷰에서 JS로 네이티브 함수를 호출하기 위한 함수를 정의한 것으로, 함수를 호출할 때 자동 완성 기능을 활용해 실수를 줄일 수 있다.

```ts
declare global {
  interface Window {
    webkit?: {
      messageHandlers?: Record<
        string,
        { postMessage?: (parameter: string) => void }
      >;
    };
  }
}
// window.webkit?. 까지 입력 시 messageHandler? 가 자동 완성됨
```

## ✏️ declare와 번들러의 시너지

```ts
const color = {
  white: "#ffffff",
  black: "#000000",
} as const;

type ColorSet = typeof color;

declare global {
  const _color: ColorSet;
}
```

- 위와 같이 전역에 `_color`라는 변수가 존재함을 타입스크립트 컴파일러에 알리면 해당 객체를 활용할 수 있다.
  ```ts
  const white = _color["white"];
  ```

하지만 타입스크립트 에러는 발생하지 않지만, 아직 ColorSet 타입을 가지고 있는 \_color 객체의 실제 데이터가 존재하지 않아 코드 실행 시 기대하는 동작과 다르게 동작할 수 있다.

이를 해결하기 위한 방법 중 하나가 번들 시점에 번들러릍 통해서 해당 데이터를 주입하는 것이다. declare global로 전역 변수를 선언하는 과정과 번들러를 통해 데이터를 주입하는 절차를 함께 활용하면 시너지를 낼 수 있다.

아래는 롤업(rollup) 번들러의 inject 모듈로 데이터를 주입하는 예시이다. inject 모듈은 import문의 경로를 분석하여 데이터를 가져오는 역할을 한다.

```md
프로젝트 레포 구조
┣ 📂node_modules
┣ 📜data.ts --------------- 색상 정의
┣ 📜index.ts
┣ 📜package-lock.json
┣ 📜package.json
┣ 📜rollup.config.js
┗ 📜type.ts --------------- 타입 정의
```

```ts
// data.ts : 색상 정의
export const color = {
  white: "#ffffff",
  black: "#000000",
} as const;
```

```ts
// index.ts
console.log(_color["white"]);
```

```ts
// rollup.config.js
import inject from "@rollup/plugin-inject";
import typescript from "@rollup/plugin-typescript";

export default [
  {
    input: "index.ts",
    output: [
      {
        dir: "lib",
        format: "esm",
      },
    ],
    plugins: [typescript(), inject({ _color: ["./data", "color"] })],
    // inject 모듈을 사용하여 _color에 해당되는 데이터를 삽입
  },
];
```

- 위 설정을 통해 번들러를 통해 만들어진 결과물을 실행하면 #ffffff가 정상적으로 출력된다.
  [참고자료](https://github.com/Coding-Village-Protector/woowahan-ts/blob/main/%5B12%EC%9E%A5%5D%20%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%20%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%20%EA%B4%80%EB%A6%AC/%EC%9D%B4%EC%84%B1%EB%A0%B9.md)

# 📝 스크립트와 설정 파일 활용하기

## ✏️ 스크립트 활용하기

## ✏️ 설정 파일 활용하기

## ✏️ 에디터 활용하기

# 📝 TS 마이그레이션

## ✏️ TS 마이그레이션의 필요성

## ✏️ 점진적인 마이그레이션

## ✏️ 마이그레이션 진행하기

# 📝 모노레포

## ✏️ 분산된 구조의 문제점

## ✏️ 통합할 수 있는 요소 찾기

## ✏️ 공통 모듈화로 관리하기

## ✏️ 모노레포의 탄생
