# 10장. 클래스 

# 개요

코드의 표현력과 그 코드로 이루어진 함수에 아무리 신경 쓸지라도 **좀 더 차원 높은 단계**까지
신경쓰지 않으면 깨끗한 코드를 얻기는 어렵다. 이 장에서는 좀 더 차원 높은 단계인 **깨끗한 클래스**를 다룬다. 

# 클래스 체계

클래스를 정의하는 표준 자바 관례에 따르면, 가장 먼저 변수 목록이 나온다. 
 * *정적(`static`) 공개(`public`) 함수*가 있다면 맨 처음에 나온다.
 * 다음으로 *정적 비공개(`private`) 변수*가 나오며, 
 * 이어서 *비공개 인스턴스 변수*가 나온다. *공개 변수*가 필요한 경우는 없다.
 * 변수 목록 다음에는 *공개 함수*가 나온다.

*비공개 함수*는 자신을 호출하는 공개 함수 직후에 넣는다. **즉, 추상화 단계가 순차적으로 내려간다. 그래서 프로그램은 신문 기사처럼 읽힌다.**

## 캡술화

때로는 테스트 코드를 작성하기 위해 변수나 유틸리티 함수를 `protected` 로 선언해
테스트 코드에 접근을 허용하기도 한다. 접근을 허용할만큼 테스트 코드는 매우 중요하다.
**하지만 그전에 비공개 상태를 유지할 온갖 방법을 강구한다. 캡슐화를 풀어주는 결정은 언제나 최후의 수단이다.**

# 클래스는 작아야 한다!

> 목록 10-2 
```java
public class SuperDashBoard {
    public Component getLastFocusedComponent()
    public void setLastFocused(Component lastFocused)
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumber() 
}
```
`SuperDashBoard` 는 메서드 수가 작음에도 **책임**이 너무 많다.

* 클래스 이름은 해당 클래스 **책임을 기술**해야 한다. 간결한 이름이 떠오르지 않으면 필경 클래스 이름이 너무 커서 그렇다. 예를 들어, 클래스 이름에 `Processor`, `Manager`, `Super` 등과 같이 모호한 단어가 있다면 클래스에다 여러 책임을 떠안겼다는 증거다.
* 클래스 설명은 만일("if"), 그리고("and"), -(하)며("or"), 하지만("but")을 사용하지 않고서 25단어 내외로 가능해야 한다.
* 위의 `SuperDashBoard` 클래스를 설명하자면 "SuperDashBoard는 마지막으로 포커스를 얻었던 컴포넌트에 접근하는 방법을 제공하며, 버전과 빌드 번호를 추적하는 메커니즘을 제공하는 클래스" 이다. "~하며"라 적힌 설명에서 보이듯이, `SuperDashBoard`가 너무 많은(2개 이상) 책임을 가지고 있다는 증거다.

## 단일 책임 원칙(SRP)

단일 책임 원칙(`Single Responsibility Principle`)은 클래스나 모듈을 **변경할 이유가 하나, 단 하나뿐이어야 한다는 원칙**이다. 

`SuperDashBoard`는 변경할 이유가 두 가지다. 
  * 첫째, `SuperDashBoard` 는 소프트웨어 버전 정보를 추적한다. 그런데 버전 정보는 소프트웨어를 출시할 때마다 달라진다.
  * 둘째, `SuperDashBoard` 는 자바 스윙 컴포넌트를 관리한다(SuperDashBoard는 최상위 GUI 윈도의 스윙 표현인 JFrame에서 파생한 클래스). 즉, 스윙 코드를 변경할 때마다 버전 번호가 달라진다. 하지만 역은 참이 아니다. 다른 코드가 바뀌어도 버전 번호는 달라진다. 즉 스윙 코드가 아닌 다른 코드가 바뀌어도 `SuperDashBoard`는 변경되어야 하므로 변경의 이유가 2가지가 된다.

```java
public class Version {
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumber()
}
```
책임, 즉 변경할 이유를 파악하려 애쓰다 보면 코드를 추상화하기도 쉬워진다. 
버전 정보를 다루는 메소드 세 개를 따로 빼내 `Version` 이라는 독자적인 클래스를 만든다. 또 **Version 클래스는 다른 어플리케이션에서 재사용하기 쉬운 구조**다.

