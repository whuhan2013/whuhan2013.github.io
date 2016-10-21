---
layout: post
title: IOS数据持久化
date: 2016-10-21
categories: blog
tags: [IOS]
description: IOS数据持久化
---



所谓的持久化，就是将数据保存到硬盘中，使得在应用程序或机器重启后可以继续访问之前保存的数据。在iOS开发中，有很多数据持久化的方案，接下来我将尝试着介绍一下5种方案：

- plist文件（属性列表）
- preference（偏好设置）
- NSKeyedArchiver（归档）
- SQLite 3
- CoreData


#### 沙盒
在介绍各种存储方法之前，有必要说明以下沙盒机制。iOS程序默认情况下只能访问程序自己的目录，这个目录被称为“沙盒”。

1.结构     
既然沙盒就是一个文件夹，那就看看里面有什么吧。沙盒的目录结构如下：

```
"应用程序包"
Documents
Library
    Caches
    Preferences
tmp
```

2.目录特性     
虽然沙盒中有这么多文件夹，但是没有文件夹都不尽相同，都有各自的特性。所以在选择存放目录时，一定要认真选择适合的目录。


```
"应用程序包": 这里面存放的是应用程序的源文件，包括资源文件和可执行文件。

  NSString *path = [[NSBundle mainBundle] bundlePath];
  NSLog(@"%@", path);
Documents: 最常用的目录，iTunes同步该应用时会同步此文件夹中的内容，适合存储重要数据。

  NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
  NSLog(@"%@", path);
Library/Caches: iTunes不会同步此文件夹，适合存储体积大，不需要备份的非重要数据。

  NSString *path = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).firstObject;
  NSLog(@"%@", path);
Library/Preferences: iTunes同步该应用时会同步此文件夹中的内容，通常保存应用的设置信息。

tmp: iTunes不会同步此文件夹，系统可能在应用没运行时就删除该目录下的文件，所以此目录适合保存应用中的一些临时文件，用完就删除。

  NSString *path = NSTemporaryDirectory();
  NSLog(@"%@", path);
```


#### plist文件
plist文件是将某些特定的类，通过XML文件的方式保存在目录中。

可以被序列化的类型只有如下几种：

－ NSArray;
－ NSMutableArray;
－ NSDictionary;
－ NSMutableDictionary;
－ NSData;
－ NSMutableData;
－ NSString;
－ NSMutableString;
－ NSNumber;
－ NSDate;


**使用方法**

```
1.获得文件路径
NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;
NSString *fileName = [path stringByAppendingPathComponent:@"123.plist"];
2.存储
NSArray *array = @[@"123", @"456", @"789"];
[array writeToFile:fileName atomically:YES];
3.读取
NSArray *result = [NSArray arrayWithContentsOfFile:fileName];
NSLog(@"%@", result);
```

**注意**

- 只有以上列出的类型才能使用plist文件存储。
- 存储时使用writeToFile: atomically:方法。 
- 其中atomically表示是否需要先写入一个辅助文件，再把辅助文件拷贝到目标文件地址。这是更安全的写入文件方法，一般都写YES。
读取时使用arrayWithContentsOfFile:方法。


#### Preference   

**使用方法**  

```
//1.获得NSUserDefaults文件
NSUserDefaults *userDefaults = [NSUserDefaults standardUserDefaults];

//2.向文件中写入内容
[userDefaults setObject:@"AAA" forKey:@"a"];
[userDefaults setBool:YES forKey:@"sex"];
[userDefaults setInteger:21 forKey:@"age"];
//2.1立即同步
[userDefaults synchronize];

//3.读取文件
NSString *name = [userDefaults objectForKey:@"a"];
BOOL sex = [userDefaults boolForKey:@"sex"];
NSInteger age = [userDefaults integerForKey:@"age"];

NSLog(@"%@, %d, %ld", name, sex, age);
```


**注意**     

- 偏好设置是专门用来保存应用程序的配置信息的，一般不要在偏好设置中保存其他数据。
- 如果没有调用synchronize方法，系统会根据I/O情况不定时刻地保存到文件中。所以如果需要立即写入文件的就必须调用synchronize方法。
- 偏好设置会将所有数据保存到同一个文件中。即preference目录下的一个以此应用包名来命名的plist文件。

