# 5장 형식 맞추기

### 들어가기에 앞서
프로그래머라면 형식을 깔끔하게 맞춰 코드를 짜야 한다.
간단한 규칙을 정하고 착실히 따라야 한다.
(Swift Lint!)

<br>

---


### 형식을 맞추는 목적
오늘 구현한 코드 스타일과 가독성 수준은 이후 유지보수 용이성과 확정성에 지속적인 영향을 미친다.
-> 형식을 맞추는 목적은 나중의 코드를 위해~

<br>

### 적절한 행 길이를 유지하라
200줄 정도인 파일로도 커다란 프로젝트를 구축할 수 있다.
큰 파일보다는 작은 파일이 이해하기도 쉽다.

#### 신문 기사처럼 작성하라
이름은 간단하면서도 설명이 가능하게,
첫 부분에서 마지막으로 갈수록 의도를 세세하게 묘사한다. (고차원 -> 저차원)

(이부분은 개인적으로 `3장 함수`에서 TO 문단처럼 읽혀야 한다와 비슷하다고 생각했습니다.)

#### 개념은 빈 행으로 분리하라
빈 행으로 분리함으로써 새로운 개념을 시작한다는 시각적 단서를 줄 수 있다.
import 문, 각 함수 사이 등..

(의미부여도 되고 가독성도 좋아진다고 생각합니다~)

`Swift`에서는 ``// MARK:`` 주석을 활용해서 구분짓는 것을 권장하고 있습니다.
`import`문은 줄바꿈 없이 사전 순으로 정렬되고 줄바꿈으로 구분 되는 경우는 다음과 같다고 합니다.
1. Module/submodule imports not under test
2. Individual declaration imports (class, enum, func, struct, var)
3. Modules imported with @testable (테스트파일 에서만)

```swift
import CoreLocation
import MyThirdPartyModule
import SpriteKit
import UIKit

import func Darwin.C.isatty

@testable import MyModuleUnderTest
```

`Swift`는 함수, Type, Extension 등 선언에서 조금 복잡한 줄 바꿈 규칙이 있는데 자세한 내용은 [Swift Style Guide](https://google.github.io/swift/#type-variable-and-function-declarations)를 참고해주세요~

#### 세로 밀집도 (연관성)
서로 밀접한 코드 행은 세로로 가까이 놓여야 한다.

(이건 객체 내부에서 통용될 수 있는 이야기겠네요. 특히 변수/상수? 객체끼리면 위의 '개념'으로 나뉠 것 같습니다.)

#### 수직거리
서로 밀접한 개념은 세로로 가까이 둬야 한다.

```swift
private func configureView() {
    self.configureNavigationView()
}

// 두 함수 사이에는 configureView와 상관 없는 코드들..

private func configureNavigationView() { ... }
```

옛날에 작성되어 있는 코드 중에는 이런식으로 되어있는 코드를 많이 보았는데 관련된 코드들끼리 붙어있지 않아서 여기저기 뒤졌던 경험이 많아서 공감이 많이 되었습니다.. 종속함수에서 언급을 해주네요.

- 변수 선언 : 변수는 사용하는 위치에 최대한 가까이 선언한다.
- 인스턴스 변수 : 클래스 맨 처음에 선언한다.
- 종속 함수 : 두 함수는 세로로 가까이 배치한다.
- 개념적 유사성 : 종속성, 변수와 그 변수를 사용하는 함수, 비슷한 동작을 수행하는 함수(테스트 코드)

#### 세로 순서
호출하는 함수 다음으로 호출되는 함수를 배치한다.
(위에서 예시로 든 `configureView() -> configureNavtionView()` 순서로)

<br>

### 가로 형식 맞추기

클린 코드에서 말해주는 가로 형식 맞추기는

- 가로 공백과 밀집도
- 가로 정렬
- 들여쓰기

다음과 같은데 해당 부분은 [Swift Style Guide](https://google.github.io/swift/#type-variable-and-function-declarations)의 `General Formatting`을 우선시하는게 좋을 것 같습니다.

<br>

### 팀 규칙
프로그래머라면 각자 선호하는 규칙이 있지만 팀에 속한다면 그것이 팀 규칙이어야 한다.
한 소스 파일에서 봤던 형식이 다른 소스 파일에도 쓰이리라는 신뢰감을 주어야 한다.
(어디는 configure, 어디는 setup..)


<br>

### 느낀 점
세로, 가로 형식 맞추기에서 공감하는 부분 (특히 종속성)도 많았고 다시 한번 `Swift Style Guide`를 읽어보는 계기도 마련해주네요. 코드를 작성하면서도 못지키는 부분도 꽤 있었습니다. 이번 기회에 다시 정독해봐야겠네요 😭
