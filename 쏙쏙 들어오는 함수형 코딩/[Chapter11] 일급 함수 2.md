# 11. 일급 함수 2

## 코드 냄새 하나와 리팩터링 두 개

### 코드의 냄새: 함수 이름에 있는 암묵적 인자

- 일급 값으로 바꾸면 표현력이 좋아짐
- 함수 본문에서 사용하는 어떤 값이 함수 이름에 나타난다면 함수 이름에 나타난다면 코드의 냄새임
- 특징
  - 거의 똑같이 구현된 함수가 있음
  - 함수 이름이 구현에 있는 다른 부분을 가리킴

### 리팩터링: 암묵적 인자를 드러내기

- 암묵적 인자가 일급 값이 되도록 함수에 인자를 추가한느 리팩터링
- 잠재적 중복을 없애고 코드의 목적을 더 잘 표현할 수 있음
- 단계
  1. 함수 이름에 있는 암묵적 인자를 확인
  2. 명시적인 인자를 추가
  3. 함수 본문에 하드 코딩된 값을 새로운 인자로 바꿈
  4. 함수를 호출하는 곳을 고침

### 리팩터링: 함수 본문을 콜백으로 바꾸기

- 함수 본문에 어떤 부분을 콜백으로 바꾸어 일급 함수로 어떤 함수에 동작을 전달할 수 있음
- 원래 있던 코드를 고차 함수로 만드는 방법
- 단계
  1. 본문에서 바꿀 부분의 앞부분과 뒷부분 확인
  2. 리팩터링할 코드를 함수로 빼냄
  3. 빼낸 함수의 인자로 넘길 부분을 또 다른 함수로 빼냄

## 카피-온-라이트 리팩터링하기

### 함수 본문을 콜백으로 바꾸기 단계

1. 본문과 앞부분, 뒷부분을 확인하기
2. 함수 빼내기
3. 콜백 빼내기

### 카피-온-라이트 단계

1. 복사본 만들기
2. 복사본 변경하기
3. 복사본 리턴하기

## 배열에 대한 카피-온-라이트 리팩터링

### 1. 본문과 앞부분, 뒷부분을 확인하기

```jsx
function arraySet(array, idx, value) {
  var copy = array.slice(); // 앞부분
  copy[idx] = value; // 본문
  return copy; // 뒷부분
}
function push(array, elem) {
  var copy = array.slice(); // 앞부분
  copy.push(elem); // 본문
  return copy; // 뒷부분
}
function drop_last(array) {
  var array_copy = array.slice(); // 앞부분
  array_copy.pop(); // 본문
  return array_copy; // 뒷부분
}
function drop_first(array) {
  var array_copy = array.slice(); // 앞부분
  array_copy.shift(); // 본문
  return array_copy; // 뒷부분
}
```

### 2. 함수 빼내기

- 앞부분과 뒷부분을 빼내기

```jsx
function arraySet(array, idx, value) {
  return withArrayCopy(array);
}

function withArrayCopy(array) {
  var copy = array.slice();
  copy[idx] = value;
  return copy;
}
```

### 3. 콜백 빼내기

```jsx
function arraySet(array, idx, value) {
  return withArrayCopy(array, function (copy) {
    copy[idx] = value;
  });
}

function withArrayCopy(array, modify) {
  var copy = array.slice();
  modify(copy);
  return copy;
}
```

## 함수를 리턴하는 함수

- 중복된 try/catch 구문을 없앴지만 모든 코드에서 withLogging()을 쓰고 있음
- 반복된 일을 없애려고 리팩터링을 적용했지만, 아직도 수동으로 withLogging()이 적용된 버전을 만들어야 함
- 고차 함수를 사용해 이런 일을 해 주는 함수 만들기

### 원래 코드

- 로그를 남겨야 하는 모든 함수에 이런 비슷한 try/catch 구문으로 감싸야 함

```jsx
try {
  saveUserData(user);
} catch (error) {
  logToSnapErrors(error);
}

try {
  fetchProduct(productId);
} catch (error) {
  logToSnapErrors(error);
}
```

### 반복되는 코드를 캡슐화

```jsx
function withLogging(f) {
  try {
    f();
  } catch (error) {
    logToSnapErrors(error);
  }
}

withLogging(function () {
  saveUserData(user);
});
```

- 로그를 남기기 위한 일반적인 시스템이 생겼지만, 여전히 문제가 있음
  - 어떤 부분에 로그를 남기는 것을 깜빡할 수 있음
  - 모든 코드에 수동으로 withLogging() 함수를 적용해야 함
- 에러를 잡아 로그를 남길 수 있는 기능이 추가된 함수를 일반 함수처럼 그냥 호출하면 좋을 듯

### 로그를 남기는 함수 만들기

```jsx
function saveUserDataWithLogging(user) {
  // 이 함수를 호출할 때 로그가 남을 것이라고 예상할 수 있음
  try {
    saveUserDataNoLogging(user); // 로그를 남기지 않는 함수이기 때문에 명확하게 이름 바꾸기
  } catch (error) {
    logToSnapErrors(error);
  }
}

function fetchProductWithLogging(product) {
  // 이 함수를 호출할 때 로그가 남을 것이라고 예상할 수 있음
  try {
    fetchProductNoLogging(productId); // 로그를 남기지 않는 함수이기 때문에 명확하게 이름 바꾸기
  } catch (error) {
    logToSnapErrors(error);
  }
}
```

- 이런 함수가 있다면 로그를 남기기 위해 withLogging() 같은 함수로 감싸야 할지 고민하지 않아도 됨
- 하지만 새로운 중복이 생김 (두 함수가 거의 똑같음)

### 함수 본문을 콜백으로 바꾸기

```jsx
function withLogging(f) {
  return function (arg) {
    try {
      f(arg);
    } catch (error) {
      logToSnapErrors(error);
    }
  };
}

var saveUserDataWithLogging = wrapLogging(saveUserDataNoLogging);
```

- 어떤 함수라도 같은 방식으로 로그를 남기는 함수로 쉽게 바꿀 수 있음
- 함수를 리턴하는 함수는 함수 팩토리와 같음