#### NSKeyedArchiver
归档在iOS中是另一种形式的序列化，只要遵循了NSCoding协议的对象都可以通过它实现序列化。由于决大多数支持存储数据的Foundation和Cocoa Touch类都遵循了NSCoding协议，因此，对于大多数类来说，归档相对而言还是比较容易实现的,常用于storyboard存储


1.遵循NSCoding协议         
NSCoding协议声明了两个方法，这两个方法都是必须实现的。一个用来说明如何将对象编码到归档中，另一个说明如何进行解档来获取一个新对象。

**遵循协议和设置属性**

```
  //1.遵循NSCoding协议 
  @interface Person : NSObject <NSCoding>

  //2.设置属性
  @property (strong, nonatomic) UIImage *avatar;
  @property (copy, nonatomic) NSString *name;
  @property (assign, nonatomic) NSInteger age;
  @end
```

**实现协议方法**


```
  //解档
  - (id)initWithCoder:(NSCoder *)aDecoder {
      if ([super init]) {
          self.avatar = [aDecoder decodeObjectForKey:@"avatar"];
          self.name = [aDecoder decodeObjectForKey:@"name"];
          self.age = [aDecoder decodeIntegerForKey:@"age"];
      }

      return self;
  }

  //归档
  - (void)encodeWithCoder:(NSCoder *)aCoder {
      [aCoder encodeObject:self.avatar forKey:@"avatar"];
      [aCoder encodeObject:self.name forKey:@"name"];
      [aCoder encodeInteger:self.age forKey:@"age"];
  }
```

**特别注意**

如果需要归档的类是某个自定义类的子类时，就需要在归档和解档之前先实现父类的归档和解档方法。即 [super encodeWithCoder:aCoder] 和 [super initWithCoder:aDecoder] 方法;



**2.使用**     
需要把对象归档是调用NSKeyedArchiver的工厂方法 archiveRootObject: toFile: 方法。

```
  NSString *file = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"person.data"];

  Person *person = [[Person alloc] init];
  person.avatar = self.avatarView.image;
  person.name = self.nameField.text;
  person.age = [self.ageField.text integerValue];

  [NSKeyedArchiver archiveRootObject:person toFile:file];
需要从文件中解档对象就调用NSKeyedUnarchiver的一个工厂方法 unarchiveObjectWithFile: 即可。

  NSString *file = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"person.data"];

  Person *person = [NSKeyedUnarchiver unarchiveObjectWithFile:file];
  if (person) {
     self.avatarView.image = person.avatar;
     self.nameField.text = person.name;
     self.ageField.text = [NSString stringWithFormat:@"%ld", person.age];
  }
```

**3.注意**    

- 必须遵循并实现NSCoding协议
- 保存文件的扩展名可以任意指定
- 继承时必须先调用父类的归档解档方法

#### SQLite3
之前的所有存储方法，都是覆盖存储。如果想要增加一条数据就必须把整个文件读出来，然后修改数据后再把整个内容覆盖写入文件。所以它们都不适合存储大量的内容。

**1.字段类型**    
表面上SQLite将数据分为以下几种类型：

- integer : 整数
- real : 实数（浮点数）
- text : 文本字符串
- blob : 二进制数据，比如文件，图片之类的

实际上SQLite是无类型的。即不管你在创表时指定的字段类型是什么，存储是依然可以存储任意类型的数据。而且在创表时也可以不指定字段类型。SQLite之所以什么类型就是为了良好的编程规范和方便开发人员交流，所以平时在使用时最好设置正确的字段类型！主键必须设置成integer


**2. 准备工作**      
准备工作就是导入依赖库啦，在iOS中要使用SQLite3，需要添加库文件：libsqlite3.dylib并导入主头文件，这是一个C语言的库，所以直接使用SQLite3还是比较麻烦的。

**3.使用**     

**创建数据库并打开**

操作数据库之前必须先指定数据库文件和要操作的表，所以使用SQLite3，首先要打开数据库文件，然后指定或创建一张表。


