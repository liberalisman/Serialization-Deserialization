# 序列化和反序列化
简单地总结一下 序列化和反序列化 的知识



### 定义以及相关概念
互联网的产生带来了机器间通讯的需求，而互联通讯的双方需要采用约定的协议，序列化和反序列化属于通讯协议的一部分。通讯协议往往采用分层模型，不同模型每层的功能定义以及颗粒度不同，例如：`TCP/IP`协议是一个四层协议，而`OSI`模型却是七层协议模型。在`OSI`七层协议模型中 **展现层（Presentation Layer）的主要功能是把应用层的对象转换成一段连续的二进制串，或者反过来，把二进制串转换成应用层的对象--这两个功能就是序列化和反序列化**。一般而言，`TCP/IP`协议的应用层对应与`OSI`七层协议模型的应用层，展示层和会话层，所以序列化协议属于`TCP/IP`协议应用层的一部分。本文对序列化协议的讲解主要基于`OSI`七层协议模型。

**序列化(Serialization)： 将数据结构或对象转换成二进制串的过程。在iOS中称为归档(Archive)
反序列化：将在序列化过程中所生成的二进制串转换成数据结构或者对象的过程。**

不同的计算机语言中，数据结构，对象以及二进制串的表示方式并不相同。

数据结构和对象：对于类似Java这种完全面向对象的语言，工程师所操作的一切都是对象`（Object）`，来自于类的实例化。在Java语言中最接近数据结构的概念，就是`POJO（Plain Old Java Object）`或者`Javabean`－－那些只有`setter/getter`方法的类。而**C二进制串：序列化所生成的二进制串指的是存储在内存中的一块数据。C语言的字符串可以直接被传输层使用，因为其本质上就是以'0'结尾的存储在内存中的二进制串**。在`Java`语言里面，二进制串的概念容易和String混淆。实际上`String `是`Java`的一等公民，是一种特殊对象`（Object）`。**对于跨语言间的通讯，序列化后的数据当然不能是某种语言的特殊数据类型。**,在`iOS`中对象转为`NSData`类型，就是序列化之后的，而`NSData`转为其他类型对象就是反序列化。

### 在iOS中序列化与反序列化
  将任何对象转NSData，这个对象都需要遵循一个协议，就是`NSCoding`协议。代码如下：
  
  ```objc
  //每个属性变量分别转码，序列化
- (void)encodeWithCoder:(NSCoder *)aCoder 
{
    [aCoder encodeObject:self.FYusername forKey:@"username"];
    [aCoder encodeObject:self.FriendlyName forKey:@"FriendlyName"];
    [aCoder encodeObject:self.phoneNum forKey:@"phoneNum"];
}
 ```
 
 ```objc
//分别把每个属性变量根据关键字进行逆转码，最后返回一个Student类的对象,反序列化
- (id)initWithCoder:(NSCoder *)aDecoder
{
    if (self = [super init]) 
    {
        self.FYusername = [aDecoder decodeObjectForKey:@"username"];
        self.FriendlyName= [aDecoder decodeObjectForKey:@"FriendlyName"];
        self.phoneNum= [aDecoder decodeObjectForKey:@"phoneNum"];
    }
    return self;
}
```
  
对象在实现NSCoding协议后，在外面使用这个对象的时候可以通过归档函数来转成NSData:

```objc
// 归档调动，序列化
NSData *contactsData=[NSKeyedArchiver archivedDataWithRootObject:ContactsArray]; 
//反序列化，转变为对象
NSObject<NSCoding> *obj=[NSKeyedUnarchiver unarchiveObjectWithFile:path];
```
其中的``NSCoder是一个编码的工具性类``，封装了对象序列化和反序列化的函数，所以实际上，我们并没有自己写序列化算法，只是遵循了这个协议让系统去调用罢了。

### iOS利用序列化和反序列化的作用
实现NSCoding的类，并序列化数据，有2个好处：
1.**序列化数据可以直接进行存储**
2.**序列化数据容易进行完全拷贝**

#### 序列化数据可以直接进行存储
在iOS中，进行存储比较快捷的方式是``NSUserDefaults``，存储方式如下：
但它支持的数据类型很有限：
``NSNumber（NSInteger、float、double），NSString，NSData，NSArray，NSDictionary，BOOL.``

```objc
[[NSUserDefaults standardUserDefaults] setObject:nickName forKey:UserDefault_NickName];
[[NSUserDefaults standardUserDefaults] synchronize];

```

一般都是些不可变的基本类型，存储其他类型时，如``NSMutableArray``等类型时，会崩溃的。
解决办法如下：

```objc
//当然，不能忽略的是，如果是自定义对象，别忘了NSCoding协议。
NSData *contactsData=[NSKeyedArchiver archivedDataWithRootObject:ContactsArray];
[[NSUserDefaults standardUserDefaults] setObject:contactsData forKey:UserDefault_ContactsArray];
[[NSUserDefaults standardUserDefaults] synchronize];
```

除了``NSUserDefaults``，另外存储``NSData``的方式可以用**归档+地址**：


```objc
[NSKeyedArchiver archiveRootObject:obj toFile:path];
```

#### 序列化数据容易进行完全拷贝：
这里简单说下使用``NSKeyedArchiver``来实现深拷贝：
主要的方法是先将某个对象转``NSData``,然后``NSData``转回赋值给新建对象：


```objc
NSData *data = [NSKeyedArchiver archivedDataWithRootObject:oldContactsArray];
NSMutableArray *newContactsArray = [NSKeyedUnarchiver unarchiveObjectWithData:data];
```
 


