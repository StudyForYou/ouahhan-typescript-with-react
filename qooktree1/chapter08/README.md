<h1>JSX에서 TSX로</h1>

# 📝 리액트 컴포넌트의 타입

## ✏️ 클래스 컴포넌트 타입

[@types/react Component 정의](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/0e9ea1e1d4cf9663452c8511564ad421f783708b/types/react/index.d.ts#L475)

```ts
interface Component<P = {}, S = {}, SS = any>
  extends ComponentLifecycle<P, S, SS> {}

class Component<P, S> {
  /** 생략 */
}

class PureComponent<P = {}, S = {}, SS = any> extends Component<P, S, SS> {}
```

- 클래스 컴포넌트가 상속받는 React.Component와 React.PureComponent의 타입 정의는 위와 같으며 P와 S는 각각 props와 상태를 의미한다.

## ✏️ 함수 컴포넌트 타입

```ts
interface WelcomeProps {
  name: string;
}

// 함수 선언식
function Welcome(props: WelcomeProps): JSX.Element {}

// 함수 표현식
const Welcome: React.FC<WelcomeProps> = ({ name }) => {};

// 함수 표현식 - React.VFC 사용(version 18 이후 삭제되었다. )
const Welcome: React.VFC<WelcomeProps> = ({ name }) => {};
```

- **리액트 v18로 넘어오면서 React.VFC가 삭제되고 React.FC에서 children이 사라졌다.**
- 앞으로는 React.VFC 대신 React.FC 또는 props 타입, 반환 타입을 직접 지정하는 형태로 타이핑해줘야 한다.

## ✏️ Children props 타입 지정

### 1. 직접 children에 대한 타입 선언해주기

```tsx
interface Props {
  children: ReactNode;
}

function Button({ children }: Props) {
  return <button>{children}</button>;
}
```

### 2. PropsWithChildren 타입 사용하기

```tsx
// React에서 제공하는 타입
type PropsWithChildren<P> = P & { children?: ReactNode | undefined };

interface Props {
  onClick: () => void;
}

function Modal({ onClick, children }: PropsWithChildren<Props>) {
  return <button onClick={onClick}>{children}</button>;
}
```

- 가장 보편적인 children 타입은 `ReactNode | undefined`가 된다.
- `PropsWithChildren` 타입은 children이 존재할 경우와 아닐 경우 모두 에러가 뜨지 않는다.

### 🚨 StrictPropsWithChildren

- [StrictPropsWithChildren 출처](https://velog.io/@kkojae91/PropsWithChildren%EB%8A%94-%EC%95%88%EC%A0%84%ED%95%9C-%ED%83%80%EC%9E%85%EC%9D%BC%EA%B9%8C)

```ts
export type StrictPropsWithChildren<P = unknown> = P & {
  children: ReactNode;
};
```

- children에 옵셔널을 제거함으로써 `children을 넘겨주지 않을 경우 에러가 띄워질 수 있도록` 설정이 가능하다.

## ✏️ render 메서드와 함수 컴포넌트의 반환 타입 - React.ReactElement vs. JSX.Element vs. React.ReactNode 활용하기

- React.ReactElement, JSX.Element, React.ReactNode 타입은 쉽게 헷갈릴 수 있다.

### ReactElement 타입

```ts
interface ReactElement<
  P = any,
  T extends string | JSXElementConstructor<any> =
    | string
    | JSXElementConstructor<any>
> {
  type: T;
  props: P;
  key: Key | null;
}
```

- React.createElement를 호출하는 형태의 구문으로 변환하면 React.createElement의 반환 타입은 ReactElement이다.
- 리액트는 실제 DOM이 아니라 가상의 DOM을 기반으로 렌더링하는데 `가상 DOM의 엘리먼트는 ReactElement 형태로 저장`된다.
- ReactElement 타입은 `JSX의 createElement 메서드 호출로 생성된 리액트 엘리먼트를 나타내는 타입`

### JSX.ELement 타입

```ts
declare global {
  namespace JSX {
    interface ELement extends React.ReactElement<any, any> {}
  }
}
```

- JSX.Element 타입은 리액트의 ReactElement를 확장하고 있는 타입이며, 글로벌 네임스페이스에 정의되어 있어 외부 라이브러리에서 컴포넌트 타입을 재정의할 수 있는 유연성을 제공한다.

  ### 💡글로벌 네임스페이스(Global Namespace)

  - 프로그래밍에서 식별자(변수, 함수, 타입 등)가 정의되는 전역적인 범위
  - 어떠한 파일이든지 해당 스코프에서 선언된 식별자는 모든 곳에서 접근할 수 있다.

### React.ReactNode 타입

```ts
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
type ReactFragment = {} | Iterable<ReactNode>;

// ReactNode 타입
type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

- `ReactNode는 리액트의 render 함수가 반환할 수 있는 모든 형태를 담고 있다.`
- JSX 형태의 문법을 때로는 string, number, null, undefined 같이 어떤 타입이든 children prop으로 지정할 수 있게 하고 싶다면 ReactNode 타입으로 children을 선언하면 된다.

### ReactNode, JSX.Element, ReactElement 사이의 포함 관계

<img src="./ReactNode.png" />

## ✏️ 사용 예시

### ReactNode 사용 예시

- 리액트의 `Composition(합성)` 모델을 활용하기 위해 prop으로 children을 많이 사용해 봤을 것이다.

  ```ts
  // React에 정의되어 있는 PropsWithChildren 타입
  type PropsWithChildren<P = unknown> = P & {
    children?: ReactNode | undefined;
  };

  // ReactNode 직접 선언
  interface MyComponentProps {
    children?: React.ReactNode;
    // ...
  }

  // PropsWithChildren
  type MyComponentProps = PropsWithChildren<MyProps>;
  ```

  - PropsWithChildren 타입도 children을 ReactNode 타입으로 선언하고 있다.

### JSX.Element 사용 예시

- 리액트 엘리먼트를 prop으로 전달 받아 render props 패턴으로 컴포넌트를 구현할 때 유용하게 활용할 수 있다.

  ```tsx
  interface Props {
    icon: JSX.Element;
  }

  const Item = ({ icon }: Props) => {
    // prop으로 받은 컴포넌트의 props에 접근할 수 있다.
    const iconSize = icon.props.size;

    return <li>{icon}</li>;
  };

  // icon prop에는 JSX.Element 타입을 가진 요소만 할당할 수 있다.
  const App = () => {
    return <Item icon={<Icon size={14} />} />;
  };
  ```

### ReactElement 사용 예시

- 앞서 살펴본 JSX.Element 예시를 확장하여 추론 관점에서 더 유용하게 활용할 수 있는 방법은 JSX.Element 대신에 ReactElement를 사용하는 것이다.
- `ReactElement의 제네릭으로 props를 설정할 수 있기 때문에 prop으로 받은 컴포넌트의 props에 접근할 때 목록이 추론`된다.

  ```tsx
  interface IconProps {
    size: number;
  }

  interface Props {
    icon: React.ReactElement<IconProps>;
  }

  const Item = ({ icon }: Props) => {
    // icon prop으로 받은 컴포넌트의 컴포넌트의 props에 접근하면, props의 목록이 추론된다.
    const iconSize = icon.props.size;
    return <li>{icon}</li>;
  };
  ```

## ✏️ 리액트에서 기본 HTML 요소 타입 활용하기

- 기존 HTML 태그의 속성 타입을 활용하여 타입을 지정하는 방법에 대해 알아보자.
- HTML 태그의 속성 타입을 활용하는 대표적인 2가지 방법은 리액트의 `DetailedHTMLProps`와 `ComponentPropsWithoutRef` 타입을 활용하는 것이다.

### DetailedHTMLProps

```ts
// DetailedHTMLProps 활용 방법
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

type ButtonProps = {
  onClick?: NativeButtonProps["onClick"];
};
```

- 특정 HTML 요소의 모든 속성(기본 HTML 속성 및 React 추가 속성)을 포함하는 타입을 정의하는 데 사용된다.
- 이를 통해 특정 HTML 요소의 타입 안전성을 보장할 수 있다.

### ComponentPropsWithoutRef

```ts
// ComponentPropsWithoutRef 활용 방법
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;
type ButtonProps = {
  onClick?: NativeButtonType["onClick"];
};
```

- ref 속성을 포함하지 않는 컴포넌트의 타입을 정의할 때 사용된다.
- 이는 주로 `forwardRef를 사용하지 않는 컴포넌트에 적합`하다.

### 언제 ComponentPropsWithoutRef를 사용하면 좋을까? [`ref`를 props로 내려줘야 하는 경우]

- 클래스 컴포넌트와 함수 컴포넌트에서 ref를 props로 받아 전달하는 방식에 차이가 있다.

  ```tsx
  class WrappedButton extends React.Component {
    constructor() {
      this.buttonRef = React.createRef();
    }

    render() {
      return (
        <div>
          <Button ref={this.buttonRef} />
        </div>
      );
    }
  }
  ```

  - 클래스 컴포넌트로 만들어진 버튼은 ref가 Button 컴포넌트의 button 태그를 그대로 바라보게 된다.
  - 🚨 하지만 함수 컴포넌트의 경우, 전달받은 ref가 Button 컴포넌트의 button 태그를 바라보지 않는다.
  - 클래스 컴포넌트에서 ref 객체는 마운트된 컴포넌트의 인스턴스를 current 속성값으로 가지지만, 함수 컴포넌트에서는 생성된 인스턴스가 없기 때문에 ref에 기대한 값이 할당되지 않는 것이다.
  - 이러한 제약을 극복하고 함수 컴포넌트에서도 ref를 전달받을 수 있도록 도와주는 것이 `React.forwardRef`메서드이다.

```tsx
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;

const Button = forwardRef<HTMLButtonElement, NativeButtonType>((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  );
});
```

- forwardRef는 2개의 제네릭 인자를 받을 수 있는데, 첫 번째는 ref에 대한 타입 정보, 두 번째는 props에 대한 타입 정보이다.
- 함수 컴포넌트의 props로 DetailedHTMLProps와 같이 ref를 포함하는 타입을 사용하게 되면, 실제로는 동작하지 않는 ref를 받도록 타입이 지정되어 예기치 않은 에러가 발생할 수 있다.
- 따라서, **HTML 속성을 확장하는 props를 설계할 때는 `ComponentPropsWithoutRef` 타입을 사용하여 ref가 실제로 forwardRef와 함께 사용될 때만 props로 전달되도록 타입을 정의하는 것이 안전하다.**
