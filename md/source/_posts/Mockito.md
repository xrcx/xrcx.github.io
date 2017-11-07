
## 为什么要使用Mock工具
在做单元测试的时候，我们会发现我们要测试的方法会引用很多外部依赖的对象，比如：（发送邮件，网络通讯，远程服务, 文件系统等等）。 而我们没法控制这些外部依赖的对象，为了解决这个问题，我们就需要用到Mock工具来模拟这些外部依赖的对象，来完成单元测试。

## 什么是Mock
所谓的mock就是创建一个类的虚假的对象，在测试环境中，用来替换掉真实的对象，以达到两大目的：

- 验证这个对象的某些方法的调用情况，调用了多少次，参数是什么等等
- 指定这个对象的某些方法的行为，返回特定的值，或者是执行特定的动作


## Mock 测试框架 - [Mockito](http://site.mockito.org/)
[GitHub](https://github.com/mockito/mockito)
Mockito 是一个基于MIT协议的开源 Java 测试框架，它能够 Mock 对象、验证结果以及为测试用例打桩。

### Mockito Android支持
```java
 dependencies {
   testCompile "org.mockito:mockito-core:2.+"
 }
```
### 用法大纲
>- mock()/ @Mock：创建模拟
>- verify() ：验证Mock对象的方法是否被调用。  Mockito.verify(objectToVerify).methodToVerify(arguments)
- when().thenReturn("")/when().thenThrow()，打桩,指定mock对象的某些方法的行为  Mockito.when(mockObject.targetMethod(args)).thenReturn(desiredReturnValue)
- when().thenAnswer() - 为回调做测试桩 
- anyInt() , anyLong() ,  anyDouble(),  anyObject(),  any(clazz)等等 ： 参数设置-任意的Int类型，任意的Long类型，任意的Double类型,任何对象，任何属于clazz的对象...等
- times() ：调用Mock对象的方法次数
- atLeastOnce()，atMost(count) ,  atLeast(count) ,  never() ：至少一次，最多次数，最少次数，从不
- doNothing()/doAnswer()/doThrow() - 指定一个方法执行特定的动作   Mockito.doAnswer(desiredAnswer).when(mockObject).targetMethod(args)
- InOrder - 按顺序验证
- spy,部分模拟，doReturn("foo").when(spy).get(0)

### 详细用法
#### mock()/ @Mock - 创建Mock对象
- mock()
```java
List mockedList = mock(List.class);  
```
- @Mock注解创建,需要使用MockitoAnnotations.initMocks(this),进行注入
```java
@Mock
List mockedList;  

@Test
public void testMock(){
    //初始化@Mock注解的对象 (进行注入)
    MockitoAnnotations.initMocks(this);
}  
```
-  @Mock注解创建,使用MockitoJUnitRunner
```java
 @RunWith(MockitoJUnitRunner.StrictStubs.class)
 public class ExampleTest {

     @Mock
     private List list;

     @Test
     public void shouldDoSomething() {
         list.add(100);
     }
 }

```
-  @Mock注解创建,使用MockitoRule
```java
 public class ExampleTest {

     //Creating new rule with recommended Strictness setting
     @Rule public MockitoRule rule = MockitoJUnit.rule().strictness(Strictness.STRICT_STUBS);

     @Mock
     private List list;

     @Test
     public void shouldDoSomething() {
         list.add(100);
     }
 }
```



#### verify() - 验证某些行为
> Mock对象将记住所有的操作，可以验证其行为

```java
//使用Mock对象
mockedList.add("one");
mockedList.clear();

//验证函数的调用次数
verify(mockedList).add("one");
//verify(mockedList).add("two"); 无此操作，验证 failed
verify(mockedList).clear();  
```

#### when().thenReturn()/when().thenThrow() - 打桩
> 在创建出Mock对象后，默认返回值为null，故可做进行打桩，每当调用该打桩后的方法或变量后，让其返回相应的值。

```java
//测试桩，在调用get(0)时返回"first"
when(mockedList.get(0)).thenReturn("first");
//调用get(1)时抛出异常
when(mockedList.get(1)).thenThrow(new RuntimeException());

//输出first
System.out.println(mockedList.get(0));
//抛出异常
//System.out.println(mockedList.get(1));
//因为get(999)没有打桩，因此输出null
System.out.println(mockedList.get(999)); 
```

##### 为连续的调用做测试桩(Stub)
通过这个注解，可实现自动注入mock对象。

```java
/*when(mockedList.get(anyInt()))
            .thenThrow(new RuntimeException())
            .thenReturn("foo");*/

//第一次调用:抛出运行时异常
System.out.println(mockedList.get(0));
//第二次调用:输出"foo"
System.out.println(mockedList.get(1));
//第三次调用:也是输出"foo"
System.out.println(mockedList.get(2));
```
替代方案，短版连续桩
```java
when(mockedList.get(anyInt()))
        .thenReturn("one", "two", "three");  
```


#### when().thenAnswer() - 为回调做测试桩
```java
when(mockedList.get(anyInt())).thenAnswer(new Answer<String>() {
    @Override
    public String answer(InvocationOnMock invocation) throws Throwable {
        //获取函数调用的参数
        Object[] args = invocation.getArguments();
        //获得Mock对象本身
        Object mock = invocation.getMock();
        return "answer===>" + mock.toString();
    }
});

System.out.println(mockedList.get(50));

//doReturn(),doThrow(),doAnswer(),doNothing(),noCallRealMethod()  
```


#### anyInt()/anyString() - 参数匹配器
> 让打桩更具灵活性，比如anyInt()将匹配所有的int值

```java
//使用内置的anyInt()参数匹配器，当调用get(int)时都返回"element"
when(mockedList.get(anyInt())).thenReturn("element");
//使用自定义的参数器(在inValid()函数中返回你自己的匹配器实现)
//when(mockedList.get(isValid())).thenReturn("element");

//输出element
System.out.println(mockedList.get(999));
//也可以验证匹配器
//verify(mockedList).get(anyInt()); 
```

#### 验证函数的确切调用次数、最少调用、从未调用
+ times(x) ,几次
+ atLeastOnce()，至少一次
+ atLeast(x)，至少x次
+ atMost(x)，最多x次
+ never()，从不
```java
mockedList.add("once");

mockedList.add("twice");
mockedList.add("twice");

mockedList.add("three times");
mockedList.add("three times");
mockedList.add("three times");

//下面两个的验证结果一样，因为verify默认验证的就是times(1)
verify(mockedList).add("once");
verify(mockedList, times(1)).add("once");

//验证具体的执行次数
verify(mockedList, times(2)).add("twice");
verify(mockedList, times(3)).add("three times");

//使用never验证，never相当于time(0)
verify(mockedList, never()).add("never happened");

//使用atLeast()/atMost
verify(mockedList, atLeastOnce()).add("three times");
verify(mockedList, atLeast(2)).add("twice");
verify(mockedList, atMost(5)).add("three times");

List mockTwo = mock(List.class);  
//验证Mock对象没有交互过
//verifyZeroInteractions(mockedList); //mockedList已交互过
verifyZeroInteractions(mockTwo); //mockTwo没有交互过
```

#### doNothing()/doAnswer()/doThrow()/doReturn()/doCallRealMethod() - 指定一个方法执行特定的动作 
> 这个功能一般是用在目标的方法是void类型的时候。

- 返回空的方法上
- Spy 对象的方法上
- 对同一个方法多次打桩，以改变在测试中间的模拟的行为。
##### doAnswer()
```java
public void loginCallbackVersion(String username, String password) { 
    if (username == null || username.length() == 0) return; 
    //假设我们对密码强度有一定要求，使用一个专门的validator来验证密码的有效性 
    if (mPasswordValidator.verifyPassword(password)) return; 
    //login的结果将通过callback传递回来。
     mUserManager.performLogin(username, password, new  NetworkCallback() { 
          @Override public void onSuccess(Object data) { 
              //update view with data
           }
           @Override public void onFailure(int code, String msg) {
             //show error msg
           }
     });
}
```

在测试环境下，我们并不想依赖mUserManager.performLogin的真实逻辑，而是让mUserManager直接调用传入的NetworkCallback的onSuccess或onFailure方法。这种指定mock对象执行特定的动作的写法如下：
`Mockito.doAnswer(desiredAnswer).when(mockObject).targetMethod(args)`,传给doAnswer()的是一个Answer对象，我们想要执行什么样的动作，就在这里面实现。

```java
Mockito.doAnswer(new Answer() {
    @Override
    public Object answer(InvocationOnMock invocation) throws Throwable { 
        //这里可以获得传给performLogin的参数 
        Object[] arguments = invocation.getArguments(); //callback是第三个参数
        NetworkCallback callback = (NetworkCallback) arguments[2]; 
        callback.onFailure(500, "Server error"); 
        return 500; 
    }
}).when(mockUserManager).performLogin(anyString(), anyString(),any(NetworkCallback.class));

```
##### doThrow()
```java
   doThrow(new RuntimeException()).when(mockedList).clear();

   //following throws RuntimeException:
   mockedList.clear();
```

#### InOrder - 按顺序验证
```java
// Multiple mocks that must be used in a particular order
 List firstMock = mock(List.class);
 List secondMock = mock(List.class);

 //using mocks
 firstMock.add("was called first");
 secondMock.add("was called second");

 //create inOrder object passing any mocks that need to be verified in order
 InOrder inOrder = inOrder(firstMock, secondMock);

 //following will make sure that firstMock was called before secondMock
 inOrder.verify(firstMock).add("was called first");
 inOrder.verify(secondMock).add("was called second");

```

#### Spy
> Mock对象只能调用stubbed方法，没有调用不了它真实的方法。
> 但Mockito可以监视一个真实的对象，这时对它进行方法调用时它将调用真实的方法，同时也可以stubbing这个对象的方法让它返回我们的期望值。
> 另外不论是否是真实的方法调用都可以进行verify验证。和创建mock对象一样，对于final类、匿名类和Java的基本类型是无法进行spy的。
##### 监视对象
监视一个对象需要调用spy(T object)方法，如：List spy = spy(new LinkedList());那么spy变量就在监视LinkedList实例。
用`doReturn|Answer|Throw()`，stubbing Spy对象的方法 ，最好不用 `when(Object)`

```java
List list = new LinkedList();

//监控一个真实的对象
List spy = spy(list);
//可以为某些函数打桩
when(spy.size()).thenReturn(100);

//在监控真实对象上使用when会报错，可以使用onReturn、Answer、Throw()函数族来进行打桩
//不能:因为当调用spy.get(0)时会调用真实对象的get(0)函数，此时会发生IndexOutOfBoundsException异常，因为真实List对象是空的
when(spy.get(0)).thenReturn("foo");
// 正确用法
doReturn("foo").when(spy).get(0);

System.out.println(spy.get(0));

//Mockito并不会为真实对象代理函数(Method)调用，实际上它会复制真实对象。
//当你在监控一个真实对象时，你想为这个真实对象的函数做测试桩，那么就是在自找麻烦。

//通过spy对象调用真实对象的函数
spy.add("one");
spy.add("two");

System.out.println(spy.get(0));
System.out.println(spy.size());

//交互验证
verify(spy).add("one");
verify(spy).add("two");
```

##### @InjectMocks
> mock对象自动注入，注意：使用了@Mock注解，必须使用@RunWith(MockitoJUnitRunner.class) 或 MockitoAnnotations.initMocks(this) 或 MockitoRule  进行mocks的初始化和注入。

**区别**

- @Mock: 创建一个Mock.
- @InjectMocks: 创建一个实例，其余用@Mock（或@Spy）注解创建的mock将被注入到用该实例中。

测试目标代码：
```java
public LoginPresenter(UserManager userManager, PasswordValidator passwordValidator) {
    this.mUserManager = userManager;
    this.mPasswordValidator = passwordValidator;
}
```
测试用例代码：

```java
public class LoginPresenterTest {

    @Rule
    public MockitoRule mockitoRule = MockitoJUnit.rule();

    @Mock
    UserManager mockUserManager;
    @Mock
    PasswordValidator mockValidator;

    @InjectMocks
    LoginPresenter loginPresenter;

    @Test
    public void testLogin() {
        Mockito.when(mockValidator.verifyPassword("xiaochuang is handsome")).thenReturn(true);
        loginPresenter.login("xiaochuang", "xiaochuang is handsome");

        verify(mockUserManager).performLogin("xiaochuang", "xiaochuang is handsome");
    }
}
```

- LoginPresenter 被 @InjectMocks 修饰，他的构造方法所需要的两个参数正好这个测试类里面有这样的field被@Mock修饰过，所以 mockUserManager 和 mockValidator就构造出了 LoginPresenter，这种叫做 `Constructor Injection`

``` java
 public class LoginPresenter {
    private UserManager mUserManager;
    private PasswordValidator mPasswordValidator;

    public LoginPresenter() {
    }
}
```

- 如果 LoginPresenter 只有一个无参构造函数，Mockito依然会将LoginPresenter里面的mUserManager和mPasswordValidator初始化为LoginPresenterTest里面的两个mock对象，mockUserManager和mockValidator。这个效果跟LoginPresenter有两个构造参数是一样的,这个叫 `Field Injection`

```java
public class LoginPresenter {
    private UserManager mUserManager;
    private PasswordValidator mPasswordValidator;

    public LoginPresenter(UserManager userManager) {
        this.mUserManager = userManager;
    }
}
```
- 如果LoginPresenter只有一个UserManager的构造方法,只有mUserManager会被初始化为mockUserManager, 而mPasswordValidator是不会初始化为mockValidator的
 
>此外还有`Property Setter Injection`，也就是通过setter方法，自动的初始化@InjectMocks对象
>
这三种injection的优先级顺序分别为： Constructor Injection > Property Setter Injection > Field Injection。

>所以不推荐使用 `@InjectMocks`,因为很难弄清楚到底会通过哪种方式inject


#### 为下一步的断言捕获参数
>ArgumentCaptor与自定义的参数匹配器相关 
>这两种技术都能用于检测外出传递到Mock对象的参数 
>rgumentCaptor更适合以下情况:
> 1. 自定义不能被重用的参数匹配器 
> 2. 仅需要断言参数值

- ArgumentCaptor
```java
//在某些场景中，不光要对方法的返回值和调用进行验证，同时需要验证一系列交互后所传入方法的参数。那么我们可以用参数捕获器来捕获传入方法的参数进行验证，看它是否符合我们的要求。
mockedList.add("Haha");
ArgumentCaptor argument = ArgumentCaptor.forClass(String.class);
verify(mockedList).add(argument.capture());
assertEquals("Haha", argument.getValue());  
```
- @Captor
```java
public class MockitoTests {

    @Rule
    public MockitoRule rule = MockitoJUnit.rule();

    @Captor
    private ArgumentCaptor<List<String>> captor;

    @Test
    public final void shouldContainCertainListItem() {
        List<String> asList = Arrays.asList("someElement_test", "someElement");
        final List<String> mockedList = mock(List.class);
        mockedList.addAll(asList);

        verify(mockedList).addAll(captor.capture());
        final List<String> capturedArgument = captor.getValue();
        assertThat(capturedArgument, hasItem("someElement"));
    }
}
```