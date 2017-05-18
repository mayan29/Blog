# 《Objective-C 高级编程：iOS 与 OS X 多线程和内存管理》读书笔记


![封面](https://github.com/Mayan29/ReadingNotes/blob/master/03.《Objective-C%20高级编程：iOS%20与%20OS%20X%20多线程和内存管理》读书笔记/DATA/pic00.png)


## 1. alloc / retain / release / dealloc 实现

```objc
id obj = [NSObject alloc];
```

上面调用 NSOjbect 类的 alloc 类方法在 NSObject.m 源代码中的实现如下：

```objc
+ (instancetype)alloc
{
	return [self allocWithZone: NSDefaultMallocZone()];
}

+ (instancetype)allocWithZone:(NSZone *)z
{
	return NSAllocateObject(self, 0, z);
}
```

通过 allocWithZone: 类方法调用 NSAllocateObject 函数分配了对象。下面我们来看看 NSAllocateObject 函数：

```objc
struct obj_layout {
	NSUInteger retained;
};

inline id
NSAllocateObject(Class aClass, NSUInteger extraBytes, NSZone *zone) {
	int size = 计算容纳对象所需内存大小；
	id new = NSZoneMalloc(zone, size);
	memset(new, 0, size);
	new = (id) & ((struct obj_layout *)new)[1];
}
```

NSAllocateObject 函数通过调用 NSZoneMalloc 函数来分配存放对象所需的内存空间，之后将该内存空间置 0，最后返回作为对象使用的指针。

NSDefaultMallocZone、NSZoneMalloc 等名称中包含的 NSZone 是什么呢？它是为防止内存碎片化而引入的结构。对内存分配的区域本身进行多重化管理，根据使用对象的目的、对象的大小分配内存，从而提高了内存管理的效率。

但是，现在的 runtime 只是简单的忽略了区域的概念，runtime 中的内存管理本身已极具效率，使用区域来管理内存反而会引起内存使用效率低下以及源代码复杂化等问题

![内存碎片化](https://github.com/Mayan29/ReadingNotes/blob/master/03.《Objective-C%20高级编程：iOS%20与%20OS%20X%20多线程和内存管理》读书笔记/DATA/pic01.png)

以下是去掉 NSZone 后简化了的源代码：

```objc
struct obj_layout {
	NSUInteger retained;
};

+ (instancetype)alloc
{
	int size = sizeof(struct obj_layout) + 对象大小；
	struct obj_layout *p = (struct obj_layout *)calloc(1, size);
	return (id)(p + 1);
}
```

alloc 类方法用 struct obj_layout 中的 retained 整数来保存引用计数，并将其写入对象内存头部，该对象内存块全部置 0 后返回

![alloc 返回对象的内存图](https://github.com/Mayan29/ReadingNotes/blob/master/03.《Objective-C%20高级编程：iOS%20与%20OS%20X%20多线程和内存管理》读书笔记/DATA/pic02.png)

获取对象的引用计数 `[obj retainCount]` 内部实现：

```objc
- (NSUInteger)retainCount
{
	return NSExtraRefCount(self) + 1;
}

inline NSUInteger
NSExtraRefCount(id anObject)
{
	return ((struct obj_layout *)anObject)[-1].retained;
}
```

由对象寻址找到对象内存头部，从而访问其中的 retained 变量

![通过对象访问对象内存头部](https://github.com/Mayan29/ReadingNotes/blob/master/03.《Objective-C%20高级编程：iOS%20与%20OS%20X%20多线程和内存管理》读书笔记/DATA/pic03.png)

因为分配时全部置 0，所以 retained 为 0，由 NSExtraRefCount(self) + 1 得出，retainCount 为 1，可以推测出，retain 方法使 retained 变量加 1，而 release 方法使 retained 变量减 1