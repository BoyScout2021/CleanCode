# 3장. 함수 

## 개요 

* 어떤 프로그램이든 가장 기본적인 단위가 **함수**다.

## 작게 만들어라!

* 함수를 만드는 첫째 규칙은 '작게!'다. 함수를 만드는 둘째 규칙은 '더 작게!'다. **40여년간 오랜 시행착오를 바탕으로 나(밥 아저씨)는 작은 함수가 좋다고 확신한다.**
* 함수가 얼마나 짧아야 하느냐고? 일반적으로 목록 3-2 보다 짧아야 한다! 목록 3-2는 목록 3-3으로 줄여야 마땅하다.

> 목록 3-3 HtmlUtil.java(re-refactored)
```java
public static String renderPageWithSetupsAndTeardowns(PageData pageData, boolean isSuite) throws Exception {
    if (isTestPage(pageData)) {
        includeSetupAndTeardownPages(pageData, isSuite);
    }
    return pageData.getHtml();
}
```
=> 목록 3-1 이 무려 59줄, 목록 3-2 가 10줄, 목록 3-3 이 불과 3줄이다(마법이다, 마법).

### 블록과 들여쓰기 

* if 문/else 문/while 문 등에 들어가는 블록은 한 줄이어야 한다는 의미다. 대개 거기서 함수를 호출한다. 
* **그러면 바깥을 감싸는 함수(enclosing function)가 작아질뿐 만 아니라**
* 블록 안에서 호출하는 함수 이름을 적절히 짓는다면, 코드를 이해하기도 쉬워진다. 

## 한 가지만 해라!

* **함수는 한 가지를 해야 한다. 그 한가지를 잘 해야 한다. 그 한 가지만을 해야 한다.**

하지만 `renderPageWithSetupsAndTearDowns` 가 세가지를 한다고 주장할 수도 있다.

1. 페이지가 테스트 페이지인지 판단한다. 
2. 그렇다면 설정 페이지와 해제 페이지를 넣는다. 
3. 페이지를 HTML로 렌더링한다.

=> 위에서 언급하는 세 단계는 지정된 함수 아래에서 추상화 수준이 하나다. **지정된 함수 이름 아래에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한가지 작업만 한다.**

## 함수 당 추상화 수준은 하나로! (쉽지 않네!)

* **함수가 확실히 '한 가지' 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 한다.**
* 한 함수 내에 추상화 수준을 섞으면 코드를 읽는 사람이 헷갈린다. 특정 표현이 근본 개념인지 아니면 세부사항인지 구분하기 어려운 탓이다. 
* 하지만 문제는 이 정도로 그치지 않는다. 근본 개념과 세부사항을 뒤섞기 시작하면, **깨어진 창문처럼 사람들이 함수에 세부사항을 점점 더 추가한다(내가 그렇게 하고 있다, 베키송 ㅜㅜ).**

### 위에서 아래로 코드 읽기: 내려가기 규칙

코드는 위에서 아래로 이야기처럼 읽혀야 좋다. 한 함수 다음에는 추상화 수준이 한 단계 낮은 함수가 온다. 즉 위에서 아래로 프로그램을 읽으면 함수 추상화 수준이 한 번에 한 단계씩 낮아진다. 이것을 **내려가기 규칙**이라 부른다. 

## Switch 문 

* switch 문은 작게 만들기 어렵다. 
* 또한 '한 가지' 작업만 하는 switch문도 만들기 어렵다. **본질적으로 switch 문은 N가지를 처리한다.** 불행하게도 switch문을 완전히 피할 방법은 없다. 
* **하지만 각 switch 문을 저차원 클래스에 숨기고 절대로 반복하지 않는 방법은 있다.** 물론 **다형성(polymorphism)** 을 이용한다.