### 인상적인 한마디: 작은 클래스가 많아도 복잡성은 비슷하니 걱정말고 쪼개자. 훨씬 바람직하다.

* 작은 클래스가 많은 시스템이든 큰 클래스가 몇 개뿐인 시스템이든 돌아가는 부품은 그 수가 비슷하다. **어느 시스템이든 익힐 내용은 그 양이 비슷하다.** 그러므로 고민할 질문은 다음과 같다. `도구 상자를 어떻게 관리하고 싶은가? 작은 서랍을 많이 두고 기능과 이름이 명확한 컴포넌트를 나눠 넣고 싶은가? 아니면 큰 서랍 몇 개를 두고 모두를 던져 넣고 싶은가?`
  * 규모가 어느 수준에 이르는 시스템은 논리가 많고도 복잡하다. **이런 복잡성을 다루려면 체계적인 정리가 필수다.**
  * 작은 클래스는 각자 맡은 책임이 하나며, 변경할 이유가 하나며, 다른 작은 클래스와 협력해 시스템에 필요한 동작을 수행한다.

## 응집도(Cohesion)

우리는 응집도가 높은 클래스를 선호한다. **응집도가 높다는 말은 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미이기 때문이다.** 

> 목록 10-4 Stack.java 응집도가 높은 클래스 
```java
public class Stack {
    private int topOfStack = 0;
    List<Integer> elements = new LinkedList<>();
    
    public int size() {
        return topOfStack;
    }
    
    public void push(int element) {
        topOfStack++;
        elements.add(element);
    }
    
    public int pop() throws PoppedWhenEmpty {
        if (topOfStack == 0) {
            throw new PoppedWhenEmpty();
        }
        int element = elements.get(--topOfStack);
        elements.remove(topOfStack);
        return element;
    }
}
```
위의 `Stack` 클래스는 응집도가 아주 높다. size()를 제외한 다른 두 메서드는 두 변수를 모두 사용한다. 

* `함수를 작게, 매개변수 목록을 짧게`라는 전략을 따르다 보면 때때로 **몇몇 메서드만이 사용하는 인스턴스 변수**가 아주 많아진다. 이는 십중팔구 **새로운 클래스로 쪼개야 한다는 신호**다. 응집도가 높아지도록 변수와 메서드를 적절히 분리해 새로운 클래스 두 세게로 쪼개준다.

## 응집도를 유지하면 작은 클래스 여럿이 나온다

클래스가 응집력을 잃는다면 쪼개라!

