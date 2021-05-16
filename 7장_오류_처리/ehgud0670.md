# 7장 오류 처리

## 개요 

## 오류 코드보다 예외를 사용하라

> 오류 코드 사용하기 

예외를 지원하지 않는 언어는 오류를 처리하고 보고하는 방법이 제한적이었다. 오류 플래그를 설정하거나 호출자에게 오류 코드를 반환하는 방법이 전부였다. 

* 이 방법의 단점: 
  * 호출자 코드가 복잡해진다. 함수를 호출한 즉시 오류를 확인해야 하기 때문이다. 불행히도 **이 단계는 잊어버리기 쉽다.**

* 이 방법의 장점: 오류를 지원하지 않는 언어에서는 오류 코드만이 수단이라서 사용하는게 장점이라면 장점이지만 오류를 지원하는 언어라면 절대 사용하면 안된다. 

오류가 발생하면 예외를 던지는 편이 낫다. 그러면 호출자 코드가 **더 깔끔**해진다. **논리가 오류 처리 코드와 뒤섞이지 않으니까.**

> 예외 사용하기 

```java
public void sendShutDown() {
    try {
        tryToShutDown();
    } catch (DeviceShutDownError e) {
        logger.log(e);        
    }
}
```
* 코드가 확실히 깔끔해졌다.
* 코드 품질도 더 나아졌다. 앞서 뒤섞였던 개념, **즉 디바이스를 종료하는 알고리즘과 오류를 처리하는 알고리즘을 분리했기 때문이다.** 이제는 각 개념을 독립적으로 살펴보고 이해할 수 있다. 

## Try-Catch-Finally 문부터 작성하라

try 블록에서 무슨 일이 생기든지 **catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다.** 
그러므로 예외가 발생할 코드를 짤 때는 **try-catch-finally 문으로 시작하는 편**이 낫다. 
그래야 try 블록에서 무슨 일이 생기든지 호출자가 기대하는 **상태를 정의하기 쉬워진다.** 
따라서 해당 부분은 **TDD로 개발**하면 깔끔할 것이다! 
다음은 TDD로 작성하는 코드이다. 

> 파일이 없으면 예외를 던지는지 알아보는 테스트
```java
@Test(expected == StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
    sectionStore.retrieveSection("invalid - file");
}
```

단위 테스트에 맞춰 아래 코드를 구현한다. 
> 실패하는 메소드 
```java
public List<RecordedGrip> retrieveSection(String sectionName) {
    // 실제로 구현할 때까지 비어 있는 더미를 반환한다. 
    return new ArrayList<RecordedGrip>();
}
```
=> 코드가 예외를 던지지 않으므로 단위 테스트는 실패한다. 잘못된 파일 접근을 시도하게 구현을 변경하자. 아래 코드는 예외를 던진다.  

> 예외를 던지는 최소한의 성공 테스트
```java
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
    } catch (Exception e) {
        throw new StorageException("retrieval error", e);
    }
    return new ArrayList<RecordedGrip>();
}
```
=> 코드가 예외를 던지므로 이제는 테스트가 성공한다. **이 시점에서 리팩터링이 가능하다.** 
catch 블록에서 예외 유형을 좁혀 실제로 FileInputStream 생성자가 던지는 FileNotFoundException을 잡아내자. 

```java
public List<RecordedGrip> retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
    } catch (FileNotFoundException e) {
        throw new StorageException("retrieval error", e);
    }
    return new ArrayList<RecordedGrip>();
}
```

이처럼 강제로 예외를 일으키는 **테스트 케이스**를 먼저 작성한 후 테스트를 통과하게 코드를 작성하는 방법(TDD)를 권장한다.
그러면 자연스럽게 **try 블록의 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워진다.**

* 트랜잭션(Transaction)이란? 

데이터베이스의 상태를 변화시키기 해서 수행하는 작업의 단위

## 미확인(unchecked) 예외를 사용하라 

(찬반이 있겠지만)확인된 예외는 사용하지 않는다. 확인된 예외는 OCP를 위반하기 때문이다. 
메서드에서 확인된 예외를 던졌는데 catch 블록이 세 단계 위에 있다면 그 사이 메서드 모두가 선언부에 해당 예외를 정의해야 한다. 
즉, 하위 단계에서 코드를 변경하면 **상위 단계 메서드 선언부를 전부 고쳐야 한다는 말**이다. 
이는 OCP를 위반하고 따라서 캡슐화가 깨진다. 
오류를 원거리에서 처리하기 위해 예외를 사용한다는 사실을 감안한다면 이처럼 확인된 예외가 캡슐화를 깨버리는 현상은 참으로 유감스럽다. 

=> 이와 같은 이유로 이 책에서는 확인된 예외가 아닌 미확인 예외를 사용하라고 주장한다.

### Swift는 어떠한가

