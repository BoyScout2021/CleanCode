# 6장. 객체와 자료구조 

## 개요

변수를 private으로 정의하는 이유는 **다른 개발자(본인 포함)들이 변수에 맘대로 의존하지 않기 위해서**다.

## 자료 추상화 

* 자료를 세세하게 공개하기보다는 추상적인 개념으로 표현하는 편이 좋다. 

> 구체적인 Point 클래스
```java
public class Point {
    public double x;
    public double y;
}
```
=> x와 y를 함께 설정하는 것이 좋은데 위의 클래스는 따로 따로 설정해야 하는 면에서 좋은 클래스가 아니다.
<br>=> 자료구조가 아닌 객체라면 내부 값을 감춰야 한다.  

> 추상적인 Point 클래스 

```java
interface Point {
    double getX();
    double getY();
    void setCartesian(double x, double y);
    double getR();
    double getTheta();
    void setPolar(double r, double theta);
}
```
=> x와 y를 함께 설정할 수 있는 점에서 위의 구체적인 Point 클래스보다 낫다. 
<br>=> 나는 위 인터페이스를 사용하면 직교좌표계를 사용하는지 극좌표계를 사용하는지 알 길이 없는게 오히려 독자에게 혼란만 가중시킨다고 생각한다(위 인터페이스를 채택한 객체는 직교좌표계, 극좌표계 관련 메소드를 모두 구현해야 한다). 따라서 더 제대로 하려면 위 인터페이스를 `CartesianPoint`, `PolarPoint` 2개의 인터페이스로 나눠야 더 합당하다고 본다.

> 구체적인 Vehicle 클래스
```java
public interface Vehicle {
    double getFuelTankCapacityInGallons();
    double getGallonsOfGasoline();
}
```
=> 단순히 내부의 값을 조회하는 함수들로 구체적인 숫자, 즉 변수값을 읽어 반환할 뿐이다. 이 값들을 이용해 더 의미있는 코드를 작성하는 것은 Vehicle를 사용하는 외부 코드이므로 좋은 설계가 아니다. 

> 추상적인 Vehicle 클래스
```java
public interface Vehicle {
    double getPercentFuelRemaining();
}
```
=> 단순히 내부값을 반환하는 것이 아닌 내부 값을 이용해 백분율이라는 추상적인 개념을 반환한다. Vehicle를 사용하는 외부의 코드는 백분율이 필요할 때 이 메소드만 호출하면 된다. 즉 Vehicle이라는 하위 객체가 자신의 변수값을 이용해 백분율이라는 추상적인 값을 상위 객체에게 반환하므로 좋은 설계다. 

* 이 항목에서 인상 깊은 한마디: **인터페이스나 조회/설정만으로는 추상화가 이뤄지지 않는다.** 개발자는 객체가 포함하는 자료를 표현할 가장 좋은 방법을 심각하게 고민해야 한다. **아무 생각없이 조회/설정 함수를 추가하는 방법이 가장 나쁘다.**

## 자료/객체 비대칭 

* (자료구조를 사용하는) 절차적인 코드는 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다. 반면, 객체 지향 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다. 
* 절차적인 코드는 새로운 자료 구조를 추가하기 어렵다. 그러러면 모든 함수를 고쳐야 한다. 객체 지향 코드는 새로운 함수를 추가하기 어렵다. 그러려면 모든 클래스를 고쳐야 한다. 

* 인상 깊은 한마디: 분별 있는 프로그래머는 모든 것이 객체라는 생각이 미신임을 잘 안다. **때로는 단순한 자료 구조와 절차적인 코드가 가장 적합한 상황도 있다.**