> 목록 3-4 Payroll.java
```java
public Money calculatePay(Employee e) throws InvalidEmployeeType {
    switch (e.type) {
        case COMISSIONED:
            return calculateCommissionedPay(e);
        case HOURLY:
            return calculateHourlyPay(e);
        case SALARIED:
            return calculateSalariedPay(e);
        default:
            throw new InvalidEmployeeType(e.type);
    }
}
```
=> 위 함수는 첫째-함수가 길다, 둘째-'한가지' 작업만 수행하지 않는다. 셋째-SRP를 위반한다. 넷째-OCP를 위반하는 4가지 문제가 있다. 
<br>=> 셋째-SRP를 위반하는 이유는 해당 함수가 다형성을 고려하지 않고 모든 직원 유형에 따라 임금 계산을 하고 있기 때문이다. 다형성을 이용해 하나의 클래스는 하나의 역할만 해야한다. `SalariedEmployee`, `HourlyEmployee` 등등으로 나눠서 `calculatePay` 를 구현해야 한다.
<br>=> 넷째-OCP를 위반하는 이유는 Employee의 유형을 추가할 때 마다할 코드를 변경해야 하기 때문이다. 다형성을 이용해 수정에는 닫혀있고 확장에는 열려있도록 하자.
<br>=> **가장 심각한 문제는 위 함수와 구조가 동일한 함수가 무한정 생기고 존재한다는 사실이다.** ex) `isPayday(Employee e, Date date)` 혹은 `deliverPay(Employee e, Money pay)`

> 목록 3-5 Employee and Factory

```java
public abstract class Employee {
    public abstract Money calculatePay();
}

_____________________________________________

public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord e) throws InvalidEmployeeType;
}

public class EmployeeFactoryImpl implements EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
        return switch (r.type) {
            case COMISSIONED -> new CommissionedEmployee(r);
            case HOURLY -> new HourlyEmployee(r);
            case SALARIED -> new SalariedEmployee(r);
            default -> throw new InvalidEmployeeType(r.type);
        };
    }
}
```
=> 3-5는 3-4를 해결한 코드다. 팩토리는 switch문을 사용해 적절한 Employee 파생 클래스의 인스턴스를 생성한다. 다형성을 이용해 실제 파생 클래스의 함수가 실행된다.
=> switch 문은 **다형적 객체를 생성**하는 코드안에서 사용함으로써 **단 한번만 작성되도록** 숨겨준다.

## 서술적인 이름을 사용하라! 

* 서술적인 이름을 붙여라. 좋은 이름이 주는 가치는 아무리 강조해도 지나치지 않다.
* 이름이 길어도 괜찮다. 겁먹을 필요없다. **길고 서술적인 이름이 짧고 어려운 이름보다 좋다.**
* 이름이 붙일때는 일관성이 있어야 한다. ex) (`includeSetupAndTeardownPages`, `includeSetupPages`, `includeSuiteSetupPage`, `includeSetPage`)

## 함수 인수 
최선은 입력 인수가 없는 경우이며, 차선은 입력 인수가 1개뿐인 경우다. 

* 함수에서 이상적인 인수 개수는 0개(무항)다. 다음은 1개(단항)이고, 다음은 2개(이항)다. **3개(삼항)는 가능한 피하는 편이 좋다. 4개 이상(다항)은 특별한 이유가 있어도 사용하면 안된다.**
* 테스트 관점에서 보면 인수는 더 어렵다. 갖가지 인수 조합으로 함수를 검증하는 테스트 케이스를 작성한다고 상상해보라! 인수가 없다면 간단하다. 인수가 하나라도 괜찮다. 인수가 2개면 조금 복잡해진다. **인수가 3개를 넘어가면 인수마다 유효한 값으로 모든 조합을 구성해 테스트하기가 상당히 부담스러워진다.(인수가 늘어날수록 테스트 케이스가 늘어나므로 인수 개수는 적을수록 좋다)**
* 출력 인수는 입력 인수보다 이해하기 어렵다. 흔히 우리는 함수에다 인수로 입력을 넘기고 반환값으로 출력을 받는다는 개념에 익숙하다. 대개 함수에서 인수로 결과를 받으리라 기대하지 않는다. **그래서 출력 인수는 독자가 허둥지둥 코드를 재차 확인하게 만든다(출력 인수 쓰지 말자, `inout` 사용하지 말자).**

