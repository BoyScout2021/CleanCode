# 객체와 자료 구조

## 자료 추상화

### 구체 타입보단 추상화된 프로토콜 활용하기

- 구체적인 Point 클래스

  ```swift
  class Point {
      var x: Double
      var y: Double
  }
  ```

  - 어떤 종류의 좌표계를 사용하는 데이터 구조인지 명확함
  - 개별적으로 값을 읽고 설정
  - 모든 구현이 노출되어 있음

- 추상적인 Point 프로토콜

  ```swift
  protocol Point {
      func getX() -> Double
      func getY() -> Double
      func setCartesian(x: Double, y: Double)
      func getR() -> Double
      func getTheta() -> Double
      func setPolar(r: Double, theta: Double)
  }
  ```

  - 어떤 종류의 좌표계를 사용하는 데이터 구조인지 알수없음
  - 메서드가 접근 정책을 강제함
    - 값을 읽을 때는 개별적으로, 값을 설정할때는 메서드를 통해 두 값을 동시에 설정해야 함
  - 내부 구현을 숨길 수 있음
  - 클라이언트는 내부 구현이 어떤지 알 필요 없이, 추상화된 프로토콜의 메서드만으로 모든 동작 수행 가능

### 추상화된 프로토콜 내부의 메서드도 추상화 단계를 높여서 구현하기

- 구체적인 Vehicle 프로토콜

  ```swift
  protocol Vehicle {
      func getFuelTankCapacityInGallons() -> Double
      func getGallonsOfGasoline() -> Double
  }
  ```

  - 연료 상태를 구체적인 단위를 가진 값으로 알려줌
  - 어떤 변수에서 값을 가져와 반환할지가 명확함

- 추상적인 Vehicle 프로토콜

  ```swift
  protocol Vehicle {
      func getPercentFuelRemaining() -> Double
  }
  ```

  - 연료 상태를 백분율이라는 추상적인 개념으로 알려줌
  - 남은 연료 상태 백분율이라는 정보를 어디서 가져오는지 명확하게 드러나지 않음

## 자료/객체 비대칭

절차적인 코드와 객체 지향 코드는 상호 보완적인 특질을 가지므로 상황에 맞게 사용해야 함

### 절차적인 코드

```swift
struct Square {
    let topLeft: Point
    let side: Double
}

struct Rectangle {
    let topLeft: Point
    let height: Double
    let width: Double
}

struct Circle {
    let center: Point
    let radius: Double
}

struct Geometry {
    enum Exception: Error {
        case noSuchShape
    }

    private let pi: Double = 3.141592

    func area(of shape: Any) throws -> Double {
        if let square = shape as? Square {
            return square.side * square.side
        } else if let rectangle = shape as? Rectangle {
            return rectangle.height * rectangle.width
        } else if let circle = shape as? Circle {
            return pi * circle.radius * circle.radius
        } else {
            throw Exception.noSuchShape
        }
    }
}
```

- 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉬움
  - Geometry에 도형의 둘레 길이를 구하는 함수를 추가하고 싶다면, 다른 도형 구조체의 변경 없이 추가할 수 있음
- 새로운 자료 구조를 추가하기 어려움
  - 새로운 도형 종류를 추가하고 싶다면, Geometry의 메서드들을 모두 고쳐야 함

### 객체 지향 코드

```swift
protocol Shape {
    func area() -> Double
}

struct Square: Shape {
    private let topLeft: Point
    private let side: Double

    func area() -> Double {
        return side * side
    }
}

struct Rectangle: Shape {
    private let topLeft: Point
    private let height: Double
    private let width: Double

    func area() -> Double {
        return height * width
    }
}

struct Circle: Shape {
    private let pi: Double = 3.141592
    private let center: Point
    private let radius: Double

    func area() -> Double {
        return pi * radius * radius
    }
}
```

