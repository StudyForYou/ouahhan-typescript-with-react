<h1>타입스크립트와 객체 지향</h1>

# 📝 TS의 객체 지향

- JS로도 객체 지향 프로그래밍을 할 수 있지만 전통적인 객체 지향 프로그래밍 언어에서 기대할 수 있는 일부 기능을 지원하지 않아 객체 지향을 온전히 구현하는 데 부족함이 있다.
- TS는 이러한 제약을 해결할 수 있는 private과 같은 접근 제어자, 추상 클래스, 추상 메서드 등을 지원한다.
- 즉, TS는 객체 지향을 구현할 수 있도록 도와주는 JS의 슈퍼셋으로 볼 수 있다.

###

# 📝 우아한형제들의 활용 방식

## ✏️ 컴포넌트 영역

- 온전히 Layout만 담당한다.

```tsx
// components/CartCloseoutDialog.tsx

import { useCartStore } from "store/modules/cart";

const CartCloseoutDialog: React.VFC = () => {
  const cartStore = useCartStore();

  return (
    <Dialog
      opened={cartStore.PresentationTracker.isDialogOpen("closeout")}
      title="마감 세일이란?"
      onRequestClose={cartStore.PresentationTracker.closeDialog}
    >
      <div
        css={css`
          margin-top: 8px;
        `}
      >
        지점별 한정 수량으로 제공되는 할인 상품입니다. 재고 소진 시 가격이
        달라질 수 있습니다. 유통기한이 다소 짧으나 좋은 품질의 상품입니다.
      </div>
    </Dialog>
  );
};

export default CartCloseoutDialog;
```

- 비즈니스 로직은 useCartStore 내부에 존재할 것이다.

## ✏️ 커스텀 훅 영역

- 컴포넌트 영역 위에서 Layout과 비즈니스 로직을 연결해준다.

```tsx
// store/cart.ts

class CartStore {
  public async add(target: RecommendProduct): Promise<void> {
    const response = await addToCart(
      addToCartRequest({
        auths: this.requestInfo.AuthHeaders,
        cartProducts: this.productsTracker.PurchasableProducts,
        shopID: this.shopID,
        target,
      })
    );

    return response.fork(
      (error, _, statusCode) => {
        switch (statusCode) {
          case ResponseStatus.FAILURE:
            this.presentationTracker.pushToast(error);
            break;
          case ResponseStatus.CLIENT_ERROR:
            this.presentationTracker.pushToast(
              "네트워크가 연결되지 않았습니다."
            );
            break;
          default:
            this.presentationTracker.pushToast(
              "연결 상태가 일시적으로 불안정합니다."
            );
        }
      },
      (message) => this.applyAddedProduct(target, message)
    );
  }
}

const [CartStoreProvider, useCartStore] = setupContext<CartStore>("CartStore");
export { CartStore, CartStoreProvider, useCartStore };
```

```tsx
// serializers/cart/addToCartRequest.ts

import { AddToCartRequest } from "models/externals/Cart/Request";
import { IRequestHeader } from "models/externals/lib";
import {
  RecommendProduct,
  RecommendProductItem,
} from "models/internals/Cart/RecommendProduct";
import { Product } from "models/internals/Stuff/Product";

interface Params {
  auths: IRequestHeader;
  cartProducts: Product[];
  shopID: number;
  target: RecommendProduct;
}

function addToCartRequest({
  auths,
  cartProducts,
  shopID,
  target,
}: Params): AddToCartRequest {
  const productAlreadyInCart = cartProducts.find(
    (product) => product.getId() === target.getId()
  );

  return {
    body: {
      items: target.getItems().map((item) => ({
        itemId: item.id,
        quantity: getItemQuantityFor(productAlreadyInCart, item),
        salePrice: item.price,
      })),
      productId: target.getId(),
      shopId: shopID,
    },
    headers: auths,
  };
}

export { addToCartRequest };
```

## ✏️ 모델 영역

- 훅 영역 위에서 객체로서 상호 협력한다.

