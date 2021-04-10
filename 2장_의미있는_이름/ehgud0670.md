# 2장 의미 있는 이름 

## 개요

소프트웨어에서 이름은 **어디나 쓰인다.** 우리는 변수, 함수, 인수, 클래스, 패키지 이 모든 것에 이름을 붙인다. 소스 파일, 디렉터리, jar 파일, war 파일, ear 파일에도 이름을 붙인다. 이렇게 모든 것에 이름에 붙여지므로 의미 있는 이름을 붙여야 한다. **왜? 결국 독자(개발자)가 읽는 것이고 이해가 잘 되면 일의 능률이 오르니깐 말이다.**

## 의도를 분명히 밝혀라

축약어를 사용하지 말고 의도가 드러나게 변수 이름을 지어라. 
다른 사람이 보기에도 의도가 분명한 변수 및 메서드 이름이라면 
그렇지 않은 경우보다 훨씬 생산성이 증가하고 심적으로도 편안하다.
**따라서 이름을 주의 깊게 샆여 더 나은 이름이 떠오르면 개선하자.**

```swift
var d: Int // 경과 시간(단위: 날짜)
```
=> days?, duration? d 가 무엇을 정확히 모른다. 경과 시간이나 날짜라는 느낌이 안든다. **따로 주석이 필요하다면 의도를 분명히 드러내지 못했다는 말이다.**

```swift
var elapsdTimeInDays: Int
var daysSinceCreation: Int
var daysSinceModification: Int
var fileAgeInDays: Int
```
=> 반면 위의 변수들은 경과시간, 생성 이후의 일수, 변경 이후의 일수, 파일 사용 기간라는 이름으로 명확히 의도를 밝히므로 독자(또 다른 개발자)가 읽기 편하다. 
<br>**의도가 드러나는 이름을 사용하면 코드 이해와 변경이 편해진다.**

> 의도가 드러나지 않은 메소드 

```swift
func getThem() -> [[Int]] {
    var list1 = [[Int]]()
    for x in self.theList {
        if x[0] == 4 {
            list1.append(x)
        }
    }
    return list1
}
```
=> 메소드가 단순하지만 무슨 일을 하는지 짐작하기 어렵다. 왜일까? **변수, 메소드의 이름이 자신의 역할을 드러내지 않는 의미가 없는 이름들이기 때문이다.**

* theList에 무엇이 들었는가? 
* theList에서 0번째 값이 어째서 중요한가?
* 값 4는 무슨 의미인가?
* 함수가 반환하는 리스트 list1을 어떻게 사용하는가?
<br>=> 위의 코드를 보고 떠오르는 질문들...

`getThem`, `list1`, `theList`, `x`, `0`, `4`: 이름이 정말 끔찍하다. 정도의 차이가 있지만 이러한 변수명들은 실제로 분명히 있고 우리도 비슷하게 의도가 드러나지 않게 작성한 경우가 있을 것이고 있다. 따라서 조심하고 지금이라도 자신이 작성한 변수명, 메소드명을 더 의도가 드러나게 수정하자(상수를 선언하거나 경우의 수가 정해진 경우 enum을 고려하자!).

> 의도가 드러난, 확실한! 메소드 

```swift
func getFlaggedCells() -> [[Int]] {
    var flaggedCells = [[Int]]()
    for cell in self.gameBoard {
        if cell[Self.STATUS_VALUE] == Self.FLAGGED {
            flaggedCells.append(cell)
        }
    }
    return flaggedCells
}
```
* 지뢰찾기 게임을 만든다고 가정하자. 그럼 theList가 게임판이라는 사실을 알고 있으므로, theList를 gameBoard로 바꿔보자. 
* 게임판에서 각 칸은 단순 배열로 표현한다. 배열 index 0번째 값은 칸 상태를 뜻한다. 값 4는 깃발이 꽃힌 상태를 가리킨다. **각 값을 상수로 빼서 이름만 붙여도** 코드가 상당히 나아진다. 

=> 코드의 단순성은 변하지 않았다. 연산자 수와 상수 수는 앞의 예와 똑같으며, 들여쓰기 단계도 동일하다. 그런데도! 코드는 **더욱 명확**해졌다(거의 마법이야, 마법).

```swift
func getFlaggedCells() -> [Cell] {
    var flaggedCells = [Cell]()
    for cell in self.gameBoard {
        if cell.isFlagged {
            flaggedCells.append(cell)
        }
    }
    return flaggedCells
}
```
=> 한 단계 더 나아가, int 배열을 사용하는 대신, **칸을 간단한 클래스(대단하다!)** 로 만들어도 되겠다. `isFlagged` 라는 좀 더 명시적인 함수를 사용해 FLAGGED 라는 **상수를 감춰도** 되겠다. 

=> 단순히 이름만 고쳤는데도 함수가 하는 일을 이해하기 쉬워졌다. 이것이 **좋은 이름이 주는 (엄청난) 위력**이다. 

## 그릇된 정보를 피하라

## 의미 있게 구분하라

## 발음하기 쉬운 이름을 사용하라 

## 검색하기 쉬운 이름을 사용하라

## 인코딩을 피하라 

## 자신의 기억력을 자랑하지 마라 

## 클래스 이름 

## 메서드 이름 

## 기발한 이름은 피하라

## 한 개념에 한 단어를 사용하라 

## 말장난을 하지 마라

## 해법 영역에서 가져온 이름을 사용하라 

