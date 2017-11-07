





## 本地单元测试
1. 配置
``` java
dependencies {
    testCompile 'junit:junit:4.12'
}
```


[slide] 

## Junit

## AndroidJUnitRunner 
是JUnit测试运行器，可以在Android设备上执行JUnit3或JUnit4中风格的测试类，兼容Espresso和UI Automator测试框架。测试运行器加载测试包和应用，运行测试并报告测试结果。
[Android 测试支持库](http://developer.android.com/tools/testing-support-library/index.html)一部分

1. 注解
- @RequiresDevice来指定该条测试只运行在物理设备，而不是模拟设备
- @SdkSupress(miniSdkVersion=18) 限制在指定的Android设备上运行
- @LargeTest @MediumTest @SmallTest 用于将测试用例按照重要程度进行分类，在进行测试的时候可以选择只运行某个类别的测试用例。
    + @SmallTest 标志该测试方法是小型测试的一部分
    + @MediumTest 标志该测试方法是中等测试的一部分
    + @LargeTest 标志该测试方法是大型测试的一部分

注意：Instrumented JUnit 4测试类在设备或仿真器上运行，必须在前面加上@RunWith(AndroidJUnit4.class)注释。

[slide]
2. 访问instrumentation的API
InstrumentationRegistry.getInstrumentation() 返回当前正在运行的Instrumentation
InstrumentationRegistry.getContext() 返回此Instrumentation软件包的上下文。
InstrumentationRegistry.getTargetContext() 返回目标应用的应用上下文。
InstrumentationRegistry.getArguments() 返回传递给此Instrumentation的参数Bundle。当你要访问命令行参数时非常有用。 

参考:http://www.cnblogs.com/JianXu/p/5175945.html

[slide] 
## Mockito

[slide] 
## Powermockito
https://my.oschina.net/jackieyeah/blog/157076
[slide] 
## Robolectric



- mockito invokeMock
- 