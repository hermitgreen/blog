# Widget

Android提供了大量的UI控件，使用这些UI控件可以减少工作量，并一定程度上美化网站（以拯救程序员的贫穷审美观）

## 控件的属性

通常，宽高是必须配置的属性

```xml
android:layout_width="xxdp"//宽度
android:layout_height="xxdp"//高度
```

width和height有两个特殊属性match_parent和warp_content，前者指定控件的大小由父布局决定，后面指定控件大小刚好包含住里面的内容

下面是一些非必须的共有属性

```xml
android:id="@+id/your_id"//添加ID
android:text="Text"//文本
android:gravity="start"//内部（文字）的对齐方式
android:visibility="visible"//可见性
android:ellipsize="end"//文本缩略方式
android:paddingxxxx="10dp"//边缘留白
```

文本对齐的可选值有top bottom start end center 可以用“|”来同时指定多个值，另外，在新标准中，left和right被start和end代替，应该避免使用它们

可见性的可选值有visible（默认），invisible，gone。visible表示控件可见，invisible表示不可见，但是仍然占据着位置，gone表示控件不再占用屏幕空间。在java代码中，通过`getVisibility()`方法获得当前可见性，通过`setVisibility(arg)`方法设置可见性，方法的参数是View.GONE等值

## 控件的使用

以Button为例

通常要使用一个控件，首先要在xml中定义这个控件，配置其的一系列属性

```xml
<Button
    android:id="@+id/button"
    android:layout_width="xxdp"
    android:layout_height="xxdp"
    android:text="Test Button"
    />
```

然后，在java代码中设置其如何运行

一般来说，声明相应的变量，由于其实例必须在Activity类的各个方法类可以被访问，故要声明为私有全局变量

```java
private Button button;
```

然后，我们要获取的实例在onCreate中被启动，故我们要在onCreate中获取实例

```java
//onCreate
button = findViewById(R.id.button);
```

`findViewById()`方法接收一个参数：ID，返回该ID对应的控件实例

最后在要使用到这个控件的情景中处理逻辑。对Button而言通常就是在onCreate中为Button注册监听器

```java
//onCreate
button.setOnClickListener(new View.OnClickListener(){
    @Override
    public void onClick(View v){
        //TODO
    }
});
```

## 常见的控件

这些控件通常是Google或其他公司编写好的，我们可以通过内置的方法去修改控件的初始属性，这意味着控件不是在xml中被编写好后就一成不变的

### TextView

TextView主要用于在界面上显示一段文本信息，需要配置文本内容

```xml
<TextView
    android:id="@+id/text_view"
    android:layout_width="xxdp"
    android:layout_height="xxdp"
    android:textSize="24sp"
    android:textColor="#00ff00"
    />
```

修改TextView的内容

```java
textView.setText("Another String");
```

### EditText

EditText用于用户输入

```xml
<EditText
    android:id="@+id/edit_text"
    android:layout_width="xxdp"
    android:layout_height="xxdp"
    android:hint="hint"
    android:maxLines="2"
    />
```

hint为提示性文本，maxLines是控件内部显示文本的最大行数

获取EditText输入的文本

```java
String inputText = editText.getText().toString;
```

### Button

按钮应该是最重要的控件之一

```xml
<Button
    android:id="@+id/button"
    android:layout_width="xxdp"
    android:layout_height="xxdp"
    android:textSize="24sp"
    android:textColor="#00ff00"
    android:textAllCaps="false"
    />
```

textAllCaps为英文自动大写转换，配置这一项关闭英文大写转换

下面来看看如何使用匿名类注册监听器

```java
Button button = findViewById(R.id.button);
button.setOnClickListener(new View.OnClickListener(){
    @Override
    public void onClick(View v){
        //TODO
    }
});
```

也可用实现接口的方式注册监听器，不过不常用

```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

    @Override
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = findViewById(R.id.button);
        button.setOnClickListener(this);
    }

    @Override
    public void onClick(View v){
        switch(v.getId()){
            case R.id.button:
            //TODO
                break;
            default:
                break;
        }
    }
}
```

