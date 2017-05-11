# 《Effective Objective-C 2.0》读书笔记


## 1. 了解 Objective-C

OC 使用`消息结构`（messaging structure）而不是`函数调用`（function calling），区别如下：

```objc
Object *obj = [Object new];
[obj performWith: parameter1 and: parameter2];
```

```objc
Object *obj = new Object;
obj->perform(parameter1, parameter2);
```

主要区别：

- 函数调用，运行时执行的代码由编译器决定；消息结构，由运行环境决定；
- 函数调用，如果调用的函数是多态的，运行时就要按照`虚方法表`（virtual table）来查出应该执行哪个函数实现；消息结构，不论是否动态，总是在运行时才会去查找所要执行的方法，实际上，编译器甚至不关心接收消息的对象是何种类型，接收消息的对象问题也要在运行时处理，其过程叫做`动态绑定`（dynamic binding）

Objective-C 的重要工作都由 runtime 而非编译器来完成，所需的全部数据结构和函数都在 runtime 组件里面，runtime 组件含有全部内存管理方法。runtime 本质上就是一种与开发者编写的代码相链接的`动态库`（dynamic library），其代码能把开发者编写的所有程序粘合起来。


## 2.在类的头文件中尽量少引入其他头文件

与 C 和 C++ 一样，OC 也使用`头文件`（header file）和`实现文件`（implementation file）来区隔代码。用 OC 编写的任何类几乎都需要引入 Foundation.h，如果包含 UI 控件，需要引入 UIKit.h

一般来说，应在某个类的头文件中使用`前向声明`（forward declaring）来提及别的类，这样做可以尽量降低类之间的耦合。

```objc
@class MYPerson;
```

循环引用（chicken-and-egg situation）：如果在各自头文件中引入对方的头文件，当解析其中一个头文件时，编译器会发现它引入了另一个头文件，而那个头文件又回过头来引用第一个头文件。使用 #import 而非 #include 指令虽然不会导致死循环，但是却意味着两个类里有一个无法被正确编译。


## 3.多用字面量语法，少用与之等价的方法

使用`字面量语法`（literal syntax）的好处是可以缩减源代码长度，使其更为易读。其本质为一种`语法糖`（syntactic sugar），以数组为例，其效果等于是先创建一个数组，然后把方括号里的所有对象都加到这个数组中。但是数组元素中不能有 nil，因为 arrayWithObjects: 方法会依次处理各个参数，直到发现 nil 为止，如果中间元素是 nil，该方法会提前结束。这个问题同样适用于字典 dictionaryWithObjectsAndKeys:

使用字面量语法创建出来的字符串、数组、字典都是不可变的，如果想变为可变的，需要复制一份：

```objc
NSMutableArray *mutable = [@[@1, @2, @3] mutableCopy];
```


## 4.多用类型常量，少用 #define 预处理指令

比如想把播放动画的时间提取为常量，使用预处理指令，也就是宏定义可以这样写：

```objc
#define ANIMATION_DURATION 0.3
```

上面的宏定义的缺点是，这样定义出来的常量没有类型信息，此外会把碰到的所有 ANIMATION_DURATION 一律替换成 0.3

```objc
static const NSTimeInterval kAnimationDuration = 0.3;
```

上面的方式定义的常量包含类型信息，其好处是清楚的描述了常量的含义。需要注意的是，如果常量局限于 .m 文件中，则在前面加字母 k；若常量在类之外可见，通常以类名为前缀，比如 MYViewAnimationDuration。

static 作用：假如声明此变量时不加 static，则编译器会为它创建一个外部符号，此时若是另一个实现文件也声明了同名变量，那么编译器会报错。

如果需要对外公开某个常量，可以声明为一个外界可见的常值变量，外界无需知道实际常量值，只需知道常量名称即可，此类常量需放在`全局符号表`（global symbol table）中，如下定义：

```objc
// .h
extern NSString *const MYStringConstant;

// .m
NSString *const MYStringConstant = @"VALUE";
```

编译器看到头文件中的 extern 关键字，无需查看其定义，即允许代码使用此常量。因为它知道，当链接成二进制文件之后，肯定能找到这个常量。

