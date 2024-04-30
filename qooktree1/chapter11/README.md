<h1>CSS-in-JS</h1>

# 📝 CSS-in-JS란?

## ✏️ CSS-in-JS와 인라인 스타일의 차이

- 더 강력한 추상화 수준을 제공한다.
- JS로 스타일을 선언적이고 유지보수할 수 있는 방식으로 표현할 수 있다.

### 인라인 스타일

- HTML 요소 내부에 직접 스타일을 적용하는 방식(HTML 태그의 style 속성을 사용)

```jsx
const textStyles = {
  color: white,
  backgroundColor: black,
};

const SomeComponent = () => <p style={textStyles}>inline style!</p>;
```

```js
// DOM 노드
<p style="color: white; background-color: black;">inline style!</p>
```

### CSS-in-JS(styled-components)

```jsx
import styled from "styled-components";
const Text = styled.div`
  color: white,
  background: black
`;

const Example = () => <Text>{Hello CSS-in-JS!}</Text>;
```

```js
// DOM 노드
<style>
  .hash136s21 {
    background-color: black;
    color:white;
  }
</style>
<p class="hash136s21">Hello CSS-in-JS!</p>
```

- 실제로 CSS가 생성되기 때문에 media query, 슈도 선택자 등과 같은 CSS 기능을 쉽게 누릴 수 있다.

#### 장점

1. 컴포넌트로 생각할 수 있다.
   - 스타일을 컴포넌트 단위로 추상화하여 생각할 수 있게 하여 변도의 스타일시트를 유지보수할 필요없이 각 컴포넌트의 스타일을 관리할 수 있다.
2. 부모와 분리할 수 있다.
   - 각 컴포넌트의 스타일은 부모와 독립적으로 동작한다.(부모에서 자동으로 상속되는 속성이 없다)
3. 스코프를 가진다.
   - CSS로 컴파일될 때 고유한 이름을 생성하여 스코프를 만들어주기 때문에 선택자 충돌을 방지할 수 있다.
4. 자동으로 벤더 프리픽스가 붙는다.
   - 자동으로 벤더 프리픽스를 추가하여 브라우저 호환성을 향상해준다.
   - `vendor prefix?`: 웹 브라우저마다 지원되는 CSS 속성이나 기능이 다를 때 특정 브라우저에서 제대로 동작하도록 하기 위해 추가되는 접두사
5. `JS와 CSS 사이에 상수와 함수를 쉽게 공유할 수 있다.`

## ✏️ CSS-in-JS 등장 배경

- 스타일링 라이브러리 종류

  - CSS Preprocessor [ sass/scss, less, stylus ]
  - CSS in JS [ styled-components, emotion ]

- 웹 환경이 변하면서 요구사항이 다양해지고 복잡도도 계속 증가했다. 특히 선택자(selector)의 복잡도가 높아짐에 따라 CSS 전처리기 방식이 등장하게 되었다.
- 또, 웹 개발에 컴포넌트/모듈 방식이 적용됨에 따라 CSS Modules를 시작으로 JS에서 CSS를 생성하는 방식이 도입되었다.

2014년 11월, 메타의 엔지니어인 크리스토퍼 쉬도는 규모가 크고 동적인 웹 애플리케이션을 유지보수하기 위해 해결해야 할 CSS의 문제점을 7가지로 분류하여 설명하면서 해결책으로 CSS-in-JS 개념을 제시했다.

