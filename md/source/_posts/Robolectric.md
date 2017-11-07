# Robolectric
- [GitHub]()
- [官网}()
  Robolectric 测试框架针对 Android 的组件（包含各种View）进行了统一的 Shadow，使得我们不再依赖模拟器或真机，直接就单元测试就可方便地测试我们的 UI。

### 配置
```java
testCompile "org.robolectric:robolectric:3.3.2"
```
第一次下载 robolectric 依赖库太慢了，给个解决方案

##### 方案一，单独下载jar，再拷贝
1. 添加配置如下
```java
testCompile "org.robolectric:robolectric:3.3.2"
testCompile group: 'org.robolectric', name: 'android-all', version: '7.1.0_r7-robolectric-0' 
```
第二句是让gradle从 https://jcenter.bintray.com 下载 android-all-7.1.0_r7-robolectric-0 依赖库

2. 拷贝jar包
从jcenter下载的库，保存在 C:\Users\{User Name}\.gradle\caches\modules-2\files-2.1\目录下。
robolectric下载的库放在本地目录 ~/.m2/repository/
所以将 C:\Users\{User Name}\.gradle\caches\modules-2\files-2.1\ 拷贝到 ~/.m2/repository/ 中。
即：拷贝C:\Users\{User Name}\.gradle\caches\modules-2\files-2.1\org.robolectric\android-all\4.1.2_r1-robolectric-0\aecc8ce5119a25fcea1cdf8285469c9d1261a352\android-all-4.1.2_r1-robolectric-0.jar
到$user.home\.m2\repository\org\robolectric\android-all\4.1.2_r1-robolectric-0\

##### 方案二，把oss.sonatype.org改成阿里云maven仓库（推荐）
```java
buildscript {
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.3'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }
        jcenter()
    }
}
```


### 简单例子
app里面有两个Activity：MainActivity和SecondActivity，MainActivity里面有一个TextView，点击一下这个TextView将跳转到SecondActivity。
测试目标方法：
``` java
public class MainActivity extends AppCompatActivity {  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  

        TextView textView = (TextView)findViewById(R.id.textView1);  
        textView.setOnClickListener(new OnClickListener() {  
            @Override  
            public void onClick(View v) {  
                startActivity(new Intent(MainActivity.this, SecondActivity.class));  
            }  
        });  
    }  
}  
```
测试用例代码：
```java
@RunWith(RobolectricTestRunner.class) 
@Config(constants = BuildConfig.class)  
public class MainActivityTest {  
    @Test  
    public void testMainActivity() {  
        // 创建 MainActivity 的 instanc，启动 Activity，默认会调用Activity的生命周期: onCreate->onStart->onResume
        MainActivity mainActivity = Robolectric.setupActivity(MainActivity.class);  
        // 触发点击事件
        mainActivity.findViewById(R.id.textView1).performClick();  

        Intent expectedIntent = new Intent(mainActivity, SecondActivity.class); 
        // 获取mainActivity对应的ShadowActivity的instance
        ShadowActivity shadowActivity = Shadows.shadowOf(mainActivity);  
        // 借助Shadow类获取启动下一Activity的Intent
        Intent actualIntent = shadowActivity.getNextStartedActivity();  
        // 校验Intent的正确性
        Assert.assertEquals(expectedIntent, actualIntent);  
    }  
} 
```
- @RunWith(RobolectricTestRunner.class) - 用Robolectric的TestRunner来跑这些test
- @Config - 来配置一些东西。
- Robolectric.setupActivity 返回的时候，这个Activity已经完成了onCreate、onStart、onResume这几个生命周期的回调了。

## 配置
### @Config配置
定制Robolectric的运行时的行为。单个测试类或方法配置Robolectric，在方法级别指定的值将覆盖在类级别指定的值。

##### 1. 配置SDK版本
```java
@Config(sdk = Build.VERSION_CODES.JELLY_BEAN)
public class SandwichTest {

    @Config(sdk = Build.VERSION_CODES.KITKAT)
    public void getSandwich_shouldReturnHamSandwich() {
    }
}
```

##### 2. 配置Application类

> Robolectric会根据manifest文件配置的Application配置去实例化一个Application类，如果你想在测试用例中重新指定，可以通过下面的方式配置

```java
@Config(application = CustomApplication.class)
public class SandwichTest {

    @Config(application = CustomApplicationOverride.class)
    public void getSandwich_shouldReturnHamSandwich() {
    }
}
```

##### 3. 指定Resource路径

> Robolectric可以让你配置manifest、resource和assets路径，可以通过下面的方式配置

```java
@Config(manifest = "some/build/path/AndroidManifest.xml",
        assetDir = "some/build/path/assetDir",
        resourceDir = "some/build/path/resourceDir")
public class SandwichTest {

    @Config(manifest = "other/build/path/AndroidManifest.xml")
    public void getSandwich_shouldReturnHamSandwich() {
    }
}
```

