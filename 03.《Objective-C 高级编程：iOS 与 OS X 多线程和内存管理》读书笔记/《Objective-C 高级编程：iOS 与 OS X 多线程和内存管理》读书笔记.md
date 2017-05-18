# 《Objective-C 高级编程：iOS 与 OS X 多线程和内存管理》读书笔记


![封面](https://github.com/Mayan29/ReadingNotes/blob/master/03.《Objective-C%20高级编程：iOS%20与%20OS%20X%20多线程和内存管理》读书笔记/DATA/pic00.png)


## 1. 对象创建与释放的内部实现

### 1.1 alloc

在 NSObject 类的 alloc 类方法上设置断点，追踪程序的执行，以下列出了执行所调用的方法和函数：

```objc
+ alloc
+ allocWithZone:
class_createInstance
calloc
```

alloc 类方法首先调用 allocWithZone: 类方法，然后调用 class_createInstance 函数，最后通过调用 calloc 来分配内存块。class_createInstance 函数的源代码可以通过 [objc4 库](https://opensource.apple.com/source/objc4/objc4-709/runtime/)中的 runtime/objc-runtime-new.mm 进行确认

### 1.2 retainCount / retain / release

retainCount / retain / release 分别调用的方法和函数：

```objc
- retainCount
__CFDoExternRefOperation
CFBasicHashGetCountOfKey
```

```objc
- retain
__CFDoExternRefOperation
CFBasicHashAddValue
```

```objc
- release
__CFDoExternRefOperation
CFBasicHashRemoveValue
（CFBasicHashRemoveValue 返回 0 时，- release 调用 dealloc）
```

各个方法都调用了 CFRuntime.c 的 __CFDoExternRefOperation 函数，下面是简化后的源代码：

```objc
int __CFDoExternRefOperation(uintptr_t op, id obj) {
	CFBasicHashRef table = 取得对象对应的散列表（obj）;
	int count;
	
	switch(op) (
	case OPERATION_retainCount:
		count = CFBasicHashGetCountOfKey(table, obj);
		return count;
	case OPERATION_retain:
		CFBasicHashAddValue(table, obj);
		return obj;
	case OPERATION_release:
		count = CFBasicHashRemoveValue(table, obj);
		return 0 == count;
	)
}
```

苹果的实现就是采用散列表（引用计数表）来管理引用计数：

![通过散列表管理引用计数](https://github.com/Mayan29/ReadingNotes/blob/master/03.《Objective-C%20高级编程：iOS%20与%20OS%20X%20多线程和内存管理》读书笔记/DATA/pic01.png)

GNUstep 将引用计数保存在对象占用内存块头部的变量中，而苹果则是保存在引用计数表的记录中。GNUstep 的实现看起来既简单又高效，但是苹果如此实现必然有它的好处：

通过内存块头部管理引用计数的好处：

- 少量代码即可完成；
- 能够统一管理引用计数用内存块与对象用内存块。

通过引用计数表管理引用计数的好处：

- 对象用内存块的分配无需考虑内存块头部；
- 引用计数表各记录中存有内存块地址，可以各个记录追溯到各对象的内存块。

第二条这一特性在调试时有着重要的作用：即使出现故障导致对象占用的内存块损坏，但只要引用计数表没有被破坏，就能够确认各内存块的位置。下面是通过引用计数表追溯对象：

![通过引用计数表追溯对象](https://github.com/Mayan29/ReadingNotes/blob/master/03.《Objective-C%20高级编程：iOS%20与%20OS%20X%20多线程和内存管理》读书笔记/DATA/pic02.png)

另外，在利用工具检测内存泄漏时，引用计数表的各记录也有助于检测各对象的持有者是否存在。

### 1.3 autorelease

顾名思义，autorelease 就是自动释放，它类似于 C 语言中自动变量
的特性。

自动变量：程序执行时，若某自动变量超出其作用域，该自动变量将被自动废弃：

```objc
{
	int a;
}
```

上面这段代码，因为超出变量作用域，自动变量 `int a` 被废弃，不可再访问。

autorelease 的具体使用方法如下：

1. 生成并持有 NSAutoreleasePool 对象；
2. 调用已分配对象的 autorelease 实例方法；
3. 废弃 NSAutoreleasePool 对象。

一般情况下，我们不会使用 NSAutoreleasePool 对象来进行开发工作，但是在大量产生 autorelease 的对象时，只要不废弃 NSAutoreleasePool 对象，那么生成的对象就不能被释放，导致内存不足。

典型的例子就是，读入大量图像的同时改变其尺寸。图像文件读入到 NSData 对象，并从中生成 UIImage 对象，改变该对象尺寸后生成新的 UIImage 对象，产生大量 autorelease 对象。

解决办法是，在适当的地方生成、持有或废弃 NSAutoreleasePool 对象：

```objc
for (int i = 0; i < 图像数; i++) {
	NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
	
	// 读入图像，产生大量 autorelease 对象
	
	[pool drain];  // autorelease 对象被一起 release
}
```

但是我们在 ARC 需要使用 @autoreleasepool 块替代 NSAutoreleasePool

autorelease 的内部就是调用 NSAutoreleasePool 方法实现的。


## 2. ARC 的内部实现

65页