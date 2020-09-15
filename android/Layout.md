# Layout

布局的作用是让各个控件有条不紊，通过多层布局的嵌套可以编写出复杂的界面，安卓中有四中最基本的布局：线性布局，相对布局，帧布局和百分比布局，除此之外还有一些用的很少的布局，在以后我用到时会更新上来。一个布局必须配置其的宽和高，表示这个布局在屏幕中占的范围

## 线性布局

通常线性布局的控件在线性方向上依次排列

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <!--一些控件-->

</LinearLayout>
```

orientation指定了线性方向，这里指定了方向为竖直方向，也可以通过改为horizontal变为横向

layout_gravity属性指定了控件在布局中的对齐方式，可选值与gravity类似，需要注意，当布局对齐方式为horizontal时，只有垂直方向的对齐可以生效，反之亦然

layout_weight属性指定控件在布局中的宽度权重，当处理控件宽度时，系统会将所有的weight相加，然后取相应控件的比例，注意：当使用weight时，width需要设置为0dp
通常，我们在编写一个界面的时候可以设定好按钮的宽度，同时把另一个控件weight设置为1，这样，另一个控件就会充满屏幕

## 相对布局

相对布局中有大量的属性，通常一个控件是相对于其他控件存在的

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <!--一些控件-->

</RelativeLayout>
```

既然是相对布局，那么相应的其布局属性也有以下几种

### 相对父布局

`layout_alignParentTop="true"`
这个属性表示位于父布局顶层，相应的属性还有xxxBottom，xxxStart等，不必多说

`layout_centerInParent="true"` 这个属性表示位于父布局的正中间

### 相对控件

`layout_above="@id/id"`
位于id对应控件的上方

`layout_below="@id/id"`
下方

`layout_toStartOf="@id/id"`
左方

`layout_toEndOf="@id/id"`
右方

还有一种是根据边缘对齐的
`layout_alignStart="@id/id"`
表示和左边缘对齐，类似的和xx边缘对齐

## 帧布局

帧布局没有方便的定位方式，所有控件默认摆放在布局的左上角，通常在Fragment中使用

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <!--一些控件-->

</FrameLayout>
```

仍然可以使用`layout_gravity`属性指定控件在布局中的对齐方式

## 百分比布局

百分比布局是新增布局，被定义在support库中，要使用百分比布局，首先在build.gradle中添加百分比库的依赖

打开app/build.gradle文件，在dependencies闭包中添加

```groovy
dependencies{
    compile 'com.android.support:percent:27.1.1'
}
```

这里的版本号需要填入你自己的版本号

修改完成后，点Sync now，同步gradle

然后在xml中使用百分比布局

```xml
<android.support.percent.PercentFrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <!--一些控件-->

</android.support.percent.PercentFrameLayout >
```

由于百分比布局并没有内置在系统SDK中，所以需要把完整的包路径写出来，然后还必须要新定义一个app的命名空间，否则无法使用百分比布局的自定义属性。其自定义属性以app:开头，而不是android:

下面以Button为例

```xml
<Button
    android:id="@+id/button"
    android:text="button"
    android:layout_gravity="center|bottom"
    app:layout_widthPercent="20%"
    app:layout_heightPercent="20%"
    />
```

可以看到，配置`layout_widthPercent`和`layout_heightPercent`属性即可确定控件大小相对于父布局的百分比