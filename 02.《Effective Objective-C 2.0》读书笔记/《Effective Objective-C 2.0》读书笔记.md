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

> 循环引用（chicken-and-egg situation）：如果在各自头文件中引入对方的头文件，当解析其中一个头文件时，编译器会发现它引入了另一个头文件，而那个头文件又回过头来引用第一个头文件。使用 #import 而非 #include 指令虽然不会导致死循环，但是却意味着两个类里有一个无法被正确编译。


## 3.多用字面量语法，少用与之等价的方法

使用`字面量语法`（literal syntax）的好处是可以缩减源代码长度，使其更为易读。其本质为一种`语法糖`（syntactic sugar），以数组为例，其效果等于是先创建一个数组，然后把方括号里的所有对象都加到这个数组中。但是数组元素中不能有 nil，因为 arrayWithObjects: 方法会依次处理各个参数，直到发现 nil 为止，如果中间元素是 nil，该方法会提前结束。这个问题同样适用于字典 dictionaryWithObjectsAndKeys:

使用字面量语法创建出来的字符串、数组、字典都是不可变的，如果想变为可变的，需要复制一份：

```objc
NSMutableArray *mutable = [@[@1, @2, @3] mutableCopy];
```