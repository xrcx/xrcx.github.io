##　JUnit 基础
[JUnit4](http://junit.org/junit4/)
#### build.gradle 配置
```java 
testCompile 'junit:junit:4.12'
```
如果你通过AndroidStudio创建一个项目，这个dependency默认是加上了的，这步可以省略。

### 注解

- @BeforeClass, 该方法所在测试类运行之前会被执行，并且在一个类中只会执行一次，可以setup一些公共的资源，该方法必须是静态的

- @Before , 该方法在每次测试方法调用前都会调用

- @Test , 说明了该方法需要测试

- @After , 该方法在每次测试方法调用后都会调用

- @AfterClass 该方法所在测试类运行结束之后会被执行，并且在一个类中也只会执行一次,可以release一些公共的资源，该方法必须是静态的

- @Ignore 忽略该方法

```java
  public class JUnitLifeCycle {

    @BeforeClass
    public static void prepareDataForTest(){
        System.out.println("------method prepareDataForTest called------\n");
    }

    @Before
    public void setUp(){
        System.out.println("------method setUp called------");
    }

    @Test
    public void test1(){
        System.out.println("------method test1 called------");
    }

    @After
    public void tearDown(){
        System.out.println("------method tearDown called------");
    }

    @AfterClass
    public static void finish(){
        System.out.println("------method finish called------");
    }
}
```
调用顺序：prepareDataForTest-->setUp-->test1-->tearDown-->setUp-->test2-->teatDown...-->finish,这样便可以保证各个独立的测试之间互不干扰，以免其它测试代码修改测试环境或者测试数据影响到其它测试代码的准确性。



#### @Test注解的异常测试和时间测试
##### 异常测试(expected)
 参数 expected 代表测试方法期望抛出指定的异常，如果运行测试并没有抛出这个异常，则 JUnit 会认为这个测试没有通过。验证被测试方法在错误的情况下是否会抛出预定异常

``` java
    public class Calculator {
        // Omit testAdd() and testMultiply() for brevity
        public double divide(double divident, double dividor) {
            if (dividor == 0) throw new IllegalArgumentException("Dividor cannot be 0");
                return divident / dividor;
        }
    }
```

   当传入的除数是0的时候，该方法应该抛出IllegalArgumentException异常

``` java
    public class CalculatorTest {
        Calculator mCalculator;

        @Before
        public void setup() {
            mCalculator = new Calculator();
        }

        // Omit testAdd() and testMultiply() for brevity

        @Test(expected = IllegalArgumentException.class)
        public void test() {
            mCalculator.divide(4, 0);
        }
    }
```

   @Test(expected = IllegalArgumentException.class)表示验证这个测试方法将抛出IllegalArgumentException异常，如果没有抛出的话，则测试失败。


##### 时间测试(timeout)
被测试方法被允许运行的最长时间应该是多少，如果测试方法运行时间超过了指定的毫秒数，则 JUnit 认为测试失败。

```java
@Test(timeout = 1000)  
    public void selfXMLReader(){  
        // ...  
}  
```

### Assert
- assertEquals(expected, actual)，验证expected的值跟actual是否一样。如果传入的是object，用equals()进行对比。

- assertEquals(expected, actual, tolerance)

    传入的expected和actual是float或double类型的，tolerance是偏差值，如果两个数的差异在这个偏差值之内，则测试通过，否者测试失败。

- assertTrue(boolean condition)，验证contidion的值是true

- assertFalse(boolean condition)，验证contidion的值是false

- assertNull(Object obj)，验证obj的值是null

- assertNotNull(Object obj)，验证obj的值不是null

- assertSame(expected, actual)，验证expected和actual是同一个对象，即指向同一个对象

- assertNotSame(expected, actual)，验证expected和actual不是同一个对象，即指向不同的对象
```java
    @Test
    public void sample_assert_same(){
        //arrange
        Object expect = new Object();
        Object expectNotSame = new Object();
        Object actual = expect;

        //assert
        Assert.assertSame(expect, actual);
        Assert.assertNotSame(expectNotSame, actual);
    }
```

- assertThat(actual, Matcher<? super T> matcher)

```java
    assertThat(0,is(1)); //失败：
    assertThat(0,is(not(1)));//通过
```

- fail() , 让测试方法失败
```java
    @Test
    public void testException(){
        FirstUtil tester= new FirstUtil();
        String expected = "Hello JUnit";
        String result = "";
        try {
            result = tester.printWithDanger(0);
        }catch (Exception e){
            fail("exception is:"+e.getMessage());
        }
        assertEquals(expected,result);
    }
```

注意：上面的每一个方法，都有一个重载的方法，可以在前面加一个String类型的参数，表示如果验证失败的话，将用这个字符串作为失败的结果报告。
eg：
``` java
assertEquals("Current user Id should be 1", 1, currentUser.id());
```
当currentUser.id()的值不是1的时候，在结果报道里面将显示"Current user Id should be 1"，这样可以让测试结果更具有可读性，更清楚错误的原因是什么。



###  参数化测试
- JUnit 允许在测试类中使用参数。该类可以包含一个测试方法，并且使用提供的不同参数来执行该方法。
- @RunWith(Parameterized.class) 注释将测试类标记为参数化测试。
- 必须包含使用注释注释的静态方法@Parameters。该方法生成并返回数组的集合。该集合中的每个项目都用作测试方法的参数。
- 可以使用@Parameter公共字段上的注释来获取测试中注入的测试值。

```java
@RunWith(Parameterized.class)
@SmallTest
public class CalculatorAddParameterizedTest {
    /**
     * @return {@link Iterable} that contains the values that should be passed to the constructor.
     * In this example we are going to use three parameters: operand one, operand two and the
     * expected result.
     */
    @Parameters
    public static Iterable<Object[]> data() {
        return Arrays.asList(new Object[][]{
                {0, 0, 0},
                {0, -1, -1},
                {2, 2, 4},
                {8, 8, 16},
                {16, 16, 32},
                {32, 0, 32},
                {64, 64, 128}});
    }

    private final double mOperandOne;
    private final double mOperandTwo;
    private final double mExpectedResult;

    private Calculator mCalculator;

    /**
     * Constructor that takes in the values specified in
     * {@link CalculatorAddParameterizedTest#data()}. The values need to be saved to fields in order
     * to reuse them in your tests.
     */
    public CalculatorAddParameterizedTest(double operandOne, double operandTwo,
            double expectedResult) {
        mOperandOne = operandOne;
        mOperandTwo = operandTwo;
        mExpectedResult = expectedResult;
    }

    @Before
    public void setUp() {
        mCalculator = new Calculator();
    }

    @Test
    public void testAdd_TwoNumbers() {
        double resultAdd = mCalculator.add(mOperandOne, mOperandTwo);
        assertThat(resultAdd, is(equalTo(mExpectedResult)));
    }

}
```


## hamcrest

- 可以有效增加 JUnit 的测试能力，用一些相对通俗的语言来进行测试。
- 需要导入 `hamcrest-library-1.3.jar` 包，如果已导入 `hamcrest-all-1.3.jar` 包则不需导入此包
- 要使用 JUnit 中的assertThat来进行断言，第一个参数表示实际值，第二个参数表示hamcrest的表达式。

### 常用的比较方式
##### 逻辑
  - allOf - 如果所有匹配器都匹配才匹配, (像 Java &&)
  - anyOf - 如果任何匹配器匹配就匹配, (像 Java ||)
  - not - 如果包装的匹配器不匹配器时匹配,反之亦然
  
##### 对象
  - equalTo - 测试对象相等指定对象，相当于Object.equals方法。
  - hasToString - 测试Object.toString方法。
  - instanceOf, isCompatibleType - 测试类型。
  - notNullValue, nullValue - 测试null。
  - sameInstance - 测试对象实例。
  - hasProperty - 测试JavaBeans属性。
#####集合
  - array - 测试一个数组元素test an array’s elements against an array of matchers。
  - hasEntry, hasKey, hasValue - 测试一个Map包含一个实体,键或者值。
  - hasItem, hasItems - 测试一个集合包含一个元素。
  - hasItemInArray - 测试一个数组包含一个元素。
##### 数字
  - closeTo - 测试浮点值接近给定的值。
  - greaterThan, greaterThanOrEqualTo, lessThan, lessThanOrEqualTo - 测试次序。
##### 文本
  - equalToIgnoringCase - 测试字符串相等忽略大小写。
  - equalToIgnoringWhiteSpace - 测试字符串忽略空白。
  - containsString, endsWith, startsWith - 测试字符串匹配。

```java
    @Test
    public void testHamcrest() {
        //首先需要静态导入：import static org.hamcrest.Matchers.*;
        //判读50是否大于20并且小于60，具体的hamcrest的比较参数可以在文档中查询
        assertThat(50, allOf(greaterThan(20),lessThan(60)));
        //判读某个字符串是否以另一个字符串结尾
        assertThat("abc.txt", endsWith("txt"));

        assertThat("foo", equalTo("foo"));
        assertThat(true, hasToString(equalTo("TRUE")));
        assertThat(Integer.class, typeCompatibleWith(Number.class));
        assertThat(cheese, is(instanceOf(Cheddar.class)));
        assertThat(myBean, hasProperty("foo"));

        assertThat(new Integer[]{1,2,3}, is(array(equalTo(1), equalTo(2), equalTo(3))));
        assertThat(myMap, hasEntry("bar", "foo"));
        assertThat(Arrays.asList("foo", "bar"), hasItem("bar"));
        assertThat(Arrays.asList("foo", "bar", "baz"), hasItems("baz", "foo"));

        assertThat(1.03, is(closeTo(1.0, 0.03)));

        assertThat("Foo", equalToIgnoringCase("FOO"));
    }
```

## TestSuite
可以通过TestSuite来组成多个测试组件。
```java
//RunWith表示这个类是一个suite的类
@RunWith(Suite.class)
//说明这个类中包含哪些测试组件
@SuiteClasses({TestA.class,
               TestB.class,
               TestCalculate.class})
public class TestSuite {
    /*
     * 测试原则：
     * 1、建议创建一个专门的source folder-->test来编写测试类代码
     * 2、测试类的包应该保持和需要测试的类一致
     * 3、测试单元中的每一个测试方法都必须可以独立执行，没有顺序
     * 4、测试方法之间不能有任何的依赖性
     */
}
```

