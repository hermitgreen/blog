# Fragment

碎片是一种可以嵌在当前活动当中的UI片段，能让程序更加合理和充分利用空间，《Android编程权威指南》推荐开发最好使用碎片进行
在我的个人理解里面，碎片像是几个Layout的组合，就和Windows上我们经常在电脑屏幕上开多个窗口一样

## 碎片的使用

首先需要碎片布局文件

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:textSize="20sp"
        android:text="This is fragment"/>

</LinearLayout>
```

碎片的width和height虽然是match_parent，但是不会充满整个屏幕，因为它的父布局是Fragment控件

接下来创建Fragment类，为解决兼容性问题，通常继承support-v4库中的Fragment

```java
public class mFragment extends Fragment {
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState){
        return inflater.inflate(R.layout.fragment,container,false);
    }
}
```

在这个方法中，将R.layout.fragment动态加载进来了

在activity_main.xml中，fragment使用类似于其他控件

```xml
<fragment
        android:id="@+id/fragment"
        android:name="com.example.hermitgreen.fragmenttest.mFragment"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="match_parent"
        />
```

注意，碎片是必须配置name属性，显式指明要添加的碎片类名的

## 动态加载碎片

通常，若是需要动态加载碎片，通过FrameLayout来划分这片区域，因为FrameLayout不需要任何定位，很适合充满区域的碎片，注意FrameLayout不需要配置name属性

```xml
<fragment
        android:id="@+id/left_fragment"
        android:name="com.example.hermitgreen.fragmenttest.LeftFragment"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="match_parent"/>

<FrameLayout
        android:id="@+id/right_layout"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="match_parent">

</FrameLayout>
```

然后向FrameLayout中添加东西即可

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        replaceFragment(new mFragment());
    }

    private void replaceFragment(Fragment fragment){
        FragmentManager fragmentManager = getSupportFragmentManager();
        FragmentTransaction transaction = fragmentManager.beginTransaction();
        transaction.replace(R.id.right_layout,fragment);
        transaction.addToBackStack(null);
        transaction.commit();
    }
}
```

在`replaceFragment()`方法中，活动中可以通过`getSupportFragmentManager()`获得FragmentManager，然后调用FragmentManager的`beginTransaction()`开启一个事务，这个事务暂且就当作类似于Intent的东西吧！接下来类似于Intent的，对这个FragmentTransaction进行修改
`replace()`方法第一个参数是要加载的碎片布局，第二个参数是该碎片布局对应的类
`addToBackStack(null)`方法开启碎片的模拟返回栈，说人话就是让碎片可以类似于Layout一样返回上一页
最后用`commit()`方法提交事务

## 碎片与活动之间通信

在活动中可以调用碎片的方法,只需要利用FragmentManager的`findFragmentById()`方法

```java
OtherFragment otherFragment = (otherFragment) getSupportFragmentManager().findFragmentById(R.id.other_fragment);
```

碎片中也可以调用活动的方法

```java
MainActivity mainActivity = (MainActivity) getActivity();
```