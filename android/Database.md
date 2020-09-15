# Database

安卓数据持久化技术主要有文件存储，Shared Preferences存储，以及SQLite数据库存储。文件存储用途比较广，主要用于存储一些简单的文本数据和二进制数据。Shared Preferences使用键值对存储数据，比文件存储方便很多。SQLite则适合大量数据存储。这里略过文件存储

## SharedPreferences

安卓主要提供三种方法获得SharedPreferences对象
SharedPreferences文件都存放在/data/data/package name/shared_prefs/目录下

1. Context类的`getSharedPreferences()`方法 接受两个参数，第一个为SharedPreferences文件的名称，若不存在则创建一个。第二个参数为指定操作模式，目前只有MODE_PRIVATE一种模式可选，表示只有当前的应用程序可以对这个文件进行读写。其余几种操作模式由于其安全性问题均以被废弃。
2. Activity类中的`getPreferences()`方法 接受一个参数，操作模式，默认将当前活动类名作为文件名
3. Preference Manager类中的`getDefaultSharedPreferences()`方法 接受一个Context参数

获取了SharedPreferences对象之后，调用`edit()`方法，获取一个`SharedPreferences.Editor`对象
调用`SharedPreferences.Editor`对象的一系列`putXXXX()`方法存储数据，比如`putString()`，这类方法接受两个参数，第一个为String型的键，第二个为对应的值
最后调用`apply()`提交数据

```java
//这里调用的是Context类的getSharedPreferences()方法
SharedPreferences.Editor editor = getSharedPreferences("data",MODE_PRIVATE).edit();
editor.putString("name","Robert");
editor.putInt("age",18);
editor.putBoolean("married",false);
editor.apply;
```

通常以xml格式存储

调用SharedPreferences对象的一系列`getXXXX()`方法获得数据，这类方法接受两个参数，第一个为键，第二个是找不到该键时的默认值

```java
SharedPreferences pref = getSharedPreferences("data",MODE_PRIVATE);
String name = pref.getString("name","");
int age = pref.getInt("age",0);
boolean isMarried = pref.getBoolean("married",false);
```

要想清除掉SharedPreferences的内容，只需要调用`clear()`方法。请谨慎使用

## SQLite数据库存储

SQLite是Android内置的轻量级关系型数据库，下面介绍管理库的两种方法

### SQLiteOpenHelper

SQLiteOpenHelper帮助类是一个抽象类，借助继承这个抽象类，可以非常简单地管理库

SQLiteOpenHelper中有两个重要的抽象方法，分别是`onCreate()`和`onUpgrade()`，分别在库创建和升级时被调用

另外，有两个重要的实例方法`getReadableDatabase()`和`getWritableDatabase()`，这两个方法都可以创建或者打开一个库，并返回一个可读写的对象，所以Readable Database也是可以写入的哟。区别仅在于，数据库不可写入，如空间不足，Readable Database将只读，而Writable Database则出现异常

SQLiteOpenHelper有两个构造函数，这里仅介绍一种。这个构造方法接受四个参数，第一个为context，第二个为String型的name，第三个为允许查询数据时返回一个自定义Cursor，一般传入null就可以了，第四个为版本号

```java
public class MyDatabaseHelper extends SQLiteOpenHelper{
    public static final String CREATE_BOOK = "create table Book ("
            +"id integer primary key autoincrement, "
            +"author text, "
            +"price real, "
            +"pages integer, "
            +"name text)";

    public static final String CREATE_CATEGORY = "create table Category ("
            +"id integer primary key autoincrement, "
            +"category_name text, "
            +"category_code integer)";

    private Context myContext;

    public MyDatabaseHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version){
        super(context,name,factory,version);
        myContext = context;
    }

    @Override
    public void onCreate(SQLiteDatabase db){
        db.execSQL(CREATE_BOOK);
        db.execSQL(CREATE_CATEGORY);
        Toast.makeText(myContext,"Create Succeed",Toast.LENGTH_LONG).show();
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion){
        db.execSQL("drop table if exists Book");
        db.execSQL("drop table if exists Category");
        onCreate(db);
    }
}
```

