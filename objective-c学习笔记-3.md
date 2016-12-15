## 5 面向对象(上)

### 5.1 类和对象

> 本质上类是自定义数据类型，是一种指针类型的变量

#### 5.1.1 定义类

> 方法前的 - 表示此方法是实例方法
>
>     1）'-' 开头
>
>     2）可以访问实例变量
>
>     3）可以调用类方法
>
>     4）可以调用自己和其他对象的方法（调用其他对象方法，要作为参数传递过来）
>
>     5）由对象调用



> 方法前的 + 表示此方法是类方法
>
> ​    1）'+' 开头
>
>     2）**不可以访问实例变量**：在一个类方法中不能访问实例变量，因为实例变量还没有分配内存，初始化，也可以说还没有实例变量，原因是还没有对象
>
>     3）类方法可以调用其他的类方法
>
>     4）类方法不可以用对象调用
>
>     5）类方法，可以调用对象方法（把对象作为方法参数传递过来）
>
>     6）**类方法中不可以再调用当前方法**（死循环）



> 对象方法和类方法可以重名

#### 5.1.2 对象的产生和使用

> 访问权限允许时，`对象->成员变量名`
>
> 从内存存储角度看，objective-c的对象和C结构体类似

#### 5.1.3 对象和指针

> `FKPerson* person = [[FKPerson alloc] init]`
>
> 这里产生了一个FKPerson对象和一个person指针变量
>
> FKPerson对象存储在堆内存
>
> person变量被保存在main()函数动态存储区，存放的是FKPerson的地址值



#### 5.1.4 self关键字

> 总是指向调用该方法的对象
>
> 最大的作用是让类中的方法访问该类的另一个方法或成员变量

> self无法再类方法中出现，因为self总是代表当前类的对象，而类方法的调用者是类而不是对象
>
> 但它代表的对象是不确定的，只有类型确定，它所代表的对象只是当前类的实例
>
> 需要等到这个方法被调用，它所代表的对象才能确定

#### 5.1.5 id类型

> id类型可以代表任何类型
>
> 通过id类型的变量调用方法时，会执行动态绑定
>
> 会跟踪对象所属的类，在运行时判断该对象所属的类，确定要动态调用的方法

### 5.2 方法详解

#### 5.2.1 方法的所属性

> 方法不能独立定义，只能在类体里定义，独立定义的只能是函数
>
> 必须属于类或者对象
>
> 方法调用必须是`[类 方法]` `[对象 方法]`形式

#### 5.2.2 形参个数可变的方法

> 长度可变的形参只能处于形参列表最后
>
> 一个方法最多包含一个可变形参

- 定义：最后一个形参后增加“,...”

  - `-(void)test:(NSString*) name,...;`

- 获取可变参数

- 详细：http://justsee.iteye.com/blog/1637173

  - va_list：是一个类型，定义指向可变参数列表的指针变量

  - va_start：是一个函数，该函数指定开始处理可变形参的列表，并让指针变量指向可变形参列表的第一个参数

  - va_end：结束处理可变形参，释放指针变量

  - var_arg：该函数返回获取指针当前指向的参数的值，并将指针指向下一个参数

  - ```objective-c
    #define va_start(ap, param) __builtin_va_start(ap, param)
    #define va_end(ap)          __builtin_va_end(ap)
    #define va_arg(ap, type)    __builtin_va_arg(ap, type)
    ```

### 5.3 成员变量

#### 5.3.1 成员变量及其运行机制

> objective-c的成员变量都是实例变量，不支持真正的类变量
>
> 成员变量（实例变量）与实例共存亡
>
> `实例->实例变量`

> 成员变量（实例变量）无需显示初始化
>
> 对象创建时，系统会执行默认初始化
>
> 基本类型默认0，指针默认nil

#### 5.3.2 模拟类变量、static

![5.3.2](/Users/Armas/Desktop/objective-c笔记/5.3.2.png)

> 通过内部全局变量模拟
>
> 在类实现部分定义static全局变量，提供一个**类方法**（set、get）暴露它
>
> static关键字声明的变量只在程序运行时初始化一次；不能放在interface中，会报错

> 例子：http://www.cnblogs.com/ygm900/p/4615635.html