Swift 는 자바의 미확인 예외는 존재하지 않고, 확인된 예외(`enum Error`)만 존재한다. 
근데 Swift의 Error는 자바처럼 OCP를 위반하는가? 
아니다. 
예외를 던진다면 Swift의 메소드 선언부에는 **오직 throws 만 추가하면 되고 어떤 Error인지는 명시하지 않아도 되기 때문이다.** 
따라서 Swift 의 확인된 예외는 OCP를 위반하지 않으므로 사용해도 된다(또 미확인 예외가 없으므로 사용할 게 이것밖에 없다). 
Swift 문서에도 `enum Error`를 사용하는 걸 권장한다.

## 예외에 의미를 제공하라

예외를 던질 때는 전후 상황을 충분히 덧붙인다. 
그러면 오류가 발생한 원인과 위치를 찾기가 쉬워진다. 
오류 메시지에 정보를 담아 예외와 함께 던진다. 
실패한 연산 이름과 실패 유형도 언급한다. 

## 호출자를 고려해 예외 클래스를 정의하라

애플리케이션에서 오류를 정의할 때 프로그래머에게 가장 중요한 관심사는 **오류를 잡아내는 방법**이 되어야 한다.

> 오류를 형편없이 분류한 사례 

외부 라이브러리를 호출하는 try-catch-finally 문을 포함한 코드로, 외부 라이브러리가 던질 예외를 모두 잡아낸다. 

```java
ACMEPort port = new ACMEPort(12);

try {
    port.open();
} catch (DeviceReponseException e) { 
    reportPortError(e);
    logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {  
    reportPortError(e);
    logger.log("Unlock exception", e);
} catch (GMXError e) { 
    reportPortError(e);
    logger.log("Device reponse exception", e);
} finally {
    //...
}
```
=> 위 코드는 중복이 심하지만 그리 놀랍지 않다(하지만 중복이 있으므로 당연히 좋은 코드가 아니다). 

왜냐하면, 원래 대다수 상황에서 우리가 오류를 처리하는 방식은 (오류를 일으킨 원인과 무관하게) 비교적 일정하기 때문이다. 
1. 오류를 기록한다. 
2. 프로그램을 계속 수행해도 좋은지 확인한다.

위의 경우 예외에 대응하는 방식이 예외 유형과 무관하게 거의 동일하기 때문에 아래처럼 간결하게 고칠수 있다. 호출하는 라이브러리 API를 감싸면서 **예외 유형 하나를 반환**하면 된다. 

> 호출하는 라이브러리 API를 감싸면서 위 코드를 간결하게 작성한 코드

```java
LocalPort port = new LocalPort(12);

try {
    port.open();
} catch (PortDeviceFailure e) { 
    reportPortError(e);
    logger.log(e.getMessage(), e);
} finally {
    //...
}
```
=> 여기서 LocalPort 클래스는 단순히 ACMEPort 클래스가 던지는 예외를 잡아 변환하는 감싸기(wrapper) 클래스일뿐이다. 

```java
public class LocalPort {
    private ACMEPort innerPort;

    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }

    public void open() {
        try {
            innerPort.open();
        } catch (DeviceReponseException e) {
            throw new PortDeviceFailure(e);
        } catch (ATM1212UnlockedException e) {
            throw new PortDeviceFailure(e);
        } catch (GMXError e) {
            throw new PortDeviceFailure(e);
        }
    }
    //...
}
```
* LocalPort 클래스처럼 ACMEPort를 감싸는 클래스는 매우 유용하다. 실제로 외부 API를 사용할 때는 감싸기 기법이 최선이다.
* 또 감싸기 기법을 사용하면 특정 업체가 API를 설계한 방식에 발목 잡히지 않는다. 프로그램이 사용하기 편리한 API를 정의하면 그만이다.
* 흔히 예외 클래스가 하나만 있어도 충분한 코드가 많다. 한 예외는 잡아내고 다른 예외는 무시해도 괜찮은 경우라면 여러 예외 클래스를 사용한다. 

## 정상 흐름을 정의하라

특수 사례 패턴(SPECIAL CASE PATTERN)으로 정상 흐름을 정의하자. 
특수 사례 패턴이란 클래스를 만들거나 객체를 조작해 특수 사례를 처리하는 방식이다.
이 방식으로 클래스나 객체가 **예외적인 상황을 캡슐화해서 처리**한다. 
그러면 **클라이언트 코드가 예외적인 상황을 처리할 필요가 없어진다.**

> 기존 코드: 예외가 논리를 따라가게 어렵게 만든다. 
```java
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
    m_total += getMealPerDiem();
}
```

> 특수 사례 패턴 적용 코드: 예외가 캡슐화되어 클라이언트 코드는 논리가 깔끔해진다. 
```java  
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```

```java
public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
        // 기본값으로 일일 기본 식비를 반환한다. 
    }
}
```

## null을 반환하지 마라 

> null을 반환하는 나쁜 코드

