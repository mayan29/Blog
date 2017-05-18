# 《Objective-C 高级编程：iOS 与 OS X 多线程和内存管理》读书笔记》读书笔记


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