此类常量必须定义，并且只能定义一次，由实现文件生成目标文件时，编译器会在`数据段`（data section）为字符串分配存储空间。链接器会把此目标文件与其他目标文件相链接，以生成最终的二进制文件。凡是用到 MYStringConstant 这个全局符号的地方，链接器都能将其解析。

前例中的动画播放时长对外公布，可以这样声明：

```objc
// .h
extern const NSTimeInterval MYViewAnimationDuration;

// .m
const NSTimeInterval MYViewAnimationDuration = 0.3;
```

## 5. 用枚举表示状态、选项、状态码

C++11 标准修订了枚举的某些特性，其中一项改动就是：可以指明用何种`底层数据类型`（underlying type）来保存枚举类型的变量，这样做的好处是，可以向前声明枚举变量了。若不指定底层数据类型，编译器不清楚底层数据类型的大小，所以在用到此枚举类型时，就不知道究竟该给变量分配多少空间。

```objc
typedef enum MYConnectionState : NSUInteger {
    MYConnectionStateDisconnected,
    MYConnectionStateConnecting,
    MYConnectionStateConnected
} MYConnectionState;

typedef NS_ENUM(NSUInteger, MYConnectionState) {
    MYConnectionStateDisconnected,
    MYConnectionStateConnecting,
    MYConnectionStateConnected
};

MYConnectionState state = MYConnectionStateConnecting;
```

```objc
typedef enum MYConnectionState : NSUInteger {
    MYConnectionStateDisconnected = 1 << 0,
    MYConnectionStateConnecting   = 1 << 1,
    MYConnectionStateConnected    = 1 << 2
} MYConnectionState;

typedef NS_OPTIONS(NSUInteger, MYConnectionState) {
    MYConnectionStateDisconnected = 1 << 0,
    MYConnectionStateConnecting   = 1 << 1,
    MYConnectionStateConnected    = 1 << 2
};

MYConnectionState state = MYConnectionStateDisconnected | MYConnectionStateConnecting;
```

还有一种枚举的用法，就是在 switch 语句里，可以这样定义：

```objc
typedef NS_ENUM(NSUInteger, MYConnectionState) {
    MYConnectionStateDisconnected,
    MYConnectionStateConnecting,
    MYConnectionStateConnected
};

switch (_currentState) {
    case MYConnectionStateDisconnected:
        // ...
        break;
    case MYConnectionStateConnecting:
        // ...
        break;
    case MYConnectionStateConnected:
        // ...
        break;
}
```

用枚举来定义状态，最好不要有 default 分支，这样的话，如果稍后又加了一种状态，那么编译器就会发出警告信息，提示新加入的状态并未在 switch 分支中处理。


## 6. 理解属性这一概念

```objc
@interface MYPerson : NSObject {

	NSString *_firstName;
	NSString *_lastName;
	
@end
```

以上写法一般是 Java 和 C++ 的写法，而 OC 代码却很少这么做。这种写法的问题是：对象布局在`编译期`（compile time）就已经固定了。只要碰到访问 _fistName 变量的代码，编译器就把其替换为`偏移量`（offset），这个偏移量是`硬编码`（hardcode），表示该变量距离存放对象的内存区域的起始地址有多远。这样做目前来看没有问题，但是如果又加了一个实例变量，那就麻烦了。比如在 _firstName 之前又多了一个实例变量：

```objc
@interface MYPerson : NSObject {

	NSString *_dateOfBirth;
	NSString *_firstName;
	NSString *_lastName;
	
@end
```

原来表示 _firstName 的偏移量现在却指向 _dataOfBirth，把偏移量硬编码于其中的那些代码都会读取到错误的值