[참고 자료](https://blog.vjeux.com/2014/javascript/react-css-in-js-nationjs.html)

1. Global Namespace

- 모든 스타일을 global에 선언하기 때문에, 별도의 class 명명 규칙을 적용해야 한다. (BEM, OOCSS, SMACSS 방법론들이 등장)

2. Dependencies(의존성)??

- 하나의 element에 여러 CSS 룰이 적용되기 때문에, 모든 스타일을 개발자가 기억해야 하는 문제

3. Dead Code Elimination(불필요한 코드 제거)

- 기능 CRUD 과정에서 불필요한 CSS를 삭제하기 어렵다.(유지보수 문제)

4. Minification(최소화)

- 애플리케이션이 커지며 CSS 클래스 이름들이 늘어나 최소화하기 어렵다.

5. Sharing Constants(상수 공유)

- CSS가 분리되어 있어 JS의 상태 값을 공유하기 어려움.(현재는 CSS Variable이 도입되어 CSS의 공식 기능으로 제공됨)

6. Non-deterministic Resolution(비결정적 해결)

- CSS 로드 순서에 따라 우선순위가 달라진다.

7. Isolation(고립)

- CSS의 외부 수정을 관리하기 어렵다.

### 🚨 하지만 과연 CSS-in-JS가 좋을까?

- 개인적인 견해로 CSS-in-JS는 서비스를 컴포넌트 기반으로 작성하고 소규모 팀일 때 사용하는 것이 유용하다는 생각이 듭니다.
- CSS-in-JS 방식은 CSS 코드를 변환하는 과정이 필요하기 때문에 실제 styled-components와 CSS Modules의 `Rendering`, `Scripting`, `Painting` 소요 시간을 비교해보면 `Scripting`에서 거의 2배에 가까운 성능차이를 보입니다. [카카오 기술블로그](https://fe-developers.kakaoent.com/2022/220210-css-in-kakaowebtoon/?fbclid=IwAR10q4zhGGIdjdd7l9VyDC9Xn0AU0UkoGeDmTyoWotpt6cxn4T6WHLMPkGg)
- 성능과 생산성을 높이기 위해서는 `CSS-Modules` 혹은 `tailwindCSS`를 이용하는 방법이 좋다고 생각합니다.

## ✏️ CSS-in-JS 사용하기

```tsx
// emotion 사용 예제
import styled from "@emotion/styled";

export const Button = styled.button<{ primary: boolean }>`
  background-color: transparent;
  border: none;
  cursor: pointer;
  font-size: inherit;
  color: ${({ primary }) => (primary ? "red" : "blue")};
`;
```

- 다음 코드와 같이 템플릿 리터럴을 활용해서 동적인 스타일을 정의하여 사용 가능하다.
  ```ts
  const RoundButton = styled(Button)``;
  const SquareButton = styled(Button)``;
  ```
  - 추가적으로 공통적인 버튼 스타일을 사용하여 확장성 있는 컴포넌트 스타일을 만들 수 있다.

# 📝 유틸리티 함수를 활용하여 styled-components의 중복 타입 선언 피하기

컴포넌트에서 사용하는 props 뿐만 아니라 styled-components로 전달되는 스타일 관련 props의 타입도 정의해줘야한다. 이때 TS에서 제공하는 Pick, Omit 같은 유틸리티 타입을 활용할 수 있다.

## ✏️ props 타입과 styled-components 타입의 중복 선언 및 문제점

```ts
interface Props {
  height?: string;
  color?: keyof typeof colors;
  isFull?: boolean;
  className?: string;
  // ...
}

export const Hr: VFC<Props> = ({ height, color, isFull, className }) => {
  return (
    <HrComponent
      height={height}
      color={color}
      isFull={isFull}
      className={className}
    />
  );
};

// height, color, isFull에 대한 타입만 사용하기 위해 StyledProps 타입을 정의하며 중복되는 코드가 발생한다.
interface StyledProps {
  height?: string;
  color?: keyof typeof colors;
  isFull?: boolean;
}

// StyledProps => Pick<Props, "height" | "color" | "isFull>
// 중복되는 타입을 피할 수 있어 중복되는 코드의 제거 및 유지보수 측면에서 긍정적인 효과를 얻을 수 있다.
const HrComponent = styled.hr<StyledProps>`
  height: ${({ height }) => height || "10px"};
  margin: 0;
  background-color: ${({ color }) => colors[color || "gray7"]};
  border: none;

  ${({ isFull }) =>
    isFull &&
    css`
      margin: 0 -15px;
    `}
`;
```