可以看到，通过调用SQLiteDatabase的`execSQL()`方法，可以执行String型的SQL语句

### 创建库

```java
MyDatabaseHelper dbHelper = new MyDatabaseHelper(this,"BookStore.db",null,1);
dbHelper.getWritableDatabase();
```

要升级这个库，只需要版本号+1

```java
MyDatabaseHelper secondDbHelper = new MyDatabaseHelper(this,"BookStore.db",null,2);
secondDbHelper.getWritableDatabase();
```

这时会调用`onUpgrade()`方法，注意要创建一个新表时，要先drop掉原来的同名表，不然如果创建时发现这张表已经存在，就会报错

### CRUD

#### 添加

SQLiteDatabase提供了一个`insert()`方法，接收3个参数，第一个为表名，第二个为未指定添加数据时为列自动赋值null，一般来说这个参数就是null，第三个为一个ContentValues对象，即数据载体，ContentValues对象的一系列put()方法的重载用于添加数据

```java
SQLiteDatabase sqLiteDatabase = databaseHelper.getWritableDatabase();
ContentValues values = new ContentValues();
values.put("name","Code");
values.put("author","yb");
values.put("pages",444);
values.put("price",1.63);
sqLiteDatabase.insert("Book",null,values);
values.clear();
```

ContentValues对象的`clear()`方法则会清除当前的值，为了数据安全应该在结束后调用

#### 更新

`update()`方法用于更新数据，接收四个参数，第一个参数仍然是表名，第二个参数为ContentValues对象，第三，四个参数用于约束更新条件

```java
SQLiteDatabase sqLiteDatabase = databaseHelper.getWritableDatabase();
ContentValues values = new ContentValues();
values.put("price",1.63);
sqLiteDatabase.update("Book",values,"name = ?",new String[]{"Code"});
```

第三个参数对应的是SQL语句的where部分，表示匹配值等同于第四个字符串对象

#### 删除

`delete()`方法用于删除数据，接收三个参数，第一个仍是表名，第二，三个参数用于约束删除条件。注意！**若不设置约束条件默认删除所有行**（删库了怎么办？当然是跑路啦）

```java
SQLiteDatabase sqLiteDatabase = databaseHelper.getWritableDatabase();
sqLiteDatabase.delete("Book","pages > ?",new String[]{"500"});
```

表示删除pages大于500的行

#### 查询

`query()`方法用于查询数据，接收七个参数：第一个仍是表名；第二个用于指定查询哪几列；第三个，四个用于指定查询哪些行；第五个指定需要去group by的列，第六个指定过滤group by后的数据；第七个用于指定查询结果的排列方式

调用`query()`方法后返回一个Cursor对象，即数据载体

```java
SQLiteDatabase sqLiteDatabase = databaseHelper.getWritableDatabase();
Cursor cursor = sqLiteDatabase.query("Book",null,null,null,null,null,null,null);
if(cursor.moveToFirst()){
    do{
        String name = cursor.getString(cursor.getColumnIndex("name"));
        Log.d("MainActivity","book name is "+name);
    }while (cursor.moveToNext());
}
cursor.close();
```

调用Cursor对象的`moveToFirst()`方法将数据的指针移动到第一行，接下来`moveToNext()`即可遍历。在相应的位置调用`getXXX()`方法去获取数据，方法接收一个位置索引参数，通过cursor的`getColumnIndex("name")`方法获取列的位置索引

### 使用原生的SQL语句

SQLiteDatabase的`execSQL()`方法可以直接执行SQL语句

唯一例外是查询数据调用的是`rawQuery()`方法

### 使用LitePal

LitePal是一款优秀的Android开源数据库框架，使用LitePal可以大大简化工作量

首先编辑`app/build.gradle`，在dependencies闭包中添加

```groovy
implementation 'org.litepal.android:core:1.4.1'
```

版本号可以自己决定

接下来在app/src/main目录下，新建litepal.xml文件，配置LitePal

