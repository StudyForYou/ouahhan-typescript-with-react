# 1. CSS-in-JS와 인라인 스타일의 차이

### 인라인 스타일이 뭐지?

HTML요소 내부에 직접 스타일을 적용하는 방식. HTML태그의 style속성을 사용하여 인라인 스타일 적용 가능

```tsx
const textStyle = {
	color: white,
	backgroundColor: black,
}

const ExampleComponent = () => {
	return (
		<p style={textStyle} inLine style! </p>
		);
	};
	
```

이 코드는 브라우저에서 DOM노드를 다음과 같이 연결한다.

```tsx
<p style= "color: white; background-color: black"> inline style! </p>
```

### CSS-in-JS는 뭐지?

```tsx
import styled from "styled-components";
const Text = styled.div`
	color: white,
	background: black,
`;

//다음처럼 사용
const Example = () => <Text>{Hello Css-in-JS}</Text>;
```

이 코드는 브라우저에서 DOM노드를 다음과 같이 연결한다

```tsx
<style>
	.hash135s21 {
		background-color: black;
		color: white;
	}
</style>

<p class="hash135s21">Hello CSS-in-JS</p>
```

인라인 스타일은 DOM노드에 속성으로 스타일을 추가하는 반면, Css-in-JS는 DOM상단에 <style>태그를 추가한다.

CSS-in-JS를 사용하면 실제로 CSS가 생성됨!

⇒ 미디어쿼리, 슈도 선택자 같은 CSS기능을 누릴 수 있음

### CSS-in-JS의 장점

- 컴포넌트로 생각할 수 있다 : 스타일을 컴포넌트 단위로 추상화하여 생각하게 해준다.
    
    ⇒별도의 스타일시트를 유지보수할 필요 없이 컴포넌트의 스타일을 관리할 수 있음
    

- 부모와 분리 가능 : 명시적으로 정의하지 않은 CSS는 부모 요소에서 자동으로 상속된다. 하지만 CSS-in-JS는 상속을 받지 않는다. 따라서 각 컴포넌트의 스타일이 부모와 독립되어 독립적으로 동작한다.

- 스코프를 가진다 : CSS는 하나의 전역 네임스페이스를 가지기 때문에 선택자 충돌 가능성이 있다. (하나의 프로젝트 내에서는 BEM같은 네이밍 컨벤션이 도움을 줄 수 있지만, 서드파티 코드를 통합할 때에는 도움이 되지 않음) CSS-in-JS는 컴파일시 고유한 이름을 생성하여 스코프를 만들어주므로 선택자 충돌을 방지할 수 있다.

- 자동으로 벤더 프리픽스가 붙는다  ⇒ 브라우저 호환성 향상

- 자바스크립트와 ↔ CSS 사이에서 상수와 함수를 쉽게 공유 가능 : 자바스크립트 변수, 상수, 함수를 스타일 코드 내에서 쉽게 사용할 수 있다

### CSS-in-JS 등장 배경

리액트 컴포넌트를 스타일링하기 위해 순수하게 CSS만 사용할 수도 있지만 스타일링 라이브러리를 적용할 수도 있다.

- CSS Preprocessor
    - sass/scss
    - less
    - stylus
- CSS-in-JS
    - styled-component
    - emotion

### CSS의 문제점

- 글로벌 네임스페이스 : 스타일이 전역 공간을 공유하므로 중복되지 않는 클래스 이름을 고민해야 함
- 의존성 : CSS의 의존성과 자바스크립트의 의존성이 달라서 사용하지 않는 스타일
- 불필요한 코드 제거 : 기능 추가·수정·삭제 과정에서 불필요한 CSS를 삭제하기 어렵다
- 최소화 : 클래스 이름을 최소화하기 어렵다
- 스타일 우선순위 문제 : CSS의 로드 순서에 따라 스타일 우선순위가 달라진다.
- Isolation : CSS의 외부 수정을 관리하기 어렵다

⇒ 이런 문제점을 보완하기 위해 CSS-in-JS라이브러리가 등장했다!

⇒ 스타일을 컴포넌트의 일부로 간주, 스타일을 포함한 컴포넌트 단위로 분리가 가능해짐

### CSS-in-JS 사용하기

- 템플릿 리터럴을 활용해서 동적인 스타일을 정의하면 된다. 먼저 props의 타입을 정의하고, 이 props를 활용해서 동적인 스타일링을 구현한다.

```tsx
//emotion CSS 정의 예제

export const Button = styled.button<{ primary: boolean }>`
  background: transparent;
  border: none;
  cursor: pointer;
  font-size: inherit;
  padding: 0;
  margin: 0;
  color: ${({ primary }) => (primary ? 'red' : 'blue')};
`;
```

```tsx
import { css, SerializedStyles } from '@emotion/react';
import styled from '@emotion/styled';

type ButtonRadius = 'xs' | 's' | 'm' | 'l';

export const buttonRadiusStyleMap: Record<ButtonRadius, SerializedStyles> = {
  xs: css`
    border-radius: ${radius.extra_small};
  `,
  s: css`
    border-radius: ${radius.small};
  `,
  m: css`
    border-radius: ${radius.medium};
  `,
  l: css`
    border-radius: ${radius.large};
  `,
};

export const Button = styled.button<{ radius: string }>`
  ${({ radius }) => css`
    /* ...기타 스타일은 생략 */
    ${buttonRadiusStyleMap[radius]}
  `}
`;
```

# 2. 유틸리티 함수로  styled-components의 중복 타입 선언 피하기

리액트는 여러 옵션을 props로 받아 유연한 컴포넌트를 구현할 수 있다.

이때 스타일 관련 props는 styled-components로 전달되는데 해당  타입을 styled-components에서도 정의해주어야 한다.

⇒ 이 때 타입스크립트의 Pick, Omit같은 유틸리티 타입을 유용하게 활용할 수 있다!


```tsx
//유틸리티 타입을 활용하지 않은 예제

interface Props {
  height?: string;
  color?: keyof typeof colors;
  isFull?: boolean;
  className?: string;
  // ...
}

export const Hr: VFC<Props> = ({ height, color, isFull, className }) => {
  // ...
  return (
    <HrComponent
      height={height}
      color={color}
      isFull={isFull}
      className={className}
    />
  );
};

//props와 똑같은 타입이지만 StyledProps를 별도로 정의해주어야 함 => 코드 중복
//props가 변경되면 StyledProps도 변경해주어야 함
interface StyledProps {
  height?: string;
  color?: keyof typeof colors;
  isFull?: boolean;
}

const HrComponent = styled.hr<StyledProps>`
  height: ${({ height }) => height || '10px'};
  margin: 0;
  background-color: ${({ color }) => colors[color || 'gray7']};
  border: none;

  ${({ isFull }) =>
    isFull &&
    css`
      margin: 0 -15px;
    `}
`;
```

```tsx
//유틸리티 타입을 활용한 예제 : 중복되는 타입 선언을 피할 수 있다! 
const HrComponent = styled.hr<Pick<Props, 'height' | 'color' | 'isFull'>>`
  // ...
`;
```
