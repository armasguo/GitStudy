## 6 面向对象（下）

### 6.1 objective-c的包装类

> objective-c扩展自C，C中的基本数据类型（int、short、float、double等）都不是对象，没有属性和方法
>
> 所以objective-c提供了**NSValue、NSNumber**封装基本数据类型，使它们具有面向对象的特征

#### 6.1.1 它们不是包装类

- 这3个类型依旧是基本类型
  - NSInteger：大致等于long型整数
  - NSUInteger：大致等于unsigned long型整数
  - CGFloat：在64位平台大致相当于double，32位平台大致相当于float
- 对于64位和类似于64位平台，NSInteger、NSUInteger分别相当于long、unsigned long；为了兼容性，整形变量最好用这两种

![6.1-1](/Users/Armas/Desktop/objective-c笔记/6.1-1.png)

- 对于64位平台，CGFloat相当于double；为了兼容性，浮点型变量最好用这种

![6.1-2](/Users/Armas/Desktop/objective-c笔记/6.1-2.png)

#### 6.1.2 nsvalue和nsnumber

- NSValue

  > 是NSNumber的父类，更通用的包装类
  >
  > 用于包装单个short、int、long、float、char、指针、对象id等数据项

- NSNumber

  > 更具体的包装类，主要用于包装C语言的数值类型
  >
  > 主要包含3类方法
  >
  > - +numberWithXxx：将基本数据类型包装
  > - -initWithXxx：先创建一个NSNumber对象，再用基本数据类型初始化
  > - -xxxValue：返回被包装的基本类型的值

> oc提供了自动装箱机制，例如将整型值直接赋值给NSNumber变量
>
> 但这种机制不完善，建议显示地将基本类型包装
>
> - 自动装箱生成的NSNumber不支持ARC
> - 不能将浮点数赋值

### 6.2 处理对象

#### 6.2.1 打印对象和description方法

> NSObject提供的description方法总是返回`<对象名：16进制的首地址>`
>
> 一般需要重写description方法，在打印对象时输出有意义的信息
>
> ```objective-c
> -(NSString*)description{
>   return [NSString stringWithFormat:@"..."];
> }
> ```

#### 6.2.2 ==和isequal方法

- ==

> 在两个变量是基本类型且是数值型，两个变量相等，使用==判断将返回真
>
> 对于指针型变量，它们必须指向同一个对象（保存的内存地址相同），使用==判断才返回真
>
> 当使用==比较类型上没有继承关系的两个指针变量时，编译器会警告

```objective-c
NSString* str1 = @"疯狂iOS";
NSString* str2 = @"疯狂iOS";
NSLog(@"str1: %p str2: %p",str1,str2);//地址相同
NSLog(@"str1是否和str2相等: %d",str1==str2);//1
NSString* str3 = [NSString stringWithFormat:@"疯狂iOS"];
NSLog(@"str3: %p",str3);//与str1、str2不同
NSLog(@"str1是否和str3相等: %d",str1==str3);//0
```

> 直接将字符串赋值给NSString变量称为直接量，该字符串被放入**常量池**，在常量池中同一个字符串直接量只有一个
>
> stringWithFormat:类方法创建的字符串对象是运行时创建，被保存在**运行时内存区（堆内存）**

- isEqual:

> isEqual是NSObject提供的实例方法，它和==没有区别
>
> 需要重写isEqual:实现自定义比较，比如NSString的isEqual:方法

```objective-c
-(BOOL)isEqual:(id)other{
  //两个对象为同一对象--自反性
  if(self == other)
    return YES;
  //other不为nil，且是FKUser实例
  if(other!=nil && [other isMemberOfClass:FKUser.class]){
    FKUser* target = (FKUser*)other;
    return [self.idStr isEqual:target.idStr];
  }
  return NO;
}
```

> 重写isEqual:要求
>
> - 自反性
>   - [x isEqual:x]一定返回真
> - 对称性
>   - [x isEqual:y]和[y isEqual:x]返回相同
> - 传递性
>   - [x isEqual:y]和[y isEqual:z]都返回真，[x isEqual:z]也返回真
> - 一致性
>   - 无论比较多少次，只要比较的信息不变，最后结果一致
> - x不为nil，返回假
>   - [x isEqual:nil]返回假，若x不为nil

### 6.3 类别与扩展