### 많이 쓰는 단항 형식

* 인수에 질문을 던지는 경우 많이 사용한다. `boolean fileExist("MyFile")` 이 좋은 예다.
* 인수를 뭔가로 변환해 결과를 반환하는 경우다. `InputStream fileOpen("MyFile")`은 String 형의 파일 이름을 InputStream으로 변환한다.
* 다소 드물게 사용하지만 그래도 아주 유용한 단항 함수 형식이 **이벤트 함수**다. `passwordAttemptFailedNtimes(int attempts)` 가 좋은 예다. iOS 의 경우, `func buttonDidTap(sender: UIButton)` 가 있다. 이벤트 함수는 조심해서 사용한다. **이벤트라는 사실이 코드에 명확히 드러나야 한다.** 
* 출력 인수는 피한다. `StringBuffer transform(StringBuffer in)`이 `void transform(StringBuffer out)`보다 좋다.   

### 플래그 인수 

* 플래그 인수는 추하다. 함수로 부울 값을 넘기는 관례는 정말로 끔찍하다. **함수가 한꺼번에 여러 가지를 처리한다고 대놓고 공표하는셈이니까 말이다.** 사용하지 말자.
* `render(true)` 는 `renderForSuite()`와 `renderForSingleTest()`라는 함수로 나눠야 마땅하다. 

### 이항 함수

* 인수가 2개인 함수는 인수가 1개인 함수보다 이해하기 어렵다.
* 이항 함수가 무조건 나쁘다는 소리가 아니다. 프로그램을 짜다보면 불가피한 경우도 생긴다. **하지만 그만큼 위험이 따른다는 사실을 이해하고 가능하면 단항함수로 바꾸도록 애써야 한다.** 예를 들어, writeField 메서드를 outputStream 클래스 구성원으로 만들어 `outputStream.writeField(name)`으로 호출한다. 아니면 outputStream을 현재 클래스 구성원 변수로 만들어 인수로 넘기지 않는다. 아니면 FieldWriter라는 새 클래스를 만들어 구성자에서 outputStream을 받고 write 메서드를 구현한다.
* 이항 함수가 적절한 경우도 있다. `Point p = new Point(0,0)` 가 좋은 예다. **하지만 여기서 인수 2개는 한 값을 표현하는 두 요소다. 두 요소에는 자연적인 순서도 있다.**

### 삼항 함수 

인수가 3개인 함수는 인수가 2개인 함수보다 훨씬 더 이해하기 어렵다(작성하지 말자). **순서, 주춤, 무시로 야기되는 문제가 두 배 이상 늘어난다.**
* 예를 들어, assertEquals(message, expected, actual)이라는 함수를 살펴보자. 첫 인수가 `expected`라고 예상하지 않았는가? 매번 함수를 볼 때마다 주춤했다가 message를 무시해야 한다는 사실을 상기했다. 무시하지 않게 3항 함수는 작성하지 말자.
* 반면 `assertEquals(1.0, amount, .001)`은 그리 음험하지 않은 삼항 함수다. 여전히 주춤하게 되지만 그만한 가치가 충분하다. **부동소수점 비교가 상대적이라는 사실은 언제든 주지할 중요한 사항이다.**

### 인수 객체 

인수가 2-3개 필요하다면 일부를 **독자적인 클래스 변수**로 선언할 가능성을 짚어본다. 

```java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```
=> 객체를 생성해 인수를 줄이는 방법이 눈속임이라 여겨질지 모르지만 **그렇지 않다.**
<br>=> 위 예제에서 **x와 y를 묶었듯이 변수를 묶어 넘기려면 이름을 붙여야 하므로 결국은 개념을 표현하게 된다(순서가 생기고 가독성이 생긴다. 해당 Point가 center라는 것도 명확해져서 쓰는 사람 입장에서 바로 사용할 수 있게끔 해준다).**