#### 5.3.3 单例(singleton)模式

> 如果一个类只需要创建一个实例，这个类就叫单例类

> 首先在类实现部分定义id类型static全局变量，赋值为nil `static id instance = nil;`
>
> 定义类方法 `+(id)instance;`
>
> ```objective-c
> +(id)instance{
>   if(!instance){
>     instance = [[super alloc] init];
>   }
>   return instance;
> }
> ```

### 5.4 隐藏和封装

#### 5.4.1 理解封装

> 将对象的状态信息（成员变量）隐藏在对象内部，外部程序只能通过该类提供的方法操作、访问内部信息

- 好处
  - 隐藏实现细节
  - 在访问方法中加入控制逻辑，限制不合理访问
  - 进行数据检查，保证对象信息的完整性
  - 便于修改，提高可维护性
- 良好的封装
  - 对象的成员变量和实现细节都隐藏
  - 将可以安全访问成员变量的方法暴露出来

#### 5.4.2 使用访问控制符

- 访问控制符：不能修饰局部变量，因为其只能被所在的方法访问

  > 范围从出现位置到下一个访问控制符或者}结束
  >
  > **只能出现在.h文件**
  >
  > 在实现文件中，虽然可以用如下方式声明在interface中，但成员变量都是private，方法也是类私有
  >
  > ```objective-c
  > @interface xxx(){
  > @public
  >     int public_int;}
  > -(void)im_in:(int)arg;
  > @end
  > ```

  - @private：当前类访问权限

    > 成员变量使用，表示只能在类内部访问
    >
    > 在类的实现部分定义成员变量，默认采用

  - @packge：与映像访问权限相同

    > 在当前类或相同映像的其它程序访问
    >
    > 相同映像--编译后生成的同一个框架或同一个执行文件
    >
    > 部分隐藏成员变量

  - @protected：子类访问权限

    > 当前类和当前类的子类任意地方访问
    >
    > 部分暴露成员变量
    >
    > 类的接口部分定义的成员变量默认采用

  - @public：公共访问权限

    > 任意地方

![5.4.2-1](/Users/Armas/Desktop/objective-c笔记/5.4.2-1.png)

![5.4.2-2](/Users/Armas/Desktop/objective-c笔记/5.4.2-2.png)

- 访问控制符的基本原则
  - 大多数成员变量都应该使用@private，对于工具方法应该在实现部分定义
  - 若某类主要用作父类，成员变量希望给子类使用，使用@protected
  - 希望暴露出来的方法应该在接口部分定义

#### 5.4.3 理解@package访问控制符

#### 5.4.4 合成存取方法

> @property
>
> 在实现部分使用@synthesize声明，现在也可省略
>
> - **使用@synthesize声明属性后，该属性对应的成员变量名不再是 `_属性名`，而是 `属性名`**
> - **当 `@synthesize name = name_2`后，该属性对应的成员变量名是 `name_2`**
>
> 定义了成员变量、setter和getter方法，可称为定义了一个property

> `synthesize property名 [=成员变量名];`