```
/**
*  打开数据库并创建一个表
*/
- (void)openDatabase {

   //1.设置文件名
   NSString *filename = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject stringByAppendingPathComponent:@"person.db"];

   //2.打开数据库文件，如果没有会自动创建一个文件
   NSInteger result = sqlite3_open(filename.UTF8String, &_sqlite3);
   if (result == SQLITE_OK) {
       NSLog(@"打开数据库成功！");

       //3.创建一个数据库表
       char *errmsg = NULL;

       sqlite3_exec(_sqlite3, "CREATE TABLE IF NOT EXISTS t_person(id integer primary key autoincrement, name text, age integer)", NULL, NULL, &errmsg);
       if (errmsg) {
           NSLog(@"错误：%s", errmsg);
       } else {
           NSLog(@"创表成功！");
       }

   } else {
       NSLog(@"打开数据库失败！");
   }
}
```


**执行指令**

使用 sqlite3_exec() 方法可以执行任何SQL语句，比如创表、更新、插入和删除操作。但是一般不用它执行查询语句，因为它不会返回查询到的数据。

```
/**
*  往表中插入1000条数据
*/
- (void)insertData {

NSString *nameStr;
NSInteger age;
for (NSInteger i = 0; i < 1000; i++) {
  nameStr = [NSString stringWithFormat:@"Bourne-%d", arc4random_uniform(10000)];
  age = arc4random_uniform(80) + 20;

  NSString *sql = [NSString stringWithFormat:@"INSERT INTO t_person (name, age) VALUES('%@', '%ld')", nameStr, age];

  char *errmsg = NULL;
  sqlite3_exec(_sqlite3, sql.UTF8String, NULL, NULL, &errmsg);
  if (errmsg) {
      NSLog(@"错误：%s", errmsg);
  }
}

NSLog(@"插入完毕！");
}
```


**查询指令**

前面说过一般不使用 sqlite3_exec() 方法查询数据。因为查询数据必须要获得查询结果，所以查询相对比较麻烦。示例代码如下：

- sqlite3_prepare_v2() : 检查sql的合法性
- sqlite3_step() : 逐行获取查询结果，不断重复，直到最后一条记录
- sqlite3_coloum_xxx() : 获取对应类型的内容，iCol对应的就是SQL语句中字段的顺序，从0开始。根据实际查询字段的属性，使用sqlite3_column_ xxx取得对应的内容即可。
- sqlite3_finalize() : 释放stmt


```
/**
*  从表中读取数据到数组中
*/
- (void)readData {
   NSMutableArray *mArray = [NSMutableArray arrayWithCapacity:1000];
   char *sql = "select name, age from t_person;";
   sqlite3_stmt *stmt;

   NSInteger result = sqlite3_prepare_v2(_sqlite3, sql, -1, &stmt, NULL);
   if (result == SQLITE_OK) {
       while (sqlite3_step(stmt) == SQLITE_ROW) {

           char *name = (char *)sqlite3_column_text(stmt, 0);
           NSInteger age = sqlite3_column_int(stmt, 1);

           //创建对象
           Person *person = [Person personWithName:[NSString stringWithUTF8String:name] Age:age];
           [mArray addObject:person];
       }
       self.dataList = mArray;
   }
   sqlite3_finalize(stmt);
}
```

**4.总结**  
总得来说，SQLite3的使用还是比较麻烦的，因为都是些c语言的函数，理解起来有些困难。不过在一般开发过程中，使用的都是第三方开源库 FMDB，封装了这些基本的c语言方法，使得我们在使用时更加容易理解，提高开发效率。

