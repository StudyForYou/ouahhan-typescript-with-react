<h1>훅</h1>

### 클래스 컴포넌트

- componentDidMount, componentDidUpdate와 같이 하나의 생명주기 함수에서만 상태 없데이트에 따른 로직을 실행시킬 수 있었다.
- 또한 모든 상태를 하나의 함수 내에서 처리하다 보니 관심사가 뒤섞이게 되었고 상태에 따른 테스트나 잘못 발생한 사이드 이펙트의 디버깅이 어려워졌다.

### 리액트 훅의 도입

- 리액트 훅이 도입되면서 함수 컴포넌트에서도 클래스 컴포넌트와 같이 컴포넌트 생명주기에 맞춰 로직을 실행할 수 있게 되었다.
- 이에 따라 비즈니스 로직을 재사용하거나 작은 단위로 코드를 분할하여 테스트하는 게 용이해졌으며 사이드 이펙트와 상태를 관심사에 맞게 분리하여 구성할 수 있게 되었다.

# 📝 리액트 훅

## ✏️ useState

### useState의 타입 정의

```ts
function useState<S>(
  initialState: S | (() => S)
): [S, Dispatch<SetStateAction<S>>];

type Dispatch<A> = (value: A) => void;
type SetStateAction<S> = S | ((prevState: S) => S);
```

- useState가 반환하는 튜플의 두 번째 요소는 상태를 업데이트할 수 있는 Dispatch 타입의 함수이다.
- Dispatch 함수의 제네릭으로 지정한 SetStateAcction에는 useState로 관리할 상태 타입인 S 또는 이전 상태 값을 받아 새로운 상태를 반환하는 함수인 (prevState: S) => S가 들어갈 수 있다.
- 이처럼 useState를 동기적으로 처리하기 위해 사용한다.

## ✏️ 의존성 배열을 사용하는 훅

### useEffect

```ts
// useEffect의 타입 정의
function useEffect(effect: EffectCallback, deps?: DependencyList): void;
type DependencyList = ReadonlyArray<any>;
type EffectCallback = () => void | Destructor;
```

- 첫 번째 인자인 EffectCallback은 Destructor를 반환하거나 아무것도 반환하지 않는 함수이다.
- `Promise 타입은 반환하지 않으므로 useEffect의 콜백 함수에는 비동기 함수가 들어갈 수 없다.`
- 두 번째 인자인 deps는 옵셔널하게 제공되며 effect가 수행되기 위한 조건을 나열한다.
- 다만 deps의 원소로 숫자나 문자열 같은 TS 기본 자료형이 아닌 객체나 배열을 넣을 때는 주의해야 한다.
  - 얕은 비교로만 판단하기 때문에 실체 객체 값이 바뀌지 않았더라도 객체의 참조 값이 변경되면 콜백 함수가 실행된다.

#### Destructor(클린업 함수)

- 컴포넌트가 언마운트될 때 실행되는 함수이다.
- deps 배열이 존재한다면, 배열의 값이 변경될 때마다 Destructor가 실행된다.

### useLayoutEffect

- 타입 정의는 useEffect와 동일하며 하는 역할의 차이만 있다.
- useEffect의 경우 레이아웃 배치와 화면 렌더링이 모두 완료된 후에 실행된다.
- `useLayoutEffect를 사용하면 화면에 해당 컴포넌트가 그려지기 전에 콜백 함수를 실행한다.`

### useMemo와 useCallback

- 이전에 생성된 값 or 함수를 기억하며, 동일한 값과 함수를 반복해서 생성하지 않도록 해주는 훅이다.
- `어떤 값을 계산하는 데 오랜 시간이 걸릴 때나 렌더링이 자주 발생하는 form에서 유용하게 사용된다.`

```ts
type DependencyList = ReadonlyArray<any>;

function useMemo<T>(factory: () => T, deps: DependencyList | undefined): T;
function useCallback<T extends (...args: any[]) => any>(
  callback: T,
  deps: DependencyList
): T;
```

- 불필요한 곳에 사용하지 않도록 하는 것이 중요하다.
- 모든 값과 함수를 useMemo와 useCallback을 사용해서 과도하게 메모이제이션하면 컴포넌트의 성능 향상이 보장되지 않을 수 있다.

#### 메모이제이션(Memoization)

- 이전에 계산한 값을 저장함으로써 같은 입력에 대한 연산을 다시 수행하지 않도록 최적화하는 기술

## ✏️ useRef