> 절차지향이 객체지향보다 더 적합한 경우가 있다. (by [우아한 객체지향 강의 - 조영호](https://www.youtube.com/watch?v=dJ5C4qRqAgA))

* 절차지향은 로직을 한눈에 보인다는 장점이 있다(ex. `Validator`).

<img width="800" alt="우아한_객체지향" src="https://user-images.githubusercontent.com/38216027/126743543-d8327731-3b31-4137-9807-3c5d5f6a6866.png">

### 함수형 프로그래밍

* 내부 변수가 자료구조 타입인 경우(ex. Array, Dictionary, Set...) 내부 변수는 private 으로 은닉화하고 클로저를 이용해 내부 변수에 접근하기
  * 함수형을 사용하므로 Side Effect 가 일어나면 안됩니다. 따라서 클로저로 해당 요소(element)의 index 값도 같이 보냅니다. 

> Participant.swift 

```swift
class Participant {   
    private var cards = [Card]()

    func searchCard(handler: (Int, Card) -> Void) {
        for (cardIndex, card) in self.cards.enumerated() {
            handler(cardIndex, card)
        }
    }
}    
```

> 사용하는 쪽 코드 

```swift 
participant.searchCard { cardIndex, card in
    if card.suit == .club {
        print(cardIndex, card)
    }
}
```

> Note: 클로저로 받은 Card 객체의 내부값을 변경 못하도록 Card 객체를 불변객체로 만듭니다


## 디미터 법칙 

* 디미터 법칙은 객체 C의 모든 메서드는 다음에 해당하는 **메서드만**을 호출해야 한다고 말한다(by 실용주의 프로그래머 & 클린 코드).
  * 자신
    ```java
    class C {
        void foo() {
            goo();
        }

        private void goo() { }
    }
    ```
  * 메서드로 넘어온 인자
    ```java
    class C {
        void foo(B b) {
            b.goo();
        }
    }    
    ```
  * 자신이 생성한 객체
    ```java
    class C {
        void foo() {
            B b = new B();
            b.goo();
        }
    }
    ```
  * 직접 포함하고 있는 객체
    ```java
    class C {
        private B b = new B(); 

        void foo() {
            b.goo();
        }
    }
    ```
=> 이외의 메서드는 호출해선 안된다. 

* 디미터 법칙은 결합도를 줄이는데 목적이 있다. 
* 디미터 법칙은 오직 객체에만 적용될뿐 **자료 구조에는 적용되지 않는다.**

## 주의사항 - 하나의 .을 강제하는 규칙이 아니다.
디미터 법칙은 객체 지향 생활 체조 원칙 중 한 줄에 점을 하나만 찍는다.로 요약되기도 한다.

```java
IntStream.of(1, 15, 3, 20)
    .filter(x -> x > 10)
    .distinct()
    .count();
```
=> 하지만 위 코드는 디미터 법칙과 객체 지향 생활 체조 원칙을 위반한 코드가 아니다.

**디미터 법칙은 결합도와 관련된 법칙이고 결합도가 문제 되는 것은 객체의 내부 구조가 외부로 노출되는 경우이다.**
위 코드는 IntStream의 내부 구조가 노출되지 않았다. 단지 IntStream을 다른 IntStream으로 변환할 뿐, 객체를 둘러싸고 있는 캡슐은 유지한다. 한 줄에 하나 이상의 점을 찍는 모든 케이스가 객체 지향 생활 체조 원칙 및 디미터 법칙을 위반하는 것은 아니다. **객체 내부 구현에 대한 어떤 정보도 외부로 노출하지 않는다면 괜찮다.**

=> 이 항목은 해당 [블로그글](https://woowacourse.github.io/javable/post/2020-06-02-law-of-demeter/)을 참고하였습니다. 

### 디미터 법칙을 위반한 코드 - 기차 충돌

디미터 법칙을 위반한 전형적인 코드의 형태는 다음과 같다.

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```
getter가 줄줄이 이어진 모습이 기차와 닮아서 열차 전복, 기차 충돌(train wreck) 이라는 단어로 표현하기도 한다.

### 잡종 구조 - 잡종을 경계하라!

### 구조체 감추기 

```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```
=> **객체의 내부 값이 필요한 이유를 알면** 위 코드처럼 디미터 법칙을 위반하지 않는 우아한 메서드가 탄생한다. 


## 자료 전달 객체(DTO)

자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스다. 이런 자료 구조체를 때로는 자료 전달 객체(Data Transfer Object, DTO)라 한다. 

### 활성 레코드 

* DTO에서 추가적으로 save나 find와 같은 탐색 함수를 제공한다.
* 활성 레코드는 자료 구조로 취급한다. 비즈니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성한다. 여기서 내부 자료는 활성 레코드의 인스턴스일 가능성이 높다(ex. **ViewModel과 Model**)

### 결론 

* 객체는 동작을 공개하고 자료를 숨긴다. 
* 자료구조는 별다른 동작없이 자료를 노출한다. 자료구조는 별도의 메서드를 만들지 않는다.
* 우수한 프로그래머는 **새로운 자료 타입을 추가하는 유연성이 필요하면 객체를 사용**하고, **새로운 동작(함수)를 추가하는 유연성이 필요하면 자료 구조와 절차적인 코드를 사용**한다. 