- 特殊指示符

  - assign

    > 进行简单赋值，不更改传入值的引用计数
    >
    > 主要用于NSInteger等基础类型，以及short、float、double、结构体等C数据类型

  - atomic(nonatomic)：存取方法是否为原子操作

    - atomic保证一个线程进入存取方法后，其它线程无法进入
    - 默认值是atomic，但atomic性能不高
    - 大多数单线程采用nonatomic

  - copy：setter方法传入的值可能在之后会被修改，若不想影响成员变量（**对于可变类型或其子类可变，比如NSString有可变子类NSMutableString**），使用copy

    - 传入的值赋值一个副本，将副本赋值给成员变量
    - 原成员变量引用计数减一

  - getter、setter：自定义存取方法名

    - 如，getter=abc，setter=xyz:（因为带参数，需要：）
    - `@property (assign,nonatomic,getter=wawa,setter=nana:) int price;`

  - readonly、readwrite

    - 默认readwrite
    - readonly表示只有getter

  - retain

    - 成员变量原来所引用的对象的引用计数减一

    - 传入的对象的引用计数加一

    - 当引用计数大于1，对象不会被回收；启用ARC后，retain很少使用

    - ```objective-c
      @interface FKWin:NSObject
      @property(nonatomic,retain) NSDate* date;
      @end
        
      --------------------------------------------
      FKWin* win = [[FKWin alloc] init];
      NSDate* date = [[NSDate alloc] init];
      //此时为1
      NSLog("date的引用计数:%ld",date.retainCount);
      //因为使用了retain指示符，date引用计数+1
      [win setDate:date];
      //此时为2
      NSLog("[win date]的引用计数:%ld",[win date].retainCount);
      //释放，date引用计数为1
      [date release];
      //此时为1
      NSLog("[win date]的引用计数:%ld",[win date].retainCount);
      ```

  - strong、weak：对传入的对象分别持有强引用和弱引用

    > 在启用ARC机制后，如果不希望被该属性引用的对象被回收，使用strong

    > 若需要保证性能，避免性能溢出，可以使用weak
    >
    > 使用weak属性时，可能其被引用的对象已经被释放
    >
    > 不过系统会自动将声明为weak的指针赋值为nil，在其指向的地址被释放后
    >
    > 所以weak指示符可有效地防止悬空指针

  - 强引用：只要该强引用指向某个对象，这个对象就不会自动回收

  - 弱引用：即使该弱引用指向某个对象，这个对象也可能被回收

  - unsafe_unretained：和weak基本相同

    - 不同之处在于，即使unsafe_unretained指针所引用的对象被回收了，unsafe_unretained指针也不会被赋值nil
    - 可能会导致崩溃，所以不如weak

#### 5.4.5 使用点语法访问属性

> 无论对象是否存在相应的成员变量，只要存在getter或者setter方法就可以通过点语法获取或者设置属性值

### 5.5 键值编码-KVC和键值监听-KVO

> 其实通过KVC操作对象属性的性能比存取方法性能更低
>
> 但KVC编程更简洁，更适合提炼一些通用性质的代码
>
> 因为KVC允许通过常量或变量字符串操作对象属性，有很高的灵活性

#### 5.5.1 简单的kvc

> 最基本的KVC由NSKeyValueCoding协议支持
>
> 最基本的操作属性的两个方法
>
> - setValue：属性值 forKey：属性名
>   - 底层执行机制
>     - 优先考虑调用setter方法，**所以当修改了setter方法名会导致`setValue:forKey`找不到setter方法而进行下一级寻找**
>     - 如果没找到getter，搜索`_属性名`的成员变量，不论其在哪儿用哪个修饰符定义，**当修改了属性名对应的成员变量名（且不是`_属性名`），导致该次搜索无果**
>     - 都没找到，会搜索`属性名`的成员变量
>     - 3条都没找到，执行该对象的 `setValue:forUndefinedKey:`方法，引起异常`NSUnknownKeyException`导致结束
> - valueForKey：属性名
>   - 底层执行机制
>     - 优先考虑调用getter方法，**所以当修改了getter方法名会导致`valueForKey:`找不到getter方法而进行下一级寻找**
>     - 如果没找到getter，搜索`_属性名`的成员变量，不论其在哪儿用哪个修饰符定义，**当修改了属性名对应的成员变量名（且不是`_属性名`），导致该次搜索无果**
>     - 都没找到，会搜索`属性名`的成员变量
>     - 3条都没找到，执行该对象的 `valueForUndefinedKey:`方法，引起异常`NSUnknownKeyException`导致结束
>
> **如果修改了setter或者getter方法名字，函数会找不到相应存取方法而搜索成员变量名**
>
> **key值不准确会导致搜索不到成员变量名**
>
> **key值要么直接是成员变量名**
>
> **如果是属性名，得保证该属性名对应的成员变量名是`_属性名`或者`属性名`**

#### 5.5.2 处理不存在的key

> 使用KVC操作属性，可能存取方法和对应的成员变量都不存在，此时会自动调用`setValue:forUndefinedKey:`或`valueForUndefinedKey:`，引发异常`NSUnknownKeyException`导致结束

> 可以在对象对应的类中重写`setValue:forUndefinedKey:`或`valueForUndefinedKey:`，甚至不需要在接口声明