> 需要为类扩展添加新的行为，虽然可以继承来实现，但有时不是最好的选择
>
> 比如NSNumber，因为他只是一个类簇（cluster）的前端类，NSNumber相当于一个抽象父类
>
> 当我们使用`[NSNumber numberWithInt:5]`方法生成的NSNumber对象只是NSNumber子类的实例
>
> 即使为NSNumber派生子类也没有意义，派生的子类对NSNumber现有的子类没有影响
>
> 此时可以用类别实现

- 类簇
  > 选定一个父类，模拟抽象类
  >
  > 以该父类派生多个子类
  >
  > 在需要使用这些子类时，面向父类编程
  >
  > 当调用父类的初始化、静态方法时，返回的是子类的实例
  >
  > 这系列类被称为**类簇（cluster）**

![6.3.1-1](/Users/Armas/Desktop/objective-c笔记/6.3.1-1.png)

#### 6.3.1 类别（category）

> OC的动态特征允许使用category为现有的类添加新方法，在**不创建子类、不访问原类源代码**的情况下
>
> 可以将**类定义模块化地分布**到多个相关文件

> 类别由接口、实现部分组成，类别文件的命名一般是`类名+类别名`
>
> - 接口
>
>   ```objective-c
>   @interface 已有类名(类别名)
>   //方法定义
>   ...
>   @end
>   ```
>   - 定义类别和类的语法区别
>     - 定义类使用的类名必须项目没有，定义类别相反
>     - 必须用**圆括号包含类别名**
>     - 类别**一般只定义方法**
>
> - 实现
>
>   ```objective-c
>   @implementation 已有类名(类别名)
>   //方法实现
>   ...
>   @end
>   ```

> - 注意
>   - 类别可以重写原有类的方法，但最好用继承的方式
>   - 通过类别为指定类添加新方法，会影响指定类及其子类
>   - 可以为一个类定义多个类别
>   - 类别通常的用法有三种
>     - 利用类别对类进行模块化设计
>     - 使用类别调用私有方法
>     - 使用类别实现非正式协议

#### 6.3.2 利用类别对类进行模块化设计

> 一般类使用*.h定义接口，.m定义实现，不能将实现分布到多个m文件
>
> 当需要将一个很大的类分模块设计时，**可以用类别的方式将类的实现分布到多个*.m文件**

```objective-c
//NSWindow.h
@interface NSWindow:NSResponser<NSAnimatablePropertyContainer,NSUserInterfaceValidations,NSUserInterfaceItemIdentification>
//category
@interface NSWindow(NSKeyboardUI)
@interface NSWindow(NSToolbarSupport)
@interface NSWindow(NSDrag)
@interface NSWindow(NSCarbonExtensions)
```

> 通过这种方式，NSWindow类按模块分不到不同*.m文件，提高项目可维护性
>
> NSWindow.m、NSWindow+NSKeyboardUI.m、NSWindow+NSToolbarSupport.m、NSWindow+NSDrag.m、NSWindow+NSCarbonExtensions.m

#### 6.3.3 使用类别来调用私有方法

> 虽然可以使用`performSelector:`调用私有方法，但这避开了编译器的语法检查，可能会产生未知错误
>
> 使用类别定义**前向引用**，也可以调用私有方法

```objective-c
//main.m
@interface VarArgs(im_in)
//声明私有方法im_in
-(void)im_in:(NSNumber*)arg;
@end

int main(int argc, const char* argv[]){
  @autoreleasepool {
    VarArgs_sub* varArgs_sub = [[VarArgs_sub alloc] init];
    [varArgs_sub im_in:[NSNumber numberWithDouble:1.31]];
    NSLog(@"public_int: %@",[varArgs_sub valueForKey:@"public_int"]);
  }
}
```

> 定义类别的目的只是为了让**编译器知道VarArgs类有`-(void)im_in:(NSNumber*)arg`这个私有方法**
>
> 所以即使改变了声明方法的返回类型、参数类型（只要不导致编译器报错，会产生警告），只要调用时按照原方法规则，最后都能成功执行

#### 6.3.4 扩展（extension）

> 扩展相当于匿名类别
>
> ```objective-c
> @interface 已有类名()
> {
>   实例变量
> }
> //方法定义
> ...
> @end
> ```

> 类别有单独的头文件和实现文件，但扩展只是临时对某个类接口扩展，统一在一个实现文件实现全部的方法
>
> **扩展可以额外增加实例变量**，使用@property、@synthesize
>
> 但定义类的类别时，不允许定义额外的实例变量

> 在实现文件和使用该类的文件中导入头文件时，需要导入整个接口和扩展部分，如`FKCar+drive.h`

### 6.4 协议（protocol）与委托