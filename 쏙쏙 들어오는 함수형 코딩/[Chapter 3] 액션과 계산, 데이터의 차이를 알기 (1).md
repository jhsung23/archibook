# 3. 액션과 계산, 데이터의 차이를 알기 (1)

## 액션과 계산, 데이터

- 함수형 프로그래머는 액션과 계산, 데이터를 구분해야 함

### 액션

- 실행 시점과 횟수에 의존함
- 부수 효과, 부수 효과가 있는 함수, 순수하지 않은 함수가 액션에 해당함
- ex) 이메일 보내기, 데이터베이스 읽기

### 계산

- 입력으로 출력을 계산함
- 순수 함수, 수학 함수가 계싼에 해당함
- ex) 최댓값 찾기, 이메일 주소가 올바른지 확인하기

### 데이터

- 이벤트에 대한 사실
- ex) 사용자가 입력한 이메일 주소, 은행 API로 읽은 달러 수량

## 액션과 계산, 데이터는 어디에나 적용할 수 있습니다

- 장보기 과정을 그린다면 다음과 같음
  | 과정                   | 액션/계산/데이터 | 이유                                                                                               |
  | ---------------------- | ---------------- | -------------------------------------------------------------------------------------------------- |
  | 냉장고 확인하기        | 액션             | 냉장고를 확인하는 시점에 따라 냉장고에 있는 제품이 다름                                            |
  | 운전해서 상점으로 가기 | 액션             | 두 번 운전해서 상점에 가면 연료가 두 배 소모됨                                                     |
  | 필요한 것 구입하기     | 액션             | 누군가 브로콜리를 구입하면 브로콜리가 다 떨어질 수도 있기 때문에 구입하는 시점이 중요              |
  | 운전해서 집으로 오기   | 액션             | 이미 집에 있다면 상점에 있는 것이 아니기 때문에 상점에서 집으로 올 수 없으므로 언제 하는 지가 중요 |

### 모든 것이 액션인것인가?

- 모든 것을 액션으로 분류하면 안 됨
- 아주 단순한 과정은 액션으로만 분류할 수 있지만, 장보기 과정은 그렇게 단순하지 않음

### 냉장고 확인하기

- 냉장고를 확인하는 일은 확인하는 시점이 중요하므로 액션임
- 냉장고에 가지고 있는 제품은 데이터 (= 현재 재고)

### 운전해서 상점으로 가기

- 운전해서 상점으로 가는 것은 복잡한 행동이고 명확히 액션
- 상점 위치나 가는 경로는 데이터

### 필요한 것 구입하기

- 구입하는 것도 액션
- 구입을 하기 위해서는 필요한 것이 무엇인지도 알아야 함
  - 필요한 재고에서 현재 재고를 제외하면 장보기 목록이 됨
- 단계를 더 세분화할 수 있음
  | 과정                    | 액션/계산/데이터 | 이유                                                                     |
  | ----------------------- | ---------------- | ------------------------------------------------------------------------ |
  | 현재 재고               | 데이터           | .                                                                        |
  | 필요한 재고             | 데이터           | .                                                                        |
  | 재고 빼기               | 계산             | 같은 입력 값일 때 항상 같은 결과값을 줌 (어떤 것을 결정하는 일은 계산임) |
  | 장보기 목록             | 데이터           | .                                                                        |
  | 목록에 있는 것 구입하기 | 액션             | .                                                                        |
- 구입하는 단계와 구입하는 것을 결정하는 단계로 나누어 액션과 계산을 분리할 수 있음

### 운전해서 집으로 오기

- 운전해서 집오는 단계도 더 나눌 수 있지만 논외

### 정리

