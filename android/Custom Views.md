# Custom Views

## 引入布局

在xml中，要引入一个布局，只需要`<include layout="@layout/name" />`

当然，被引用的布局要小于当前布局

## 创建自定义控件

要创建自定义控件，首先要创建继承自Layout的类，以LinearLayout为例

```java
public class CustomLayout extends LinearLayout {

    public CustomLayout (Context context, AttributeSet attrs) {
        super(context, attrs);
        LayoutInflater.from(context).inflate(R.layout.custom, this);
        //在这里处理custom.xml的各种逻辑
        //TODO
    }
}
```

在构造函数中，使用LayoutInflater对Layout进行动态加载，`LayoutInflater.from()`方法构建出一个LayoutInflater对象，context为上下文。然后调用`inflate()`方法加载布局，`inflate()`方法的第一个参数为布局文件，第二个参数为布局文件的父布局，由于我们的控件直接在当前页面使用，父布局传入this。注意，不能new一个新的LayoutInflater对象

要使用，只需要以"包名.类名"的方式，和正常控件一样使用

```xml
<com.example.packageName.CustomLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    />
```