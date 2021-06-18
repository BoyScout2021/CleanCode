# 시스템 

## 개요

## 도시를 세운다면?

## 시스템 제작과 시스템 사용을 분리하라

```java
public Service getService() {
    if (service == null) {
        service = new MyServicImpl(...); // 모든 상황에 적합한 기본값일까?            
    }
    return service;
}
```

### Main 분리

### 팩토리

### 의존성 주입 

```java
MyService myService = (MyService)(jndiContext.lookup("NameOfMyService"));
```

## 확장

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

### 횡단(cross-cutting) 관심사

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


## 순수 자바 AOP 프레임워크 

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

```java
XmlBeanFactory bf = new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```

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


## AspectJ 관점 



## 테스트 주도 시스템 아키텍처 구축 

## 의사 결정을 최적화하라 

## 명백한 가치가 있을 때 표준을 현명하게 사용하라 

## 시스템은 도메인 특화 언어가 필요하다 

## 결론
