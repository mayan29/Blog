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


## 11. 52页