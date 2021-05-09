# 6장 객체와 자료 구조

변수를 private으로 정의해놓고 왜 get, set 역할의 함수를 public하게 정의해서 외부에 노출시킬까?

<br>

## 자료 추상화

단순히 변수 사이에 함수라는 계층을 넣는다고 구현이 저절로 감춰지는 것은 아니다.
추상화를 해야한다.

```swift
// 단순히 변수값을 반환
protocol VehicleFuelInterface {
    func getFuelTankCapapcityInGallons() -> Double
    func getGallonsOfGasoline() -> Double
}

// 정보가 어떻게 오는지 전혀 드러나지 않음.
protocol VehicleFuelInterface {
    func getPercentFuelRemaining() -> Double
}
```
<br>

## 자료/객체 비대칭

객체 지향 코드에서 어려운 변경은 절차적인 코드에서 쉽고,
절차적인 코드에서 어려운 변경은 객체 지향 코드에서 쉽다.

새로운 함수가 아니라 새로운 자료 타입이 필요한 경우 -> 클래스와 객체 지향 기법
새로운 자료 타입이 아니라 새로운 함수가 필요한 경우 -> 절차적인 코드와 자료구조

2번째에 해당하는 예를 들자면,
공통적인 작업을 `protocol`로 빼놓고 다른 객체에서 채택해 사용하는 경우 프로토콜에 새 함수를 추가하려면 채택하고 있는 모든 객체에서 수정이 일어나야 한다.

(자료구조..에 대한 개념은 struct와 같다고 보면 되는건가?)
<br> 

## 디미터 법칙

클래스 C의 메서드 f()는 다음과 같은 객체의 메서드만 호출해야 한다.
- 클래스 C
- f가 생성한 객체
- f로 넘어온 객체
- C 인스턴스 변수에 저장된 객체

근데 저 객체의 메서드가 반환하는 객체의 메서드는 호출하면 안된다고 한다.

### 기차 충돌

```swift
// 이럴 경우에는 자료 구조로 작성하자..
let outputDir = ctxt.getOptions().getScratchDir().getAbsoultePath()
    
// 자료구조인 경우
let outputDir = ctxt.options.scratchDir.absoultePath

// 적어도 이렇게
let opts = ctxt.getOptions()
let scratchDir = opts.getScratchDir()
let outputDir = scratchDir.getAbsoultePath()
```

### 잡종 구조

### 구조체 감추기
만약 위의 ctxt, options, scratchDir이 객체로 사용된다면 어떻게 구성해야 좋을까?

가져오는게 아닌 해당 객체에게 **뭔가를 하라고** 말하도록 바꾼다.
`getScratchDir() -> createScratchFileStream(classFileName)`

<br>

## 자료 전달 객체
(여기서 나오네요.) 
자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스 == `Struct`

~~`Swift`에서는 굳이 예제 목록 6-7처럼 복잡하게 get변수명() 처리가 할 필요가 없이 `struct`를 쓰면 해결된다.~~

<br>

## 결론

여기서는 객체와 자료 구조(+ 절차적인 코드)를 상황에 따라서 유연성있게 설계해야한다고 한다.

Swift에서는 `class`, `struct`를 선택하는 기준은 [Swift 공식 문서](https://developer.apple.com/documentation/swift/choosing_between_structures_and_classes)를 참고하면 될 것 같습니다.
