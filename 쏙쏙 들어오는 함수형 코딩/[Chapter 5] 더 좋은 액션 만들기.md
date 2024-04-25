# 5. 더 좋은 액션 만들기

## 비즈니스 요구 사항과 설계를 맞추기

### 요구 사항에 맞춰 더 나은 추상화 단계 선택하기

- 기계적인 리팩터링이 항상 최선의 구조를 만드는 것은 아님
- 다음 코드에는 비즈니스 요구 사항으로 봤을 때 맞지 않는 부분이 있음
  ```tsx
  function gets_free_shipping(total, item_price) {
    return item_price + total >= 20;
  }
  ```
  - 요구 사항은 장바구니에 담긴 제품을 주문할 때 무료 배송인지 확인하는 것
  - 그러나 함수를 보면 장바구니로 무료 배송을 확인하지 않고 제품의 합계와 가격으로 확인하고 있음
  - 비즈니스 요구 사항과 다른 인자
  - 함수의 시그니처를 `gets_free_shipping(cart)`로 변경하는 리팩터링 필요

## 비즈니스 요구 사항과 함수를 맞추기

- 장바구니 값을 인자로 받아 합계가 20보다 크거나 같은지 알려주는 함수
- 별도의 계산 함수를 사용해 금액 합계를 구함
  ```tsx
  // BEFORE
  function gets_free_shipping(total, item_price) {
    return item_price + total >= 20;
  }

  // AFTER
  function gets_free_shipping(cart) {
    return calc_cart_total(cart) >= 20;
  }
  ```
  - 바꾼 함수는 합계와 제품 가격 대신 장바구니 데이터를 사용함
  - 장바구니는 전자상거래에서 많이 사용하는 엔티티 타입이기 때문에 비즈니스 요구 사항과 잘 맞음
- 단, 함수의 동작을 바꿨기 때문에 엄밀히 말하면 위 변경은 리팩터링이라고 할 수 없음

## 원칙: 암묵적 입력과 출력은 적을수록 좋습니다

- 인자가 아닌 모든 입력은 암묵적 입력, 리턴값이 아닌 모든 출력은 암묵적 출력임
  - 암묵적 입력과 출력이 없는 함수를 계산이라 부름
- 계산을 만들기 위해 암묵적 입력과 출력을 없애는 원칙은 액션에도 적용할 수 있음
  - 액션에서 모든 암묵적 입력과 출력을 없애지 않더라도 암묵적 입력과 출력을 줄이면 좋음
- 어떤 함수에 암묵적 입력과 출력이 있다면 다른 컴포넌트와 강하게 연결된 컴포넌트이며 다른 곳에서 사용할 수 없기 때문에 모듈이 아님
- 암묵적 입력과 출력을 명시적으로 바꿔 모듈화된 컴포넌트로 만들 수 있음

### 암묵적 입력과 출력이 있는 함수는 조심해서 사용해야 한다

- 앞 예제에 전역 변수를 변경하는 함수가 있었음. 다른 곳에서도 이 전역 변수를 사용할 수 있기 때문에, 원하는 값을 얻으려면 다른 코드가 실행되지 않는지 확인이 필요함
- 암묵적 출력으로 발생하는 일을 원할 때만 쓸 수도 있음 예를 들어 DOM을 바꾸는 암묵적 출력이 있는 함수를 쓸 때 DOM을 바꾸는 것만 필요 없는 경우, 결과는 필요하지만 다른 곳에 영향을 줄 수도 있게 됨
- 암묵적 입력과 출력이 있는 함수는 아무 때나 실행할 수 없기 때문에 테스트하기 어려움
  - 모든 입력값을 설정하고 테스트를 돌린 후에 모든 출력값을 확인해야 함
  - 입력과 출력이 많을 수록 테스트는 어려워짐
- 계산은 암묵적 입력과 출력이 없기 때문에 테스트하기 쉬움

## 암묵적 입력과 출력 줄이기

- 암묵적 입력은 인자라는 명시적 입력으로 변경할 수 있음
  ```tsx
  // BEFORE
  function update_shipping_icons() {
    var buy_buttons = get_buy_buttons_dom();
    for (var i = 0; i < buy_buttons.length; i++) {
      var button = buy_buttons[i];
      var item = button.item;
      var new_cart = add_item(shopping_cart, item.name, item.price);
      //  ^^^^^^^^^^^^^ 전역 변수 읽는 부분 (암묵적 입력)
      if (gets_free_shipping(new_cart)) {
        button.show_free_shipping_icon();
      } else {
        button.hide_free_shipping_icon();
      }
    }
  }

  // AFTER
  function update_shipping_icons(cart) {
    // ^^^^^ 전역 변수에 직접 접근하지 말고 인수로 전달
    var buy_buttons = get_buy_buttons_dom();
    for (var i = 0; i < buy_buttons.length; i++) {
      var button = buy_buttons[i];
      var item = button.item;
      var new_cart = add_item(cart, item.name, item.price);
      // ^^^^ 전역 변수가 아닌 인자에 접근
      if (gets_free_shipping(new_cart)) {
        button.show_free_shipping_icon();
      } else {
        button.hide_free_shipping_icon();
      }
    }
  }
  ```

