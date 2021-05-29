# 단위 테스트 

## 테스트는 유연성, 유지보수성, 재사용성을 제공한다.

코드에 유연성, 유지보수성, 재사용성을 제공하는 버팀목이 바로 **단위 테스트**다. 이유는 간단한다. 테스트 케이스가 있으면 변경이 두렵지 않기 때문이다. 변경을 하는 중에 틀린 부분이 있으면 단위 테스트가 해당 로직은 틀렸다고 매번 알려주기 때문이다. 마치 QA가 여러 기능을 검증하는 것처럼 테스트 코드가 틀렸다고 **계속 피드백을 주니깐** 너무 믿음직하고, 두려운 마음이 사라지기 때문이다.

## 깨끗한 테스트 코드 

깨끗한 테스트 코드는 가독성이 매우 중요하다. 가독성은 실제 코드보다 테스트 코드에 더욱 중요할 수 있다. 
**테스트 코드는 최소의 표현으로 많은 것을 나타내야 한다.**

* 여느 코드와 마찬가지로 테스트 코드도 명료성, 단순성, 풍부한 표현력이 필요하다.

### 리팩토링 하기 전의 테스트 코드 
```java
public void testGetPageHierarchyAsXml() throws Exception {
    crawler.addPage(root, PathParser.parse("PageOne"));
    crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
    crawler.addPage(root, PathParser.parse("PageTwo"));

    request.setResource("root");
    request.addInput("type", "pages");
    Responder responder = new SerializedPageResponder();
    SimpleResponse response =
                (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
    String xml = response.getContent();

    assertEquals("test/xml", response.getContentType());
    assertSubString("<name>PageOne</name>", xml);
    assertSubString("<name>PageTwo</name>", xml);
    assertSubString("<name>ChildOne</name>", xml);
}

public void testGetPageHierarchyAsXmlDoesntContainSymbolicLinks() throws Exception {
    WikiPage pageOne = crawler.addPage(root, PathParser.parse("PageOne"));
    crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
    crawler.addPage(root, PathParser.parse("PageTwo"));

    PageData data = pageOne.getData();
    WikiPageProperties properties = data.getProperties();
    WikiPageProperty symLinks = properties.set(SymbolicPage.PROPERTY_NAME);
    symLinks.set("SymPage", "PageTwo");
    pageOne.commit(data);
        
    request.setResource("root");
    request.addInput("type", "pages");
    Responder responder = new SerializedPageResponder();
    SimpleResponse response =
                (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
    String xml = response.getContent();

    assertEquals("test/xml", response.getContentType());
    assertSubString("<name>PageOne</name>", xml);
    assertSubString("<name>PageTwo</name>", xml);
    assertSubString("<name>ChildOne</name>", xml);
    assertNotSubString("SymPage", xml);
}

public void testGetDataAsHtml() throws Exception {
    crawler.addPage(root, PathParser.parse("TestPageOne"), "test page");
        
    request.setResource("TestPageOne");
    request.addInput("type", "data");
    Responder responder = new SerializedPageResponder();
    SimpleResponse response =
                (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
    String xml = response.getContent();

    assertEquals("test/xml", response.getContentType());
    assertSubString("test page", xml);
    assertSubString("<Test", xml)
}
```
* crawler 쪽 코드는 테스트 코드와 무관하며 테스트 코드의 의도만 흐린다. 
* responder를 생성하는 코드, response를 수집해 변환하는 코드 모두 잡음에 불과하다. 
* resource와 인수에서 요청 URL을 만드는 어설픈 코드(`request.setResource("TestPageOne");
    request.addInput("type", "data");`)도 있다.

=> **이러한 코드는 읽는 사람을 고려하지 않는다.** 불쌍한 독자들은 온갖잡다하고 무관한 코드를 이해한 후라야 간단한 테스트를 이해한다.

### 목록 9-2: 리팩토링한 테스트 코드  
```java
public void testGetPageHierarchyAsXml() throws Exception {
    makePages("PageOne", "PageOne.ChildOne", "PageTwo");

    submitRequest("root", "type:pages");

    assertResponseIsXML();
    assertResponseContains("<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
}

public void testGetPageHierarchyAsXmlDoesntContainSymbolicLinks() throws Exception {
    WikiPage pageOne = makePage("PageOne");
    makePages("PageOne.ChildOne", "PageTwo");

    addLinkTo(page, "PageTwo", "SymPage");

    submitRequest("root", "type:pages");

    assertResponseIsXML();
    assertResponseContains("<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
    assertReponseDoseNotContain("SymPage");
}

public void testGetDataAsHtml() throws Exception {
    makePageWithContent("TestPageOne", "test page");

    submitRequest("TestPageOne", "type:data");

    assertResponseIsXML();
    assertResponseContains("test page", "<Test");
}
```
* BUILD-OPERATE-CHECK 패턴이 위와 같은 테스트 구조에 적합하다. 
  * 첫 부분은 테스트 자료를 만든다. (BUILD)  `makePages`, `addLinkTo`
  * 두 번째 부분은 테스트 자료를 조작한다. (OPERATE) `submitRequest`
  * 세 번째 부분은 조작한 결과가 올바른지 확인한다 (CHECK) `assertResponseIsXML`

