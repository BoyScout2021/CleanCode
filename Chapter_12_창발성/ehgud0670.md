# 12장. 창발성 

## 창발적 설계로 깔끔한 코드를 구현하자

## 단순한 설계 규칙 1: 모든 테스트를 실행하라

## 단순한 설계 규칙 2~4: 리팩터링  

## 중복을 없애라 

```java
int size() {}
boolean isEmpty() {}
```

```java
boolean isEmpty() {
    return 0 == size();
}
```

```java
public void scaleToOneDimension(
        float desiredDimension, float imageDimension) {
    if (Math.abs(desiredDimension - imageDimension < errorThreshold)) {
        return;
    }
    float scalingFactor = desiredDimension / imageDimension;
    scalingFactor = (float) (Math.floor(scalingFactor * 100) * 0.01f);
    
    RenderedOp newImage = ImageUtilities.getScaledImage(
            image, scalingFactor, scalingFactor);
    image.dispose();
    System.gc();
    image = newImage;
}

public synchronized void rotate(int degrees) {
    RenderedOp newImage = ImageUtilities.getRotatedImage(
            image, degrees);
    image.dispose();
    System.gc();
    image = newImage;
}
```


```java
public void scaleToOneDimension(
        float desiredDimension, float imageDimension) {
    if (Math.abs(desiredDimension - imageDimension < errorThreshold)) {
        return;
    }
    float scalingFactor = desiredDimension / imageDimension;
    scalingFactor = (float) (Math.floor(scalingFactor * 100) * 0.01f);
    replaceImage(ImageUtilities.getScaledImage(
            image, scalingFactor, scalingFactor));
}

public synchronized void rotate(int degrees) {
    replaceImage(ImageUtilities.getRotatedImage(
            image, degrees););
}

private void replaceImage(RenderedOp newImage) {
    image.dispose();
    System.gc();
    image = newImage;
}
```

```java
public class VacationPolicy {
    public void accrueUSDivisionVacation() {
        // 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
        // ...
        // 휴가 일수가 미국 최소 법정 일수를 만족하는지 확인하는 코드 
        // ...
        // 휴가 일수를 급여 대장에 적용하는 코드
        // ...
    }

    public void accrueEUDivisionVacation() {
        // 지금까지 근무한 시간을 바탕으로 휴가 일수를 계산하는 코드
        // ...
        // 휴가 일수가 유럽연합 최소 법정 일수를 만족하는지 확인하는 코드 
        // ...
        // 휴가 일수를 급여 대장에 적용하는 코드
        // ...
    }
}
```

```java
abstract public class VacationPolicy {
    public void accrueVacation() {
        calculateBaseVacationHours();
        alterForLegalMinimums();
        applyToPayroll();
    }
    
    private void calculateBaseVacationHours() { /* ... */ };
    abstract protected void alterForLegalMinimums();
    private void applyToPayroll() { /* ... */ };
}

public class USVacationPolicy extends VacationPolicy {
    @Override protected void alterForLegalMinimums() {
        // 미국 최소 법정 일수를 사용한다.
    }
}

public class EUVacationPolicy extends VacationPolicy {
    @Override protected void alterForLegalMinimums() {
        // 유럽연합 최소 법정 일수를 사용한다.
    }
}
```
## 표현하라

## 클래스와 메서드 수를 최소로 줄여라 

## 결론