FMDB地址:[FMDB的gitHub地址](https://github.com/ccgus/fmdb)


#### CoreData的简单使用

**准备工作**   

- 创建数据库
    + 新建文件，选择CoreData -> DataModel
    + 添加实体（表），Add Entity
    + 给表中添加属性，点击Attributes下方的‘+’号
- 创建模型文件
    + 新建文件，选择CoreData -> NSManaged Object subclass
    + 根据提示，选择实体
- 通过代码，关联数据库和实体

```
- (void)viewDidLoad {
   [super viewDidLoad];

   /*
    * 关联的时候，如果本地没有数据库文件，Ｃoreadata自己会创建
    */

   // 1. 上下文
   NSManagedObjectContext *context = [[NSManagedObjectContext alloc] init];

   // 2. 上下文关连数据库

   // 2.1 model模型文件
   NSManagedObjectModel *model = [NSManagedObjectModel mergedModelFromBundles:nil];

   // 2.2 持久化存储调度器
   // 持久化，把数据保存到一个文件，而不是内存
   NSPersistentStoreCoordinator *store = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:model];

   // 2.3 设置CoreData数据库的名字和路径
   NSString *doc = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
   NSString *sqlitePath = [doc stringByAppendingPathComponent:@"company.sqlite"];

   [store addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:[NSURL fileURLWithPath:sqlitePath] options:nil error:nil];

   context.persistentStoreCoordinator = store;
   _context = context;

}
```


#### CoreData的基本操作（CURD）

**增删改查**

```
-(IBAction)addEmployee{

   // 创建一个员工对象 
   //Employee *emp = [[Employee alloc] init]; 不能用此方法创建
   Employee *emp = [NSEntityDescription insertNewObjectForEntityForName:@"Employee" inManagedObjectContext:_context];
   emp.name = @"wangwu";
   emp.height = @1.80;
   emp.birthday = [NSDate date];

   // 直接保存数据库
   NSError *error = nil;
   [_context save:&error];

   if (error) {
       NSLog(@"%@",error);
   }
}
读取数据 - Read

  -(IBAction)readEmployee{

      // 1.FetchRequest 获取请求对象
      NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"Employee"];

      // 2.设置过滤条件
      // 查找zhangsan
      NSPredicate *pre = [NSPredicate predicateWithFormat:@"name = %@",
                          @"zhangsan"];
      request.predicate = pre;

      // 3.设置排序
      // 身高的升序排序
      NSSortDescriptor *heigtSort = [NSSortDescriptor sortDescriptorWithKey:@"height" ascending:NO];
      request.sortDescriptors = @[heigtSort];

      // 4.执行请求
      NSError *error = nil;

      NSArray *emps = [_context executeFetchRequest:request error:&error];
      if (error) {
          NSLog(@"error");
      }

      //NSLog(@"%@",emps);
      //遍历员工
      for (Employee *emp in emps) {
          NSLog(@"名字 %@ 身高 %@ 生日 %@",emp.name,emp.height,emp.birthday);
      }
  }
修改数据 - Update

-(IBAction)updateEmployee{
   // 改变zhangsan的身高为2m

   // 1.查找到zhangsan
   // 1.1FectchRequest 抓取请求对象
   NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"Employee"];

   // 1.2设置过滤条件
   // 查找zhangsan
   NSPredicate *pre = [NSPredicate predicateWithFormat:@"name = %@", @"zhangsan"];
   request.predicate = pre;

   // 1.3执行请求
   NSArray *emps = [_context executeFetchRequest:request error:nil];

   // 2.更新身高
   for (Employee *e in emps) {
       e.height = @2.0;
   }

   // 3.保存
   NSError *error = nil;
   [_context save:&error];

   if (error) {
       NSLog(@"%@",error);
   }
}
删除数据 - Delete

-(IBAction)deleteEmployee{

   // 删除 lisi

   // 1.查找lisi
   // 1.1FectchRequest 抓取请求对象
   NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"Employee"];

   // 1.2设置过滤条件
   // 查找zhangsan
   NSPredicate *pre = [NSPredicate predicateWithFormat:@"name = %@",
                       @"lisi"];
   request.predicate = pre;

   // 1.3执行请求
   NSArray *emps = [_context executeFetchRequest:request error:nil];

   // 2.删除
   for (Employee *e in emps) {
       [_context deleteObject:e];
   }

   // 3.保存
   NSError *error = nil;
   [_context save:&error];

   if (error) {
       NSLog(@"%@",error);
   }

}
```

好像都是oc代码


**参考链接**  

[iOS中几种数据持久化方案](http://www.jianshu.com/p/7616cbd72845)

[CoreData的使用](http://www.jianshu.com/p/6e048f7c5812)