=> 잡다고 세세한 코드를 거의 다 없앴다는 사실에 주목하자. **테스트 코드는 본론에 돌입해 진짜 필요한 자료 유형과 함수만 사용한다.**

> 도메인에 특화된 테스트 언어 

목록 9-2는 도메인에 특화된 언어(DSL)로 테스트 코드를 구현하는 기법을 보여준다. 흔히 쓰는 시스템 조작 API를 사용하는 대신 **API 위에다 함수와 유틸리티를 구현한 후 그 함수와 유틸리티를 사용한다.**

* 목록 9-2 중 도메인에 특화된 테스트 코드 예 
  * `assertResponseIsXML` => 기존 JUnit API의 assertEquals를 사용하는 것을 넘어 Response가 XML인지 검사하는 도메인에 특화된 테스트 코드로 발전함
  * `assertResponseContains` => 기존 JUnit API의 assertSubString을 사용하는 것을 넘어 Response가 특정 스트링값들을 포함하는지 검사하는 도메인에 특화된 테스트 코드로 발전함

## 이중 표준 

테스트 코드는 단순하고, 간결하고, 표현력이 풍부해야 하지만, **실제 코드만큼 효율적일 필요는 없다.**
실제 환경이 아니라 **테스트 환경에서 돌아가는 코드이기 때문**이다.

### 리팩터링 하기 전 코드 
```java
@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
    hw.setTemp(WAY_TOO_COLD);
    controller.tic();
    assertTrue(hw.heaterState());
    assertTrue(hw.blowerState());
    assertFalse(hw.coolerState());
    assertFalse(hw.hiTempAlarm());
    assertTrue(hw.loTempAlarm());
}
```
=> 위 코드는 점검하는 상태 이름과 상태 값을 확인하느라 눈길이 이리저리 흩어진다. **heaterState라는 상태를 보고서는 왼쪽으로 눈길을 돌려 assertTrue를 읽는다(아래도 똑같이 행동 반복).**  따분하고 미덥잖다. 테스트 코드를 읽기가 어렵다. 

### 리팩터링 한 후 코드 
```java
@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
    wayTooCold();
    assertEquals("HBchL", hw.getState());
}
```
=> 대문자는 켜짐(on)이고, 소문자는 꺼짐(off)을 뜻한다. 문자는 항상 {heater, blower, cooler, hi-temp-alarm, lo-temp-alarm} 순서다. 
<br>=> 그릇된 정보를 피하라의 규칙 위반에 가깝지만 여기서는 적절해 보인다. **일단 의미만 안다면 눈길이 문자열을 따라 움직이며 결과를 재빨리 판단**한다.
<br>=> 빠르게 판단되고 이해하기 쉽다는 면에서 가독성이 높은 코드이다.  

* hw 변수의 getState 메소드는 실제 프로덕션 객체의 메소드가 아닌 MockControlHardware의 메소드이다. 위 코드처럼 읽기 쉬운 테스트를 하기 위해 가짜 객체에 만든 메소드다. 

> MockControlHardware.java
```java
public String getState() {
    String state = "";
    state += heater ? "H" : "h";
    state += blower ? "B" : "b";
    state += cooler ? "C" : "c";
    state += hiTempAlarm ? "H" : "h";
    state += loTempAlarm ? "L" : "l";
    return state;
}
```
=> 이렇게 StringBuffer를 사용하지 않고 직접 스트링을 연결하는 것은 메모리에 효율적이지 않다.

```swift
public String getState() {
    StringBuffer stateBuffer = new StringBuffer();
    if (heater) {
        stateBuffer.append("H");
    } else {
        stateBuffer.append("h");
    }

    if (blower) {
        stateBuffer.append("B");
    } else {
        stateBuffer.append("b");
    }

    if (cooler) {
        stateBuffer.append("C");
    } else {
        stateBuffer.append("c");
    }
    
    if (hiTempAlarm) {
        stateBuffer.append("H");
    } else {
        stateBuffer.append("h");
    }
    
    if (loTempAlarm) {
        stateBuffer.append("L");
    } else {
        stateBuffer.append("l");
    }

    return stateBuffer.toString();
}
```
=> 하지만 StringBuffer를 사용하면 위 예시처럼 코드가 직관적이지 않다(삼항 연산자는 제 판단에 위 사항에선 사용하지 못합니다. if 로 분기처리 할 수밖에 없습니다).