![在类中新增另一个实例变量前后的数据布局图](https://github.com/Mayan29/ReadingNotes/blob/master/02.《Effective%20Objective-C%202.0》读书笔记/DATA/pic01.png)

如果代码使用了编译期间计算出来的偏移量，那么在修改类定义之后必须重新编译，否则就会出错。

OC 的做法是，把实例变量当做一种存储偏移量所用的`特殊变量`（special variable），交由`类对象`（class object）保管，偏移量会在运行期查找，如果类的定义变了，那么存储的偏移量也就变了，所以总能使用正确的偏移量，甚至可以在运行期向类中新增实例变量。

这个问题还有一种解决方法，就是尽量不要直接访问实例变量，应该通过存取方法来做。OC 这门语言就是根据名称自动创建出存取方法，也就是@property语法。

所有属性都声明为 nonatomic 是因为：在 iOS 中使用同步锁开销较大，一般不要求属性必须是原子的，因为这并不能保证线程安全，若要实现线程安全，还需采用更为深层的锁定机制才行。例如，一个线程在连续多次读取某属性值的过程中有别的线程在同时改写改值，那么即便将属性声明为 atomic 也还是会读到不同的属性值。


## 7. 在对象内部尽量直接访问实例变量

直接访问实例变量和通过属性访问区别如下：

- 直接访问实例变量的速度更快，由于不经过 OC 的方法派发（method dispatch），编译器所生成的代码会直接访问保存对象实例变量的那块内存；

- 直接访问实例变量不会调用设置方法，比如在 ARC 下直接访问一个声明为 copy 的属性，那么并不会拷贝该属性，只会保留新值并释放旧值；

- 直接访问实例变量不会触发 KVO 通知；

- 通过属性访问有助于排查与之相关的错误

总结为：在对象内部读取数据时，直接通过实例变量来读；写入数据时，通过属性 set 方法来写。


## 8. 以类族模式隐藏实现细节

`类族`（class cluster）是一种很有用的模式，可以隐藏`抽象基类`（abstract base class）背后的实现细节，例如 UIButton 的类方法：

```objc
+ (UIButton *)buttonWithType:(UIButtonType)type;
```

该方法所返回的对象，其类型取决于传入的按钮类型，然而，不管返回什么类型的对象，他们都继承同一个基类：UIButton，这么做的意义是，UIButton 的使用者无须关心创建出来的按钮具体属于哪个子类，只需明白如何创建按钮，设置属性即可。

举一个例子，创建一个 MYPerson 抽象类，MYStudent、MYTeacher、MYWorker 每一个子类都继承 MYPerson

```objc
#import <Foundation/Foundation.h>

@interface MYPerson : NSObject


typedef NS_ENUM(NSUInteger, MYPersonType) {

    MYPersonTypeStudent,
    MYPersonTypeTeacher,
    MYPersonTypeWorker,
};


@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSUInteger age;



+ (instancetype)personWithType:(MYPersonType)type;


- (void)introduceMyself;

@end
```

```objc
#import "MYPerson.h"
#import "MYStudent.h"
#import "MYTeacher.h"
#import "MYWorker.h"

@implementation MYPerson


+ (instancetype)personWithType:(MYPersonType)type
{
    switch (type) {
        case MYPersonTypeStudent:
            return [[MYStudent alloc] init];
            break;
        case MYPersonTypeTeacher:
            return [[MYTeacher alloc] init];
            break;
        case MYPersonTypeWorker:
            return [[MYWorker alloc] init];
            break;
    }
}


- (void)introduceMyself
{
    NSLog(@"%@", self.class);
}


@end
```

方法调用，打印结果为 MYWorker

```objc
MYPerson *person = [MYPerson personWithType:MYPersonTypeWorker];
[person introduceMyself];
```

## 9. 理解 objc_msgSend 的作用

### 9.1 基本消息传递

```objc
void objc_msgSend(id self, SEL cmd, ...)
```

objc_msgSend 函数依据`接受者`（receiver）和`选择子`（selector）的类型来调用适当的方法，该方法需要在接受者所属的类中搜寻其`方法列表`（list of methods），如果能找到与选择子名称相符的方法，就跳至其实现代码。如果找不到，就沿着继承体系继续向上查找，找到合适的方法再跳转。如果最终还是找不到，那就执行`消息转发`（message forwarding）

这么说来，想调用一个方法似乎需要很多步骤，但是，objc_msgSend 会将匹配结果缓存在`快速映射表`（fast map）里面，每个类都有这样一块缓存，如果稍后还向该类发送与选择子相同的消息，那么执行起来就很快了。

### 9.2 其他消息传递

#### objc\_msgSend\_stret

如果待发送的消息要返回结构体，可交由此函数处理。只有当 CPU 的寄存器能够容纳得下消息返回类型时，这个函数才能处理。如果返回的结构体太大了，就由另一个函数执行派发，会通过分配在栈上的某个变量来处理消息所返回的结构体。

#### objc\_msgSend\_fpret

如果消息返回的是浮点数，那么可交由此函数处理。通常所用的 objc_msgSend 在这种情况下并不适合，这个函数是为了处理 x86 等架构 CPU 中某些奇怪状况。

#### objc\_msgSendSuper

如果要给超类发消息，例如 `[super message:parameter]`，那么就交由此函数处理。也有另外两个与 objc_msgSend_stret 和 objc_msgSend_fpret 等效的函数，用于处理发给 super 的相应消息。

### 9.3 尾调用优化

每个类里都有一张表格，其中的指针都会指向这个函数，而选择子的名称则是查表时所用的 key，objc_msgSend 等函数的原理是，通过这张表格来寻找应该执行的方法。

原型的样子和 objc_msgSend 函数很像，这是利用`尾调用优化`（tail-call optimization）技术（函数最后一项操作是调用另外一个函数），编译器会生成跳转至另一个函数所需的指令码，而且不会向调用堆栈中推入新的`栈帧`（frame stack）。这项优化对 objc_msgSend 非常关键，如果不这么做的话，每次调用 OC 方法之前，都需要为调用 objc_msgSend 函数准备栈帧。此外，若不优化，还会过早的发生`栈溢出`（stack overflow）现象。


## 10. 理解消息转发机制

在编译期间向类发送了其无法解读的消息并不会报错，因为在运行期可以继续向类中添加方法。当对象接收到无法解读的消息后，就会启动`消息转发`（message forwarding）机制。

消息转发分为两大阶段

#### 动态方法解析（dynamic method resolution）

征询接收者，所属的类，看是否能动态添加方法，以处理当前这个`未知的选择子`（unknown selector）

#### 完整的消息转发机制（full forwarding mechanism）

首先，接受者看看有没有其他对象能处理这条消息，如果有，运行时会把消息转给那个对象，消息转发过程结束，一切正常。若没有`备援的接收者`（replacement receiver）则启动完整的消息转发机制，运行时会把与消息有关的全部细节封装到 NSInvocation 对象中，再给接受者最后一次机会，令其设法解决当前还未处理的这条消息。

### 10.1 动态方法解析

对象在收到无法解读的消息后，首先将调用其所属类的下列类方法：

```objc
+ (BOOL)resolveInstanceMethod:(SEL)selector
```

如果尚未实现的方法是类方法，那么会调用：

```objc
+ (BOOL)resolveClassMethod:(SEL)selector
```

该方法的参数就是那个未知的选择子，其返回值为 BOOL 类型，表示这个类是否能新增一个实例方法用以处理此选择子。

使用这种方法的前提是：相关方法的实现代码已经写好，只等着运行时动态插在类里面就可以了。

举个例子，调用 Person 的 fly 方法

```objc
Person *p = [[Person alloc] init];
    
SEL sel = NSSelectorFromString(@"fly");
[p performSelector:sel];
```    

```objc
#import "Person.h"
#import <objc/runtime.h>

@implementation Person


+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    
    NSString *selString = NSStringFromSelector(sel);
    if ([selString isEqualToString:@"fly"]) {
        
        class_addMethod(self, sel, (IMP)fly, "v@:");
        
        return YES;
    }
    return [super resolveInstanceMethod:sel];
}

void fly(id self, SEL _cmd) {
    
    NSLog(@"抱歉，人是不能飞的");
}

@end
```

### 10.2 备援的接收者

如果上一步没有找到自定义的方法实现，这一步中，运行时系统会问它，能不能把这条消息转给其他接收者来处理

在上例中，如果 `resolveInstanceMethod:` 没有方法实现，则通过 `forwardingTargetForSelector:` 方法指定其他类来实现

```objc
#import "Person.h"
#import "Bird.h"

@interface Person ()

@property (nonatomic, strong) Bird *bird;

@end

@implementation Person

- (Bird *)bird
{
    if (!_bird) {
        _bird = [[Bird alloc] init];
    }
    return _bird;
}

- (id)forwardingTargetForSelector:(SEL)aSelector
{
    
    NSString *selString = NSStringFromSelector(aSelector);
    if ([selString isEqualToString:@"fly"]) {
        
        return self.bird;
    }
    return [super forwardingTargetForSelector:aSelector];
}

@end
```

不用在 Bird.h 中暴露方法名，在 Bird.m 中实现即可

```objc
#import "Bird.h"

@implementation Bird

- (void)fly
{
    NSLog(@"我能飞");
}

@end
```

### 10.3 完整的消息转发

如果转发算法已经来到这一步的话，则启用完整的消息转发机制。

```objc
#import "Person.h"
#import "Bird.h"

@interface Person ()

@property (nonatomic, strong) Bird *bird;

@end

@implementation Person

- (Bird *)bird
{
    if (!_bird) {
        _bird = [[Bird alloc] init];
    }
    return _bird;
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
{
    NSMethodSignature *signature = [super methodSignatureForSelector:aSelector];
    
    if (!signature) {
        if ([Bird instancesRespondToSelector:aSelector]) {
            signature = [Bird instanceMethodSignatureForSelector:aSelector];
        }
    }
    return signature;
}

- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    if ([Bird instancesRespondToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:self.bird];
    }
}

@end
```

接收者在每一步均有机会处理消息，步骤越往后，处理消息的代价就越大，最好能在第一步就处理完，这样的话，运行时就可以将此方法缓存取来了。若想在第三步里把消息转给备援的接收者，那还不如把转发操作提前到第二步。因为第三步只是修改了调用目标，这项改动放在第二步执行会更为简单，不然的话，还得创建并处理完整的 NSInvocation



## 11. 自定义初始化方法

```objc
#import "Person.h"

@implementation Person


- (instancetype)initWithName:(NSString *)name
{
    self = [super init];
    if (self) {
        NSLog(@"我的名字为：%@", name);
    }
    return self;
}

- (instancetype)init
{
    @throw [NSException exceptionWithName:NSInternalInconsistencyException reason:@"Person 类必须使用 initWithName: 方法初始化" userInfo:nil];
}

@end
```


## 12. 实现 description 方法

调试程序时，经常需要打印并查看对象信息，比如打印数组信息：

```objc/Users/may-g/Desktop/立刻删除/立刻删除
NSArray *arr = @[@1, @2, @3, @4];
NSLog(@"%@", arr);
```

打印结果为

```objc
(
    1,
    2,
    3,
    4
)
```

但是如果打印自定义的类的时候却是这样的：

```
<Person: 0x608000001530>
```

上面这种内容不太有用，所以我们应该在自己的类中覆写 description 方法：

```objc
#import "Person.h"

@implementation Person


- (NSString *)description
{
    return [NSString stringWithFormat:@"<%@ : %p>, name = \"%@\" age = %lu", self.class, self, self.name, self.age];
}

@end
```

创建并打印自定义类：

```objc
Person *p = [[Person alloc] init];
p.name = @"mayan";
p.age = 26;
    
NSLog(@"%@", p);
```

打印结果如下

```objc
<Person : 0x600000025560>, name = "mayan" age = 26
```

以前还需要覆写 `debugDescription` 方法才能在断点时自定义打印，现在仅仅设置好 `description` 就可以了


## 13. 理解 NSCopying 协议

对象的拷贝通常使用 `copy` 方法完成，如果想令自己的类支持拷贝操作，那么就要实现 NSCopying 协议

```objc
- (id)copyWithZone:(NSZone *)zone;
```

为何会出现 NSZone 呢？因为以前开发中，会据此把内存分为不同的`区`（zone），而对象会创建在某个区里面。现在不用了，每个程序只有一个`默认区`（default zone），所以实现这个方法，不必担心 zone 参数。

```objc
#import "Person.h"

@interface Person () <NSCopying>

@end

@implementation Person


- (instancetype)initWithName:(NSString *)name andAge:(NSInteger)age
{
    if (self = [super init]) {
        _name = [name copy];
        _age  = age;
    }
    return self;
}

- (id)copyWithZone:(NSZone *)zone
{
    return [[[self class] allocWithZone:zone] initWithName:_name andAge:_age];
}

@end
```

## 14. 协议、委托、分类

OC 语言有一项特性叫做`协议`（protocol），它与 Java 的`接口`（interface）类似。定义一套接口，某对象若想接受另一个对象的委托，则需遵从此接口，以便成为其`委托对象`（delegate），而这另一个对象则可以给其委托对象回传一些信息，也可以在发生相关事件时通知委托对象。

此模式可将数据和业务逻辑解耦，比方说，用户界面里有个显示数据所用的视图，那么，此视图只应包含显示数据所需的逻辑代码，而不应决定要显示何种数据以及数据之间如何交互问题。视图对象的属性中，可以包含负责数据与事件处理的对象。这两种对象分别称为`数据源`（data source）与`委托`（delegate）

利用`分类`（Category）机制，我们无须继承子类即可直接为当前类添加方法，而在其他语言中，需通过继承子类来实现。由于 OC 运行时是高度动态的，所以才能支持这一特性。


## 15. 以自动释放池块降低内存峰值

```objc
for (int i = 0; i < 100000; i++) {

    [self doSomethingWithInt:i];
}
```

如上所示代码，如果 `doSomethingWithInt:` 方法要创建临时对象，那么这些对象很可能放在自动释放池里，比方说，它们可能是一些临时字符串。但是，即使这些对象在调用完方法之后就不再使用，但是它们依然处于存活状态。这样一来，在执行 for 循环时，应用程序所占内存量就会持续上涨，等所有临时对象都释放之后，内存用量又突然下降。

如果把循环内的代码包裹在自动释放池块中，那么在循环中自动释放的对象就会放在这个池，而不是线程的主池里面：

```objc
for (int i = 0; i < 100000; i++) {
        
    @autoreleasepool {
        [self doSomethingWithInt:i];
    }
}
```

这样，应用程序在执行循环时的内存峰值就会降低，但是是否应该用池来优化效率，完全取决于具体的应用程序。首先得监控内存用量，判断其中有没有需要解决的问题，别着急优化。尽管自动释放池块的开销不大，但是毕竟还是有的，所以尽量不要建立额外的自动释放池。


## 16. 理解 block

当前多线程编程的核心就是`块`（block）和`大中枢派发`（Grand Central Dispatch，GCD），block 是一种在 C、C++、OC 中使用的`词法闭包`（lexical closure）

![在类中新增另一个实例变量前后的数据布局图](https://github.com/Mayan29/ReadingNotes/blob/master/02.《Effective%20Objective-C%202.0》读书笔记/DATA/pic02.png)

- 在存放 block 对象的内存区域中，首个变量是 isa，指向 Class 对象的指针；
- 最重要的就是 invoke 变量，这是个函数指针，指向 block 的实现代码；
- descriptor 变量是指向结构体的指针，其中声明了 block 对象的总体大小，还声明了 copy 和 dispose 这两个辅助函数所对应的函数指针，辅助函数在拷贝和丢弃 block 对象时运行，其中会执行一些操作，比如，前者要保留捕获的对象，后者将之释放；
- block 还会把它所捕获的所有变量都拷贝一份，这些拷贝放在 descriptor 变量后面，捕获了多少个变量，就要占据多少内存空间。拷贝的并不是对象本身，而是指向这些对象的指针变量，原因是执行 block 时，要从内存中把这些捕获到的变量读出来。


## 17. 多用派发队列，少用同步锁

防止多线程因为执行同一份代码而出问题，通常使用锁来实现某种同步机制。

```objc
@synchronized(self) {
    // do something
}
```

这种写法的缺点是，滥用同步锁会降低代码效率，因为共用同一个锁的那些同步块，都必须按顺序执行，如果在 self 对象上频繁加锁，那么程序可能要等另一段与此无关的代码执行完毕，才能继续执行当前代码。

166页