# 12장. 창발성 

## 창발적 설계로 깔끔한 코드를 구현하자

켄트 벡이 제시한 **단순한 설계 규칙 네 가지**가 소프트웨어 설계 품질을 크게 높여준다고 믿는다. 
이 4가지 규칙을 따르면 SRP나 DIP와 같은 원칙을 적용하기 쉽고, 우수한 설계의 **창발성**을 촉진한다고 믿는다. 
켄트 백은 다음 규칙을 따르면 **설계는 '단순하다'고 말한다.** 

* 모든 테스트를 수행한다.
* 중복을 없앤다.
* 프로그래머 의도를 표현한다. 
* 클래스와 메서드 수를 최소로 줄인다. 

## 단순한 설계 규칙 1: 모든 테스트를 실행하라

* 테스트를 철저히 거쳐 모든 테스트 케이스를 항상 통과하는 시스템이 `테스트가 가능한 시스템` 이다. 테스트 하지 않으면 `테스트가 가능한 시스템`이 아니다. 
  * 논란의 여지는 있지만, 검증(테스트)이 불가능한 시스템은 **절대 출시하면 안 된다.**
* 테스트가 가능한 시스템을 만들려고 애쓰면 설계 품질이 더불어 높아진다. **SRP를 준수하는 클래스는 테스트가 훨씬 더 쉽다.** 따라서 테스트에 신경쓰면 자연히 SRP를 지키는 클래스가 많이 나올 것이다.  
* 결합도가 높으면 테스트 케이스를 작성하기 어렵다. 그러므로, 테스트 케이스를 많이 작성할수록 **개발자는 DIP와 같은 원칙을 적용하고, 의존성 주입, 인터페이스, 추상화 등과 같은 도구를 사용해 결합도를 낮춘다.** 따라서 설계 품질은 더욱 높아진다.
* 따라서 `테스트 케이스를 만들고 계속 돌려라`라는 간단하고 단순한 규칙을 따르면 시스템은 **낮은 결합도와 높은 응집력이라는 객체 지향 방법론이 지향하는 목표**를 저절로 달성한다. 즉, 설계 품질이 높아진다. 

## 단순한 설계 규칙 2~4: 리팩터링  

* 테스트 케이스를 모두 작성했다면 코드를 점진적으로 리팩터링 해나간다. **코드를 리팩토링하면서 시스템이 깨질까 걱정할 필요가 없다. 테스트 케이스가 있으니까, 피드백을 받을 수 있으니까 말이다!**
* 리팩터링 단계에서는 소프트웨어 설계 품질을 높이는 기법이라면 무엇이든 적용해도 괜찮다. 응집도를 높이고, 결합도를 낮추고, 관심사를 분리하고, 시스템 관심사를 모듈로 나누고, 함수와 클래스 크기를 줄이고, 더 나은 이름을 선택하는 등 다양한 기법을 동원한다.

## 중복을 없애라 

중복은 우수한 설계의 커다란 적이다. 중복을 없애자.

```java
int size() {}
boolean isEmpty() {}
```

=> 각 메서드를 따로 구현하는 방법도 있지만 `isEmpty` 메서드에서 `size` 메서드를 이용하면 코드를 중복해 구현할 필요가 없어진다.

```java
boolean isEmpty() {
    return 0 == size();
}
```
=> 이렇게 말이다. 


```java
public void scaleToOneDimension(
        float desiredDimension, float imageDimension) {
    if (Math.abs(desiredDimension - imageDimension < errorThreshold)) {
        return;
    }
    float scalingFactor = desiredDimension / imageDimension;
    scalingFactor = (float) (Math.floor(scalingFactor * 100) * 0.01f);
    
    RenderedOp newImage = ImageUtilities.getScaledImage(
            image, scalingFactor, scalingFactor);
    image.dispose();
    System.gc();
    image = newImage;
}

public synchronized void rotate(int degrees) {
    RenderedOp newImage = ImageUtilities.getRotatedImage(
            image, degrees);
    image.dispose();
    System.gc();
    image = newImage;
}
```
=> `sizeToOneDimension` 메서드와 `rotate` 메서드를 살펴보면 일부 코드가 동일하다. 다음과 같이 코드를 정리해 중복을 제거한다. 