- 리액트에서 요소에 focus를 설정하거나 특정 컴포넌트의 위치로 스크롤하는 등 `DOM을 직접 선택해야 하는 경우 사용된다.`

### 타입 정의

- 세 종류의 타입 정의를 가지고 있으며 오버로딩되어 있다.

  ```ts
  /**
   * 1. 제네릭으로 HTMLInputElement | null 사용시 타입 반환
   [example]
   useRef<number>(0)
   useRef<HTMLInputElement | null>(null);
  */
  function useRef<T>(initialValue: T): MutuableRefObject<T>;

  /**
   * 2.제네릭으로 HTMLInputElement, 인자로 null 사용시 타입 반환
   [example]
   useRef<number>(null);
   useRef<HTMLInputElement>(null);
   */
  function useRef<T>(initialValue: T | null): RefObject<T>;

  /** 
   * 3. 초기값 없이 useRef 사용시 타입 반환
   [example]
   useRef<HTMLInputElement>();
  */
  function useRef<T = undefined>(): MutuableRefObject<T | undefined>;

  interface MutableRefObject<T> {
    current: T; // current 값 변경 가능
  }

  interface RefObject<T> {
    readonly current: T | null; // current 값 변경 불가능 (readonly)
  }
  ```

  - 주의해야 할 점은 RefObject 타입은 current 속성을 직접 수정하려고 하면 안되지만 안의 하위 프로퍼티인 `value`는 여전히 수정이 가능하다. 따라서, `current.value` 값을 수정하는 경우는 shallow하기 때문에 가능하다.

### 자식 컴포넌트에 ref 전달하기

- ref라는 속성의 이름은 리액트에서 `DOM 요소 접근`이라는 특수한 목적으로 사용되기 때문에 props를 넘겨주는 방식으로 전달할 수 없다. 따라서 리액트 컴포넌트에서 ref를 prop으로 전달하기 위해서는 `forwardRef`를 사용해야 한다.
- **다만 이름을 ref가 아닌 inputRef 등의 다음 이름을 사용한다면 forwardRef를 사용하지 않아도 된다.**

  ```tsx
  import { useRef } from 'react';

  interface Props {
    name: string;
  }

  const MyInput = forwardRef<HTMLInputElement, Props>((props, ref) => {
    return (
      <div>
        <label>{props.name}</label>
        <input ref={ref} />
      </div>
    )
  })

  const Component = () => {
    const ref = useRef<HTMLInputElement>(null);
    return <MyInput ref={ref}>;
  }
  ```

- forwardRef의 타입 정의

  ```ts
  function forwardRef<T, P = {}>(
    render: ForwardRefRenderFunction<T, P>
  ): ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>;

  // forwardRef에 인자로 넘겨주는 콜백함수의 타입 정의
  // P는 일반적인 리액트 컴포넌트에서 자식 컴포넌트로 넘겨주는 props 타입
  // T는 ref로 전달하려는 요소의 타입
  interface ForwardRefRenderFunction<T, P = {}> {
    (props: P, ref: ForwardedRef<T>): ReactElement | null;
    displayName?: string | undefined;
    defaultProps?: never | undefined;
    propTypes?: never | undefined;
  }

  type ForwardedRef<T> =
    | ((instance: T | null) => void)
    | MutableRefObject<T | null>
    | null;
  ```

  - useRef의 반환타입은 `MutableRefObject<T>` 또는 `RefObject<T>`가 될 수 있다고 했다.
  - 하지만 `ForwardedRef에는 오직 MutableRefObject`만 들어올 수 있다.
  - `MutableRefObject`가 `RefObject`보다 넓은 범위의 타입을 가지고 있기 때문에, 부모 컴포넌트에서 ref를 어떻게 선언했는지와 관계없이 자식 컴포넌트가 해당 ref를 수용할 수 있다.

### useImperativeHandle

- ref에 할당되는 값을 `DOM 객체가 아닌 컴포넌트 내부에서 커스터마이징한 객체로 변경`할 수 있다.
- 즉, 자식 컴포넌트에서 노출하고 싶은 ref 객체를 따로 정의할 수 있는 셈이다.