> 목록 10-5 PrintPrimes.java
```java
public class PrintPrimes {
    public static void main(String[] args) {
        final int M = 1000; // 구할 소수의 개수 
        final int RR = 50;
        final int CC = 4;
        final int WW = 10;
        final int ORDMAX = 30;
        int P[] = new int[M + 1];
        int PAGENUMBER;
        int PAGEOFFSET;
        int ROWOFFSET;
        int C;
        int J; // 현재의 값. 
        int K; // 현재의 index 
        boolean JPRIME;
        int ORD;
        int SQUARE;
        int N;
        int MULT[] = new int[ORDMAX + 1]; // 소인수들의 배수. ex) 3 * 3, 3 * 5, 5 * 7 ...

        J = 1;
        K = 1;
        P[1] = 2;
        ORD = 2;
        SQUARE = 9;

        while (K < M) {
            do {
                J = J + 2;
                // 1. 해당 값이 SQUARE과 같을 때, Mult 배열의 새로운 index에 새로운 값을 추가한다. 
                if (J == SQUARE) {
                    ORD = ORD + 1;
                    SQUARE = P[ORD] * P[ORD];
                    MULT[ORD - 1] = J;
                }
                N = 2;
                JPRIME = true;
                while (N < ORD && JPRIME) {
                    // 2. 해당 값이 Mult 배열의 n index의 값보다 클때, Mult 배열의 기존 index에 새로운 값을 업데이트한다. 
                    while (MULT[N] < J) {
                        MULT[N] = MULT[N] + P[N] + P[N];
                    }
                    // 3. 해당 값이 Mult 배열의 n index의 값과 같을 때, 비로소 소수가 아님을 판정한다. 
                    if (MULT[N] == J) {
                        JPRIME = false;
                    }
                    N = N + 1;
                }
            } while (!JPRIME);

            K = K + 1;
            // 4. 3번의 경우가 아닌 모든 해당 값들은 소수가 판단하여 primes 배열에 추가한다. 
            P[K] = J;
        }
        {
            PAGENUMBER = 1;
            PAGEOFFSET = 1;
            while (PAGEOFFSET <= M) {
                System.out.println("The First " + M +
                        " Prime Numbers --- Page" + PAGENUMBER);
                System.out.println();
                for (ROWOFFSET = PAGEOFFSET; ROWOFFSET < PAGEOFFSET + RR; ROWOFFSET++) {
                    for (C = 0; C < CC; C++) {
                        if (ROWOFFSET + C * RR <= M) {
                            System.out.format("%10d", P[ROWOFFSET + C * RR]);
                        }
                    }
                    System.out.println();
                }
                System.out.println("\f");
                PAGENUMBER = PAGENUMBER + 1;
                PAGEOFFSET = PAGEOFFSET + RR * CC;
            }
        }
    }
}
```
=> 함수가 하나뿐인 위 프로그램은 엉망진창이다. 들여쓰기가 심하고, 이상한 변수가 많고, 구조가 빡빡하게 결합되었다. **최소한 여러 함수로 나눠야 마땅하다.** 

* 목록 10-6에서 목록 10-8까지는 목록 10-5를 작은 함수와 클래스로 나눈 후 함수와 클래스와 변수에 **좀 더 의미있는 이름**을 부여한 결과다.

> 목록 10-6 PrimePrinter.java 
```java
public class PrimePrinter {
    public static void main(String[] args) {
        final int NUMBER_OF_PRIMES = 1000;
        int[] primes = PrimeGenerator.generate(NUMBER_OF_PRIMES);

        final int ROWS_PER_PAGE = 50;
        final int COLUMNS_PER_PAGE = 4;
        RowColumnPagePrinter tablePrinter = new RowColumnPagePrinter(
                ROWS_PER_PAGE,
                COLUMNS_PER_PAGE,
                "The First" + NUMBER_OF_PRIMES + " Prime Numbers"
        );
        tablePrinter.print(primes);
    }
}
```

> 목록 10-7 RowColumnPagePrinter.java
```java
public class RowColumnPagePrinter {
    private int rowsPerPage;
    private int columnsPerPage;
    private int numbersPerPage;
    private String pageHeader;
    private PrintStream printStream;

    public RowColumnPagePrinter(int rowsPerPage,
                                int columnsPerPage,
                                String pageHeader) {
        this.rowsPerPage = rowsPerPage;
        this.columnsPerPage = columnsPerPage;
        this.pageHeader = pageHeader;
        numbersPerPage = rowsPerPage * columnsPerPage;
        printStream = System.out;
    }

    public void print(int data[]) {
        int pageNumber = 1;
        for (int firstIndexOnPage = 0;
             firstIndexOnPage < data.length;
             firstIndexOnPage += numbersPerPage) {
            int lastIndexOnPage = Math.min(firstIndexOnPage + numbersPerPage - 1, data.length - 1);
            printPageHeader(pageHeader, pageNumber);
            printPage(firstIndexOnPage, lastIndexOnPage, data);
            printStream.println("\f");
            pageNumber++;
        }
    }

    private void printPageHeader(String pageHeader, int pageNumber) {
        printStream.println(pageHeader + " --- Page " + pageNumber);
        printStream.println();
    }

    private void printPage(int firstIndexOnPage, int lastIndexOnPage, int[] data) {
        int firstIndexOfLastRowOnPage = firstIndexOnPage + rowsPerPage - 1;
        for(int firstIndexInRow = firstIndexOnPage;
            firstIndexInRow <= firstIndexOfLastRowOnPage;
            firstIndexInRow++) {
            printRow(firstIndexInRow, lastIndexOnPage, data);
            printStream.println();
        }
    }

    private void printRow(int firstIndexInRow, int lastIndexOnPage, int[] data) {
        for (int column = 0; column < columnsPerPage; column++) {
            int index = firstIndexInRow + column * rowsPerPage;
            if (index <= lastIndexOnPage) {
                printStream.format("%10d", data[index]);
            }
        }
    }

    private void setOutput(PrintStream printStream) {
        this.printStream = printStream;
    }
}
```

