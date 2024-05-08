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
- [참고자료](https://github.com/Coding-Village-Protector/woowahan-ts/blob/main/%5B12%EC%9E%A5%5D%20%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%20%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%20%EA%B4%80%EB%A6%AC/%EC%9D%B4%EC%84%B1%EB%A0%B9.md)

# 📝 스크립트와 설정 파일 활용하기

TS 프로젝트에서 script와 tsconfig 등을 잘 활용하면 개발 생산성을 높일 수 있다.

## ✏️ 스크립트 활용하기

### 실시간으로 타입을 검사하자

- 에디터가 가능한 빠르게 타입 에러를 감지하지만 프로젝트 규모가 커지거나 컴퓨터 성능이 떨어지면 타입 에러를 알려주는 속도가 느려진다.
- 이럴 때 아래의 스크립트를 사용하여 실시간으로 에러를 확인할 수 있다.

```powershell
$ yarn tsc --noEmit --incremental --w
# tsc: TS 컴파일러
# noEmit: JS로 된 출력 파일을 생성하지 않도록 설정
# incremental: 증분 컴파일을 활성화하여 컴파일 시간을 단축
# w: 파일 변경 사항을 모니터링
```

#### 증분 컴파일?

- 매번 모든 대상을 컴파일하는 것이 아니라 변경 사항이 있는 부분만을 컴파일하는 것 (컴파일 시간 감소)

`package.json`의 scripts에 추가하여 키워드로 사용 가능하다.

```json
// package.json
{
  "scripts": {
    "type-check": "tsc --noEmit --incremental -w"
  }
}
```

```powershell
$ npm run type-check
```

### 타입 커버리지 확인하기

```powershell
$ npx type-coverage --detail
```

- 현재 프로젝트의 타입 커버리지와 any를 사용하고 있는 변수의 위치를 나타낸다.
- 타입 커버리지: 변수 전체의 몇 퍼센트가 타입이 지정되어 있는 지 확인 가능

## ✏️ 설정 파일 활용하기

### TS 컴파일 속도 높이기

- `incremental` 속성을 활용하여 TS 컴파일 속도를 높일 수 있다.
- 속성을 `true`로 설정하면 증분 컴파일이 활성화되어 변경된 부분만 컴파일하게 된다.

```json
// 1. tsconfig에 추가하는 방법
{
  "compilerOptions": {
    ...
    incremental: true
  }
}
```

```powershell
# 2. 스크립트 활용하는 방법
$ yarn tsc --noEmit --incremental --diagnostic
```

## ✏️ 에디터 활용하기

### 에디터에서 TS 서버 재시작하기

- 정의된 타입이 있는 객체인데도 import되지 않거나 자동 완성 기능이 동작하지 않는 경우에는 TS 서버를 재실행하자.
- VSCode에서는 `Ctrl + Shift + P`를 누르고 `Restart TS Server` 클릭
- WebStorm에서는 에디터 오른쪽 하단 바에 위치한 TypeScript 메뉴에서 `Restart TypeScript Service` 클릭

# 📝 TS 마이그레이션

## ✏️ TS 마이그레이션의 필요성

TS를 새로운 기술 스택으로 도입하게 되면 다음과 같이 2가지 방법이 있다.

1. 기존 프로젝트를 TS로 마이그레이션
2. TS 프로젝트로 새로 구축

- JS를 무조건 TS로 마이그레이션하는 것보다 프로젝트의 규모와 특성 및 내외부 여건을 종합적으로 고려하여, 선택하는 것이 옳다.

## ✏️ 점진적인 마이그레이션

- 작은 부분부터 시작하여 점차 범위를 넓혀가며 마이그레이션을 진행하는 것이 옳다.
- 진입 장벽이 낮아지고 프로젝트의 전반적인 동작을 안정적으로 유지할 수 있다.

## ✏️ 마이그레이션 진행하기

다음과 같은 단계를 거친다.

### 1. TS 개발 환경을 설정하고, 빌드 파이프라인에 TS 컴파일러를 통합한다.

- tsconfig.json 파일에서 allowJS를 true로 noImplicitAny를 false로 설정한다.

### 2. 작성된 JS 파일을 TS 파일로 변환한다.

- 필요한 타입과 인터페이스를 하나씩 정의하며 함수 시그니처를 추가해나간다.

### 3. tsconfig.json 파일에서 allowJS를 false로 변경하고, noImplicitAny를 true로 설정하여 타입이 명시되지 않은 부분이 없는지 점검한다.

- 스크립트를 활용하는 것도 하나의 방법이다.

# 📝 모노레포

- 여러 프로젝트를 관리하는 경우에는 일반적으로 개별 프로젝트마다 별도의 repository를 생성하여 관리한다.
- 만약 프로젝트들의 공통된 요소를 찾아 통합한다면 더 효율적으로 관리할 수 있다.

## ✏️ 분산된 구조의 문제점

- 여러 개의 repo에 프로젝트를 위한 Jest, Babel, TS 등의 설정 파일을 별도로 구성하면 각각 반복되는 작업이 늘어나지만 이를 합치면 시간적인 부분에서 메리트가 발생한다.
- 또, 프로젝트 관리 측면에서 하나의 코드가 오류가 발생하면 다른 프로젝트에서도 오류를 수정해야하는 번거로움이 생길 수 있다.
- `공통 컴포넌트를 왜 사용하는지 생각해보면 알 수 있는 문제인 것 같다.`

## ✏️ 통합할 수 있는 요소 찾기

- 프로젝트 내에서 공통으로 통합할 수 있는 요소를 찾고 만약 각 파일의 소스코드가 같지 않다면 통합을 위해 일부 수정해야 한다.

## ✏️ 공통 모듈화로 관리하기

- 소스코드를 수정한 다음 모듈화를 통해 통합할 수 있다.
- npm과 같은 패키지 관리자를 활용하여 공통 모듈을 생성하고 관리한다면 각 프로젝트에서 간편하게 모듈과 의존성을 맺고 사용할 수 있게 된다.
- 새로운 프로젝트를 시작하더라도 모듈을 통해 코드를 재사용할 수 있고 유지보수도 쉬워진다.
- 하지만, 공통 모듈에 변경이 발생한다면 해당 모듈을 사용하는 프로젝트에서도 추가 작업이 필요할 수 있다.
- 또한 공통 모듈의 개수가 늘어나면 관리해야 할 repo도 그만큼 늘어난다.

## ✏️ 모노레포의 탄생

### 모노레포란

- **버전 관리 시스템에서 여러 프로젝트를 하나의 레포지토리로 통합하여 관리하는 소프트웨어 개발 전략**

### 다른 소프트웨어 개발 전략

- 모놀리식: 다양한 기능을 가진 프로젝트를 하나의 레포지토리로 관리하는 구조
- 폴리레포: 거대한 프로젝트를 작은 프로젝트의 집합으로 나누어 관리하는 구조

### 모노레포의 장점

- 불필요한 코드 중복을 줄여준다.
- 별도의 패키지 관리를 통해 모듈을 게시하지 않아도 된다.
- 기능 변화를 쉽게 추적하고 의존성을 관리할 수 있게 된다.

### 모노레포의 단점

- 시간이 지나며 repo가 거대해질 수 있다. -> 빌드 속도가 느려진다.
- 하나의 repo에 여러 팀의 이해관계가 얽혀있다면 소유권과 권한 관리가 복잡해질 수 있다.
- 각 프로젝트나 모듈의 소유권을 명확히 정의하고 규칙을 설정해야 하는 과정이 별도로 필요하다.