```tsx
// models/Cart.ts

export interface AddToCartRequest {
  body: {
    shopId: number;
    items: { itemId: number; quantity: number; salePrice: number }[];
    productId: number;
  };
  headers: IRequestHeader;
}

export class RecommendProduct {
  public getId(): number {
    return this.id;
  }

  public getName(): string {
    return this.name;
  }

  public getThumbnail(): string {
    return this.thumbnailImageUrl;
  }

  public getPrice(): RecommendProductPrice {
    return this.price;
  }

  public getCalculatedPrice(): number {
    const price = this.getPrice();
    return price.sale?.price ?? price.origin;
  }

  public getItems(): RecommendProductItem[] {
    return this.items;
  }

  public getType(): string {
    return this.type;
  }

  public getRef(): string {
    return this.ref;
  }

  constructor(init: any) {
    this.id = init.id;
    this.name = init.displayName;
    this.thumbnailImageUrl = init.thumbnailImageUrl;
    this.price = {
      sale: init.displayDiscounted
        ? {
            price: Math.floor(init.salePrice),
            percent: init.discountPercent,
          }
        : null,
      origin: Math.floor(init.retailPrice),
    };
    this.type = init.saleUnit;
    this.items = init.items.map((item) => {
      return {
        id: item.id,
        minQuantity: item.minCount,
        price: Math.floor(item.salePrice),
      };
    });
    this.ref = init.productRef;
  }

  private id: number;
  private name: string;
  private thumbnailImageUrl: string;
  private price: RecommendProductPrice;
  private items: RecommendProductItem[];
  private type: string;
  private ref: string;
}
```

## ✏️ API 레이어 영역

- 모델 영역 위에서 API를 해석하여 모델로 전달한다.

```tsx
// apis/Cart.ts

// APIResponse는 데이터 로드에 성공한 상태와 실패한 상태의 반환 값을 제네릭하게 표현해주는 API 응답 객체이다
// (APIResponse<OK, Error>)
interface APIResponse<OK, Error> {
  // API 응답에 성공한 경우의 데이터 형식
  ok: OK;
  // API 응답에 실패한 경우의 에러 형식
  error: Error;
}

export const addToCart = async (
  param: AddToCartRequest
): Promise<APIResponse<string, string>> => {
  return (await GatewayAPI.post<IAddCartResponse>("/v3/cart", param)).map(
    (data) => data.message
  );
};
```

# 📝 캡슐화와 추상화

- 프로젝트 설계의 궁극적인 목표는 객체들이 유기적으로 협력하게끔 만들어서 적절하게 도메인 분리를 하는 것이다.
- 이를 위해 `캡슐화`는 중요한 도구가 될 수 있다. `추상화`는 객체들을 모델링하는 과정 자체라고 할 수 있는데, 이 객체들을 좀 더 사람이 인지할 수 있도록 적합한 설계를 하는 것이 곧 추상화다.

### 캡슐화

- 캡슐화란 다른 객체 내부의 데이터를 꺼내와서 직접 다루지 않고, 해당 객체에서 처리할 행위를 따로 요청함으로써 협력하는 것이다.
- 앞서 컴포넌트 역시 객체라고 했다. 그렇다면 컴포넌트의 내부 데이터인 상태(state)가 바로 캡슐화의 대상이 될 수 있다. 결국 컴포넌트 내의 상태와 prop을 잘 다루는 것도 캡슐화의 개념에 부합하는 것이다.

# 📝 정리

- 객체 지향 패러다임의 원론적인 부분에 집중하기 보다는 실제 코드를 작성하며 객체 지향을 경험해보는 것이 중요하다. 우리는 이미 객체 지향을 구현하고 있다는 것을 명심하자.
- 생산성 측면에선 단지 코드를 빨리 작성하는게 도움이 될수도 있다. 객체지향은 유지보수를 위한 개념이므로, 유지보수가 필요하지 않다면 꼭 따를 필요가 없을지도 모른다. 어디에나 반드시 객체 지향이 적용되어야 하는 것은 아니자만 객체 지향 관점으로 개발하는 것은 중요하다. 좋은 추상화를 어떻게 만들지 고민하며 개발한다면 변경에 유연하고 유지보수하기 쉬운 설계를 구축할 수 있을 것이다
