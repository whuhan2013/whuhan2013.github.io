---
layout: post
title: Android高性能ORM数据库DBFlow入门 
date: 2016-6-12
categories: blog
tags: [数据库]
description: Android高性能ORM数据库DBFlow入门 
---


DBFlow，综合了 ActiveAndroid, Schematic, Ollie,Sprinkles 等库的优点。同时不是基于反射，所以性能也是非常高，效率紧跟greenDAO其后。基于注解，使用apt技术，在编译过程中生成操作类，使用方式和ActiveAndroid高度相似，使用简单。
 
特性：
 
1、无缝支持多个数据库；
 
2、使用annotation processing提高速度；
 
3、ModelContainer类库可以直接解析像JSON这样的数据；
 
4、增加灵活性的丰富接口。
 
 
 
github仓库：[https://github.com/Raizlabs/DBFlow](https://github.com/Raizlabs/DBFlow)
 
 
 
DBFlow在国内可能用的人不是很多，所以中文介绍很少，所以就有了这篇文章，接下来就让我们一起学习DBFlow。
 
**一、引入依赖、初始化**
 
需要引入apt和maven，配置项目的 build.gradle
 
```
buildscript {
  repositories {
    jcenter()
  }
  dependencies {
    classpath 'com.android.tools.build:gradle:2.0.0-beta6'
    classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
  }
}

allprojects {
  repositories {
    jcenter()
    maven { url "https://jitpack.io" }
  }
} 
```

配置app的build.gradle

``` 
apply plugin: 'com.android.application'
apply plugin: 'com.neenbedankt.android-apt'
def dbflow_version = "3.0.0-beta4"
android {
  compileSdkVersion 23
  buildToolsVersion "23.0.2"

  defaultConfig {
    applicationId "cn.taoweiji.dbflowexample"
    minSdkVersion 14
    targetSdkVersion 23
    versionCode 1
    versionName "1.0"
  }
  buildTypes {
    release {
      minifyEnabled false
      proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
  }
}

dependencies {
  compile fileTree(dir: 'libs', include: ['*.jar'])
  apt "com.github.Raizlabs.DBFlow:dbflow-processor:${dbflow_version}"
  compile "com.github.Raizlabs.DBFlow:dbflow-core:${dbflow_version}"
  compile "com.github.Raizlabs.DBFlow:dbflow:${dbflow_version}"
} 
```


需要在Application的onCreate对DBFlow进行初始化

``` 
public class MyApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        FlowManager.init(this);
    }
} 
```


记得修改AndroidManifest.xml

```
<application
    android:name=".MyApplication"
../>
```


**二、数据库创建、表创建**
 

**定义数据库**
 
我这里定义了一个名称叫做AppDatabase的数据库，可以根据自己的喜欢进行定义。
 
```
@Database(name = AppDatabase.NAME, version = AppDatabase.VERSION)
public class AppDatabase {
  //数据库名称
  public static final String NAME = "AppDatabase";
  //数据库版本号
  public static final int VERSION = 1;
} 
```

**创建数据库对象**
 
必须继承BaseModel，BaseModel包含了基本的数据库操作（save、delete、update、insert、exists），看下面代码可以发现这个表是关联上面定义的数据库，People的id是自增的id。
 
```
@ModelContainer
@Table(database = AppDatabase.class)
public class People extends BaseModel {
    //自增ID
    @PrimaryKey(autoincrement = true)
    public Long id;
    @Column
    public String name;
    @Column
    public int gender;
} 
```

编写完数据表对象后，点击Android studio的build->Make Project(Mac的童鞋直接command+F9)就会使用apt进行了编译，
 
查看目录（我的people类放在cn.taoweiji.dbflowexample.db）
 
app/build/generated/source/apt/debug/cn/taoweiji/dbflowexample/db
 
就可以看到自动生成 People_Adapter、People_Container、People_Table，其中People_Table在后面使用有很大的作用，建议详细看看它的结构。


**注意**  

如果配置好了后，Make Project后，却没有生成_Table, GeneratedDatabaseHolder, _DataBase, _Adapter等，检查都无误后，可以检查一下：

```
android {
    compileSdkVersion 23
    buildToolsVersion "23.0.2"
    }
```

参考链接:[体验Android ORM之DBFlow - 漫漫求学中的孩儿 - 博客频道 - CSDN.NET](http://blog.csdn.net/qduningning/article/details/51128283)


**三、增删改**
 
由于数据表对象继承了BaseModel，已经包含了很多的操作
 
```
People people = new People();
people.name = "Wiki";
people.gender = 1;
people.save();
//people.update();
//people.delete();
Log.e("Test", String.valueOf(people.id));
```


删除、更新等操作就自己体验，这里就不多说了。
 
**四、查询**


``` 
//返回所有查询结果
List<People> peoples = new Select().from(People.class).queryList();
//返回单个查询结果
People people = new Select().from(People.class).querySingle();
//查询gender = 1的所有People
List<People> peoples2 = new Select().from(People.class).where(People_Table.gender.eq(1)).queryList(); 
DBFlow的查询方式借鉴ActiveAndroid的，但是比ActiveAndroid功能还要强大。
```

**四、事务、批量保存**
 
事务是一个数据必须具备的，如果保存10000条数据，一条一条保存必然是很慢的，所以就需要用到事务，批量保存。DBFlow的事务非常的强大，同时使用也很复杂，这里就简单介绍批量保存，更多内容请查看官方文档
 
https://github.com/Raizlabs/DBFlow/blob/master/usage/Transactions.md

```
List<People> peoples = new ArrayList<>();
for (int i = 0; i < 1000; i++) {
    People people = new People();
    people.name = "Wiki";
    people.gender = 1;
    peoples.add(people);
}
//实时保存，马上保存
new SaveModelTransaction<>(ProcessModelInfo.withModels(peoples)).onExecute();
//异步保存，使用异步，如果立刻查询可能无法查到结果
//TransactionManager.getInstance().addTransaction(new SaveModelTransaction<>(ProcessModelInfo.withModels(peoples))); 
```


**五、数据库升级（增加表、增加字段等）**
 
如果是新增表无需做特别的处理，直接修改AppDatabase的版本号即可。
 
如果需要新增字段，除了需要修改AppDatabase的版本号外，还需要做特殊的处理，DBFlow的描述是：Migrations。
 
例子：对People新增一个email字段
 
第1步，修改数据库版本号
 
```
@Database(name = AppDatabase.NAME, version = AppDatabase.VERSION)
public class AppDatabase {
  //数据库名称
  public static final String NAME = "AppDatabase";
  //数据库版本号，这里修改2
  public static final int VERSION = 2;
} 
```

第2步，需要修改数据表对象结构，增加email

``` 
@ModelContainer
@Table(database = AppDatabase.class)
public class People extends BaseModel {
    //自增ID
    @PrimaryKey(autoincrement = true)
    public Long id;
    @Column
    public String name;
    @Column
    public int gender;
    @Column
    public String email;
} 
```

第3步，执行第二步之后，需要build(Android studio的build->Make Project、Mac的童鞋直接command+F9)，通过apt更新People_Table，接下来编写Migrations
 
```
@Migration(version = 2, database = AppDatabase.class)
public class Migration_2_People extends AlterTableMigration<People> {

    public Migration_2_People(Class<People> table) {
        super(table);
    }

    @Override
    public void onPreMigrate() {
        addColumn(SQLiteType.TEXT, People_Table.email.getNameAlias().getName());
    }
} 
```


类名可以更加自己喜欢定义，我个人的规则是，按照数据库版本号和需要更新的数据表来命名，需要注意是：version = 2
 
数据库升级就大功告成了。
 
总结：这篇文章只是简单介绍了DBFlow的基本功能使用，DBFlow还有很多很厉害的功能，比如多数据库支持、Powerful Model Caching等，而且还支持Kotlin语言（运行在Java虚拟机的新语言）。我只使用过greenDAO、activeAndroid、afinal、DBFlow数据库，所以在我看来，DBFlow是我用过的数据库当中最好用的数据库，性能也很好，使用非常简单，高度推荐。
 
我在github上共享一下DBFlow的配置
 
[https://github.com/taoweiji/DBFlowExample](https://github.com/taoweiji/DBFlowExample)


**原文链接**

[Android高性能ORM数据库DBFlow入门 - 陶伟基Wiki - 博客园](http://www.cnblogs.com/taoweiji/p/5246540.html)
 
 