### 인수 목록 
때로는 인수 개수가 가변적인 함수도 필요하다. `String.format` 메서드가 좋은 예다.  

```java
String.format("%s worked %.2f hours.", name, hours);
```

```java
public String format(String format, Object... args);
```

=> 위 예제처럼 가변 인수 전부를 동등하게 취급하면 List 형 인수 하나로 취급할 수 있다. **이런 논리로 따져보면 String.format은 사실상 이항 함수다.**

```java
void monad(Integer... args);
void dyad(String name, Integer... arsg);
void triad(String name, int conut, Integer... args);
```
=> 가변인수를 취하는 모든 함수에 같은 원리가 적용된다. 가변 인수를 취하는 함수는 삼항 함수까지만 사용하자.

### 동사와 키워드

* 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야 한다. ex) `write(name)`, `writeField(name)`
* 키워드를 추가하자. `assertEquals` 보단 `assertEquals(expected: expected, actual: actual)` 이 더 좋다. 이러면 인수 순서를 기억할 필요가 없어진다.

## 부수 효과를 일으키지 마라! 

* 함수에서 한 가지를 하겠다고 약속하고 남몰래 다른 일을 하는 행위이다. 따라서 예측 불가능하다. 

```java
public class UserValidator {
    private Cryptographer cryptographer;
    
    public boolean checkPassword(String userName, String password) {
        User user = UserGateway.findByName(userName);
        if (user != User.NULL) {
            String codedPhrase = user.getPhraseEncodedByPassword();
            String phrase = cryptographer.decrypt(codedPhrase, password);
            if ("Valid Password".equals(phrase)) {
                Session.initialize();
                return true;
            }
        }
        return false;
    }
}
```
=> 여기서, 함수가 일으키는 부수 효과는 `Session.initialize()` 호출이다. 따라서 함수 이름만 보고 함수를 호출하는 사용자는 사용자를 인증하면서 **기존 세션 정보를 지워버릴 위험**에 처한다. 해당 checkPassword 함수는 세션을 초기화해도 괜찮은 경우에서만 가능하다. **자칫 잘못 호출하면 의도하지 않게 세션 정보가 날아간다. 즉 자신이 목적한대로 기능이 동작한것이 아니므로 버그다.**
<br>=> 즉 `checkPasswordAndInitializeSession`이라는 이름이 훨씬 좋다, 함수가 한 가지 역할을 한다는 규칙을 위반하지만 말이다.

### 출력 인수 

* 우리가 보통 인수를 입력 인수라고 생각하는데 출력 인수로 사용하는것 자체가 **부수 효과**다. 

일반적으로 우리는 인수를 함수 **입력**으로 해석한다. 따라서 인수를 **출력으로 사용하는 함수는 어색하고 오해하기 쉽다.**

```java
appendFooter(s);
```

```java
public void appendFooter(StringBuffer report);
```

=> 이 함수는 무언가에 s를 바닥글로 첨부할까? 아니면 s에 바닥글에 첨부할까? 인수 s는 입력일까 출력일까?
<br>=> 함수 선언부를 찾아보고 나서야 알게되지만, 함수 선언부를 찾아보는 행위는 코드를 보다가 주춤하는 행위와 동급이다. 즉 거슬리므로 생산성에 방해되므로 이렇게 작성하지 말아야 한다.

> 객체 지향 언어에서는 출력 인수를 사용할 필요가 거의 없다. 

출력 인수로 사용하라고 설계한 변수가 바로 this(프로퍼티) 이기 때문이다. 

```java
report.appendFooter()
```

=> 이런 식으로, 함수에서 상태가 변경해야 한다면 **함수가 속한 객체 상태를 변경하는 방식**을 택한다.
<br>=> 상위 객체의 동작이 아닌 하위 객체(본인)의 동작이 되도록 해라.

