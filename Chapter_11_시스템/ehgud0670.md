# 11장. 시스템 

## 개요

`복잡성은 죽음이다. 개발자에게서 생기를 앗아가며, 제품을 계획하고 제작하고 테스트하기 어렵게 만든다.`

- 레이 오지(Ray Ozzie), 마이크로소프트 CTO

## 도시를 세운다면?

* 도시를 세우는 것은 한 사람으로 불가능하다. 한 사람이 아닌 각 분야를 관리하는 팀이 있어야 도시와 같은 거대한 시스템이 제대로 돌아간다.
* 도시가 돌아가는 또 다른 이유는 **적절한 추상화와 모듈화** 때문이다. 
따라서 전체적인 구조는 이해하지 못하더라도 개인과 개인이 관리하는 구성요소(`Component`)는 효율적으로 돌아간다.

이 장에서는 **높은 추상화 수준, 즉 시스템 수준에서도 깨끗함을 유지하는 방법**을 살펴본다.

## 시스템 제작과 시스템 사용을 분리하라

제작(construction)은 사용(use)은 아주 다르다는 사실을 명심한다. 

`소프트웨어 시스템은 (애플리케이션 객체를 제작하고 의존성을 서로 '연결'하는) 준비 과정과 (준비 과정 이후에 이어지는) 런타임 로직을 분리해야 한다.`

* 시작 단계는 모든 애플리케이션이 풀어야 할 관심사(concern)이다. **관심사 분리**는 우리 분야에서 가장 오래되고 가장 중요한 설계 기법 중 하나다. 

아래 코드는 관심사를 분리하지 않은 예의 코드다. 
```java
public Service getService() {
    if (service == null) {
        service = new MyServicImpl(...); // 모든 상황에 적합한 기본값일까?            
    }
    return service;
}
```
=> `초기화 지연(Lazy Initialzation)` 혹은 `계산 지연(Lazy Evaluation)`이라는 기법이다. 

> 초기화 지연의 장점

* 실제로 필요할 때까지 객체를 생성하지 않으므로 불필요한 부하가 걸리지 않는다.
* 어떤 경우에도 null 포인터를 반환하지 않는다.

> 초기화 지연의 단점

* getService 메서드가 MyServiceImpl과 (위에서는 생략한) 생성자 인수에 명시적으로 의존하는 문제가 있다. 
* 테스트도 문제다. `MyServiceImpl`이 무거운 객체라면 단위 테스트에서 `getService` 메서드를 호출하기 전에 적절한 테스트 전용 객체(Test Double이나 Mock 객체)를 service 필드에 할당해야 한다. 
* 일반 런타임 로직에다 객체 생성 로직을 섞어놓은 탓(service가 null 인 경로, service가 null이 아닌 경로)에 모든 실행 경로도 테스트해야 한다(이는 또한 SRP를 위반한다).
* **무엇보다 MyServiceImpl이 모든 상황에 적합한 객체인지 모른다는 사실이 가장 큰 문제다.** 주석도 그렇게 달았다.

설정 논리는 일반 실행 논리와 분리해야 한다. 그래야 모듈성이 높아진다. 

### Main 분리

<img width="549" alt="main에서 분리" src="https://user-images.githubusercontent.com/38216027/123529282-a95ce300-d729-11eb-8eec-f5b28ac4c50a.png">

* main 함수에서 시스템에 필요한 객체를 생성한 후 이를 애플리케이션에 넘긴다. (생성)
* 애플리케이션은 그저 객체를 사용할 뿐이다. (사용)

=> 모든 의존성 방향이 단 방향으로 main 쪽에서 어플리케이션 쪽으로 향한다.

### 팩토리

**객체가 생성되는 시점을 애플리케이션이 결정할 필요한 경우도 있다.**
예를 들어, 주문처리 시스템에서 애플리케이션은 LineItem 인스턴스를 생성해 Order에 추가하는 경우이다(LineItem 인스턴스는 사용자가 아이템을 고를 때마다 동적으로 생산되어야 한다).

<img width="579" alt="팩토리로 생성 분리" src="https://user-images.githubusercontent.com/38216027/123529279-a5c95c00-d729-11eb-80a8-952f0f5cc61b.png">