```java
public void registerItem(Item item) {
    if (item != null) {
        ItemRegistry registry = persistentStore.getItemRegistry();
        if (registry != null) {
            Item existing = registry.getItem(item.getID());
            if (existing.getBillingPeriod().hasRetailOwner()) {
                existing.register(item);
            }
        }
    }
}
```
=> 위 코드는 나쁜 코드다! null을 반환하는 코드는 일거리를 늘릴뿐만 아니라 호출자에게 문제를 떠넘긴다. 누구 하나라도 null 확인을 빼먹는다면 애플리케이션이 통제 불능에 빠질지도 모른다. 

* 실제로 위 코드에는 persistentStore가 null인지 검사하지 않는데, 해당 객체가 null이라면 NullPointerException이 발생한다.

> null을 반환하는 나쁜 코드 2

```java
List<Employee> employees = getEmployees();
if (employees != null) {
    for(Employee e : employees) {
        totalPay += e.getPay();
    }
}
```

> 위 코드를 수정한 버전: null이 아닌 빈 객체를 반환한다. 
```java
List<Employee> employees = getEmployees();
for(Employee e : employees) {
    totalPay += e.getPay();
}
```

```java
public List<Employee> getEmployees() {
    if ( /* 직원이 없다면 */ ) {
        return Collections.emptyList();
    }
}
```

=> 이처럼 코드를 변경하면 코드도 깔끔해질뿐더러 NullPointerException이 발생할 가능성도 줄어든다. 

### Swift는 어떠한가

* nil을 반환해도 되지 않는 코드라면 당연히 nil을 반환하지 말자. nil을 반환하면 클라이언트 단에서 옵셔널을 통한 nil 검사를 해야 되고, 클라이언트 단에서 만약에라도 forced unwrapping을 한다면 앱 크래쉬가 발생하기 때문이다.
* 반환하는 값이 컬렉션 타입이라면 위의 자바코드처럼 빈 컬렉션을 반환하는 것도 좋은 방법이다. 
* 반환하는 값이 컬렉션 타입이 아니고 nil이 고려될 수 밖에 없다면 nil을 반환하자. Swift는 nil을 반환해도 된다. **왜냐하면 클라이언트 단에서 옵셔널을 통한 nil 체크를 하면 되기 때문이다.** 하지만 클라이언트 단에서 **forced unwrapping은 하지말도록 무조건 조심**하자!

## null을 전달하지 마라 

* null을 전달하면 메소드 내에서 NullPointerException이 발생할 가능성이 있기 때문이다. 

> null을 전달하는 코드 
```java
public class MetircsCalculator {
    public double xProjection(Point p1, Point p2) {
        return (p2.x - p1.x) * 1.5;       
    }
    //...
}
```

```java
calculator.xProjection(null, new Point(12, 13));
```
=> 이렇게 null을 전달하면 NullPointerException이 발생한다. 

> 개선안 1: 메소드 내에서 null 검사해서 예외 던지기

```java
public class MetircsCalculator {
    public double xProjection(Point p1, Point p2) {
        if (p1 == null || p2 == null) {
            throw InvalidArgumentException(
                "Invalid argument for MetricsCalculator.xProjection");
        }
        return (p2.x - p1.x) * 1.5;       
    }
}
```

> 개선안 2: 메소드 내에서 assert 문 이용하기 

```java
public class MetircsCalculator {
    public double xProjection(Point p1, Point p2) {
        assert p1 != null : "p1 should not be null";
        assert p2 != null : "p2 should not be null";
        return (p2.x - p1.x) * 1.5;       
    }
}
```

### Swift는 어떠한가

* nil을 전달하는 경우 guard을 이용해 nil 처리하자.

```swift
public class MetircsCalculator {
    public func xProjection(p1: Point?, p2: Point?) -> Double? {
        guard let p1 = p1, let p2 = p2 else { return nil }
        return (p2.x - p1.x) * 1.5
    }
}
```

* 자바처럼 assert를 사용한 방법도 있겠다.

```swift
public func xProjection(p1: Point?, p2: Point?) -> Double? {
    assert(p1 != nil, "p1 should not be null")
    assert(p2 != nil, "p1 should not be null")
    return (p2!.x - p1!.x) * 1.5
}
```

* 하지만 애초에 인수에 옵셔널을 추가하지 않는게 Best다(그럴 수 있는 인수라면!!). 

```swift
public func xProjection(p1: Point, p2: Point) -> Double {
    return (p2.x - p1.x) * 1.5
}
```

## 결론 

* 깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다. 이 둘은 **상충하는 목표가 아니다.** 
  * 오류 처리를 프로그램 논리와 분리해 독자적인 사안으로 고려하면 튼튼하고 깨끗한 코드를 작성할 수 있다.
  * 오류 처리를  프로그램 논리와 분리하면 **독립적인 추론이 가능해지며 유지보수성도 크게 높아진다.**
