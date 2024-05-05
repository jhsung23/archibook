# 6. 변경 가능한 데이터 구조를 가진 언어에서 불변성 유지하기 (1)

## 동작을 읽기, 쓰기 또는 둘 다로 분류하기

- 동작을 읽기 또는 쓰기 또는 둘 다 하는 것으로 분류할 수 있음
- 읽기 동작은 데이터를 바꾸지 않고 정보를 꺼내는 것
  - 데이터가 바뀌지 않기 때문에 다루기 쉬움
  - 인자에만 의존해 정보를 가져오는 읽기 동작이라면 계산임
- 쓰기 동작은 어떻게든 데이터를 바꾸는 것
  - 바뀌는 값은 어디서 사용될지 모르기 때문에 바뀌지 않도록 원칙이 필요함

### 장바구니 동작

1. 제품 개수 가져오기 → 읽기
2. 제품 이름으로 제품 가져오기 → 읽기
3. 제품 추가하기 → 쓰기
4. 제품 이름으로 제품 빼기 → 쓰기
5. 제품 이름으로 제품 구매 수량 바꾸기 → 쓰기

- 쓰기 동작은 불변성 원칙에 따라 구현해야 함
- 불변성 원칙을 카피-온-라이트(copy on write)라고 함
- 자바스크립트는 변경 가능한 데이터 구조를 사용하기 때문에 불변성 원칙을 적용하려면 직접 구현해야 함

## 카피-온-라이트 원칙 세 단계

1. 복사본 만들기
2. 복사본 변경하기
3. 복사본 리턴하기

```tsx
function add_element_last(array, elem) {
  // array를 변경하려고 함
  var new_array = array.slice(); // 1. 복사본 만들기
  new_array.push(elem); // 2. 복사본 바꾸기
  return new_array; // 3. 복사본 리턴하기
}
```

- 위 함수는 어떻게 동작하는가? 어떻게 기존 배열을 변경하지 않았는가?
  1. 배열을 복사했고 기존 배열은 변경하지 않음
  2. 복사본은 함수 범위에 있기 때문에 다른 코드에서 값을 바꾸기 위해 접근할 수 없음
  3. 복사본을 변경하고 나서 함수를 리턴함. 이후에는 값을 바꿀 수 없음
- add_element_last() 함수는 데이터를 바꾸지 않았고 정보를 리턴했기 때문에 읽기임
- 카피-온-라이트는 쓰기를 읽기로 바꿈

## 카피-온-라이트로 쓰기를 읽기로 바꾸기

```tsx
function remove_item_by_name(cart, name) {
  var idx = null;
  for (var i = 0; i < cart.length; i++) {
    if (cart[i].name === name) {
      idx = i;
    }
  }
  if (idx !== null) {
    cart.splice(idx, 1);
  }
}
```

- cart.splice() 메서드는 배열에서 항목을 삭제하는 메서드임
- 위 함수는 cart.splice()를 통해서 장바구니를 변경함
- 만약 remove_item_by_name() 함수에 전역변수 shopping_cart를 넘기면 전역변수인 장바구니가 변경됨

### remove_item_by_name()에 카피-온-라이트 적용하기

1. 장바구니 복사하기

   ```tsx
   function remove_item_by_name(cart, name) {
     var new_cart = cart.slice(); // cart를 복사하여 지역변수에 저장
     var idx = null;
     for (var i = 0; i < cart.length; i++) {
       if (cart[i].name === name) {
         idx = i;
       }
     }
     if (idx !== null) {
       cart.splice(idx, 1);
     }
   }
   ```

2. 복사본 변경하기

   ```tsx
   function remove_item_by_name(cart, name) {
     var new_cart = cart.slice();
     var idx = null;
     for (var i = 0; i < new_cart.length; i++) {
       if (new_cart[i].name === name) {
         idx = i;
       }
     }
     if (idx !== null) {
       new_cart.splice(idx, 1); // cart의 복사본을 수정
     }
   }
   ```

3. 복사본 리턴하기

   ```tsx
   function remove_item_by_name(cart, name) {
     var new_cart = cart.slice();
     var idx = null;
     for (var i = 0; i < new_cart.length; i++) {
       if (new_cart[i].name === name) {
         idx = i;
       }
     }
     if (idx !== null) {
       new_cart.splice(idx, 1);
     }
     return new_cart; // 복사본을 수정하여 리턴
   }
   ```

