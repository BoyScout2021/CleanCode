# 3장. 함수 

## 개요 

* 어떤 프로그램이든 가장 기본적인 단위가 **함수**다.

## 작게 만들어라!

```java
public static String renderPageWithSetupsAndTeardowns(PageData pageData, boolean isSuite) throws Exception {
    if (isTestPage(pageData)) {
        includeSetupAndTeardownPages(pageData, isSuite);
    }
    return pageData.getHtml();
}
```

## 한 가지만 해라!

## 함수 당 추상화 수준은 하나로!

### 위에서 아래로 코드 읽기: 내려가기 규칙

## Switch 문 

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

```java
public abstract class Employee {
    public abstract Money calculatePay();
}

public interface EmployeeFactory {
    public Employee makeEmployee(EmployeeRecord e) throws InvalidEmployeeType;
}
```


## 서술적인 이름을 사용하라! 

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

### 출력 인수 

```java
appendFooter(s);
```

```java
public void appendFooter(StringBuffer report);
```

## 명령과 조회를 분리하라!

```java
public boolean set(String attribute, String value);
```

```java
if (set("username", "unclebob"))...
```

```java
if (attributeExists("username") {
    setAttribute("username", "unclebob");
}
```

## 오류코드보다 예외를 사용하라!

```java
if (deletePage(page) == E_OK)
```

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

## 반복하지 마라! 

## 함수를 어떻게 짜죠?

## 결론 