```xml
<?xml version="1.0" encoding="utf-8"?>
<litepal>

    <dbname value="BookStore" ></dbname>

    <version value="1" ></version>

    <list>
        <mapping class="com.example.litepaltest.Book"></mapping>
    </list>

</litepal>
```

其中dbname用于指定数据库名，version指定数据库版本号，list用于指定映射模型

然后在AndroidManifest中的application下，添加

```xml
android:name="org.litepal.LitePalApplication"
```

LitePal使用ORM（对象关系映射）管理数据，这意味着可以使用面对对象的思想去管理这个库，而不用SQL语句。

定义一个类作为数据类型

```java
public class Book {

    private int id;
    private String author;
    private double price;
    private int pages;
    private String name;

    //AS可以通过Alt+Ins快捷键生成getter和setter方法
    public void setId(int id) {this.id = id;}
    public void setAuthor(String author) {this.author = author;}
    public void setPrice(double price) {this.price = price;}
    public void setPages(int pages) {this.pages = pages;}
    public void setName(String name) {this.name = name;}

    public int getId() {return id;}
    public String getAuthor() {return author;}
    public double getPrice() {return price;}
    public int getPages() {return pages;}
    public String getName() {return name;}
}
```

定义好的模型类需要添加到映射模型列表中，即litepal.xml的list下，注意需要完整的路径

```xml
<mapping class="com.example.litepaltest.Book"></mapping>
```

#### 创建库

```java
LitePal.getDatabase();
```

#### 修改库

想修改数据库，只需修改模型类，如添加一个新的变量，然后将版本号+1即可。要添加一个新表，也只需新建一个表，然后添加到映射模型列表

#### 添加数据

首先需要让模型类继承自DataSupport类
然后只需要创建出模型类的实例，然后调用一系列的`setXXXX`方法，最后调用`save()`方法保存即可

```java
Book book = new Book();
book.setName("YB");
book.save();
```

#### 更新

对于已经存储在数据库中的数据，即已经调用过`save()`方法的，只需要对相应的实例重新设置，然后再次save即可

```java
Book book = new Book();
book.setName("YB");
book.save();
book.setName("upup");
book.save();
```

另外一种直接修改数据库中数据

```java
Book book = new Book();
book.setPrice(12);
book.setPress("Anchor");
book.updateAll("name = ? and author = ?","The Lost Symbol","Dan Brown");
```

`updateAll()`方法指定约束条件

注意`updateAll()`方法不能将数据设为默认值，要将某一列设为默认值，需要调用`setToDefault()`方法，然后传入相应的列名

```java
book.setToDefault("pages");
```

#### 删除

对于已经存储在数据库中的数据，即已经调用过`save()`方法的，可以直接调用`delete()`，然后再次save即可

也可通过DataSupport类的`deleteAll()`方法，第一个参数为模型类，第二个，三个参数为约束条件

```java
DataSupport.deleteAll(Book.class,"price < ?","15");
```

注意，**如果不指定约束条件，将删除整个表中的数据**

#### 查询

首先看看怎么获取整个表

```java
List<Book> books = DataSupport.findAll(Book.class);
```

`findAll()`方法接收一个模型类参数，并返回所有的实例类

除此之外还有连缀查询，使用连缀查询时，末尾需要调用`find()`方法，接收一个模型类作为参数

- `select()`用于指定查询哪几列的数据
  `java List books = DataSupport.select("name","author").find(Book.class);`
- `where()`用于指定约束条件
  `java List books = DataSupport.where("pages > ?","500").find(Book.class);`
- `order()`指定排序方式，desc表示降序，asc表示升序
  `java List books = DataSupport.order("price desc").find(Book.class);`
- `limit()`指定查询结果的数量，如只查前三条，则`limit(3)`
  通常与`offset()`连用，指定偏移量，如
  `java DataSupport.limit(3).offset(1).find(Book.class);`
  则表示查询第2,3,4条数据

以上连缀查询可以进行任意的排列组合从而实现精确查找

当然，也可以通过`findBySQL()`方法进行SQL语句的原生查询，返回的是Cursor对象