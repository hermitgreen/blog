# ListView

几乎每个APP都会用到滚动控件，淘宝的商品页，知乎的信息页等等，这是最重要而最难用的一种控件，所以从常用控件中单独提出来写用法
滚动控件主要有两种，一种是安卓原带的ListView，另外一种是新增的RecyclerView，当然，在8012年的今天，我们更应该多关注RecyclerView。

## 初始化

```xml
<ListView
    android:id="@+id/list_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    />
```

ListView的数据传入需要适配器完成，比较好用的适配器之一是ArrayAdapter

```java
String[] data = {"some data"};
ArrayAdapter<String> adapter = new ArrayAdapter<String>(MainActivity.this, android.R.layout.simple_list_item_1, data);
```

以字符串数据为例，这里由于数据为字符串，故将ArrayAdapter的泛型指定为String，ArrayAdapter有多个构造函数的重载，这里使用的第一个参数为上下文，第二个参数为布局的id，simple_list_item_1是一个安卓内置的布局文件，里面只有一个TextView，用于显示一段文本，第三个是数据

最后将适配器传入

```java
ListView listView = findViewById(R.id.list_view);
listView.setAdapter(adapter);
```

## 定制界面

和《第一行代码》一样，以一个显示水果的界面来作为例子

首先我们要有一个实体类来描述水果的信息

```java
public class Fruit {

    private String name;
    private int imageId;

    public Fruit(String name, int imageId) {
        this.name = name;
        this.imageId = imageId;
    }

    public String getName(){
        return name;
    }

    public int getImageId(){
        return imageId;
    }
}
```

然后需要一个布局文件去确定信息怎么展示，创建fruit_item.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ImageView
        android:id="@+id/fruit_image"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <TextView
        android:id="@+id/fruit_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:layout_maginStart="10dp" />
</LinearLayout>
```

接下来创建一个自定义适配器去管理要传入的信息

```java
public class FruitAdapter extends ArrayAdapter<Fruit> {

    private int resourceId;

    public FruitAdapter (Context context, int FruitViewResourceId, List<Fruit> objects) {
        super(context, FruitViewResourceId,objects);
        resourceId = FruitViewResourceId;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        Fruit fruit = getItem(position);
        View view = LayoutInflater.from(getContext()).inflate(resourceId,parent,false);

        ImageView fruitImage = view.findViewById(R.id.fruit_image);
        TextView fruitName = view.findViewById(R.id.fruit_name);

        fruitImage.setImageResource(fruit.getImageId());
        fruitName.setText(fruit.getName());

        return view;
    }
}
```

这段代码理解起来比较困难，首先从FruitAdapter看起，作为继承ArrayAdapter的构造函数，在这里将FruitViewResourceId记录下来，这是每一个水果的布局文件，也就是fruit_item.xml，以便后面动态加载布局文件

`getView()`方法在每一个Fruit Item被划到屏幕内时会被触发，这个时候，首先通过通过`getItem(position)`得到被划进来的是哪个Fruit实体，然后利用`LayoutInflater.from().inflate()`方法动态加载对应的布局，在ListView中应使用`getContext()`得到Context，inflate的第一个参数是我们自定义的布局文件，而第三个参数为false，表示不为这个生成的View添加父布局，因为这个View是要放到ListView中的。

最后为这个View添加好信息，需要注意，这里应该调用view的`findViewById()`方法，因为每一个Fruit实体的图像和名称是不同的

总结一下就是，先取得水果布局文件ID，然后得到水果信息，根据水果信息构造出水果展示的布局，再把它加入到ListView中

最后在onCreate中使用FruitAdapter

```java
Fruit apple = new Fruit("apple", R.drawable.apple_pic);
fruitList.add(apple);
//etc..
FruitAdapter adapter = new FruitAdapter(MainActivity.this, R.layout.fruit_item, fruitList);
listView.setAdapter(adapter);
```

## 优化

在动态加载布局时，假如用户是重复划动的，可以重用之前加载好的View

```java
View view;
if(convertView != null){
    view = convertView;
} else {
    view = LayoutInflater.from(getContext()).inflate(resourceId,parent,false);
}
```

利用ViewHolder缓存控件实例

```java
class ViewHolder {
    ImageView fruitImage;
    TextView fruitName;
}

View view;
ViewHolder viewHolder;
if(convertView != null){
    view = convertView;
    viewHolder = (ViewHolder)view.getTag();
} else {
    view = LayoutInflater.from(getContext()).inflate(resourceId,parent,false);
    viewHolder = new ViewHolder();
    viewHolder.fruitImage.view.findViewById(R.id.fruit_image);
    viewHolder.fruitName.view.findViewById(R.id.fruit_name);
    view.setTag(viewHolder);
}
viewHolder.fruitImage.setImageResource(fruit.getImageId());
viewHolder.fruitName.setText(fruit.getName());
```

注意，setImageResource和setText仍然需要执行，因为fruit_item.xml中并没有这些信息

## 点击事件

ListView的点击监听器类似于Button的

```java
listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
    @Override
    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
        Fruit fruit = fruitList.get(position);
        //TODO
    }
});
```

## RecyclerView

RecyclerView相当于一个增强版的ListView，属于新增控件，要使用RecyclerView首先打开app/build.gradle文件，在denpendencies闭包中添加`compile 'com.android.support:recyclerview-v7:xx'`，这里的xx填入你自己的版本号，然后就可以使用了

```xml
<android.support.v7.widget.RecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    />
```

### 适配器

通常RecyclerView使用的适配器为自带的RecyclerView.Adapter。以《第一行代码》中的示例为例

```java
public class FruitAdapter extends RecyclerView.Adapter<FruitAdapter.ViewHolder> {

