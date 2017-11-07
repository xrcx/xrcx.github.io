# PowerMock
- [GitHub](https://github.com/powermock/powermock)
- [官网](http://powermock.github.io/)

## 为什么要使用PowerMock
现如今比较流行的Mock工具如 jMock 、EasyMock 、Mockito等都有一个共同的缺点：不能mock静态、final、私有方法等。而PowerMock能够完美的弥补以上三个Mock工具的不足。

## PowerMock简介
PowerMock 也是一个单元测试模拟框架，它是在其它单元测试模拟框架的基础上做出的扩展。通过提供定制的类加载器以及一些字节码篡改技巧的应用，PowerMock 现了对静态方法、构造方法、私有方法以及 Final 方法的模拟支持，对静态初始化过程的移除等强大的功能。因为 PowerMock 在扩展功能时完全采用和被扩展的框架相同的 API, 熟悉 PowerMock 所支持的模拟框架的开发者会发现 PowerMock 非常容易上手。PowerMock 的目的就是在当前已经被大家所熟悉的接口上通过添加极少的方法和注释来实现额外的功能。

## 入门
### 配置
```java
dependencies {
    testCompile 'junit:junit:4.12'
    // PowerMock brings in the mockito dependency
    testCompile 'org.powermock:powermock-module-junit4:1.6.5'
    testCompile 'org.powermock:powermock-module-junit4-rule:1.6.5'
    testCompile 'org.powermock:powermock-api-mockito:1.6.5'
    testCompile 'org.powermock:powermock-classloading-xstream:1.6.5'
}
```

### 注解：
- @RunWith
- @PrepareForTest - 是声明需要进行mock的静态类
- @PowerMockIgnore - 声明package路径，表示不使用PowerMockito来加载所声明的package路径的类。

> 注：如果你的测试用例里没有使用注解 `@PrepareForTest`，那么可以不用加注解 `@RunWith(PowerMockRunner.class)`，
> 反之亦然。当你需要使用PowerMock强大功能(Mock静态、final、私有方法等)的时候，就需要加注解 `@PrepareForTest`。

**声明**

- 使用注解方式
```java
@RunWith(PowerMockRunner.class)
@PrepareForTest({YourClassWithEgStaticMethod.class})  
```
- 注解和 Rule 的方式
```java
@PrepareForTest({YourClassWithEgStaticMethod.class})

@Rule
public PowerMockRule rule = new PowerMockRule();
```
**声明多个静态类**

`@PrepareForTest({Example1.class, Example2.class, ...})`

## 基本用法预览
- PowerMockito.mock() 
    + 创建 mock 对象  
    + `PowerMockito.mock(File.class);`

- PowerMockito.when().thenReturn() 
    + 指定 mock 对象的某些方法的行为
    + `PowerMockito.when(file.exists()).thenReturn(true);`

- PowerMockito.whenNew().withArguments().thenReturn() 
    + 模拟构造函数 
    + `PowerMockito.whenNew(File.class).withArguments(directoryPath).thenReturn(directoryMock);`

- PowerMockito.mockStatic(Utils.class)
    + 模拟静态
    + `PowerMockito.mockStatic(CommonExample.class);`

- PowerMockito.verifyNew().withArguments();
    + 验证指定构造函数是否被调用    
    + `PowerMockito.verifyNew(File.class).withArguments(directoryPath);`   

- PowerMockito.verifyStatic();
    - 验证静态方法是否被调用
    - `PowerMockito.verifyStatic();`

- PowerMockito.verifyPrivate().invoke()
    + 验证私有方法是否被调用
    + `verifyPrivate(commonExample).invoke("isExist");`
    
## 基本用法详解
### 1. 普通Mock：Mock参数传递的对象
> 普通Mock不需要加 `@RunWith` 和 `@PrepareForTest` 注解

测试目标代码：
```java
public class CommonExample {
    public boolean callArgumentInstance(File file) {
        return file.exists();
    }
}
```
测试用例代码：
```java
public class CommonExampleTest {
    @Test
    public void testCallArgumentInstance() {
        // 创建出一个mock对象
        File file = PowerMockito.mock(File.class);    
        CommonExample commonExample = new CommonExample();
        // 指定这个mock对象具体的行为
        PowerMockito.when(file.exists()).thenReturn(true);
        // 将mock对象作为参数传递个测试方法，执行测试方法
        Assert.assertTrue(commonExample.callArgumentInstance(file));
    }
}
```
### 2. 模拟构造方法
> 需要添加注解
> - @Rule
> - @PrepareForTest ,里面的类是需要mock的new对象代码所在的类
> 
> 可以很好的模拟构造函数,从而使被测代码中 new 操作返回的对象可以被随意定制，会很大程度的提高单元测试的效率

测试目标代码：
```java
public class CommonExample {
    public boolean createDirectoryStructure(String directoryPath) {
        File directory = new File(directoryPath);
        if (directory.exists()) {
            String msg = "\"" + directoryPath + "\" 已经存在.";
            throw new IllegalArgumentException(msg);
        }
        return directory.mkdirs();
    }
}
```
测试用例代码：
```java

public class CommonExampleTest {
    @Rule
    public PowerMockRule rule = new PowerMockRule();

    @Test
    @PrepareForTest(CommonExample.class)
    public void testMockConstructorMethod() throws Exception {
        final String directoryPath = "seemygod";
        //创建File的模拟对象
        File directoryMock = PowerMockito.mock(File.class);

        //在当前测试用例下,当出现new File("seemygod")时,就返回模拟对象
        PowerMockito.whenNew(File.class).withArguments(directoryPath).thenReturn(directoryMock);

        //当调用模拟对象的exists时,返回false
        PowerMockito.when(directoryMock.exists()).thenReturn(false);
        //当调用模拟对象的mkdirs时,返回true
        PowerMockito.when(directoryMock.mkdirs()).thenReturn(true);
        Assert.assertTrue(new CommonExample().createDirectoryStructure(directoryPath));
        //验证new File(directoryPath);  是否被调用过
        PowerMockito.verifyNew(File.class).withArguments(directoryPath);
    }
}
```

### 3. Mock final方法
> 需要添加注解
> - @Rule
> - @PrepareForTest(ClassDependency.class)，注解里写的类是需要mock的final方法所在的类

测试目标代码：
```java
public class CommonExample {
    public final boolean callFinalMethod() {
        return false;
    }
}
```
测试用例代码：
```java
public class CommonExampleTest {
    @Rule
    public PowerMockRule rule = new PowerMockRule();

    @Test
    @PrepareForTest(CommonExample.class)
    public void testCallFinalMethod() throws Exception {
        CommonExample commonExample = PowerMockito.mock(CommonExample.class);
        PowerMockito.when(commonExample.callFinalMethod()).thenReturn(true);
        Assert.assertTrue(commonExample.callFinalMethod());
    }
}
```


### 4. Mock 静态方法
> 需要添加注解
> - @Rule
> - @PrepareForTest(ClassDependency.class)，注解里写的类是静态方法所在的类

测试目标代码：
``` java
public class CommonExample {
     public static String callStaticMethod() {
        return UUID.randomUUID().toString();
    }
}
```
测试用例代码：
``` java
public class PowerMockSampleTest {
    @Rule
    public PowerMockRule rule = new PowerMockRule();

    @Test
    @PrepareForTest(CommonExample.class)
    public void testCallStaticMethod() throws Exception {
        PowerMockito.mockStatic(CommonExample.class);
        PowerMockito.when(CommonExample.callStaticMethod()).thenReturn("FAKE UUID");
        Assert.assertEquals("FAKE UUID", CommonExample.callStaticMethod());
        //验证静态方法是否被调用
        PowerMockito.verifyStatic();
    }
}
```


### 5.Mock 私有方法
> 需要添加注解
> - @Rule
> - @PrepareForTest(ClassDependency.class)，注解里写的类是私有方法所在的类

测试目标方法：
```java
public class CommonExample {
    public boolean callPrivateMethod() {
        return isExist();
    }
    private boolean isExist() {
        return false;
    }
}
```
测试用例代码：

``` java
public class PowerMockSampleTest {
    @Rule
    public PowerMockRule rule = new PowerMockRule();

    @Test
    @PrepareForTest(CommonExample.class)
    public void testCallPrivateMethod() throws Exception {
        CommonExample commonExample = spy(new CommonExample());

        PowerMockito.when(commonExample.callPrivateMethod()).thenCallRealMethod();
        PowerMockito.when(commonExample, "isExist").thenReturn(true);
        Assert.assertTrue(commonExample.callPrivateMethod());
        verifyPrivate(commonExample).invoke("isExist");
    }
}
```



### 6. Mock 私有变量
> 不需要加注解@PrepareForTest和@RunWith，而是使用Whitebox来mock私有变量mState并注入你预设的变量值。

测试目标方法：
```java
public class CommonExample {
    private static final int STATE_NOT_READY = 0;
    private static final int STATE_READY = 1;
    private int mState = STATE_NOT_READY;

    public boolean doSomethingIfStateReady() {
        if (mState == STATE_READY) 
            return true;
        else  
            return false;
    }
}
```
测试用例代码：

``` java
public class PowerMockSampleTest {
    @Rule
    public PowerMockRule rule = new PowerMockRule();

    @Test
    public void testPrivateProperty() throws Exception {
        CommonExample sample = new CommonExample();
        Whitebox.setInternalState(sample, "mState", 1);
        Assert.assertTrue(sample.doSomethingIfStateReady());
    }
}
```

## PowerMock简单实现原理
1. 当某个测试方法被注解 `@PrepareForTest` 标注以后，在运行测试用例时，会创建一个新的 `org.powermock.core.classloader.MockClassLoader` 实例，然后加载该测试用例使用到的类（系统类除外）。

2. PowerMock会根据你的mock要求，去修改写在注解 `@PrepareForTest` 里的class文件（当前测试类会自动加入注解中），以满足特殊的mock需求。例如：去除'final方法的final标识，在静态方法的最前面加入自己的虚拟实现等。

3. 如果需要mock的是系统类的final方法和静态方法，PowerMock不会直接修改系统类的class文件，而是修改调用系统类的class文件，以满足mock需求。