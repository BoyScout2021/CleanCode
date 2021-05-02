# 주석 

## 개요

주석은 거짓말을 한다. **불행하게도 주석이 언제나 코드를 따라가지는 않는다. 아니 따라기지 못한다.** 주석이 코드에서 분리되어 점점 더 부정확한 도아로 변하는 사례가 너무도 흔하다. 

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

=> 위에 제시한 주석은 코드에서 사용한 **정규표현식이 시간과 날짜를 듯한다고 설명**한다.
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

### 주절거리는 주석
### 같은 이야기를 중복하는 주석 
### 오해할 여지가 있는 주석
### 의무적으로 다는 주석
### 이력을 기록하는 주석
### 있으나 마나 한 주석
### 무서운 잡음
### 함수나 변수로 표현할 수 있다면 주석을 달지 마라 
### 위치를 표시하는 주석
### 닫는 괄호에 다는 주석
### 공로를 돌리거나 저자를 표시하는 주석
### 주석으로 처리한 코드 
### HTML 주석
### 전역 정보
### 너무 많은 정보 
### 모호한 관계
### 함수 헤더
### 비공개 코드에서 Javadocs
### 예제