```java
public void scaleToOneDimension(
        float desiredDimension, float imageDimension) {
    if (Math.abs(desiredDimension - imageDimension < errorThreshold)) {
        return;
    }
    float scalingFactor = desiredDimension / imageDimension;
    scalingFactor = (float) (Math.floor(scalingFactor * 100) * 0.01f);
    replaceImage(ImageUtilities.getScaledImage(
            image, scalingFactor, scalingFactor));
}

public synchronized void rotate(int degrees) {
    replaceImage(ImageUtilities.getRotatedImage(
            image, degrees););
}

private void replaceImage(RenderedOp newImage) {
    image.dispose();
    System.gc();
    image = newImage;
}
```
=> 아주 적은 양이지만 공통적인 코드를 새 메서드로 뽑고 보니 클래스가 SRP를 위반한다. 
그러므로 새로 만든 `replaceImage` 메서드를 다른 클래스로 옮겨도 좋겠다. 
그러면 새 메서드의 가시성이 높아진다. 
**따라서 다른 팀원이 새 메서드를 좀 더 추상화해 다른 맥락에서 재사용할 기회를 포착할지도 모른다.** 

TEMPLATE METHOD 패턴은 고차원 중복을 제거할 목적으로 자주 사용하는 기법이다. 아래가 템플릿 메서드의 예이다. 
```java
public class VacationPolicy {
    public void accrueUSDivisionVacation() {
        // 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
        // ...
        // 휴가 일수가 미국 최소 법정 일수를 만족하는지 확인하는 코드 
        // ...
        // 휴가 일수를 급여 대장에 적용하는 코드
        // ...
    }

    public void accrueEUDivisionVacation() {
        // 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
        // ...
        // 휴가 일수가 유럽연합 최소 법정 일수를 만족하는지 확인하는 코드 
        // ...
        // 휴가 일수를 급여 대장에 적용하는 코드
        // ...
    }
}
```
=> 최소 법정 일수를 계산하는 코드만 제외하면 두 메서드는 거의 동일하다. 
최소 법정 일수를 계산하는 부분만 아래 코드처럼 TEMPLATE METHOD 패턴을 적용해 눈에 들어오는 중복을 제거한다. 

```java
abstract public class VacationPolicy {
    public void accrueVacation() {
        calculateBaseVacationHours();
        alterForLegalMinimums();
        applyToPayroll();
    }
    
    private void calculateBaseVacationHours() { /* ... */ };
    abstract protected void alterForLegalMinimums();
    private void applyToPayroll() { /* ... */ };
}

public class USVacationPolicy extends VacationPolicy {
    @Override protected void alterForLegalMinimums() {
        // 미국 최소 법정 일수를 사용한다.
    }
}

public class EUVacationPolicy extends VacationPolicy {
    @Override protected void alterForLegalMinimums() {
        // 유럽연합 최소 법정 일수를 사용한다.
    }
}
```
=> 하위 클래스는 중복되지 않는 정보만 제공해 accrueVacation 알고리즘에서 빠진 '구멍'을 메운다.

## 표현하라

개발자가 코드를 명백하게 짤수록 다른 사람이 그 코드를 이해하기 쉬워진다. 그래야 결함이 줄어들고 유지보수 비용이 적게 든다. 

1. 우선, 좋은 이름을 선택한다. **이름과 기능이 완전히 딴판인 클래스나 함수로 유지보수 담당자를 놀라게 해서는 안된다.**
2. 함수와 클래스 크기를 가능한 줄인다. 
3. 표준 명칭을 사용한다. 예를 들어, **디자인 패턴은 의사소통과 표현력 강화가 주요 목적이다.** 클래스가 COMMAND나 VISITOR와 같은 표준 패턴을 사용해 구현된다면 클래스 이름에 패턴 이름을 넣어준다. 그러면 다른 개발자가 클래스 설계 의도를 이해하기 쉬워진다. 
4. 단위 테스트 케이스를 꼼꼼히 작성한다. 

하지만 표현력을 높이는 가장 중요한 방법은 **노력**이다. 나중에 읽을 사람을 고려해 조금이라도 읽기 쉽게 만들려고 충분히 고민하자.

## 클래스와 메서드 수를 최소로 줄여라 

SRP를 준수한다고 클래스와 메서드를 극단으로 잘게 나누는 것은 득보다 실이 많아지는 행동이다. 
따라서 이 규칙은 **함수와 클래스 수**를 가능한 줄이라고 제안한다.
하지만 이 규칙은 간단한 설계 규칙 네 개 중 우선 순위가 가장 낮다. 
다시 말해, 클래스와 함수 수를 줄이는 작업도 중요하지만, **테스트 케이스를 만들고 중복을 제거하고 의도를 표현하는 작업이 더 중요하다는 뜻**이다.

## 결론

경험이 제일 중요하긴 하다. 하지만 단순한 설계 4 규칙을 따르다면 (오랜 경험 후에야 익힐) 우수한 기법과 원칙을 단번에 활용할 수 있다. 