- 넓이를 반환하는 메서드를 가진 추상화된 Shape 프로토콜을 도형 구조체들이 채택하는 형태
- 넓이를 계산하는 역할을 수행하는 Geometry가 필요없어짐
- 기존 함수를 변경하지 않으면서 새로운 자료 구조를 추가하기 쉬움
  - 새로운 도형 종류를 추가하고 싶다면, 다른 도형 구조체의 변경 없이 추가할 수 있음
- 새로운 함수를 추가하기 어려움
  - 둘레 길이와 같은 새로운 계산 로직을 추가하고 싶다면, Shape 프로토콜을 채택한 모든 도형 구조체의 코드를 고쳐야 함

## 디미터 법칙(최소 지식 원칙)

꼭 필요한 최소한의 객체와만 상호작용해야한다는 법칙

### 상호작용해도 되는 객체

- 객체 자체
- 메서드에서 생성한 객체
- 메서드에 매개변수로 전달된 객체
- 객체에 속하는 구성요소

### 디미터 법칙을 위반하는 경우

- **기차 충돌**

  ```swift
  let outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath()
  ```

  여러 객차가 한줄로 이어진 기차처럼 보이는 코드

  ```swift
  let opts = ctxt.getOptions()
  let scratchDir = opts.getScratchDir()
  let outputDir = scratchDir.getAbsolutePath()
  ```

  포함관계를 확실히 알 수 있도록 위와같이 개선

  > 위의 코드들은 디미터 법칙을 위반하였는가?
  >
  > ctxt, Options, ScratchDir이 객체인지 자료구조인지에 따라 다르다.
  >
  > 객체라면 내부 구조를 숨겨야 하므로 위반, 자료 구조라면 내부 구조를 노출하므로 법칙 적용 x

- **잡종 구조**

  중요한 기능을 수행하는 함수와 공개 변수, 공개 조회/설정 함수를 모드 가지는 절반은 객체, 절반은 자료 구조인 코드

- **구조체 감추기**

  만약 ctxt, Options, ScratchDir이 객체라면 내부 구조를 감춰야 함

  ```swift
  let outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath()
  let outputFile = outputDir + "/" + String(describing: Self.self) + ".swift"
  let url = URL(string: outputFile)!
  let outputStream = OutputStream(url: url, append: false)
  ```

  outputDir를 사용하는 부분의 코드를 보면, outputDir 경로에 파일을 생성하는 게 목적임을 알 수 있음

  다른 객체들의 내부 구조를 드러내서 얻은 outputDir를 외부에서 사용하기보단, outputDir를 사용하는 동작을 ctxt 내부에서 하도록 책임을 줘야 함

  ```swift
  let className = String(describing: Self.self)
  let outputStream = ctxt.createScratchFileStream(for: className)
  ```

## 자료 전달 객체

### DTO?

- Data Transfer Object
- 자료 구조체의 전형적인 형태인 공개 변수만 있고 함수가 없는 형태
- 데이터 베이스와 통신하거나 소켓에서 받은 메세지의 구문 분석에 유용
- 데이터베이스에 저장된 가공되지 않은 정보를 애플리케이션 코드에서 사용할 객체로 변환하는 일련의 단계에서 가장 처음으로 사용하는 구조체

### 활성 레코드

- DTO의 특수한 형태
- 공개 변수가 있거나 비공개 변수에 조회/설정 함수가 있는 자료 구조, save나 find 같은 탐색 함수도 제공
- 활성 레코드에 비즈니스 메서드를 추가해 객체로 취급하면 안됨 (잡종 구조)
- 활성 레코드는 자료 구조로 취급하고, 비즈니스 로직을 가지면서 활성 레코드의 인스턴스를 가지는 객체는 따로 생성해야 함

## 결론

새로운 자료 타입을 추가하는 유연성이 필요하다? -> 객체 지향 코드

새로운 동작을 추가하는 유연성이 필요하다? -> 절차적인 코드

무조건 한쪽이 더 낫다고 생각하기보단, 직면한 문제에 최적인 해결책을 선택할 줄 아는 개발자가 되어야 한다.