```objective-c
//xxx.m
-(void)setValue:(id) value forUndefinedKey:(id) key{
  NSLog(@"key：%@不存在",key);
}
```

> 为什么重写的方法是隐藏的，还能被调用呢
>
> **objective-c不存在绝对的隐藏的方法**
>
> **任何方法都可以通过NSObject提供的`performSelector:`或`performSelector:withObject:`调用**

#### 5.5.3 处理nil

> 对于基本类型是不接受nil值的，如果使用`setValue:forKey:`设置nil，会引发`NSInvalidArgumentException`异常
>
> 异常的引发是因为程序自动执行了对象的`setNilValueForKey:`
>
> 所以，可以重写对象的`setNilValueForKey:`

```objective-c
//xxx.m
-(void)setNilValueForKey:(id) key{
  if([key isEqualToString:@"price"]){
    price = 0;
  }else{
    [super setNilValueForKey:key];
  }
}
```

#### 5.5.4 key路径

> KVC还可以操作复合属性
>
> 如FKOrder对象包含FKItem类型的item属性
>
> FKItem对象又包含name、price属性
>
> KVC可以通过item.name、item.price这种key路径来操作FKItem的name、price属性

> - setValue:forKeyPath:	根据Key路径设置
> - valueForKeyPath:   根据key路径获取

```objective-c
//main
VarArgs_sub* varArgs_sub = [[VarArgs_sub alloc] init];
[varArgs_sub setValue:[[VarArgs alloc] init] forKey:@"varArgs"];
        [varArgs_sub setValue:@"9999" forKeyPath:@"varArgs.test"];
        NSLog(@"test: %@",[varArgs_sub valueForKeyPath:@"varArgs.test"]);
```

> **注意**：没有对应的keyPath的异常方法，keyPath是嵌套调用`setValue:forKey:`和`valueForKey:`，重写异常处理去需要访问的对象实现部分重写

#### 5.5.5 键值监听kvo

> 参考：http://ningandjiao.iteye.com/blog/2009729

> iOS分为数据模型组件和视图组件，数据模型组件维护状态数据，视图组件负责显示状态数据
>
> 需要状态数据改变时，视图组件动态更新
>
> - 实现方案
>
>   - 数据模型组件持有视图组件的引用
>
>     > 状态数据改变，回调视图组件的特定方法，视图组件通过该方法动态更新
>
>     > 不足：应该是视图组件依赖数据模型组件，而这种方式造成了双向依赖
>
>   - 利用NSNotificationCenter
>
>     > 数组模型自己作为消息生产者，视图组件作为消息消费者
>
>     > 不足：都需要与iOS的消息中心耦合

> 使用KVO(Key Value Observing，键值监听)机制更好
>
> 由NSKeyValueObserving协议提供支持，NSObject的子类都可使用协议中的方法
>
> - 协议包含的方法
>   - `addObserver:forKeyPath:options:context:`
>     - 注册一个监听器监听指定的key路径
>   - `removeObserver:forKeyPath:`
>     - 删除指定key路径的监听器
>   - `removeObserver:forKeyPath:context:`
>     - 删除指定key路径的监听器，在指定context下

> 使用KVO机制，视图组件重写`observeValueForKeyPath:ofObject:change:context:`并作为数据模型组件的监听器
>
> 当key路径对应属性值改变，便回调监听器中实现的方法，监听器得到最新修改的数据，从而动态更新自身
>
> - KVO编程步骤
>   - 为被监听对象（一般是数据模型组件）注册监听器
>     - `addObserver:forKeyPath:options:context:`
>   - 重写监听器的方法
>     - `observeValueForKeyPath:ofObject:change:context:`
>     - **change的类型比较复杂**

### 5.6 对象初始化

#### 5.6.1 为对象分配空间

> - 调用alloc类方法（来自NSObject）
>   - 系统为该对象的所有实例变量分配内存空间
>   - 将每个实例变量的值重置为0，整型置为0，浮点型置为0.0，BOOL置为NO，指针变量置为nil

#### 5.6.2 初始化方法与对象初始化

- 初始化方法模板

  ```objective-c
  -(id)init
  {
    //调用父类init，将init得到的对象赋值给self对象
    //如果self不为nil，表名父类init方法成功执行
    if(self = [super init]){
      //执行初始化...
    }
    return self;
  }
  ```

