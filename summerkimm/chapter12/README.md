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

## 12.2 스크립트와 설정 파일 활용하기

### 1. 스크립트 활용하기

실시간으로 타입 에러 확인하기

```
yarn tsc -noEmit -incremental -w
npx tsc -noEmit -incremental -w
```

- noEmit : 자바스크립트로 된 출력 파일을 생성하지 않도록 하는 옵션
- incremental : 증분 컴파일을 활성화하여 컴파일 시간을 단축할 수 있게 하는 옵션
- w : 파일 변경 사항을 모니터링한다는 뜻

  > **증분 컴파일**  
  > 매번 모든 대상을 컴파일하는 것이 아니라 변경 사항이 있는 부분만을 컴파일하는 것을 말하며, 이를 활용하면 컴파일 시간을 줄일 수 있다.

#### 타입 커버리지 확인하기

타입스크립트 통제하에 프로그램이 잘 돌아가고 있는지 확인이 필요할 때 유용함

```
npx type-coverage --detail
```

현재 프로젝트의 타입 커버리지와 any를 사용하고 있는 변수의 위치를 확인 할 수 있다

### 2. 설정 파일 활용하기

#### 타입스크립트 컴파일 속도 높이기

tsconfig의 incremental 속성을 true로 설정하면 증분 컴파일이 활성화되어 매번 모든 대상을 컴파일 하는 것이 아니라 변경된 부분만 컴파일된다. 이로써 컴파일타임을 줄일 수 있다.

```json
// tsconfig에 추가
{
  "compilerOptions": {
    ...
    incremental: true
  }
}
```

```
npx tsc -noEmit -incremental -diagnostics
```

### 3. 에디터 활용하기

에디터에서 타입 스크립트 서버 재시작하기  
VSCode에서 Restart TS server 기능을 지원함

- mac : command + shift + P
- window: ctrl + shift + P

## 12.3 타입스크립트 마이그레이션

### 1. 타입스크립트 마이그레이션의 필요성

우아한 형제들에서는 타입스크립트로 프로젝트를 새로 구축한 사례가 기존 프로젝트를 타입스크립트로 마이그레이션한 사례보다 많다.

- 기존 코드를 마이그레이션하는 것은 기존 코드의 구조적인 한계가 드러날 수 있음
- 새로운 설계는 상황에 따라 비즈니스 요구 사항의 변화를 반영할 수 있음

결론 : 무작정 마이그레이션하는 것보다 여러 조건을 따져서 가장 효율적인 방법을 택하자

### 2. 점진적인 마이그레이션

결론 : 우선순위를 설정하고 작은 부분부터 시작하여 점진적으로 범위를 넓혀가자

### 3. 마이그레이션 진행하기

1. 타입스크립트 개발 환경을 설정하고, 빌드 파이프라인에 타입스크립트 컴파일러를 통일한다.  

   **tsconfig.json -> allowJS: true, noImplicitAny: false**

  - allowJS는 자바스크립트 파일을 컴파일할 때 사용하는 옵션.    
  기본 자바스크립트 함수를 타입스크립트에서 임포트하거나 반대로 타입스크립트 함수를 자바스크립트에서 임포트할 수 있게 해줌
  - noImplicitAny는 암시적 any 타입이 있을 때 오류가 발생하게 하는 옵션.  
  타입을 점진적으로 추가하는 과정에서는 오류가 발생하지 않도록 false로 설정해야 함

2. 작성된 자바스크립트 파일을 타입스크립트 파일로 변환함.    
    이 단계에서 필요한 타입과 인터페이스를 하나씩 정의하며 함수 시그니처를 추가해나간다.

3. 기존 자바스크립트 파일을 모두 타입스크립트로 변환하는 작업이 완료되면 tsconfig.json 파일에서 allowJS를 false, noImplicitAny를 true로 설정하여 타입이 명시되지 않은 부분이 있는지 확인

## 12.4. 모노레포
여러 프로젝트를 관리하는 경우 일반적으로 개별 프로젝트마다 별도의 레포지토리를 생성하여 관리함. 이때 프로젝트마다 공통된 요소가 있다면 이를 통합하여 효율적으로 관리할 수 있다. 

### 1. 분산된 구조의 문제점 
만약 프로젝트에 필요한 기능이 다른 프로젝트에 존재한다면 해당 기능을 복사 붙여넣기를 통해 빠르게 구현할 수 있음   
- 문제점 
  - 새로운 버그가 발견되거나 기능 확장을 위해 해당 기능을 수정해야 하는 경우 프로젝트의 개수만큼 반복적인 수정 작업을 해야함
  - 특정 라이브러리에 문제가 생기거나 더 이상 사용되지 않는 경우에도 일일이 대응해야 함 

- 해결책  
  반복되는 코드를 통합해 한 곳에서 프로젝트를 관리할 수 있도록 해야 함 

### 2. 통합할 수 있는 요소 찾기 
프로젝트 내에서 공통으로 통합할 수 있는 요소를 찾아 통합한다.

### 3. 공통 모듈화로 관리하기
npm과 같은 패키지 관리자를 활용하여 공통 모듈을 생성하고 관리하면 각 프로젝트에서 간편하게 모듈과 의존성을 맺고 사용할 수 있고, 유지보수도 쉬워짐 

### 4. 모노레포의 탄생 
모노레포 : 버전 관리 시스템에서 여러 프로젝트를 하나의 레포지토리로 통합하여 관리하는 소프트웨어 개발 전략    
- 모놀리식 기법 
  - 다양한 기능을 가진 프로젝트를 하나의 레포지토리로 관리하는 기법    
  - 단점 : 코드 간의 직접적인 의존이 발생하여 일부 로직만 변경될 때도 전체 프로젝트에 영향을 줌. 설계적인 픅면과 빌드 및 배포 등에서 효율적이지 못함   

- 폴리레포 방식 : 거대한 프로젝트를 작은 프로젝트의 집합으로 나누어 관리하는 방식
- 모노레포 방식 : 하나의 레포지토리로 모든 것을 관리하는 방식 

#### 모노레포로 관리했을 때의 장점
- Lint, CI/CD 등 개발 환경 설정까지도 통합적으로 관리하기 때문에 불필요한 코드 중복을 줄여줌 
- 공통 모듈도 동일한 프로젝트 내에서 관리되므로 별도의 패키지 관리자가 필요 없음    
#### 모노레포로 관리했을 때의 단점
- 모노레포로 프로젝트를 관리할 때 시간이 지나면서 레포지토리가 거대해질 수 있음
- 하나의 레포지토리에 여러 팀의 이해관계가 얽혀 있으면 소유권과 권한 관리가 복잡해질 수 있음    
-> 각 프로젝트나 모듈의 소유권을 명확히 정의하고 규칙을 설정해야 하는 과정이 별도로 필요함 