```tsx
const ChildComponent = forwardRef((props, ref) => {
  const inputRef = useRef();

  // useImperativeHandle을 사용하여 부모 컴포넌트에 노출할 메서드를 정의합니다.
  useImperativeHandle(ref, () => ({
    focus: () => {
      childRef.current.focus();
    },
  }));

  return <input ref={inputRef} />;
});

function ParentComponent() {
  // 자식 컴포넌트의 ref를 생성합니다.
  const childRef = useRef();

  // 자식 컴포넌트의 메서드를 부모 컴포넌트에서 사용합니다.
  const handleFocus = () => {
    childRef.current.focus();
  };

  return (
    <div>
      {/* 자식 컴포넌트를 렌더링하고 ref를 전달합니다. */}
      <ChildComponent ref={childRef} />
      {/* 버튼을 클릭하면 자식 컴포넌트의 focus 메서드가 호출됩니다. */}
      <button onClick={handleFocus}>Focus</button>
    </div>
  );
}

export default ParentComponent;
```

### useRef의 여러 가지 특성

- 자식 컴포넌트를 저장하는 변수로 활용
- 상태가 변경되더라도 불필요한 리렌더링을 피할 수 있다.

```tsx
const Banner = ({ autoplay }: { autoplay: boolean }) => {
  const isAutoPlayPause = useRef(false);

  if (autoplay) {
    const keepAutoPlay = !touchPoints[0] && !isAutoPlayPause.current;
  }

  return (
    <>
      {autoplay && (
        <Button
          aria-label="자동 재생 일지 정지"
          onClick={() => {
            // isAutoPlayPause는 렌더링에 영향을 미치지 않고 로직에만 영향을 주기 때문에 상태로 사용해서 불필요한 렌더링을 유발할 필요가 없다.
            isAutoPlayPause.current = true;
          }}
        />
      )}
    </>
  );
};
```

### 🔖 훅의 규칙

1. `최상위 레벨(top-level)에서 호출되어야 한다.`
   - 조건문, 반복문, 중첩함수, 클래스 등의 내부에서는 호출이 불가능하다.
   - 반환문(return) 앞에 있어야 한다.
2. `함수 컴포넌트나 커스텀 훅 등의 리액트 컴포넌트 내에서만 호출되어야 한다.`

# 📝 커스텀 훅

- 리액트 기본 훅 useState, useRef 등의 훅에 더해 컴포넌트 로직을 함수로 뽑아내 재사용 가능하게 만든 사용자 정의 훅을 `커스텀 훅`이라 말한다.

### 규칙

- 리액트 컴포넌트 내에서만 사용 가능하다.
- 이름은 반드시 `use`로 시작해야 한다.

## ✏️ 나만의 훅 만들기

```jsx
// hooks/useInput.jsx
import { useState, useCallback } from "react";

const useInput = (initialValue) => {
  const [val, setVal] = useState(initialValue); // 인자로 받은 값을 초기값으로 지정하여 관리
  const onChange = (e) => setVal(e.target.value);

  // input의 값 value와
  // 해당 값을 수정할 수 있는 함수 onChange를 반환하는 훅
  return { val, onChange };
};

export default useInput;
```

```jsx
// App.jsx
import useInput from "./hooks/useInput";

function App() {
  const { val, onChange } = useInput("");
  return (
    <>
      <h1>{val}</h1>
      <input onChange={onChange} value={val} />
    </>
  );
}

export default App;
```

## ✏️ TS로 커스텀 훅 강화하기

- JSX에서 TSX로 변환

  ```tsx
  // hooks/useInput.tsx
  import { useState, useCallback } from "react";

  // 🚨 ERROR : Parameter 'initialValue' implicitly has an 'any' type.
  const useInput = (initialValue) => {
    const [val, setVal] = useState(initialValue);

    // 🚨 ERROR : Parameter 'e' implicitly has an 'any' type.
    const onChange = (e) => setVal(e.target.value);
    return { val, onChange };
  };

  export default useInput;
  ```

  - 이벤트 객체 e의 타입은 유추하기 힘들기 때문에 IDE를 활용하여 TS 컴파일러(tsc)가 현재 사용하고 있는 이벤트 객체의 타입을 유추해서 알려주므로 유용하다.
  - `타입을 알고 싶은 부분 위에 마우스 커서를 가져다되면 타입을 유추할 수 있다.`

- 타입 정의 후 코드

  ```tsx
  import { ChangeEvent, useCallback, useState } from "react";

  const useInput = (initialValue: string) => {
    const [val, setVal] = useState(initialValue);
    const onChange = (e: ChangeEvent<HTMLInputElement>) => {
      setVal(e.target.value);
    };
    return { val, onChange };
  };

  export default useInput;
  ```