##### 4. 使用第三方Library Resources

> 当Robolectric测试的时候，会尝试加载所有应用提供的资源，但如果你需要使用第三方库中提供的资源文件，你可能需要做一些特别的配置。不过如果你使用gradle来构建Android应用，这些配置就不需要做了，因为Gradle Plugin会在build的时候自动合并第三方库的资源，但如果你使用的是Maven，那么你需要配置libraries变量

```java
@RunWith(RobolectricTestRunner.class)
@Config(libraries = {
    "build/unpacked-libraries/library1",
    "build/unpacked-libraries/library2"
})
public class SandwichTest {
}
```

##### 5. 使用限定的资源文件
> Android会在运行时加载特定的资源文件，如根据设备屏幕加载不同分辨率的图片资源、根据系统语言加载不同的string.xml，在Robolectric测试当中，你也可以进行一个限定，让测试程序加载特定资源.多个限定条件可以用破折号拼接在在一起。

```java
/**
 * 使用qualifiers加载对应的资源文件
 *
 * @throws Exception
 */
@Config(qualifiers = "zh-rCN")
@Test
public void testString() throws Exception {
    final Context context = RuntimeEnvironment.application;
    assertThat(context.getString(R.string.app_name), is("单元测试Demo"));
}
```

### Properties文件
如果觉得通过注解配置上面的东西麻烦，也可以把以上配置放在一个Properties文件之中，然后通过@Config指定配置文件，比如，首先创建一个配置文件robolectric.properties，然后把robolectric.properties文件放到src/test/resources目录下，运行的时候，会自动加载里面的配置

    # 放置Robolectric的配置选项:
    sdk=21
    manifest=some/build/path/AndroidManifest.xml
    assetDir=some/build/path/assetDir
    resourceDir=some/build/path/resourceDir

### 系统属性配置
- robolectric.offline：true代表关闭运行时获取jar包
- robolectric.dependency.dir：当处于offline模式的时候，指定运行时的依赖目录
- robolectric.dependency.repo.id：设置运行时获取依赖的Maven仓库ID，默认是sonatype
- robolectric.dependency.repo.url：设置运行时依赖的Maven仓库地址，默认是https://oss.sonatype.org/content/groups/public/
- robolectric.logging.enabled：设置是否打开调试开关

以上设置可以通过Gradle进行配置，如：

```java
android {
    testOptions {
        unitTests.all {
            systemProperty 'robolectric.dependency.repo.url', 'https://local-mirror/repo'
            systemProperty 'robolectric.dependency.repo.id', 'local'
        }
    }
}
```

## 使用大纲
- Robolectric.buildActivity() - 可以测试生命周期
- ​

|                             | 说明           | 例子   |
| --------------------------- | ------------ | ---- |
| Robolectric.buildActivity() | 初始化的Activity |    Robolectric.buildActivity(MyAwesomeActivity.class).create().start().get()  |
|                             |              |      |
|                             |              |      |
|                             |              |      |
|                             |              |      |
|                             |              |      |
|                             |              |      |
|                             |              |      |
|                             |              |      |

## 详情
### Robolectric.buildActivity()
- 基本用法
```java
ActivityController controller = Robolectric.buildActivity(MyAwesomeActivity.class).create().start();
Activity activity = controller.get();
// assert that something hasn't happened
activityController.resume();
// assert it happened!
```
- 测试完整的创建生命周期
```java
Activity activity = Robolectric.buildActivity(MyAwesomeActivity.class).create().start().resume().visible().get();
```
- 用Intent 启动 Activity
```java
Intent intent = new Intent(Intent.ACTION_VIEW);
Activity activity = Robolectric.buildActivity(MyAwesomeActivity.class).withIntent(intent).create().get();
```
- 恢复保存的实例状态
```java
Bundle savedInstanceState = new Bundle();
Activity activity = Robolectric.buildActivity(MyAwesomeActivity.class)
                               .create()
                               .restoreInstanceState(savedInstanceState)
                               .get();
```

###  Robolectric.setupActivity()
创建 Activity 的instance，当Robolectric.setupActivity返回的时候，这个Activity已经完成了onCreate、onStart、onResume这几个生命周期的回调了。

### performClick() 模拟用户的点击
> 使用clickOn需要在buildActivity后调用visible方法，否则会得到一个错误。

```java
LoginActivity activity = Robolectric.buildActivity(LoginActivity.class).create().visible().get();
activity.findViewById(R.id.textView1).performClick();
```

### getNextStartedActivity() - 取得最近启动的Activity


[Robolectric使用教程](http://blog.hanschen.site/2016/12/10/robolectric.html)