#### 5.6.3 便利的初始化方法

> 通常以init开开头，允许自定义传入参数
>
> 在判断语句中，一般还是 `self = [super init]`
>
> 如果完全包含另一个初始化方法，可以是
>
> `self = [self initxx1:xx2 xx3:xx4...]`

### 5.7 类的继承

#### 5.7.1 继承的特点

> **被继承的类称为父类、基类或超类**
>
> **子类是一种特殊的父类，所以父类包含的范围比子类大**
>
> **父类是大类，子类是小类**
>
> **每个类最多只有一个直接父类**，可以有多个间接父类

> 用扩展(extends)来描述子类和父类的关系更准确，父类派生(derive)子类
>
> 子类扩展父类可以得到
>
> - 全部成员变量（**非private**）
>   - **接口部分不允许定义与父类接口部分重名的成员变量，即使父类接口部分定义的private的成员变量**
>   - **如果父类在实现部分定义成员变量，子类在接口可以定义与之重名的成员变量**
>   - 反之，子类也可以在自己的**实现部分定义**与父类接口重名的成员变量，只是这时**父类的成员变量被覆盖了**
> - 全部方法（包括初始化方法，**不包括工具方法**）

#### 5.7.2 重写父类的方法

> 子类包含与父类同名方法
>
> 也称为覆盖(override)
>
> 方法签名、形参标签要完全相同

#### 5.7.3 super关键字

> super用于限定对象调用它从父类继承得到的属性或方法
>
> 不能在类方法使用，类方法调用者只能是类本身而不是对象

### 5.8 多态

> 指针变量的类型有两个
>
> - 编译时
>   - 声明变量使用的类型决定
> - 运行时
>   - 实际赋给该变量的对象类型决定
>
> 如果运行时和编译时类型不一致，就会出现多态(Polymorphism)

#### 5.8.1 多态性

```objective-c
FKBase* polymorphism = [[FKSubclass alloc] init];
//执行继承的base
[polymorphism base];
//调用重写的test
[polymorphism test];
//因为编译类型是FkBase，FKBase没有sub方法，出错
[polymorphism sub];
//将任何类型的指针变量赋值给id类型变量，这样编译就不会出错，运行时调用赋值的类型方法
id dyna = polymorphism;
[dyna sub];
```

> 子类是一种特殊父类，可以不用类型转换将子类对象直接赋值给父类指针变量，称为**向上转型(upcasting)**

> `FKBase* polymorphism = [[FKSubclass alloc] init];`
>
> 如上语句，当运行时调用该指针变量的方法，其方法行为表现出子类的行为特征（存在覆盖，则调用子类重写的方法）
>
> 可能出现相同类型变量调用同一个方法时呈现多种不同的行为特征（取决于赋值的对象类型），即多态

> 虽然父类类型的指针变量接受子类对象赋值后，指向的对象确实包含子类独有的方法，但编译类型仍然是父类类型，所以编译时是无法调用的（可以通过performSelector:调用）
>
> 这时需要将该指针变量赋值给id类型的变量，再调用时就不会编译报错

#### 5.8.2 指针变量的强制类型转换

> **除了id类型的变量，指针类型变量只能调用编译时类型的方法，无法调用运行时类型的方法（即使它实际指向的对象包含该方法）**
>
> 可以通过强制类型转换将指针变量转换为运行时类型
>
> 强制转换只是改变该指针变量编译时类型，其实际指向的类型不会改变，若随意转换在调用方法时可能会出错

#### 5.8.3 判断指针变量的实际类型

> 为了保证程序正常运行，可以在类型转换前判断对象和指针变量类型是否一致
>
> - -(BOOL)isKindOfClass:class:
>   - 判断该对象是否为class或其子类的实例
> - -(BOOL)isSubclassOfClass:class:
>   - 判断该对象是否为class子类的实例

```objective-c
NSObject* hello = @"";
NSLog(@"%d",[hello isKindOfClass:[NSObject class]]);//YES
NSLog(@"%d",[hello isKindOfClass:[NSString class]]);//YES
NSLog(@"%d",[hello isKindOfClass:[NSDate class]]);//NO
```