| 순서 | 액션                    | 계산      | 데이터      | 비고                                                                                             |
| ---- | ----------------------- | --------- | ----------- | ------------------------------------------------------------------------------------------------ |
| 1    | 냉장고 확인             | .         | 현재 재고   | 냉장고 확인 액션의 결과는 현재 재고                                                              |
| 2    | 운전해서 상점으로 가기  | .         | .           | .                                                                                                |
| 3    | .                       | .         | 필요한 재고 | .                                                                                                |
| 4    | .                       | 재고 빼기 | 장보기 목록 | 재고 빼기는 입력 데이터가 두 개(현재 재고, 필요한 재고) 필요하며, 계산한 결과는 장보기 목록이 됨 |
| 5    | 목록에 있는 것 구입하기 | .         | .           | 목록에 있는 것 구입은 장보기 목록을 입력값으로 사용                                              |
| 6    | 운전해서 집으로 오기    | .         | .           | .                                                                                                |

- 이렇게 반복하면 액션과 계산, 데이터를 더 많이 찾을 수 있고 풍부한 모델을 만들 수 있음
- 계속 나누다 보면 점점 더 복잡해진다고 생각할 수 있지만, 액션에 숨어 있는 다른 액션이나 계산 또는 데이터를 발견하기 위해 나눌 수 있는 만큼 나누는 것이 좋음
  - ex) 냉장고 확인하기 = 냉장실 확인하기 + 냉동실 확인하기 = … + … + …
  - ex) 목록에 있는 것 구입하기 = 장바구니 담기 + 계산하기 = … + … + …

## 장보기 과정에서 배운 것

1. 액션과 계산, 데이터는 어디에나 적용할 수 있다
2. 액션 안에는 계산과 데이터, 또 다른 액션이 숨어 있을지도 모른다
   - 액션을 더 작은 액션과 계산, 데이터로 나누고 나누는 것을 언제 멈춰야 할지 아는 것이 중요함
3. 계산은 더 작은 계산과 데이터로 나누고 연결할 수 있음
   - 계산을 더 작은 계산으로 나누는 것이 좋을 때도 있음
     - 계산을 나누면 첫 번째 계산의 결과가 두 번째 계산의 입력이 됨
4. 데이터는 데이터만 조합할 수 있다
   - 데이터는 다른 영향을 주지 않는 그냥 데이터임
   - 데이터 찾는 일을 먼저 해야 하며 데이터를 찾았다면 동작에 대해 많은 것을 알 수 있음
5. 계산은 때로 우리 머릿속에서 일어난다
   - 계산 단계가 있지만 잘 보이지 않는 이유는 계산이 우리 사고 과정에 녹아있기 때문
   - 어떤 단계에서 무엇인가 결정해야 할 것이 있는지 또는 무엇인가 계획해서 방법을 찾아야 할 것이 있는지 스스로에게 질문
   - 결정과 계획은 계산이 될 가능성이 높음

## 데이터에 대해 자세히 알아보기

### 데이터는 무엇인가요?

- 이벤트에 대한 사실
- 일어난 일의 결과를 기록한 것

### 데이터를 어떻게 구현하나요?

- 자바스크립트에서는 기본 데이터 타입으로 구현
- 숫자, 문자, 배열, 객체

### 어떻게 데이터에 의미를 담을 수 있나요?

- 데이터 구조로 의미를 담을 수 있음
- 데이터 구조로 도메인을 표현할 수 있음

### 불변성

- 함수형 프로그래머는 불변 데이터 구조를 만들기 위해 두 가지 원칙을 사용함
  1. 카피-온-라이트(copy on write): 변경할 때 복사본을 만듦
  2. 방어적 복사(defensive copy): 보관하려고 하는 데이터의 복사본을 만듦

### 데이터의 예

- 구입하려는 음식 목록, 이름, 전화 번호, 음식 조리법

### 데이터의 장점은 무엇인가요?

- 역설적으로 데이터 자체로 할 수 있는 것이 없기 때문에 좋음
- 데이터는 데이터 그대로 이해할 수 있음

### 데이터의 단점은 무엇인가요?

- 유연하게 해석할 수 있다는 점은 장점이지만, 해석이 반드시 필요하다는 점은 단점
- 계산은 해석하지 않아도 실행할 수 있지만 해석하지 않은 데이터는 쓸모없는 바이트임