## 명령과 조회를 분리하라!

함수는 
  1. 뭔가를 수행하거나 
  2. 뭔가에 답하거나 
  3. 둘 중 하나만 해야한다. 

### 둘 다 하면 혼란을 초래한다. 예를 보자

```java
public boolean set(String attribute, String value);
```
=> 이 함수는 이름이 attribute인 속성을 찾아 값을 value로 설정한 후 성공하면 true를 반환하고 실패하면 false를 반환한다. value 설정하는 것을 수행하고 해당 수행에 대한 결과를 답한다. 둘 다 하는 함수다. 이 함수를 사용한 아래 코드를 보자.


```java
if (set("username", "unclebob"))...
```
=> if 문에 넣고 보면 형용사로 느껴진다. 그래서 if문은 "username 속성이 unclebob으로 설정되어 있다면....."으로 읽힌다. **"username을 unclebob으로 설정하는데 성공하면..." 으로 읽히지 않는다.** 해결책은 아래 코드와 같이 명령과 조회를 분리해 혼란을 애초에 뿌리 뽑는 방법이다. 

```java
if (attributeExists("username") {
    setAttribute("username", "unclebob");
}
```

## 오류코드보다 예외를 사용하라!

명령 함수에서 오류 코드를 반환하는 방식도 명령/조회 분리 규칙을 위반한다. 동사/형용사 혼란을 일으키지 않지만 오류 코드를 반환하는 방식의 문제는 **여러 단계로 중첩되는 코드를 야기한다는 점**이다. 
* 또 오류 코드를 반환하면 호출자는 **오류 코드를 곧바로 처리해야 한다는 문제**가 있다(코드가 더러워진다).

```java
if (deletePage(page) == E_OK)
```
=> 오류 코드를 반환하는 방식은 여러 단계로 중첩되는 코드를 야기한다. 

* 대신 Exception을 사용하고, try/catch 문을 사용하자. 여러 단계로 중첩되는 코드를 한방에 해결한다. 처리를 한 방에 모아준다. 오류 코드 대신 예외를 사용하면 **오류 처리 코드가 원래 코드에서 분리되므로 코드가 깔끔해진다.** 

```java
try { 
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
} catch (Exception e) {
    logger.log(e.getMessage());
}
```

### Try/Catch 블록 뽑아내기

* try/catch 블록은 원래 추하다. 따라서 아래처럼 try/catch 블록을 별도 함수로 뽑아내자. `deletePageAndAllReferences(Page page)`, `logError(Exception e)` 

```java
public void delete(Page page) {
    try {
        deletePageAndAllReferences(page);
    } catch (Exception e) {
        logError(e);
    }
}

private void deletePageAndAllReferences(Page page) throws Exception {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) {
    logger.log(e.getMessage());
}
```

### 오류 처리도 한가지 작업이다. 

함수는 `한 가지` 작업만 해야 한다. 오류 처리도 `한 가지` 작업에 속하므로 오류를 처리하는 함수는 오류만 처리해야 한다. 함수에 try가 있다면 catch/finally로 함수를 끝내야 한다. 그 아래에 코드를 작성하면 안된다!

### Error.java 의존성 자석

```java
public enum Error {
    OK,
    INVALID,
    NO_SUCH,
    LOCKED,
    OUT_OF_RESOURCES,
    WAITING_FOR_EVENT;
}
```
=> 위와 같은 클래스는 의존성 자석이다. Error enum이 변한다면 Error enum을 사용하는 클래스 전부를 다시 컴파일하고 다시 배치해야 한다. 그래서 Error 클래스 변경이 어려워진다. 따라서 오류 코드 대신 예외를 사용하자. **새 예외는 Exception에서 파생된다. 따라서 재컴파일/재배치 없이도 새 예외 클래스를 추가할 수 있다.**

## 반복하지 마라! 

## 함수를 어떻게 짜죠?

## 결론 