## 원칙: 설계는 엉켜있는 코드를 푸는 것이다

- 함수를 사용하면 자연스레 관심사를 분리할 수 있음
- 함수는 인자로 넘기는 값과 그 값을 사용하는 방법을 분리
- 크고 복잡한 것이 더 잘 만들어진 것이 아니라, 언제든 쉽게 조합할 수 있는 분리된 것이 좋음
- 함수에 특별한 문제가 없어도 꺼낼 것이 있다면 분리하는 것이 좋음 → 더 좋은 설계

### 재사용하기 쉽다

- 함수는 작으면 작을 수록 재사용하기 쉬움
- 하는 일이 적고 함수를 호출할 때 가정을 많이 하지 않아도 됨

### 유지보수하기 쉽다

- 작은 함수는 쉽게 이해할 수 있고 유지보수하기 쉬움
- 코드가 작기 때문에 올바른지 아닌지 명확하게 이해 가능

### 테스트하기 쉬움

- 작은 함수는 테스트하기 좋음
- 한 가지 일만 하기 때문에 한 가지만 테스트하면 됨

## add_item()을 분리해 더 좋은 설계 만들기

```tsx
function add_item(cart, name, price) {
  var new_cart = cart.slice();
  new_cart.push({
    name,
    price,
  });
  return new_cart;
}
```

- 위 코드는 잘 동작하며, 장바구니에 제품을 추가하는 간단한 일만 하는 것 같지만 하는 일이 많은 함수임
  1. 배열 복사하기
  2. item 객체 만들기
  3. 복사본에 item 추가하기
  4. 복사본 리턴하기

### add_item() 함수는 cart와 item 구조를 모두 알고 있다

- item 구조만 알고 있는 함수와 cart 구조만 알고 있는 함수로 나눌 수 있음
  ```tsx
  function make_cart_item(name, price) {
    return { name, price };
  }

  function add_item(cart, item) {
    var new_cart = cart.slice();
    new_cart.push(item);
    return new_cart;
  }
  ```
  - cart와 item을 독립적으로 확장할 수 있음
    - ex. cart의 자료 구조 변경 용이

## 카피-온-라이트 패턴을 빼내기

- 위 add_item() 함수는 cart와 item에 특화된 함수가 아님
  - 일반적인 배열에 새로운 항목을 추가해 주는 역할을 함
  - 즉, 일반적인 배열과 항목에 쓸 수 있지만 이름은 일반적이지 않은 상황
    - 함수 이름과 동작 일치 필요

### 일반적인 이름으로 바꾼 코드

- 장바구니와 제품에만 쓸 수 있는 함수가 아닌 어떤 배열이나 항목에도 쓸 수 있는 이름으로 변경
  ```tsx
  function add_element_last(array, elem) {
    var new_array = array.slice();
    new_array.push(elem);
    return new_array;
  }

  function add_item(cart, item) {
    return add_element_last(cart, item);
  }
  ```
  - add_element_last() 함수는 재사용할 수 있는 유틸리티 함수가 됨

## 위 리팩터링과 관련된 질문

### 비즈니스 규칙과 장바구니 기능은 어떤 차이가 있는가? 전자상거래라면 장바구니에 관한 것이 곧 비즈니스 규칙이 아닌가?

- 장바구니는 대부분의 전자상거래 서비스에서 사용하는 일반적인 개념이고 동작하는 방식도 모두 비슷함
- 그러나 무료 배송 규칙은 특정 기업에서만 운영하는 특별한 규칙임

### 비즈니스 규칙과 장바구니에 대한 동작에 모두 속하는 함수도 있을 수 있나?

- 있을 수 있음
- 그러나 계층 관점에서 보면 code smell임
- 비즈니스 규칙에서 장바구니가 배열인지 알아야 한다면 문제가 될 수 있음
- 비즈니스 규칙은 장바구니 구조와 같은 하위 계층보다 빠르게 변경되기 때문
  - 설계를 진행하면서 분리해야 함