* 종합하자면, StringBuffer를 사용 안하면 테스트 코드가 직관적이지만 메모리에 효율적이지 않고, StringBuffer를 사용하면 메모리에 효율적이지만 테스트 코드가 직관적이지 않다. 우린 무엇을 택해야 할까?
* 저자는 StringBuffer를 택하지 않고 단순히 문자열을 연결하는 방식을 권한다. 왜냐하면 테스트 코드, 테스트 환경은 자원이 제한적일 가능성이 낮기 때문이다. 이것이 **이중 표준의 본질**이다. 실제 환경에서는 절대로 안 되지만, **테스트 환경에서는 전혀 문제없는 방식이 있다.** 대개 메모리나 CPU 효율과 관련 있는 경우다. 코드의 깨끗함과는 철처히 무관하다. 

## 테스트 당 assert 하나 

* 테스트 코드를 짤 때는 메소드 당 assert문 단 하나만을 사용해야 한다는 학파가 있다. **이유는 결론이 하나라서 코드를 이해하기 쉽고 빠르기 때문이다.**
* 따라서 저자는 아래처럼 한 개의 테스트 메소드를 두 개의 메소드로 나눴는데 확실히 테스트 당 assert 하나이다. 하지만 테스트를 분리하면 **중복되는 코드가 많아진다는 단점**이 있다. TEMPLATE METHOD 패턴을 사용하면 중복을 제거할 수 있지만 배보다 배꼽이 더 큰 방법이고, 따라서 저자는 코드가 중복되는 것 보다는 assert 문을 여럿 사용하는 편이 좋다고 생각한다고 한다. 
* 결론적으로 assert문 개수는 최대한 줄여한 좋다는 생각이다. 

```java
public void testGetPageHierarchyAsXml() throws Exception {
    givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
    
    whenRequestIsIssued("root", "type:pages");
    
    thenResponseShouldBeXML();
}

public void testGetPageHierarchyHasRightTags() throws Exception {
    givenPages("PageOne", "PageOne.ChildOne", "PageTwo");
    
    whenRequestIsIssued("root", "type:pages");
    
    thenResponseShouldContain(
            "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>"
    );
}
```

=> given-when-then 이라는 관례를 사용했다. 코드가 그 이전보다 한층 더 읽기 쉬워졌다. 하지만 중복 코드가 많아진다는 단점도 생긴다. 

## 테스트 당 개념 하나 

`테스트 함수마다 한 개념만 테스트하라`라는 규칙이 더 어울린다. 따라서 아래 코드는 바람직하지 못한 테스트 함수다. 독자적인 개념 세 개를 테스트하므로 **독자적인 테스트 세 개로 쪼개야** 마땅하다.

```java
public void testAddMonths() {
    SerialDate d1 = SerialDate.createInstance(31, 5, 2004);

    SerialDate d2 = SerialDate.addMonths(1, d1);
    assertEquals(30, d2.getDayOfMonth());
    assertEquals(6, d2.getMonth());
    assertEquals(2004, d2.getYYYY());

    SerialDate d3 = SerialDate.addMonths(2, d1);
    assertEquals(31, d3.getDayOfMonth());
    assertEquals(7, d3.getMonth());
    assertEquals(2004, d3.getYYYY());
    
    SerialDate d4 = SerialDate.addMonths(1, SerialDate.addMonths(1, d1));
    assertEquals(30, d4.getDayOfMonth());
    assertEquals(7, d4.getMonth());
    assertEquals(2004, d4.getYYYY());
}
```

## F.I.R.S.T.

* Fast(빠르게)
* Independent(독립적으로)
* Repeatable(반복가능하게)
* Self-Validating(자가검증하는)
* Timely(적시에)

=> 이 중에서 `독립적으로`가 제일 공감된다. 예전에 어떤 회사 동료분이 테스트 케이스를 확보할 때 테스트 하는데 사용하는 라이브러리 때문에 각 테스트가 의존되게끔 작성하였다. 이 의존성 때문에 결과를 예측하기 어려웠고 심지어 런타임 에러가 발생했었다. 하지만 라이브러리를 사용하면서도 충분히 메소드 간에 독립적으로 사용할 수 있었다. 아니면 또 다른 해결방법이 있을터였다. 하지만 그는 부주의하게 테스트 코드를 짰고 내가 의문을 제기했어도 고치려 들지 않았다. 
<br>=> 저자가 말한것처럼 이러한 테스트 케이스는 쓰레기에 불과하다는 말에 공감한다.  
