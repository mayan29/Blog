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

![在类中新增另一个实例变量前后的数据布局图](ddd)
