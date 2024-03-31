**5장. 타입 활용하기** 내용에 대한 간단한 퀴즈입니다~!!

**⚠️ 주의: 풀어보기 전에 다른 사람 댓글 보지 않기!**

### 1. Infer를 사용하여 다음 타입을 만들어 보세요

#### 1-1. ElementType 타입
- 배열의 요소 타입을 추출하여 반환
- 배열이 아닌 타입이 들어오면 never 타입을 반환
```ts
type ElementType<T> = /* 구현할 곳 */;

const numArr = [1, 2, 3];
const strArr = ["a", "b", "c"];

type NumberElementType = ElementType<typeof numArr>; // 'number' 타입으로 추론되어야 함
type StringElementType = ElementType<typeof strArr>; // 'string' 타입으로 추론되어야 함
```

#### 1-2. ReturnType 타입
- 함수 타입의 리턴 타입을 추출하여 반환
- 함수 타입이 아닌 타입이 들어오면 any를 반환
```ts
type ReturnType<T extends (...args: any) => any> = /* 구현할 곳 */

function add(a: number, b: number): number {
  return a + b;
}
function formatPoint(point: number): string {
  return point.toFixed(1)
}

type Result1 = ReturnType<typeof add>; // 'number' 타입으로 추론되어야 함
type Result2 = ReturnType<typeof formatPoint>; // 'string' 타입으로 추론되어야 함
```

### 2. 유틸리티 타입 혹은 커스텀 유틸리티 타입의 예시를 2가지 이상 말해주세요
- 기왕이면 이미 아는 타입 말고 새로운 타입을 말해주세요!

### 3. 코드에 존재하는 오류를 찾아주세요
- 책 178p에 있는 코드입니다. 오류가 있길래 문제로 가져와봤습니다!
```ts
type ColorType = typeof keyof theme.colors; 
type BackgroundColorType = typeof keyof theme.backgroundColor; 
type FontSizeType = typeof keyof theme.fontSize; 

interface Props {
  color?: ColorType; 
  backgroundColor?: BackgroundColorType;
  fontSize?: FontSizeType;
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void | Promise<void>;
}
```
