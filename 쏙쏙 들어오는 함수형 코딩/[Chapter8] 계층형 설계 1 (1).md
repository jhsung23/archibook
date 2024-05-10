# 8. 계층형 설계 1 (1)

## 소프트웨어 설계란 무엇입니까?

- 코드를 만들고, 테스트하고, 유지보수하기 쉬운 프로그래밍 방법을 선택하기 위해 미적 감각을 사용하는 것

## 계층형 설계란 무엇인가요?

- 계층형 설계는 소프트웨어를 계층으로 구성하는 기술
- 각 계층에 있는 함수는 바로 아래 계층에 있는 함수를 이용해 정의함
- 각 계층은 예시로 다음과 같이 구분할 수 있음
  - 비즈니스 규칙 - gets_free_shipping(), cartTax()
  - 장바구니를 위한 동작들 - remove_item_by_name(), calc_total(), add_item(), setPriceByName()
  - 카피-온-라이트 - removeItems(), add_element_last()
  - 언어에서 지원하는 배열 관련 기능 - slice()
- 각 계층을 정확히 구분하기는 어려움
- 잘 구분하기 위해서 구분하기 위한 다양한 변수를 찾고 어떻게 해야 하는지 알아야 함

## 설계

### 전문가의 저주

- 본인의 전문 분야를 잘 알지만, 설명은 잘 하지 못하는 것
- 설명하지 못하는 이유는 복잡하기 때문임

### 계층형 설계 감각을 키우기 위한 입력

- 함수 본문
- 계층 구조
- 함수 시그니처

### 계층형 설계 감각을 키우기 위한 출력

- 조직화
- 구현
- 변경

## 계층형 설계 패턴

### 패턴 1: 직접 구현

- 직접 구현된 함수를 읽을 때, 함수 시그니처가 나타내는 문제를 함수 본문에서 적절한 구체화 수준에서 해결해야 함
- 너무 구체적이라면 코드에서 나는 냄새

### 패턴 2: 추상화 벽

- 호출 그래프에 어떤 계층은 중요한 세부 구현을 감추고 인터페이스를 제공함
- 인터페이스를 사용하여 코드를 만들면 높은 차원으로 생각할 수 있음
- 고수준의 추상화 단계만 생각하면 됨

### 패턴 3: 작은 인터페이스

- 시스템이 커질수록 비즈니스 개념을 나타내는 중요한 인터페이스는 작고 강력한 동작으로 구성하는 것이 좋음
- 다른 동작도 직간접적으로 최소한의 인터페이스를 유지하면서 정의해야 함

### 패턴 4: 편리한 계층

- 계층형 설계 패턴과 실천 방법은 개발자의 요구를 만족시키면서 비즈니스 문제를 잘 풀 수 있어야 함
- 그냥 계층을 추가하면 안 되고, 코드와 그 코드가 속한 추상화 계층은 작업할 때 편리해야 함

## 패턴 1: 직접 구현

- 아래는 넥타이 하나를 사면 무료로 넥타이 클립을 하나 더 주는 코드

```jsx
function freeTieClip(cart) {
  var hasTie = false;
  var hasTieClip = false;
  for (var i = 0; i < cart.length; i++) {
    var item = cart[i];
    if (item.name === 'tie') {
      hasTie = true;
    }
    if (item.name === 'tie clip') {
      hasTieClip = true;
    }
  }
  if (hasTie && !hasTieClip) {
    var tieClip = make_item('tie clip', 0);
    return add_item(cart, tieClip);
  }
  return cart;
}
```

- 위 코드는 많은 기능이 있음
  - 장바구니를 돌면서 항목을 체크하고 무엇인가를 결정하고 있음
  - 어떤 설계 원칙을 가지고 설계한 코드가 아님
- 계층형 설계 패턴 중 직접 구현을 따르지 않은 코드
- freeTieClip()는 알아야 할 필요가 없는 구체적인 내용을 담고 있음
  - 예를 들어 마케팅과 관련된 함수가 장바구니는 배열이라는 사실(for문을 돌고 있기 때문에)을 알아야할 필요가 없음

