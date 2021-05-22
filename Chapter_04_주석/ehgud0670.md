# 주석 

## 개요

주석은 거짓말을 한다. **불행하게도 주석이 언제나 코드를 따라가지는 않는다. 아니 따라기지 못한다.** 주석이 코드에서 분리되어 점점 더 부정확한 고아로 변하는 사례가 너무도 흔하다. 

## 주석은 나쁜 코드를 보완하지 못한다

코드에 주석을 추가하는 일반적인 이유는 **코드 품질이 나쁘기 때문**이다.
주석을 다는데 시간을 쏟지 말고, 코드 품질을 높이도록 하자!

## 코드로 의도를 표현하라!

> before 

```java
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다. 
if ((employee.flags & HOURLY_FLAG) && (employee.age > 65))
```

> after

```java
if (employee.isEligibleForFullBenefits())
```

=> 몇 초만 더 생각하면 **주석보다 코드로** 대다수 의도를 표현할 수 있다. 많은 경우 주석으로 달려는 설명을 함수로 만들어 표현해도 충분하다. 

## 좋은 주석

### 법적인 주석

**법적인 이유**로 특정 주석을 넣어야 하는 경우가 있다. 예를 들어, 각 소스 파일 첫머리에 주석으로 들어가는 저작권 정보와 소유권 정보는 필요하고도 타당하다.

```java
// Copyright (C) ... 이하 생략
```

### 정보를 제공하는 주석

때로는 기본적인 정보를 주석으로 제공하면 편리하다. 

```java
// kk:mm:ss EEE, MMM dd, yyyy 형식이다. 
Pattern timeMatcher = Pattern.compile(
    "\\d*:\\d*:\\d* \\w*, \\w* \\d*, \\d*");
)
```

=> 위에 제시한 주석은 코드에서 사용한 **정규표현식이 시간과 날짜를 뜻한다고 설명**한다.
물론 시각과 날짜를 변환하는 클래스를 만들어 코드를 옮겨주면 더 좋고 더 깔끔하겠다. 그러면 주석이 필요 없어진다.

### 의도를 설명하는 주석

주석은 구현을 이해하게 도와주는 선을 넘어 결정에 깔린 의도까지 설명한다. 이러한 주석은 코드를 작성한 글쓴이의 의도 및 결정을 알 수 있어 코드를 이해하는데 도움이 된다. 

```java
public void testConcurrentAddWidgets() throws Exception {
    WidgetBuilder widgetBuilder = new WidgetBuilder(new Class[] {BoldWidget.class});
    String text = "'''bold text'''";
    ParentWidget parent = new BoldWidget(new MockWidgetRoot(), "'''bold text'''");
    AtomicBoolean failFlag = new AtomicBoolean();
    failFlag.set(false);

    // 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다.
    for(int i = 0; i < 25000; i++) {
        WidgetBuilderThread widgetBuilderThread = new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
        Thread thread = new Thread(widgetBuilderThread);
        thread.start();
    }
    assertEquals(false, failFlag.get());
}
```

### 의도를 명료하게 밝히는 주석

일반적으로는 인수나 반환값 자체를 명확하게 만들면 더 좋겠지만, 인수나 반환값이 **표준 라이브러리나 변경하지 못하는 코드에 속한다면** 의미를 명료하게 밝히는 주석이 유용하다.

```java
assertTrue(a.compareTo(a) == 0); // a == a
assertTrue(a.compareTo(b) != 0); // a != b
assertTrue(a.compareTo(b) == -1); // a < b
```

### 결과를 경고하는 주석

다른 프로그래머에게 결과를 경고할 목적으로 주석을 사용한다. 
다음은 테스트하는데 시간이 너무 들어 특정 테스트 케이스를 꺼야 하는 이유를 설명하는 주석이다. 

```java
// 여유 시간이 충분하지 않다면 실행하지 마십시오.
```

### TODO 주석  

때로는 '앞으로 할 일'을 //TODO 주석으로 남겨두면 편하다. **하지만 TODO 주석을 어떤 용도로 사용하든 시스템에 나쁘 코드를 남겨 놓는 핑계가 되어서는 안된다.**

=> // TODO 를 나쁜 코드를 남겨놓은 핑계로 적지 말자. 오직 앞으로 할 일을 적어야 할 때만 사용하자.

### 중요성을 강조하는 주석

자칫 **대수롭지 않다고 여겨질** 뭔가의 중요성을 강조하기 위해서도 주석을 사용한다.

```java
String listItemContent = match.group(3).trim();
// 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
// 문자열은 시작 공백이 있으면 다른 문자열로 인식되기 때문이다.
new ListItemWidgets(this, listItemContent, this.level + 1);
return buildList(text.substring(match.end()));
```

### 공개 API에서 Javadocs 

공개 API를 구현한다면 반드시 훌륭한 Javadocs를 작성한다. 

## 나쁜 주석

대다수 주석이 이 범주에 속한다. ㅠㅠ

### 주절거리는 주석

더 구체적인 내용 및 설명이 없어서 필요없어진 주석을 말한다. 답을 알아내려면 다른 코드를 뒤져보는 수밖에 없다. 이러한 주석은 바이트만 낭비할 뿐이다.