> 목록 10-8 PrimeGenerator.java
```java
public class PrimeGenerator {
    private static int[] primes;
    private static ArrayList<Integer> multiplesOfPrimeFactors;

    protected static int[] generate(int n) {
        primes = new int[n];
        multiplesOfPrimeFactors = new ArrayList<>();
        set2AsFirstPrime();
        checkOddNumbersForSubsequentPrimes();
        return primes;
    }

    private static void set2AsFirstPrime() {
        primes[0] = 2;
    }

    private static void checkOddNumbersForSubsequentPrimes() {
        int primeIndex = 1;
        for (int candidate = 3;
            primeIndex < primes.length;
            candidate += 2) {
            if(isPrime(candidate)) {
                primes[primeIndex++] = candidate;
            }
        }
    }

    private static boolean isPrime(int candidate) {
        if (isLeastRelevantMutipleOfNextLargerPrimeFactor(candidate)) {
            multiplesOfPrimeFactors.add(candidate);
            return false;
        }
        return isNotMultipleOfAnyPreviousPrimeFactor(candidate);
    }

    private static boolean isLeastRelevantMutipleOfNextLargerPrimeFactor(int candidate) {
        int nextLargerPrimeFactor = primes[multiplesOfPrimeFactors.size()];
        int leastRelevantMultiple = nextLargerPrimeFactor * nextLargerPrimeFactor;
        return candidate == leastRelevantMultiple;
    }

    private static boolean isNotMultipleOfAnyPreviousPrimeFactor(int candidate) {
        for (int n = 1; n < multiplesOfPrimeFactors.size(); n++) {
            if (isMultipleOfNthPrimeFactor(candidate, n)) {
                return false;
            }
        }
        return true;
    }

    private static boolean isMultipleOfNthPrimeFactor(int candidate, int n) {
        return candidate == smallestOddNthMultipleNotLessThanCandidate(candidate, n);
    }

    private static int smallestOddNthMultipleNotLessThanCandidate(int candidate, int n) {
        int multiple = multiplesOfPrimeFactors.get(n);
        while (multiple < candidate) {
            multiple += 2 * primes[n];
        }
        multiplesOfPrimeFactors.set(n, multiple);
        return multiple;
    }
}
```

길이가 늘어난 이유는 여러 가지다.

* 첫째, 리팩터링한 프로그램은 좀 더 길고 서술적인 변수 이름을 사용한다. 
* 둘째, 리팩터링한 프로그램은 코드에 주석을 추가하는 수단으로 함수 선언과 클래스 선언을 활용한다. 
* 셋째, 가독성을 높이고자 공백을 추가하고 형식을 맞추었다. 

### 원래의 프로그램이 세가지 책임으로 나눠졌다