=> 여기서도 마찬가지로 모든 의존성이 main에서 OrderProcessing 애플리케이션으로 향한다. 

### 의존성 주입 

사용과 제작을 분리하는 강력한 메커니즘 중 하나가 **의존성 주입**이다. 의존성 주입은 제어 역전(Inversion Of Control)을 의존성 관리에 적용한 메커니즘이다. 

* 제어 역전에서는 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 넘기게 된다. 의존상 관리 맥락에서 객체는 **의존성 자체를 인스턴스로 만드는 책임은 지지 않는다. 대신에 이런 책임을 다른 '전담' 메커니즘에 넘겨야만 한다. 그렇게 함으로써 제어를 역전한다.**  

```java
MyService myService = (MyService)(jndiContext.lookup("NameOfMyService"));
```
=> JNDI 검색은 의존성을 주입을 '부분적으로' 구현한 기능이다. 호출하는 객체는 의존성을 능동적으로 해결해야 한다. 

진정한 의존성 주입은 여기서 한 걸음 더 나가간다. 클래스가 의존성을 해결하려 시도하지 않는다. **클래스는 완전히 수동적이다.** 대신에 의존성을 주입하는 방법으로 **설정자(setter) 메서드**나 **생성자 인수**를 (혹은 둘 다를) 제공한다.

초기화 지연으로 얻는 장점을 포기해야 하는 걸까? 이 기법은 DI를 사용하더라도 때론 여전히 유용하다. 먼저 대다수 DI 컨테이너는 필요할 때까지는 객체를 생성하지 않고, 대부분은 계산 지연이나 비슷한 최적화에 쓸 수 있도록 **팩토리를 호출하거나 프록시를 생성하는 방법을 제공**한다.

## 확장

소프트웨어 시스템(아키텍처)은 물리적인 시스템과 다르게 점진적으로 발전이 가능하다. 
먼저 관심사를 적절히 분리하지 못한 아키텍처 예를 소개한다. EJB1과 EJB2다.

영속적으로 저장될 Bank 클래스에 필요한 엔티티 빈을 살펴보자. 엔티티 빈은 관계형 자료, 즉 테이블 행을 표현하는 객체로, 메모리에 상주한다. 

* iOS의 CoreData를 예를 들면 Bank 클래스는 CoreData 프레임워크의 `NSManagedObject`를 상속받은 클래스라 할 수 있겠다.

> 목록 11-1 Bank EJB용 EJB2 지역 인터페이스
```java
public interface BankLocal extends java.ejb.EJBLocalObject {
    String getStreetAddr1() throws EJBException;
    String getStreetAddr2() throws EJBException;
    String getCity() throws EJBException;
    String getState() throws EJBException;
    String getZipCode() throws EJBException;
    void setStreetAddr1(String street1) throws EJBException;
    void setStreetAddr2(String street2) throws EJBException;
    void setCity(String city) throws EJBException;
    void setState(String state) throws EJBException;
    void setZipCode(String zip) throws EJBException;
    Collection getAccounts() throws EJBException;
    void setAccounts(Collection accounts) throws EJBException;
    void addAccount(AccountDTO accountDTO) throws EJBException;
}
```

> 목록 11-2 상응하는 EJB2 엔티티 빈 구현
```java
public abstract class Bank implements java.ejb.EntityBean {
    // 비즈니스 논리...
    public abstract String getStreetAddr1();
    public abstract String getStreetAddr2();
    public abstract String getCity();
    public abstract String getState();
    public abstract String getZipCode();
    public abstract void setStreetAddr1(String street1);
    public abstract void setStreetAddr2(String street2);
    public abstract void setCity(String city);
    public abstract void setState(String state);
    public abstract void setZipCode(String zip);
    public abstract Collection getAccounts();
    public abstract void setAccounts(Collection accounts);

    public void addAccount(AccountDTO accountDTO) throws NamingException {
        InitialContext context = new InitialContext();
        AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
        AccountLocal account = accountHome.create(accountDTO);
        Collection accounts = getAccounts();
        accounts.add(account);
    }

    //EJB 컨테이너 논리
    public abstract void setId(Integer id);
    public abstract Integer getId();
    public Integer ejbCreate(Integer id) { ... }
    public Post ejbPostCreate(Integer id) { ... }
    // 나머지도 구현해야 하지만 일반적으로 
    public void setEntityContext(EntityContext ctx) { }
    public void unsetEntityContext() { }
    public void ejbActivate() { }
    public void ejbPassivate() { }
    public void ejbLoad() { }
    public void ejbStore() { }
    public void ejbRemove() { }
}
```