### 제품이 있는지 확인하는 함수가 있다면 설계를 개선할 수 있습니다

- 장바구니 안에 제품이 있는지 확인하는 함수가 있다면, 저수준의 반복문을 직접 쓰지 않았을 것임
- 저수준의 코드는 추출해야 할 가능성이 높음

```jsx
function freeTieClip(cart) {
  var hasTie = isInCart(cart, 'tie');
  var hasTieClip = isInCart(cart, 'tie clip');

  // 생략 ...
}

function isInCart(cart, name) {
  for (var i = 0; i < cart.length; i++) {
    if (cart[i].name === name) {
      return true;
    }
  }
  return false;
}
```

### 호출 그래프를 만들어 함수 호출을 시각화하기

- 함수에서 사용하는 다른 함수와 언어 기능을 호출 그래프로 그릴 수 있음

```markdown
freeTieClip()
├── array index // 언어 기능
├── for loop // 언어 기능
├── make_item() // 함수 호출
└── add_item() // 함수 호출
```

- 직접 만든 함수와 언어 기능은 추상화 수준이 다름
- 반복문과 배열 인덱스를 참조하는 기능은 더 낮은 추상화 단계임
  ```markdown
  freeTieClip()
  ├── make_item() // 함수 호출
  ├── add_item() // 함수 호출
  ├────── array index // 언어 기능
  └────── for loop // 언어 기능
  ```
- 한 함수에서 서로 다른 추상화 단계를 사용하면 코드가 명확하지 않아 읽기 어려움

### 직접 구현 패턴을 사용하면 비슷한 추상화 계층에 있는 함수를 호출합니다

- 서로 다른 추상화 단계에 있는 기능을 사용하면 직접 구현 패턴이 아님
- 개선된 코드의 호출 그래프
  ```markdown
  freeTieClip()
  ├── isInCart() // 함수 호출
  ├── make_item() // 함수 호출
  └── add_item() // 함수 호출
  ```
- freeTieClip()이 사용하는 모든 함수는 장바구니가 배열인지 몰라도 됨
  - 이는 함수가 모두 비슷한 계층에 있다는 것을 의미함
- 함수가 모두 비슷한 계층에 있다면 직접 구현했다고 할 수 있음

### remove_item_by_name() 함수 그래프 그려보기

```jsx
function remove_item_by_name(cart, name) {
  var idx = null;
  for (var i = 0; i < cart.length; i++) {
    if (cart[i].name === name) {
      idx = i;
    }
  }
  if (idx !== null) {
    return removeItems(cart, idx, 1);
  }
  return cart;
}
```

```
remove_item_by_name()
├── array index
├── for loop
└── removeItems()
```

- freeTieClip() 함수와 비교했을 때 remove_item_by_name() 함수는 isInCart() 함수가 속해있는 계층과 유사함
  - isInCart()와 마찬가지로 for loop와 array index를 가리키기 때문
  - 같은 계층을 가리키는 것은 같은 계층에 있어도 좋다는 정보임

### 같은 계층에 있는 함수는 같은 목적을 가져야 합니다

- 계층의 목적은 각 계층에 있는 함수의 목적과 같음
- 각 계층은 추상화 수준이 다르기 때문에 어떤 계층에 있는 함수를 읽거나 고칠 때 낮은 수준의 구체적인 내용은 신경 쓰지 않아도 됨
  - 예를 들어 장바구니 비즈니스 규칙 계층에 있는 함수를 쓸 때, 장바구니가 배열로 구현되어 있다는 것과 같은 구체적인 내용은 신경 쓰지 않아도 됨
- 다이어그램은 함수가 호출하는 것을 있는 그대로 표현한 것이기 때문에 함수를 어떤 계층에 놓을지 바로 알 수 있음
  - 다이어그램은 코드를 높은 차원에서 볼 수 있는 좋은 도구임