* PrimePrinter: main 함수 하나만 포함하며 실행 환경을 책임진다. 호출 방식이 달라지면 클래스도 바뀐다. 예를 들어, 프로그램을 [SOAP 서비스](https://ko.wikipedia.org/wiki/SOAP)로 바꾸려면 PrimePrinter를 고쳐준다.
* RowColumnPagePrinter: 숫자 목록을 주어진 행과 열에 맞춰 페이지에 출력하는 클래스다. 출력하는 모양새를 바꾸려면 RowColumnPagePrinter 클래스를 고쳐준다. 
* PrimeGenerator: 소수 목록을 생성하는 방법을 안다. 소수를 계산하는 알고리즘이 바뀐다면 PrimeGenerator 클래스를 고쳐준다.   

### 인상적인 한마디: 테스트를 통해 구조를 바꾼다.  

* 가장 먼저, 원래 프로그램의 정확한 동작을 검증하는 테스트 슈트를 작성했다.
* 그런 다음, 한번에 하나씩 수차례에 걸쳐 조금씩 코드를 변경했다. 코드를 변경할 때마다 **테스트를 수행해 원래 프로그램과 동일하게 동작하는지 확인**했다.

# 변경하기 쉬운 클래스 

깨끗한 시스템은 클래스를 체계적으로 정리해 변경에 수반하는 위험을 낮춘다. 즉 SideEffect 없이 변경하기 쉬운 클래스다. 

> 목록 10-9 변경이 필요해 '손대야' 하는 클래스 
```java
public class Sql {
    public Sql(String table, Column[] columns)
    public String create() 
    public String insert(Object[] fields)
    public String selectAll()
    public String findByKey(String keyColumn, String keyValue) 
    public String select(Column column, String pattern) 
    public String select(Criteria criteria)
    public String preparedInsert() {}
    private String columnList(Column[] columns)
    private String valueList(Object[] fields, final Column[] columns)
    private String selectWithCriteria(String criteria)
    private String placeholderList(Column[] columns)
}
```
=> 아직 미완성이라 update문과 같은 일부 SQL 기능을 지원하지 않는다. **update 문을 지원할 시점이 오면 클래스에 '손대어' 고쳐야 한다.** 문제는 코드에 손대면 위험이 생긴다는 것이다. 다른 코드를 망가뜨릴 잠정적인 위험이 존재하고 그래서 테스트도 완전히 다시 해야 한다.

* 새로운 SQL 문을 지원하려면 반드시 Sql 클래스에 손대야 한다. 
* 또한 기존 SQL 문 하나를 수정할 때도 반드시 Sql 클래스에 손대야 한다. 
<br>=> 이렇게 변경할 이유가 두 가지이므로 Sql 클래스는 SRP를 위반한다. 

단순히 구조적인 관점에서도 Sql은 SRP를 위반한다. 메서드를 쭉 훑어보면 selectWithCriteria라는 비공개 메서드가 있는데, 이 메서드는 select문을 처리할때만 사용한다. 

> 목록 10-10 닫힌 클래스 집합 
```java
abstract public class Sql {
    public Sql(String table, Column[] columns) 
    abstract public String generate()
}

public class CreateSql extends Sql {
    public CreateSql(String table, Column[] columns)
    @Override public String generate()
}

public class SelectSql extends Sql {
    public SelectSql(String table, Column[] columns) 
    @Override public String generate()
}

public class InsertSql extends Sql {
    public InsertSql(String table, Column[] columns) 
    @Override public String generate()
    private String valueList(Object[] fields, final Column[] columns) 
}

public class SelectWithCriteriaSql extends Sql {
    public SelectWithCriteriaSql(String table, Column[] columns) 
    @Override public String generate()
}

public class SelectWithMatchSql extends Sql {
    public SelectWithMatchSql(String table, Column[] columns) 
    @Override public String generate()
}

public class FindByKeySql extends Sql {
    public FindByKeySql(String table, Column[] columns) 
    @Override public String generate()
}

public class PreparedInsertSql extends Sql {
    public PreparedInsertSql(String table, Column[] columns)
    @Override public String generate()
    private String placeholderList(Column[] columns)
}

public class Where {
    public Where(String criteria)
    public String generate()
}

public class ColumnList {
    public ColumnList(Column[] columns)    
    public String generate()
}
```

각 클래스는 극도로 단순하다. 
**코드는 순식간에 이해된다.** 
함수 하나를 수정했다고 다른 함수가 망가질 위험도 사실상 사라졌다. 
테스트 관점에서 모든 논리를 구석구석 증명하기도 쉬워졌다. 
**클래스가 서로 분리되었기 때문이다.** update 문을 추가할 때 기존 클래스를 수정할 필요없이 Sql 클래스에서 **새 클래스 UpdateSql을 상속받아 거기에 넣으면 된다.** SRP도 준수하고 있고 OCP도 제공한다.

* 이상적인 시스템이라면 새 기능을 추가할 때 **시스템을 확장할 뿐 기존 코드를 변경하지는 않는다.**

## 변경으로부터 격리 

OOP에는 구체적인(`concrete`) 클래스와 인터페이스 타입(스위프트의 프로토콜 타입)이 있다. 

 * 구체적인 클래스는 상세한 구현(코드)을 포함한다.
 * 반면에 추상(`abstract`) 클래스(및 인터페이스 타입)는 개념만 포함한다.  

**요구사항은 변하기 마련이다. 따라서 코드도 변하기 마련이다.**
근데 상세한 구현에 의존하는 클래스는 구현(코드)이 바뀌면 위험에 빠진다. 
그래서 인터페이스와 추상 클래스를 사용해 구현이 미치는 상황을 격리한다.
인터페이스의 함수가 바뀌지 않는 한, 요구사항이 바뀌어도 구현이 격리되었기 때문에 위험에 빠지지 않는다. 

### 인터페이스 타입은 테스트에 유용하다. Test Double!

상세한 구현에 의존하는 코드는 테스트하기 어렵다. 
예를 들어 `Portfolio` 라는 객체가 있고 해당 클래스가 5분마다 값이 달라지는 `TokyoStockExchange` API를 사용한다고 했을 때, 해당 API를 이용하는 `Portfolio` 코드를 테스트하기란 쉽지 않다.

따라서 아래 코드처럼 인터페이스 타입을 이용해보자.
```java
public interface StockExchange {
    Money currentPrice(String symbol);
}
```
=> 테스트할 때 5분마다 값이 달라지는 API를 대신하는 가짜 객체를 생성하기 위해 `StockExchange` 라는 인터페이스를 생성하고 메서드를 추가한다.

다음으로 `StockExchange` 인터페이스를 구현하는 `TokyoStockExchange` 클래스를 구현한다. 또 Portfolio 생성자를 수정해 `StockExchange` 참조자를 인수로 받는다. 

```java
public class Portfolio {
    private StockExchange exchange;

    public Portfolio(StockExchange exchange) {
        this.exchange = exchange;
    }
    
    // ...
}
```

이제 `TokyoStockExchange` 클래스를 흉내내는 테스트용 클래스(Test Double)를 만들 수 있다. 테스트용 클래스는 StockExchange 인터페이스를 구현하며 **고정된 주가를 반환한다.** 

```java
public class PortfolioTest {
    private FixedStockExchangeStub exchange;
    private Portfolio portfolio;

    @Before
    protected void setUp() throws Exception {
        exchange = new FixedStockExchangeStub();
        exchange.fix("MSFT", 100);
        portfolio = new Portfolio(exchange);
    }
    
    @Test
    public void GivenFiveMSFTTotalShouldBe500() throws Exception {
        portfolio.add(5, "MSFT");
        Assert.assertEquals(500, portfolio.value());
    }
}
```
=> 테스트에서 마이크로소프트 주식을 다섯 주를 구입한다면 테스트용 클래스는 언제나 100불을 반환한다. 우리 테스트용 클래스는 단순히 미리 정해놓은 표 값(`exchange.fix("MSFT", 100)`)만 참조한다. 그러므로 우리는 **전체 포트폴리오 총계가 500불인지 확인하는 테스트 코드(`GivenFiveMSFTTotalShouldBe500`)를 작성할 수 있다.** 

* 위와 같은 테스트가 가능할 정도로 시스템의 결합도를 낮추면 **유연성과 재사용성도 더욱 높아진다.** 시스템 요소가 서로 잘 격리되어 있으면 각 요소를 이해하기도 더 쉬워진다.

### DIP: 의존성 역전 원칙

이렇게 결합도를 최소로 줄이면 자연스럽게 또 다른 클래스 원칙인 DIP(`Dependency Inversion Principle`) 를 따르는 클래스가 나온다. 본질적으로 **DIP는 클래스가 상세한 구현이 아니라 추상화에 의존해야 한다는 원칙**이다. 