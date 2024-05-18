# 10. 일급 함수 1 (2)

## 반복문 예제: 먹고 치우기

### 음식을 준비하고 먹는 일을 하는 코드와 지저분한 식기를 설거지하는 코드

```jsx
for (var i = 0; i < foods.length; i++) {
  var food = foods[i];
  cook(food);
  eat(food);
}

for (var i = 0; i < dishes.length; i++) {
  var dish = dishes[i];
  wash(dish);
  dry(dish);
  putAway(dish);
}
```

### 코드를 함수로 만들기

- 설명할 수 있는 이름을 붙임

```jsx
function cookAndEatFoods() {
  for (var i = 0; i < foods.length; i++) {
    var food = foods[i];
    cook(food);
    eat(food);
  }
}

function cleanDishes() {
  for (var i = 0; i < dishes.length; i++) {
    var dish = dishes[i];
    wash(dish);
    dry(dish);
    putAway(dish);
  }
}

cookAndEatFoods();
cleanDishes();
```

### 두 반복문에서 최대한 문법적으로 비슷한 부분을 찾아 하나로 만들기

- 구체적인 지역변수 이름을 일반적으로 바꾸기

```jsx
function cookAndEatFoods() {
  for (var i = 0; i < foods.length; i++) {
    var item = foods[i]; // food -> item
    cook(item);
    eat(item);
  }
}

function cleanDishes() {
  for (var i = 0; i < dishes.length; i++) {
    var item = dishes[i]; // dish -> item
    wash(item);
    dry(item);
    putAway(item);
  }
}

cookAndEatFoods();
cleanDishes();
```

- 함수 이름에 있는 암묵적 인자를 드러내기

```jsx
function cookAndEatArray(array) {
  // 명시적인 배열 인자 추가
  for (var i = 0; i < array.length; i++) {
    // foods를 인자로 받기
    var item = array[i];
    cook(item);
    eat(item);
  }
}

function cleanArray(array) {
  // 명시적인 배열 인자 추가
  for (var i = 0; i < array.length; i++) {
    // dishes를 인자로 받기
    var item = array[i];
    wash(item);
    dry(item);
    putAway(item);
  }
}

cookAndEatArray(foods); // 인자로 전달
cleanArray(dishes); // 인자로 전달
```

- 반복문 안에 있는 본문 분리하기

```jsx
function cookAndEatArray(array) {
  //     ^^^^^^^^^^   함수 이름에 있는 암묵적인 인자가 두 함수의 차이임
  for (var i = 0; i < array.length; i++) {
    var item = array[i];
    cookAndEat(item); // 빼낸 함수를 호출
  }
}

function cookAndEat(food) {
  cook(food);
  eat(item);
}

function cleanArray(array) {
  //     ^^^^^      함수 이름에 있는 암묵적인 인자가 두 함수의 차이임
  for (var i = 0; i < array.length; i++) {
    var item = array[i];
    clean(item); // 빼낸 함수를 호출
  }
}

function clean(dish) {
  wash(item);
  dry(item);
  putAway(item);
}

cookAndEatArray(foods); // 인자로 전달
cleanArray(dishes); // 인자로 전달
```

- 함수 이름을 일반적인 이름으로 바꾸고 명시적인 인자로 표현

```jsx
function operateOnArray(array, f) {
  // 함수 이름을 일반적인 이름으로 바꾸고 명시적인 인자 추가
  for (var i = 0; i < array.length; i++) {
    var item = array[i];
    f(item); // 본문에서 새 인자 사용
  }
}

function cookAndEat(food) {
  cook(food);
  eat(item);
}

function clean(dish) {
  wash(item);
  dry(item);
  putAway(item);
}

operateOnArray(foods, cookAndEat); // 호출하는 코드에 인자 추가
cleanArray(dishes, clean); // 호출하는 코드에 인자 추가
```

- 배열과 함수를 인자로 받는 고차함수 forEach 사용하기

```jsx
function cookAndEat(food) {
  cook(food);
  eat(item);
}

function clean(dish) {
  wash(item);
  dry(item);
  putAway(item);
}

forEach(foods, cookAndEat);
forEach(dishes, clean);
```

→ 이러한 리팩터링을 함수 본문을 콜백으로 바꾸기라고 함

## 리팩터링: 함수 본문을 콜백으로 바꾸기

- 함수 본문을 콜백으로 바꾸기라는 리팩터링으로 중복을 없앨 수 있음

### 중복이 있는 코드

- 모든 곳에서 같은 형태로 catch 구문을 사용함

```jsx
try {
  saveUserData(user);
} catch (error) {
  logToSnapErrors(error);
}

try {
  fetchProduce(productId);
} catch (error) {
  logToSnapErrors(error);
}
```

### 1. 본문과 본문의 앞부분과 뒷부분을 구분하기

```jsx
try {
  // 앞부분
  saveUserData(user); // 본문
} catch (error) {
  // 뒷부분
  logToSnapErrors(error); // 뒷부분
} // 뒷부분

try {
  // 앞부분
  fetchProduce(productId); // 본문
} catch (error) {
  // 뒷부분
  logToSnapErrors(error); // 뒷부분
} // 뒷부분
```

### 2. 전체를 함수로 빼내기

```jsx
function withLogging() {
  try {
    saveUserData(user);
  } catch (error) {
    logToSnapErrors(error);
  }
}

function withLogging() {
  try {
    fetchProduce(productId);
  } catch (error) {
    logToSnapErrors(error);
  }
}
```

### 3. 본문 부분을 빼낸 함수의 인자로 전달한 함수로 바꾸기

```jsx
function withLogging(f) {
  // f는 함수를 의미
  try {
    f(); // 원래 본문이 있던 곳에서 인자로 받은 함수를 호출
  } catch (error) {
    logToSnapErrors(error);
  }
}

withLogging(function () {
  // 본문을 전달
  saveUserData(user);
});
```

## 이것은 무슨 문법인가요?

### 1. 전역으로 정의하기

- 함수를 전역적으로 정의하고 이름을 붙임
- 함수 정의를 할 때 가장 많이 쓰는 방법으로, 붙인 이름으로 프로그램 어디서나 쓸 수 있음

```jsx
function saveCurrentUserData() {
  saveUserData(user);
}

withLogging(saveCurrentUserData);
```

### 2. 지역적으로 정의하기

- 함수를 지역 범위 안에서 정의하고 이름을 붙임
- 이름을 가지고 있지만, 범위 밖에서는 쓸 수 없음
- 지역적으로 쓰고 싶지만 이름이 필요할 때 유용

```jsx
function someFunction() {
  var saveCurrentUserData = function () {
    saveUserData(user);
  };
  withLogging(saveCurrentUserData);
}
```

### 3. 인라인으로 정의하기

- 함수를 사용하는 곳에서 바로 정의함
- 문맥에서 한 번만 쓰는 짧은 함수에 쓰면 좋음

```jsx
withLogging(function () {
  saveUserData(user);
});
```

## 왜 본문을 함수로 감싸서 넘기나요?

```jsx
withLogging(function () {
  saveUserData(user);
});
```

- 함수를 만들어 감싼 이유는 코드가 바로 실행되면 안 되기 때문
- 함수의 실행을 미루는 일반적인 방법임
- 감싼 함수를 호출하기 전까지는 실행되지 않음
- 즉, withLogging() 함수에서 만든 try/catch 문맥 안에서 실행하도록 코드를 감싼 것
