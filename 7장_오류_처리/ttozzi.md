# 오류 처리

깨끗한 코드와 오류 처리의 연관성?

오류 처리 코드로 인해 프로그램 논리를 이해하기 어려워질 수 있음

## 깔끔하게 오류를 처리하는 방법

### 오류를 표현하는 코드보다 언어에서 제공하는 예외처리 사용하기

```swift
final class DeviceController {
    ...
    func sendShutDown() {
        let handle = getHandle(DEV1)
        if handle != .invalid {
            retrieveDeviceRecord(handle)
            if record.getStatus() != .suspended {
                pauseDevice(handle)
                clearDeviceWorkQueue(handle)
                closeDevice(handle)
            } else {
                os_log("device suspended")
            }
        } else {
            os_log("invaild handle: \(handle)")
        }
    }
    ...
}
```

논리가 오류 처리 코드와 섞이면 안된다.

오류를 표현하는 코드를 따로 만들어 사용하기보단, 언어에서 제공하는 예외처리를 활용해야 한다.

```swift
final class DeviceController {
    ...
    func sendShutDown() {
        do {
            try tryToShutDown()
        } catch let error {
            os_log("\(error.localizedDescription)")
        }
    }
    
    func tryToShutDown() throws {
        let handle = try getHandle(DEV1)
        let record = try retrieveDeviceRecord(handle)
        
        pauseDevice(handle)
        clearDeviceWorkQueue(handle)
        closeDevice(handle)
    }
    
    func getHandle(_ id: DeviceID) throws -> DeviceHandle {
        ...
        guard handle != .invaild else {
            throw DeviceError.invalidHandle
        }
        return handle
    }
    
    func retrieveDeviceRecord(_ handle: DeviceHandle) throws -> DeviceRecord {
        ...
        guard record.getStatus() != .suspended else {
            throw DeviceError.suspended
        }
        return record
    }
    ...
}
```

주요 로직과 예외처리를 명확하게 구분하여 코드 가독성이 좋아졌다.

### Try-Catch-Finally 문부터 작성?

TDD?

에러가 발생하는 로직을 작성할 땐 try-catch 예외처리 코드부터 먼저 짜라?

자연스럽게 try 블록 내부의 로직 부분을 먼저 작성하게 되므로 예외처리와 독립적으로 분리된 코드를 짜게 된다?

### 미확인 예외를 사용?

???

### 예외에 의미를 제공하기

- 왜 에러가 발생했는지 충분한 정보를 담기

### 호출자를 고려해 예외 클래스 정의하기

- 여러가지 라이브러리에서 제공하는 에러를 구분지어 사용하지 말고, 클라이언트단에서 사용할 에러로 래핑해서 한 종류의 에러만 사용하기
  - **예제 작성하기**
- 외부 API를 그대로 사용하지 않고 한 단계 래핑하는 이유?
  - 외부 API를 감싸면 외부 라이브러리와 프로그램 사이의 의존성을 줄일 수 있음
  - 래퍼 클래스에 주입을 통해 외부 API와의 독립적인 테스트가 가능
  - 특정 API의 인터페이스에 고정되지 않는 유연한 구조의 구현이 가능해짐

### 정상 흐름을 정의하라?

```swift
do {
    let expenses = try expenseReportDAO.getMeals(employee.getID())
    m_total += expenses.getTotal()
} catch {
    m_total += getMealPerDiem()
}
```

`getMeals` 메서드에서 청구한 식비가 존재한다면 `m_total` 에 그 값을 더하고, 없다면 예외를 발생시켜 `getMealPerDiem` 를 호출해 기본값을 더해주도록 함

불필요한 예외처리가 논리를 따라가기 어렵게 만듦

```swift
let expenses = expenseReportDAO.getMeals(employee.getID())
m_total += expenses.getTotal()
```

에러를 특수한 상황으로 처리할 필요가 없다면, 예외처리보단 예외적인 상황을 캡슐화하여 로직에 녹여내는게 좋음

### null 반환, 전달 피하기..???

적절하게 잘 사용해라?

Swift 에도 해당 되는가

### 결론

오류 처리를 프로그램 논리와 분리하여 가독성, 안정성을 모두 챙긴 깨끗한 코드를 작성하자

