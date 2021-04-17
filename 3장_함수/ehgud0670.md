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

### 많이 쓰는 단항 형식 

### 플래그 인수 

### 이항 함수 

### 삼항 함수 


### 인수 객체 

```java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```

### 인수 목록 

```java
String.format("%s worked %.2f hours.", name, hours);
```

```java
public String format(String format, Object... args);
```

### 동사와 키워드 

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