## 앞에서 만든 카피-온-라이트 동작은 일반적입니다

### .splice() 메서드를 일반화

```tsx
function removeItems(array, idx, count) {
  var copy = array.slice();
  copy.slice(idx, count);
  return copy;
}
```

### remove_item_by_name() 함수에서 removeItems() 호출

```tsx
function remove_item_by_name(cart, name) {
  var idx = null;
  for (var i = 0; i < cart.length; i++) {
    if (cart[i].name === name) {
      idx = i;
    }
  }
  if (idx !== null) {
    return removeItems(cart, idx, 1); // removeItems() 함수가 배열을 복사하기 때문에 본 함수에서 복사본을 만들 필요 없음
  }
  return cart; // 변경이 없다면 복사본을 만들지 않아도 됨
}
```

## 쓰기를 하면서 읽기도 하는 동작은 어떻게 해야 할까요?

- 어떤 동작은 읽고 변경하는 일을 동시에 함
  - 이런 동작은 값을 변경하고 리턴함
- shift() 메서드의 경우 값을 바꾸는 동시에 배열에 첫 번째 항목을 리턴함
  - 변경하면서 읽는 동작임
- 위 동작을 카피-온-라이트로 변경하기 위한 두 가지 접근 방법
  1. 읽기와 쓰기 함수로 각각 분리한다
  2. 함수에서 값을 두 개 리턴한다
  - 선택할 수 있다면 책임이 확실히 분리되는 1번 방법이 더 좋은 방법임

## 쓰면서 읽기도 하는 함수를 분리하기

### 읽기와 쓰기 동작으로 분리하기

- shift() 메서드의 읽기 동작은 값을 단순히 리턴하는 동작
- 배열에 첫 번째 항목을 리턴함
- 즉, 배열에 첫 번째 항목을 리턴하는 계산 함수를 만들면 됨
  - 읽기 동작만 할 뿐, 아무것도 바꾸지 않음
  - 숨겨진 입력이나 출력이 없기 때문에 이 함수는 계산임
  ```tsx
  function first_element(array) {
    return array[0];
  }
  ```
  - first_element() 함수는 배열을 바꾸지 않는 읽기 함수이기 때문에 카피-온-라이트를 적용할 필요 없음
- shift() 메서드의 쓰기 동작은 새로 만들 필요 없음
  - 단 리턴 값은 사용하지 않음
  ```tsx
  function drop_first(array) {
    array.shift();
  }
  ```

### 쓰기 동작을 카피-온-라이트로 바꾸기

- drop_first() 함수는 인자로 들어온 값을 변경하는 쓰기임
- 값을 변경하는 대신 카피-온-라이트 적용
  ```tsx
  function drop_first(array) {
    var array_copy = array.slice();
    array_copy.shift();
    return array_copy;
  }
  ```
- 읽기와 쓰기를 분리하는 접근 방법은 분리된 함수를 따로 쓸 수 없기 때문에 더 좋은 접근 방법임

## 값을 두 개 리턴하는 함수로 만들기

### 동작을 감싸기

- shift() 메서드를 바꿀 수 있도록 새로운 함수로 감쌈
  ```tsx
  function shift(array) {
    return array.shift();
  }
  ```

### 읽으면서 쓰기도 하는 함수를 읽기 함수로 바꾸기

- 인자를 복사한 후 복사한 값의 첫 번째 항목을 지우고, 지운 첫 번째 항목과 변경된 배열을 함께 리턴함
  ```tsx
  function shift(array) {
    var array_copy = array.slice();
    var first = array_copy.shift();
    return {
      // 값 2개를 리턴하기 위해 객체 사용
      first: first,
      array: array_copy,
    };
  }
  ```

### 다른 방법

- 첫 번째 접근 방식을 사용해 두 값을 객체로 조합하는 방법
  ```tsx
  function shift(array) {
    return {
      first: first_element(array),
      array: drop_first(array),
    };
  }
  ```
- 첫 번째 접근 방법으로 만든 두 함수는 모두 계산이기 때문에 쉽게 조합할 수 있음
  - 조합해도 이 함수는 계산임