    private List<Fruit> mFruitList;

    //内部类ViewHolder
    static class ViewHolder extends RecyclerView.ViewHolder{
        View fruitView;
        ImageView fruitImage;
        TextView fruitName;

        public ViewHolder(View view){
            super(view);
            fruitView = view;
            fruitImage = view.findViewById(R.id.fruitImage);
            fruitName = view.findViewById(R.id.fruitName);
        }
    }

    public FruitAdapter(List<Fruit> fruitList){
        mFruitList = fruitList;
    }

    //以下是三个RecyclerView.Adapter的方法的重写
    @Override
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext()).inflate
                (R.layout.fruit_item,parent,false);

        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(ViewHolder holder, int position){
        Fruit fruit = mFruitList.get(position);
        holder.fruitName.setText(fruit.getName());
        holder.fruitImage.setImageResource(fruit.getImageID());
    }

    @Override
    public int getItemCount(){
        return mFruitList.size();
    }
}
```

ViewHolder的作用在上面就已经提过了。`onCreateViewHolder()`，`onBindViewHolder()`和`getItemCount()`三个方法必须被重写。
在`onCreateViewHolder()`中，构造了一个view并返回一个viewHolder，注意，这里的LayoutInflater有第三个参数false，原理同ListView。
`onBindViewHolder()`会在每个子项被滑到屏幕内时被执行，可以通过position参数得到当前项的实例，并设置实例的Name和Image `getItemCount()`用于获取Recycler有多少项

RecyclerView的使用和ListView类似

```java
RecyclerView recyclerView = findViewById(R.id.recyclerView);

LinearLayoutManager layoutManager = new LinearLayoutManager(this);
recyclerView.setLayoutManager(layoutManager);

FruitAdapter fruitAdapter = new FruitAdapter(fruitList);
recyclerView.setAdapter(fruitAdapter);
```

LinearLayoutManager用于指定RecyclerView的布局方式，他的参数就是this，因为RecyclerView是在当前页面呈现的。将LayoutManager传入recyclerView，再将携带了信息的Adapter也传入，就实现了类似ListView的视觉效果

### LayoutManager

之所以说RecyclerView强于ListView，其中一点就是LayoutManager的多样性 LinearLayoutManager为线性布局，默认为纵向滚动，还可以实现横向滚动，只需要在设置了LayoutManager之前

```java
LinearLayoutManager layoutManager = new LinearLayoutManager(this);
layoutManager.setOrientation(LinearLayoutManager.HORIZONTAL)
recyclerView.setLayoutManager(layoutManager);
```

StaggeredGridLayoutManager可以实现瀑布流布局，瀑布流布局子项的宽度是根据布局的列数来确定的，所以宽度需要设置为match_parent，在代码中

```java
RecyclerView recyclerView = findViewById(R.id.recyclerView);

StaggeredGridLayoutManager layoutManager = new StaggeredGridLayoutManager(3,StaggeredGridLayoutManager.VERTICAL);
recyclerView.setLayoutManager(layoutManager);

FruitAdapter fruitAdapter = new FruitAdapter(fruitList);
recyclerView.setAdapter(fruitAdapter);
```

StaggeredGridLayoutManager的构造函数接收两个参数，第一个参数是布局的列数，第二个参数是布局的方向

GridLayoutManager是网格布局，使用则同上。但是网格布局没有瀑布流布局好用

### 点击事件

与ListView很不一样的一点是，RecyclerView不能用`setOnItemClickListener()`注册子项的监听器，而只能在子项的View中去注册
在RecyclerView的`onCreateViewHolder()`中注册点击事件

```java
@Override
public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    View view = LayoutInflater.from(parent.getContext()).inflate
                (R.layout.fruit_item,parent,false);

    final ViewHolder holder = new ViewHolder(view);

    //给整个View设置监听器
    holder.fruitView.setOnClickListener(new View.OnClickListener(){
        @Override
        public void onClick(View v) {
            int position = holder.getAdapterPosition();
            Fruit fruit = mFruitList.get(position);
            //TODO
        }
    });

    //给Image组件设置监听器
    holder.fruitImage.setOnClickListener(new View.OnClickListener(){
        @Override
        public void onClick(View v) {
            //TODO
        }
    });

    return new holder;
}
```

在监听器中，可以通过ViewHolder的`getAdapterPosition()`得到被点击的子项的位置，调用List的`get(position)`方法可以得到被点击的实例