기술 개념에는 기술 이름을 붙이는 게 가장 적합한 선택이다.  
모든 이름을 문제 영역(domain)에서 가져오는 정책은 현명하지 못하다.
같은 이름(도메인용으로 사용되는 이름)을 다른 이름(기술용어로 사용되는 이름)으로 오해하던 동료들이 매번 물어봐야 하기 때문이다.
따라서 부담 갖지말고 코드를 읽는 사람도 프로그래머라는 사실을 명심하고, 전산용어, 알고리즘 이름, 패턴 이름, 수학 용어등을 사용하자(해도 괜찮다).

전산용어를 사용한 에:
* VISTOR 패턴에서 따온 `AccountVisitor`
* JobQueue 개념에서 따온 `enqueue(job: Job())`

내 생각: 기술 용어와 중복되지는 않지만 다른 예에서도 도메인이름을 사용하는 것이 적절하지 못한 경우가 있다. 
예를 들어 애니메이션 되는 뷰가 있다고 해보자. 해당 뷰의 아이콘 이미지가 토끼라고 변수명이나 클래스명을 `Rabbit`으로 짓는건 현명하지 못하다.
나중에 거북이로 아이콘이 바뀌면 `Turtle`이라고 이름을 바꿀 것인가? 
애니메이팅 되고 있는 뷰의 이름은 `AnimationView`가 적절할 것이다. 

## 문제 영역에서 가져온 이름을 사용하라 

적절한 `프로그래밍 용어`가 없다면 문제 영역(domain)에서 이름을 가져온다. 그러면 코드를 보수하는 프로그래머가 도메인 전문가에게 의미를 물어 파악할 수 있다.   

## 의미 있는 맥락을 추가하라 

스스로 의미가 분명한 이름도 있다. 하지만 대다수 이름은 그렇지 못하고 주변의 맥락을 파악해야 의미를 알아차리는 경우다.
**그래서 클래스, 함수, 이름 공간에 넣어 맥락을 부여한다. 모든 방법이 실패하면 마지막 수단으로 접두어를 붙인다.**
예를 들어, firstName, lastName, street, houseNumber, city, state, zipcode라는 변수가 있다.
변수를 훑어보면 주소라는 사실을 금방 알아챈다. 하지만 어느 메서드가 state(주)라는 변수하나만 사용하면 state(상태)와 헷갈릴 가능성이 농후하다.

**addr라는 접두어를 추가해** addrFirstName, addrLastName, addrState라 쓰면 맥락이 좀 더 분명해진다.
변수가 좀 더 큰 구조에 속하다는 사실이 적어도 독자(개발자)에게는 분명해진다.
또 **Address라는 클래스를 생성하면 더 좋다.** 그러면 변수가 좀 더 큰 개념에 속하다는 사실이 **컴파일러**에게도 분명해진다(가독성을 위해서라도 응집성을 높이자). 

> 맥락이 불분명한 변수 

```swift
private func printGuessStatistics(candidate: Character, count: Int) {
    var number: String
    var verb: String
    var pluralModifier: String
    
    if(count == 0) {
        number = "no"
        verb = "are"
        pluralModifier = "s"
    } else if (count == 1) {
        number = "no"
        verb = "are"
        pluralModifier = "s"
    } else {
        number = "no"
        verb = "are"
        pluralModifier = "s"
    }
    
    let guessMessage = "There \(verb) \(number) \(candidate) \(pluralModifier)"
    print(guessMessage)
}
```

> 맥락이 분명한 변수 

```swift
public class GuessStatisticsMessage {
    private var number: String
    private var verb: String
    private var pluralModifier: String
    
    init() {
        self.number = ""
        self.verb = ""
        self.pluralModifier = ""
    }
    
    private func createPluralDependentMessageParts(count: Int) {
        if(count == 0) {
            thereAreNoLetters()
        } else if(count == 1) {
            thereIsOneLetter()
        } else {
            thereAreManyLetters(count: count)
        }
    }
    
    private func thereAreNoLetters() {
        self.number = "no"
        self.verb = "are"
        self.pluralModifier = "s"
    }
    
    private func thereIsOneLetter() {
        self.number = "1"
        self.verb = "is"
        self.pluralModifier = "s"

    }
    
    private func thereAreManyLetters(count: Int) {
        self.number = "\(count)"
        self.verb = "are"
        self.pluralModifier = "s"
    }
}
```

## 불필요한 맥락을 없애라

일반적으로 짧은 이름이 긴 이름보다 좋다. **단, 의미가 분명한 경우에 한해서다(즉, 의미가 분명하지 않은 대부분의 경우에는 긴 이름이 좋다).**
accountAddress와 customerAddress 는 Address 클래스 인스턴스로는 좋은 이름이나 클래스 이름으로는 적합하지 못하다.
Address는 클래스 이름으로 적합하다. 포트 주소, MAC 주소, 웹 주소를 구분해야 한다면 PostalAddress, MAC, URI라는 이름도 괜찮겠다. 

* 불필요한 맥락이 포함된 경우들 
  * 고급 휘발유 충전소(Gas Station Deluxe)라는 애플리케이션을 짠다고 모든 클래스 이름을 GSD라고 시작하는 경우 
  <br>=> 진짜 끔찍하다. IDE에서 클래스 이름을 검색하려고 G를 누르면 모오든 클래스 이름이 열거될 것이기 때문이다. 보통 회사 이름을 줄이거나 도메인 이름을 약자로 접두사로 클래스 이름에 추가하던데 그러지 말자. 모두가 고생한다. 

## 마치면서 

여느 코드 개선 노력과 마찬가지로 이름 역시 나름대로 바꿨다가는 누군가 질책할지도 모른다. **그렇다고 코드를 개선하려는 노력을 중단해서는 안 된다.** 우리는 프로그래머고 유지보수를 위해 높은 코드 품질을 지향해야 한다. 이름 잘 짓자!