* 비즈니스 논리가 EJB2 애플리케이션 컨테이너에 강하게 결합되는 문제가 있다. 
* 클래스를 생성할 때는 컨테이너에서 파생해야 되며 컨테이너가 요구하는 다양한 생명주기 메서드도 제공해야 한다. 즉, 너무 무겁다(heavy).
* 이렇게 비즈니스 논리가 덩치 큰 컨테이너와 밀접하게 결합된 탓에 독자적인 단위 테스트가 어렵다. **컨테이너를 흉내내는 쉽지 않은 일을 하거나** 많은 시간을 낭비하려 EJB와 테스트를 실제 서버에 배치해야 한다. 그래서 EJB2 코드는 프레임워크 밖에서 **재사용하기란 사실상 불가능하다(이게 제일 큰 단점).**

### 횡단(cross-cutting) 관심사

횡단 관심사는 책 내용도 좋지만 jojoldu님의 이 [글](https://jojoldu.tistory.com/71)을 읽으면 더 잘 이해하실 수 있을 겁니다. 

## 자바 프록시

> 목록 11-3 JDK 프록시 예제

```java
// Bank.java (패키지 이름을 감춘다)
import java.util.Collection;

// 은행 추상화
public interface Bank {
    Collection<Account> getAccounts();
    void setAccounts(Collection<Account> accounts);
}

// BankImpl.java
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

// 추상화를 위한 POJO("Plain Old Java Object") 구현 
public class BankImpl implements Bank {
    private List<Account> accounts;

    @Override
    public Collection<Account> getAccounts() {
        return accounts;
    }

    @Override
    public void setAccounts(Collection<Account> accounts) {
        this.accounts = new ArrayList<>();
        for (Account account: accounts) {
            this.accounts.add(account);
        }
    }
}

// BankProxyHandler.java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.util.Collection;

// 프록시 API가 필요한 "InvocationHandler"
public class BankProxyHandler implements InvocationHandler {
    private Bank bank;

    public BankProxyHandler(BankImpl bank) {
        this.bank = bank;
    }

    // InvocationHandler에 정의된 메서드
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        if (methodName.equals("getAccounts")) {
            bank.setAccounts(getAccountsFromDatabase());
            return bank.getAccounts();
        } else if (methodName.equals("setAccounts")) {
            bank.setAccounts((Collection<Account>) args[0]);
        } else {
            ...
        }
    }

    // 세부사항은 여기에 이어진다.
    protected Collection getAccountsFromDatabase() { ... }
    protected void setAccountsFromDatabase(Collection<Account> accounts) { ... }
}

// 다른 곳에 위치하는 코드 
Bank bank = (Bank) Proxy.newProxyInstance(
        Bank.class.getClassLoader(),
        new Class[] {Bank.class},
        new BankProxyHandler(new BankImpl()));
```
* POJO로 BankImpl을 단순하게 구성했다. DB 연동에 필요한 실제 세부사항은 InvocationHandler에서 구현되었다. 즉 관심사를 분리하여 결합도가 없어졌다.
* 하지만 코드가 상당히 많으며 제법 복잡하다. 다시 말해서, 프록시를 사용하면 깨끗한 코드를 작성하기 어렵다. 또한 프록시는 (진정한 AOP 해법에 필요한) 시스템 단위로 실행 '지점'을 명시하는 메커니즘도 제공하지 않는다.

## 순수 자바 AOP 프레임워크 

다행히 프록시 코드는 판박이라 **도구로 자동화**할 수 있다. 스프링은 비즈니스 논리를 POJO로 구현한다. 

> POJO

* POJO는 순수하게 도메인에 초첨을 맞춘다. POJO는 엔터프라이즈 프레임워크에 (그리고 다른 도메인에도) 의존하지 않는다. 따라서 테스트가 개념적으로 더 쉽고 간단한다.

```xml
<beans>
    ...
    <bean id="appDataSource"
    class="org.apache.commons.dbcp.BasicDataSource"
    destory-method="close"
    p:driverClassName="com.mysql.jdbc.Driver"
    p:url="jdbc:mysql://localhost:3306/mydb"
    p:username="me"/>

    <bean id="bankDataAccessObject"
    class="com.example.banking.persistence.BankDataAccessObject"
    p:dataSource-ref="appDataSource"/>

    <bean id="bank"
    class="com.example.banking.model.Bank"
    p:dataAccessObject-ref="bankDataAccessObject"/>
    ...
</beans>
```
=> Bank 도메인 객체는 자료 접근자 객체로 프록시되었으며, 자료 접근자 객체는 JDBC 드라이버 자료 소스로 프록시되었다(책의 그림은 화살표 방향이 잘못되었다. AppDataSource의 화살표는 Bank가 아닌 BankDataAccessObject로 향해야 한다). 

```java
XmlBeanFactory bf = new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```
=> 애플리케이션에서 DI 컨테이너에게 (XML 파일에 명시된) 시스템 내 최상위 객체를 요청하려면 위의 코드가 필요하다. 
즉, 스프링 관련 자바 코드가 거의 필요 없으므로 애플리케이션은 **사실상 스프링과 독립적**이다. 
**EJB2 시스템이 지녔던 강한 결합이라는 문제가 모두 사라진다.**

### EJB3

EJB3는 **XML 설정 파일**과 **자바 5 애너테이션 기능**을 사용해 횡단 관심사를 **선언적으로 지원**하는 스프링 모델을 따른다.

```java
@Entity
@Table(name = "Banks")
public class Bank implements java.io.Serializable {
    @Id @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;

    @Embeddable // Bank의 데이터베이스 행에 '인라인으로 포함된' 객체
    public class Address {
        protected String streetAddr1;
        protected String streetAddr2;
        protected String city;
        protected String state;
        protected String zipCode;
    }

    @Embedded
    private Address address;

    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, mappedBy = "bank")
    private Collection<Account> accounts = new ArrayList<>();
    
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }
    
    public void addAccount(Account account) {
        account.setBank(this);
        accounts.add(account);
    }
    
    public Collection<Account> getAccounts {
        return accounts;
    }

    public void setAccounts(Collection<Account> accounts) {
        this.accounts = accounts;
    }
}
```

원래 EJB2 코드보다 위 코드가 훨씬 더 깨끗하다. 
* 일부 상세한 엔티티 정보는 애너테이션에 포함되어 그대로 남아있지만, 모든 정보가 애너테이션 속에 있으므로 **코드 자체는 깔끔하고 깨끗하다.** 즉, **그만큼 코드를 테스트하고 개선하고 보수하기가 쉬워졌다.**
* 애너테이션에 들어있는 영속성 정보는 일부든 전부든, 필요하다면, XML 배치 기술자로 옮겨도 괜찮다. **그러면 진짜로 순수한 POJO만 남는다.**

## AspectJ 관점 

너무 자바 이야기다. 생략한다.

## 테스트 주도 시스템 아키텍처 구축 

```
최선의 시스템 구조는 각기 POJO(또는 다른)객체로 구현되는 모듈화된 관심사 영역(도메인)으로 구성된다.
이렇게 서로 다른 영역은 해당 영역 코드에 최소한의 영향을 미치는 관점이나 유사한 도구를 사용해 통합한다. 
이런 구조 역시 코드와 마찬가지로 테스트 주도 기법을 적용할 수 있다.
```

## 의사 결정을 최적화하라 

관심사를 모듈로 분리한 POJO 시스템은 기민함을 제공한다. 이런 기민함 덕택에 구조에 대한 정책을 결정하기 제일 적절한 때까지 최대한 늦츨 수 있다. 반면에 결합도가 높은 시스템이라면 기민하게 개발하는 것이 불가능하므로 애초에 먼저 결정해야 하는 단점이 있다.  

## 명백한 가치가 있을 때 표준을 현명하게 사용하라

생략

## 시스템은 도메인 특화 언어가 필요하다 

생략

## 결론

시스템 역시 깨끗해야 한다. **깨끗하지 못한 아키텍처는 도메인 논리를 흐리며 기민성을 떨어트린다.** 관심사를 분리해 기민성이 높은 깨끗한 아키텍쳐를 만들도록 하자.  
