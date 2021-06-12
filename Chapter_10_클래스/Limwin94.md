# 10장 클래스

## 클래스 체계
표준 자바 관례에 따라서 설명을 하고 있는데,, `Swift Style Guide`에서는 

> The order of types, variables, and functions in a source file, and the order of the members of those types, can have a great effect on readability. However, there is no single correct recipe for how to do it; different files and different types may order their contents in different ways.

소스 파일의 유형, 변수 및 함수의 순서와 해당 유형의 멤버 순서는 가독성에 큰 영향을 미칠 수 있습니다. **그러나 올바른 하나의 방법은 없습니다.** 다른 파일들과 다른 타입들은 다른 방식들로 내용을 정렬 할 수 있습니다.

그리고 밑에서는 위와 같은 구성원들의 논리적 순서를 결정할때 `MARK`주석을 사용해서 해당 그룹에 대한 설명을 제공하는것도 권장하고 있었습니다.

```swift
class MovieRatingViewController: UITableViewController {

  // MARK: - View controller lifecycle methods

  override func viewDidLoad() {
    // ...
  }

  override func viewWillAppear(_ animated: Bool) {
    // ...
  }

  // MARK: - Movie rating manipulation methods

  @objc private func ratingStarWasTapped(_ sender: UIButton?) {
    // ...
  }

  @objc private func criticReviewWasTapped(_ sender: UIButton?) {
    // ...
  }
}
```

책에서 이야기하는 요지는 추상화 단계가 아래로 내려가도록 배치하면 신문 기사마냥 읽힌다는 것이라고 생각하면 되겠네요.

### 캡슐화
변수와 함수는 가능한 공개하지 않는 편이 좋지만,, 테스트를 위해서 패키지내에서만 동작하도록 protected로 선언 할 수도 있다. <br>
-> `private(set)`같은 방식을 고려해봅시다.

<br>

## 클래스는 작아야 한다.
함수와 마찬가지로 클래스 또한 최대한 **작게** 만들어야한다.
클래스에 대한 규모는 **책임**으로 측정한다.

클래스 이름은 해당 클래스 책임을 기술해야 한다. 네이밍부터가 크기를 줄이는 첫번째 관문이다.

간결한 이름이 떠오르지 않는다면 책임이 너무 큰 클래스라는것..

### 단일 책임 원칙(SRP)
= 클래스나 모듈을 변경할 이유가 단 하나뿐이어야 한다.

그래서 변경할 이유를 파악하려 하다 보면 추상화하기도 쉬워지게 된다.
(그 다른 이유들을 묶어 따로 빼면 다른 하나의 클래스가 나올 수 있다.)

다목적 클래스 몇 개로 이루어진 시스템은 수정이 일어날 때 너무 많은 것을 봐야한다.
작은 클래스 여럿이 서로 협력하것이 더 좋은 시스템이다.

+) 이전에 조영호님의 객체지향 강의를 들을 기회가 있었는데요.<br>
`책임 주도 개발`에 관해서 설명을 해주셨는데 객체에 대해 다시 생각할 수 있는 좋은 시간이었습니다. 여기서 설명하기에는 너무 길어서 인터넷에 찾아보시거나 조영호님의 저서를 읽어보시면 될 것 같습니다 :)

### 응집도
클래스는 인스턴스 변수 수가 적어야 한다.
클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야 한다.

메서드가 변수를 많이 사용할수록 메서드와 클래스는 응집도가 높아진다.

근데 몇몇 메서드만이 사용하는 인스턴스 변수가 생기면 -> 새로운 클래스로 쪼개기가 가능하다.

### 응집도를 유지하면 작은 클래스 여럿이 나온다.
큰 함수를 작은 함수로 나누다 보면 (그리고 이 함수들이 몇몇 변수만 사용한다면) 작은 클래스 여럿으로 분리할 수 도 있다.

<br>

## 변경하기 쉬운 클래스

### 변경으로부터 격리
상세한 구현에 의존하는 클래스는 구현이 바뀌면 위험에 빠지므로
-> `protocol`을 사용해 구현이 미치는 영향을 격리할 수 있다.

또 구체화된 클래스는 테스트가 어렵다. <br> protocol과 같이 추상화를 해놓으면 실제 서비스 되고 있는 클래스를 흉내내는 테스트용 클래스를 만들 수 있게 된다. (테스트용 클래스는 고정된 결과값을 내놓는다던지..)

이렇게 결합도를 낮추면
- 유연성과 재사용성 확보
- 각 시스템 요소가 다른 요소로부터 잘 격리
- 각 시스템 요소를 이해하기 쉬워짐
- DIP 원칙을 따르는 클래스가 나오기 쉬움

---

<br>
이거랑 아키텍쳐 패턴까지 결합해서 봐야한다고 생각하니 난이도가 올라가는 것 같습니다..