```java
// 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였다는 의미다. 
```
=> 누가 모든 기본값을 읽어 들이는가? `loadProperties.load` 를 호출하기 전에 읽어 들이는가? 아니면 `loadProperties.load`가 예외를 잡아 기본값을 읽어 들인 후 예외를 던져주는가? 아니면 `loadProperties.load` 가 파일을 읽어 들이기 전에 모든 기본값부터 읽어들이는가? 충분한 설명이 없어 해당 주석은 독자에게 도움이 되지 않고 오히려 궁금중만 불러 일으킨다.  

### 같은 이야기를 중복하는 주석 && 오해할 여지가 있는 주석

코드 내용을 그대로 중복한 주석은 때문에 자칫하면 코드보다 주석을 읽는 시간이 더 오래 걸린다. 
```java
// this.closed 가 true일 때 반환되는 유틸리티 메서드다. 
// 타임아웃에 도달하면 예외를 던진다. 
public synchronized void waitForClose(final long timeoutMillis) throws Exception {
    if(!closed) {
        wait(timeoutMillis);
        if(!closed) {
            throw new Exception("MockResponseSender could not be closed");
        }
    }
}
```
=> 주석이 코드보다 읽기도 쉽지도 않다. 실제로 코드보다 부정확해 **독자가 함수를 대충 이해하고 넘어가게 만든다.** 
<br>=> 또 위의 코드는 오해할 여지가 있는 주석이다. `this.closed` 가 true 로 변하는 순간에 메서드는 반환되지 않는다. `this.closed` 가 **true 여야** 메서드는 반환된다. 또 타임아웃을 기다리면 무작정 예외를 던지는 것도 아니다. 타임아웃이 지나고, **그래도 `this.closed`가 true가 아니면** 예외를 던진다. 이처럼, 해당 주석들은 내용이 정확하지 않아 코드를 오해할 여지가 있다.

### 의무적으로 다는 주석

의무적으로 다는 주석은 오히려 코드만 헷갈리게 만들며, 거짓말할 가능성을 높이며, 잘못된 정보를 제공할 여지만 만든다. 

### 이력을 기록하는 주석

소스 코드 관리 시스템이 있으므로 이력을 기록하는 주석을 남기지 말자. 오히려 소스 코드 관리 시스템과 내용이 다르면 혼란만 가중할 뿐이다. 완전히 제거하는 편이 좋다.

### 있으나 마나 한 주석

주석으로 분풀이를 표현하지 말고 리팩토링하자.

### 함수나 변수로 표현할 수 있다면 주석을 달지 마라 

* 주석 대신 코드로 의도를 표현해라!

> before 

```java
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다. 
if ((employee.flags & HOURLY_FLAG) && (employee.age > 65))
```

> after

```java
if (employee.isEligibleForFullBenefits())
```

### 위치를 표시하는 주석

```java
// Actions ////////////////////////////////////////
```
=> 너무 자주 사용하지 않는다면 배너는 눈에 띄며 주의를 환기해주는 유용한 주석이다. 그러므로 반드시 필요할 때만, 아주 드물게 사용하는 편이 좋다. 
<br>=> **하지만 배너를 남용하면 독자가 흔한 잡음으로 여겨 무시한다.**

### 닫는 괄호에 다는 주석

닫는 괄호에 주석을 달아야겠다는 생각이 든다면 대신 함수를 줄이려 시도하자.

* 아 너무 공감된다. 내가 봤던 5000줄 짜리 코드에 각 루프문, if 문, try문을 구분하기 위해 끝에 주석을 남긴다. 이러한 방식은 나중에 코드를 고칠때도 방해가 된다. 고치다가 주석 때문에 해당 줄의 코드가 주석화 되는 경우도 허다하기 때문이다(개빡침).
함수를 잘게 나누어 이런 주석을 달지 말자. 

### 공로를 돌리거나 저자를 표시하는 주석

```java
/* 릭이 추가함 */
```
=> 소스 코드 관리 시스템으로 확인이 가능하므로 해당 내용의 주석을 달지 말자. 

### 주석으로 처리한 코드 

소스 코드 관리 시스템으로 이전 코드들이 저장되므로 **절대 코드를 주석처리 하지말자.**
실수로 주석을 풀어 메소드가 깨어나서 동작하는 일이 훨씬 위험하다.

### 모호한 관계

주석과 주석이 설명하는 코드는 둘 사이 관계가 명백해야 한다.

```java
/*
 * 모든 픽셀을 담을 만큼 충분한 배열로 시작한다(여기에 필더 바이트를 더한다).
 * 그리고 헤더 정보를 위해 200바이트를 더한다. 
 */
this.pngBytes = new byte[(this.width + 1) * (this.height * 3) + 200];
```
* 여기서 필터 바이트란 무엇일까? +1과 관련이 있을까? 아니면 *3과 관련이 있을까? 아니면 둘 다?
* 한 픽셀이 한 바이트인가? 
* 200을 추가하는 이유는?

=> 주석을 다는 목적은 (낮은 품질의)코드만으로 설명이 부족해서다. 코드로 다 설명이 되도록 리팩토링하자.

### 예제

리팩터링 버전 코드에서 `determineIterationLimit()` 메소드가 잘못되었습니다.
`double iterationLimit = Math.sqrt(crossedOut.length);` 가 아닌 `double iterationLimit = Math.sqrt(crossedOut.length) + 1.0;`이 되어야 합니다. 
