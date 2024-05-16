# 10. 일급 함수 1 (1)

## 코드의 냄새: 함수 이름에 있는 암묵적 인자

- 다음의 경우는 코드 냄새임
  - 문자열만 다르고 동일한 함수
  - 문자열이 함수 이름에 그대로 들어있는 함수
- 즉, 필드를 결정하는 문자열이 함수 이름에 있는 것
- 함수 이름에 있는 일부가 인자처럼 동작함
- 이 냄새를 함수 이름에 있는 암묵적 인자라고 함
- 값을 명시적으로 전달하지 않고 함수 이름의 일부로 전달함

## 리팩터링: 암묵적 인자를 드러내기

- 암묵적 인자를 명시적인 인자로 바꾸는 것
- 리팩터링 단계
  1. 함수 이름에 있는 암묵적 인자 확인
  2. 명시적인 인자를 추가
  3. 함수 본문에 하드 코딩된 값을 새로운 인자로 변경
  4. 함수를 부르는 곳을 고침

```jsx
// 리팩터링 전
function setPriceByName(cart, name, price) {
  //        ^^^^^ 함수 이름에 있는 price가 암묵적 인자임
  var item = cart[name];
  var newItem = objectSet(item, 'price', price);
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}

cart = setPriceByName(cart, 'shoe', 13);
cart = setQuantityByName(cart, 'shoe', 3);
cart = setShippingByName(cart, 'shoe', 0);
cart = setTaxByName(cart, 'shoe', 2.34);

// 리팩터링 후
function setFieldByName(cart, name, field, value) {
  //                                ^^^^^  ^^^^^ 명시적인 인자를 추가하고 원래 인자는 더 일반적인 이름으로 변경
  var item = cart[name];
  var newItem = objectSet(item, field, value);
  //                            ^^^^^  ^^^^^ 새로운 인자 사용
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}

cart = setFieldByName(cart, 'shoe', 'price', 13);
cart = setFieldByName(cart, 'shoe', 'quantity', 3);
cart = setFieldByName(cart, 'shoe', 'shipping', 0);
cart = setFieldByName(cart, 'shoe', 'tax', 2.34);
```

- 리팩터링 전에는 필드명이 함수 이름에 암묵적으로 있었지만, 리팩터링 후 암묵적인 이름은 인자로 넘길 수 있는 값이 됨
- 값은 변수나 배열에 담을 수 있어 일급이라 부르며, 일급 값은 언어 전체에 어디서나 쓸 수 있음

## 일급인 것과 일급이 아닌 것을 구별하기

### 자바스크립트에는 일급이 아닌 것과 일급인 것이 섞여 있습니다.

다른 언어를 사용해도 마찬가지입니다.

- 자바스크립트에서 수식 연산자, 반복문, 조건문, try/catch 블록은 일급이 아님
- 일급이 아닌 것을 일급으로 바꾸는 방법을 아는 것이 중요함
- 함수명 일부를 값처럼 쓸 수 없음
  - 함수명은 일급이 아니기 때문
  - 그래서 함수명의 일부를 인자로 바꿔 일급으로 만들 수 있음
- 일급으로 바꾸는 기술은 함수형 프로그래밍에서 중요함

## 필드명을 문자열로 사용하면 버그가 생기지 않을까요?

- 문자열에 오타가 있으면 버그로 이어질 수 있음
- 이 문제를 해결하기 위해 컴파일 타임에 검사하거나 런타임에 검사하는 방법이 있음
- 컴파일 타임에 검사하는 방법은 정적 타입 시스템에서 사용하는 방법임
  - 자바스크립트는 정적 타입 시스템 언어가 아니지만 타입스크립트로 문자열이 사용할 수 있는 필드인지 확인할 수 있음
- 런타임 검사는 컴파일 타임에 동작하지 않고 함수를 실행할 때마다 동작함
  - 필드명이 올바른지 확인하는 코드를 작성해서 검사할 수 있음
  ```jsx
  if (!validItemFields.includes(field)) {
    throw 'Not a valid item field: ' + field;
  }
  ```

## 일급 필드를 사용하면 API를 바꾸기 더 어렵나요?

- 엔티티 필드명을 일급으로 만들어 사용하는 것이 세부 구현을 밖으로 노출하는 것이 아닐까?
- 장바구니 제품은 어떤 필드명과 함께 추상화 벽 아래에서 정의한 것인데, 추상화 벽 위에 있는 사람들에게 전달되는 것은 추상화 원칙을 위반하는 것이 아닐까?
- API 문서에 필드명을 명시하면 영원히 필드명을 바꾸지 못하는 것 아닐까?
- 필드명은 계속 유지해야 하지만 구현이 외부에 노출된 것은 아님
- 만약 내부에서 정의한 필드명이 바뀐다 해도 사용하는 사람들이 원래 필드명을 그대로 사용하게 할 수 있음

```jsx
var validItemFields = ['price', 'quantity', 'shipping', 'tax', 'number'];
var translations = { quantity: 'number' };

function setFieldByName(cart, name, field, value) {
  if (!validItemFields.includes(field)) {
    throw 'Not a valid item field: ' + field;
  }
  if (translations.hasOwnProperty(field)) {
    // 원래 필드명을 새로운 필드명으로 단순히 바꿔줌
    field = translations[filed];
  }
  var item = cart[name];
  var newItem = objectSet(item, field, value);
  var newCart = objectSet(cart, name, newItem);
  return newCart;
}
```

## 객체와 배열을 너무 많이 쓰게 됩니다

- 자바스크립트를 사용한다면 전보다 객체를 더 많이 쓴다고 느낄 수 있음
- 중요한 것은 데이터를 사용할 때 임의의 인터페이스로 감싸지 않고 그대로 사용한다는 점임
- 인터페이스를 잘 만들면 데이터를 정해진 방법으로만 쓸 수 있음
- 데이터를 데이터 그대로 사용하는 것의 중요한 장점은 여러 가지 방법으로 해석할 수 있다는 점임
- 데이터가 미래에 어떤 방법으로 해석될지 미리 알 수 없기 때문에 필요할 때 알맞은 방법으로 해석할 수 있어야 함
- 이는 데이터 지향이라는 중요한 원칙
  - 데이터 지향: 이벤트와 엔티티에 대한 사실을 표현하기 위해 일반 데이터 구조를 사용하는 프로그래밍 형식

## 어떤 문법이든 일급 함수로 바꿀 수 있습니다

- 자바스크립트는 일급이 아닌 것이 많음
- - 연산자는 변수에 할당할 수 없지만 즉, 일급이 아니지만 + 연산자와 같은 함수를 만들 수 있음

    ```jsx
    function plus(a, b) {
      return a + b;
    }
    ```
- 자바스크립트에서 함수는 일급 값임
- 위 코드는 + 연산자를 일급 값으로 만든 것이며 단순히 + 연산을 하는 함수기 때문에 쓸데없어 보일 수 있으나, 일급으로 만들면 강력한 힘이 생김
