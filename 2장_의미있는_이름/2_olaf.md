# 2장  의미 있는 이름



## 의도를 분명히 밝혀라, 그릇된 정보를 피하라.

* 변수의 존재 이유, 수향 기능, 사용방법을 이름에 포함하자.
* 매직넘버를 지양하고 상수에는 이름을 지어서 의미를 포함하자.
* 코드는 문장을 읽듯이 읽혀야 하는데, 잘못된 이름으로 인해 해독하는 경우를 피하자.
* 약어를 사용하게되면 다른 사람들이 이해하기 힘들다.



## 의미 있게 구분하라

* 매개변수의 이름, 레이블에 의미를 부여하자.
* 불용어를 지양하자.
  * `ProductInfo`, `ProductData`이 둘의 차이는 뭘까?
  * `NameString`, `Name` -> `String`은 중복이다.



## 발음하기 쉬운 이름을 사용하라

* 변수 이름이 너무 길어서 약어를 사용한다면, 변수 이름에 포함된 단어들을 추상화 해서 발음하기 쉽게 바꾸자
* `genymdhms` -> `generationTimeStamp`



## 인코딩을 피하라

이 부분은 무슨 말을 하는지 잘 이해 못함.



## 클래스 이름

* 명사나 명사구가 적합하다.

* **`Manager`, `Helper` `Processor`, `Data`, `Info`를 피하자.**

* > Protocol 네이밍 컨벤션
  >
  > - **Protocols that describe \*what something is\* should read as nouns** (e.g. `Collection`).
  > - **Protocols that describe a \*capability\* should be named using the suffixes `able`, `ible`, or `ing`** (e.g. `Equatable`, `ProgressReporting`).



## 메서드 이름

* 동사나 동사구가 적합하다.

* > ex) 변환하는 메소드를 만들 때
  >
  > `func convert() -> Double` ( O )
  >
  > `fucn converter() -> Double` ( X )



## 한 개념에 한 단어를 사용하라

* 서버에 요청하는 메소드 이름을 지을 때, 어느 클래스에서는 `fetch` 또 어느 클래스에서는 `retrieve` 로 요청을 한다.
* 혼자서 개발할 때는 별 문제 없겠지만, 다른 사람이 볼 때는 의도를 파악하기 어려울 수 있음.
* 통일해서 사용하자.