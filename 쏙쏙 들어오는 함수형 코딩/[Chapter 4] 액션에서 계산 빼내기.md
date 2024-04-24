# 4. 액션에서 계산 빼내기

## 테스트하기 쉽게 만들기

```tsx
function update_tax_dom() {
  set_tax_dom(shopping_cart_total * 0.1); // 장바구니 금액 합계에 10%를 곱한 후 dom 업데이트
}
```

- 비즈니스 규칙인 total \* 0.1을 테스트하려면,
  - 전역 변숫값(shopping_cart_total)을 설정해야 함
  - set_tax_dom을 호출했기 때문에 DOM에서 값을 가져와야 함
- 따라서 테스트를 더 쉽게 하려면
  - DOM 업데이트와 비즈니스 규칙이 분리돼야 함
  - 전역 변수가 없어야 함

## 함수에는 입력과 출력이 있습니다

- 모든 함수는 입력과 출력이 있음
  - 입력: 함수가 계산을 하기 위한 외부 정보
  - 출력: 함수 밖으로 나오는 정보나 어떤 동작
- 함수를 부르는 이유는 결과가 필요하기 때문이며 결과를 얻으려면 입력이 필요함

### 입력과 출력은 명시적이거나 암묵적일 수 있습니다

- 인자는 명시적인 입력이고 리턴값은 명시적인 출력임
- 그러나 암묵적으로 함수로 들어가거나 나오는 정보도 있음

```tsx
var total = 0;
function add_to_total(amount) {
  // 인자는 명시적 입력
  console.log('Old total: ' + total); // 전역 변수를 읽는 것은 암묵적 입력
  total += amount; // 전역 변수를 바꾸는 것은 암묵적 출력
  return total; // 리턴값은 명시적 출력
}
```

### 함수에 암묵적 입력과 출력이 있으면 액션이 됩니다

- 함수에서 암묵적 입력과 출력을 없애면 계산이 됨
- 암묵적 입력은 함수의 인자로 바꾸고, 암묵적 출력은 함수의 리턴값으로 바꾸면 됨

## 액션에서 계산 빼내기

1. 계산에 해당하는 코드를 분리
2. 입력값은 인자로 출력값은 리턴값으로 변경

### 서브루틴 추출하기

- BEFORE
  ```tsx
  function calc_cart_total() {
    shopping_cart_total = 0;
    for (var i = 0; i < shopping_cart.length; i++) {
      var item = shopping_cart[i];
      shopping_cart_total += item.price;
    }

    set_cart_total_dom();
    update_shipping_icons();
    update_tax_dom();
  }
  ```
- AFTER
  ```tsx
  function calc_cart_total() {
    calc_total(); // 새로 만든 함수 호출

    set_cart_total_dom();
    update_shipping_icons();
    update_tax_dom();
  }

  function calc_total() {
    shopping_cart_total = 0;
    for (var i = 0; i < shopping_cart.length; i++) {
      var item = shopping_cart[i];
      shopping_cart_total += item.price;
    }
  }
  ```
- 새로 만든 함수는 아직 액션이기 때문에 계산으로 바꿔야 함
- 계산으로 바꾸려면 어떤 입력과 출력이 있는지 확인해야 함
  ```tsx
  function calc_total() {
    shopping_cart_total = 0; // 전역 변숫값을 바꾸는 암묵적 출력
    for (var i = 0; i < shopping_cart.length; i++) {
      // 전역 변숫값을 읽는 암묵적 입력
      var item = shopping_cart[i];
      shopping_cart_total += item.price; // 전역 변숫값을 바꾸는 암묵적 출력
    }
  }
  ```
- 전역 변수 대신 지역 변수를 사용하고 지역 변숫값을 리턴
  ```tsx
  function calc_cart_total() {
    shopping_cart_total = calc_total();

    set_cart_total_dom();
    update_shipping_icons();
    update_tax_dom();
  }

  function calc_total(cart) {
    // 명시적 입력
    var total = 0; // 전역 변수 대신 지역 변수 사용
    for (var i = 0; i < cart.length; i++) {
      var item = cart[i];
      total += item.price;
    }
    return total; // 명시적 출력
  }
  ```

## 액션에서 또 다른 계산 빼내기

### 함수 추출하기

- BEFORE
  ```tsx
  function add_item_to_cart(name, price) {
    shopping_cart.push({
      name,
      price,
    });
    calc_cart_total();
  }
  ```
- AFTER
  ```tsx
  function add_item_to_cart(name, price) {
    add_item(name, price); // 새로 만든 함수 호출

    calc_cart_total();
  }

  function add_item(name, price) {
    shopping_cart.push({
      name,
      price,
    });
  }
  ```
- 새로 만든 함수는 전역 변수인 shopping_cart를 변경하기 때문에 아직 액션임
- 전역 변수 대신 지역 변수를 읽고 지역 변수를 변경
  ```tsx
  function add_item(cart, name, price) {
    cart.push({
      name,
      price,
    });
  }
  ```
- 전역 배열을 변경하는 암묵적 출력을 명시적 출력으로 변경 → 복사본을 만들고 복사본에 추가해 리턴해야 함(카피-온-라이트)
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

## 함수형 코드 관련 궁금증

- 코드가 더 많아진 것 같은데 잘한 것인지? 코드가 적어야 더 좋은 것 아닌가?
  - 일반적으로 더 적은 코드가 좋은 것은 맞지만, 새로운 함수를 만들면 시간이 지나면서 함수로 분리한 것에 장점을 얻을 수 있음
  - 코드를 테스트하기 쉬워졌고 재사용하기 좋아짐
  - 테스트 코드는 더 짧아짐
- 재사용성을 높이고 쉽게 테스트할 수 있는 것이 함수형 프로그래밍의 전부인가?
  - 동시성이나 설계, 데이터 모델링 측면에서 좋은 점들이 있음
- 다른 곳에서 쓰지 않더라도 계산으로 분리하는 것이 중요한가?
  - 물론임
  - 함수형 프로그래밍의 목적 중에 어떤 것을 분리해서 더 작게 만들려고 하는 것도 있음
  - 작은 것은 테스트하기 쉽고 재사용하기 쉽고 이해하기 쉽기 때문
- 계산으로 바꾼 함수 안에서 아직도 변수를 변경(new_cart.push())하고 있다. 함수형 프로그래밍에서는 모든 것이 불변값이어야 한다는데 뭐가 맞냐?
  - 불변값은 생성된 다음에 바뀌면 안 되는 값임
  - 생성할 때는 초기화가 필요함
    - 초깃값이 있어야 하는 배열이 필요하고 이후에 바꾸지 않는다고 해도 초깃값을 넣기 위해 시작 부분에 값을 배열에 넣어줘야 함
  - 지역 변수를 변경하는 곳은 나중에 초기화할 값으로 새로 생성하는 것이며 지역 변수이기 때문에 함수 밖에서 접근할 수 없음

## 계산 추출을 단계별로 알아보기

1. 계산 코드를 찾아 빼낸다 - 원래 코드에서 빼낸 부분에 새 함수를 호출하도록 변경한다
2. 새 함수에 암묵적 입력과 출력을 찾는다
3. 암묵적 입력은 인자로 암묵적 출력은 리턴값으로 바꾼다 - 인자와 리턴값은 바뀌지 않는 불변값이어야 한다