对Button来说，也经常在它的监听器中去处理其他的控件的逻辑，比如处理一段输入文本

```java
private EditText editText;
private Button button;
//onCreate
editText = findViewById(R.id.edit_text);
button = findViewById(R.id.button);
button.setOnClickListener(new View.OnClickListener(){
    @Override
    public void onClick(View v){
        String inputText = editText.getText.toString();
        //TODO
    }
});
```

### ImageView

ImageView是用于在界面上展示图片的一个控件，图片通常放在以“drawable”开头的目录下，后缀“-xhdpi”指分辨率的高低程度

ImageView需要配置图片路径

```xml
<ImageView
    android:id="@+id/image_view"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@drawable/img_1"
    />
```

修改ImageView的图片

```java
imageView.setImageResource(R.drawable.img_2);
```

### ProgressBar

ProgressBar用于显示一个进度条

```xml
<ProgressBar
    android:id="@+id/progress_bar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    />
```

默认为圆形进度条，可以通过配置style来变成水平进度条

```xml
<ProgressBar
    android:id="@+id/progress_bar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    style="?android:attr/progressBarStyleHorizontal"
    android:max="100"
    />
```

这里的max可以自己设定表示进度条的进度最大值

获取进度值

```java
int progress = progressBar.getProgress();
```

更改进度

```java
progressBar.setProgress(progress);
```

`setProgress()`方法接收一个小于max的int型数

当完成时，使进度条消失

```java
progressBar.setVisibility(View.GONE);
```

### AlertDialog

AlertDialog可以在当前界面弹出一个对话框，能够屏蔽掉其他控件。他的使用有点特殊，由于是弹出窗口，所以并不是在xml中定义，而是在java代码中通过`AlertDialog.Builder()`构造，其只有一个参数，指定Dialog的上下文

```java
AlertDialog.Builder dialog = new AlertDialog.Builder(MainActivity.this);

dialog.setTitle("Title");
dialog.setMessage("Message");
dialog.setCancelable(false);

dialog.setPositiveButton("OK", new DialogInterface.OnClickListener(){
    @Override
    public void onClick(DialogInterface dialog, int which){
        //TODO
    }
});

dialog.setNegativeButton("Cancel", new DialogInterface.OnClickListener(){
    @Override
    public void onClick(DialogInterface dialog, int which){
        //TODO
    }
});

dialog.show();
```

通过setxxx可以为Dialog设置一系列的属性，一般必须要设置的有Title，Message，接收一个字符串，和Cancelable，接收一个布尔值，为了达成屏蔽下层控件的目的，通常Cancelable都是设置成false的

接下来设置PositiveButton和NegativeButton，其结构和Button的onClickListener相似，不过多一个参数，第一个参数为字符串，是按钮的文本信息。第二个所用的内部类是`DialogInterface.onClickListener()`，而不是Button所用的`View.onClickListener()`

最后调用`show()`方法把Dialog展示出来

### ProgressDialog

ProgressDialog用于弹出一个带进度条的对话框，与AlertDialog类似地，同样具有屏蔽其他控件的功能，有效地避免了加载阶段用户的乱操作。ProgressDialog直接使用`ProgressDialog()`构造，其只有一个参数，指定Dialog的上下文

```java
ProgressDialog progressDialog = new ProgressDialog(MainActivity.this);

progressDialog.setTitle("Title");
progressDialog.setMessage("Message");
progressDialog.setCancelable(true);

progressDialog.show();
```

属性与上类似。需要注意的是，由于ProgressDialog是没有类似于AlertDialog一样的Button的，所以当setCancelable为false时，必须在处理完数据之后，**调用`progressDialog.dismiss()`，来关闭ProgressDialog**，否则用户无法关闭这个对话框！

### CheckBox

复选框

```xml
<CheckBox
    android:id="@+id/check_box"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    />
```

设置是否勾选

```java
checkBox.setChecked(true);//or false
```

获取勾选状态

```java
Boolean isChecked = checkBox.isChecked();
```