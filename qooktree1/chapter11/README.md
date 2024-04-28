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
   - `벤더 프리픽스란?`: 웹 브라우저마다 지원되는 CSS 속성이나 기능이 다를 때 특정 브라우저에서 제대로 동작하도록 하기 위해 추가되는 접두사
5. JS와 CSS 사이에 상수와 함수를 쉽게 공유할 수 있다.

## ✏️ CSS-in-JS 등장 배경

## ✏️ CSS-in-JS 사용하기

# 📝 유틸리티 함술르 활용하여 styled-components의 중복 타입 선언 피하기

## ✏️ props 타입과 styled-components 타입의 중복 선언 및 문제점
