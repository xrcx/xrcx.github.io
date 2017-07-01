# DataBinding #
## 绑定事件处理方法 ##
[参考链接](http://blog.csdn.net/crazyman2010/article/details/53590007 "参考链接")

（一）Method References（方法引用）

1. 类似于在布局文件中定义Android:onClick属性
2. 在编译的时候就绑定了
3. 注意只要签名一致，即自己定义的方法的签名(signature)要和处理事件用的listener完全一样

#### 示例 ####
1. 创建响应方法

		public class MyHandler {
		    public void myOnClick(View view){
		        Toast.makeText(view.getContext(),"hello",Toast.LENGTH_SHORT).show();
		    }
		    public boolean myOnLongClick(View view){
		        Toast.makeText(view.getContext(),"hello world",Toast.LENGTH_SHORT).show();
		        return true;
		    }
		}

2. 在布局文件中设置

		<layout xmlns:android="http://schemas.android.com/apk/res/android">
	    <data>
	        <variable name="handlers" type="com.heyy.databingexample.MyHandler"/>
	    </data>
	    <LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:orientation="vertical">
	        <TextView
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:text="测试事件绑定"
	            android:onClick="@{handlers::myOnClick}"
	            android:onLongClick="@{handlers::myOnLongClick}"/>
	    </LinearLayout>
	</layout>
3. 在代码中进行绑定

		@Override
   	 	protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        ActivityMainBinding activityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);
	        activityMainBinding.setHandlers(new MyHandler());
    	}

（二）Listener Bindings（监听绑定）

- 在事件发生以后才绑定的
- 不再要求签名一致，而只是要求返回值一样

1. 创建响应方法
   多了一个参数

		public class MyHandler {
		    public void myOnClick(View view, String name) {
		        Toast.makeText(view.getContext(), "hello " + name, Toast.LENGTH_SHORT).show();
		    }
		}

2. 在布局文件中设置

    注意，传递String参数时使用了"CrazyMan&quot，引号等需要使用HTML字符实体。

		<layout xmlns:android="http://schemas.android.com/apk/res/android">
	    <data>
	        <variable name="handlers" type="com.heyy.databingexample.MyHandler"/>
	    </data>
	    <LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:orientation="vertical">
	
	        <TextView
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:text="测试事件绑定"
	            android:onClick="@{(view)->handlers.myOnClick(view,&quot;CrazyMan&quot;)}"
	            android:onLongClick="@{handlers::myOnLongClick}"/>
	    </LinearLayout>
	</layout>
3. 在代码中绑定

		@Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        ActivityMainBinding activityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main);
	        activityMainBinding.setHandlers(new MyHandler());
	    }


#### 两种方法的比较 ####

1. Method References方法要求比较低(gradle plugin 1.5+)，Listener Bindings要求高(gradle plugin 2.0+)。
1. Method References要求响应方法的签名和事件listener中的签名完全一致，Listener Bindings只要求返回值一致。
1. Method References在编译时绑定，Listener Bindings在运行时才会执行绑定lambda表达式。
1. Method References不可以自定义参数，Listener Bindings可以自定义参数。
1. Listener Bindings可以编写简单的代码逻辑。