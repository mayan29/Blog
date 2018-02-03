# 零基础学 React Native？看这篇就够了！


## 1. 简介

### Native 的优点

- Native 的控件有更好的体验；
- Native 有更好的手势识别；
- Native 有更合适的线程模型，尽管 Web 端的 Web Worker 可以解决一部分问题，但是图像解码和文本渲染仍无法多线程渲染，影响了 Web 的流畅性。

### React Native 简介

- Facebook 于 2015 年 9 月 15 日发布 [React Native](https://github.com/facebook/react-native)。
- 目前在 iOS 上支持 iOS 8.0 以上版本，Android 上支持 Android 4.1 以上版本。
- 环境部署参照 [React Native 中文网站](https://reactnative.cn/docs/0.51/getting-started.html#content)


## 2. 基础语法

### 2.1. Hello World

```js
import React, { Component } from 'react';
import { StyleSheet, Text } from 'react-native';

export default class HelloWorld extends Component<{}> {
  render() {
    return (
      <Text style={styles.helloWorldStyle}>
        Hello World!
      </Text>
    );
  }
}

const styles = StyleSheet.create({
  helloWorldStyle: {
    fontSize: 20,
    textAlign: 'center',
    margin: 100,
  }
});
```

### 2.2. View

React Native 的核心机制之一就是虚拟 DOM：可以在内存中创建虚拟 DOM 元素。React Native 利用虚拟 DOM 来减少对实际 DOM 的操作从而提升性能。传统的创建方式可读性并不好，于是 React Native 发明了 JSX，利用我们熟悉的 HTML 语法来创建虚拟 DOM。在实际开发中，JSX 在产品打包阶段都已经编译成纯 JavaScript，JSX 的语法不会带来任何性能影响。因此 JSX 本身并不是高深的技术，可以说是一个比较高级、直观的语法糖。

```js
import React, { Component } from 'react';
import { StyleSheet, View } from 'react-native';

export default class ViewTest extends Component<{}> {
  render() {
    return (
      <View style={styles.container}>
        <View style={styles.innerContainer}>

        </View>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex : 1,  // 占满整个屏幕
    marginTop : 20,  // 上边距
    backgroundColor : 'red',
  },
  innerContainer: {
    width : 200,
    height : 200,
    backgroundColor : 'green',
  }
});
```

### 2.3. FlexBox 布局

#### FlexBox 简介

FlexBox（The Flexible Box Module，弹性盒模型），意为`弹性布局`，通过弹性的方式来对齐和分布容器中内容的空间，使其能适应不同屏幕，为盒装模型提供最大的灵活性。其主要思想是：让容器有能力让其子控件能够改变其宽度、高度，以最佳方式填充可用空间。React Native 中的 FlexBox 是这个规范的一个子集。

HTML 页面采用 CSS 布局，基于盒子模型，但是对于特殊布局非常不方便。CSS 常规布局是基于块和内联流方向，FlexBox 布局是基于 flex-flow 流。

#### FlexBox 容器属性 - 主轴方向

```js
container: {
  flex : 1,  
  marginTop : 20,  
  backgroundColor : 'red',
  flexDirection : 'row-reverse'  // 子控件主轴方向
},
```

其属性一共有四种：

| 属性 | 功能 |
| --- | --- |
| column | 主轴为垂直方向，起点在上边（默认）|
| column-reverse | 主轴为垂直方向，起点在下边 |
| row | 主轴为水平方向，起点在左边 |
| row-reverse | 主轴为水平方向，起点在右边 |

![主轴方向](https://github.com/Mayan29/Blog/blob/master/Blog/Images/image007.png)

#### FlexBox 容器属性 - 对齐方式

```js
container: {
  flex : 1, 
  marginTop : 20,  
  backgroundColor : 'red',
  flexDirection : 'row',
  justifyContent : 'space-around',  // 子控件对齐方式
},
```  

其属性一共有五种：

| 属性 | 功能 |
| --- | --- |
| flex-start | 起始位置对齐（默认）|
| flex-end | 结束位置对齐 |
| center | 中间位置对齐 |
| space-between | 平均分布，两端对齐 |
| space-around | 平均分布，两端保留一半空间 |

![对齐方式](https://github.com/Mayan29/Blog/blob/master/Blog/Images/image008.png)

#### FlexBox 容器属性 - 侧轴对齐方式

```js
container: {
  flex : 1,
  marginTop : 20,
  backgroundColor : 'red',
  flexDirection : 'row',
  justifyContent : 'flex-start',
  alignItems : 'center',  // 子控件侧轴对齐方式
},
```

其属性一共有五种：

| 属性 | 功能 |
| --- | --- |
| stretch | 未设置高度或者设置auto，将占满整个容器的高度（默认） |
| flex-start | 交叉轴的起点对齐 |
| flex-end | 交叉轴的终点对齐 |
| center | 交叉轴的中点对齐 |
| baseline | 项目第一行文字的基线对齐 |

#### FlexBox 容器属性 - 换行

```js
container: {
  flex : 1,
  marginTop : 20,
  backgroundColor : 'red',
  flexDirection : 'row',
  justifyContent : 'flex-start',
  alignItems : 'flex-start',
  flexWrap : 'wrap',  // 子控件是否换行
},
```

其属性一共有三种：

| 属性 | 功能 |
| --- | --- |
| nowrap | 不换行（默认） |
| wrap | 换行 |
| wrap-reverse | 换行，排在第一行上面，成为第一行（不常用） |

#### FlexBox 容器属性 - 占据比例

```js
// 子控件 1
innerContainer1: {  
  flex : 1,
  fontSize: 20,
  textAlign: 'center',
  backgroundColor : 'green',
},
// 子控件 2
innerContainer2: {  
  flex : 3,
  fontSize: 20,
  textAlign: 'center',
  backgroundColor : 'blue',
},
// 子控件 3
innerContainer3: {  
  flex : 1,
  fontSize: 20,
  textAlign: 'center',
  backgroundColor : 'orange',
},
```

![占据比例](https://github.com/Mayan29/Blog/blob/master/Blog/Images/image009.png)

#### FlexBox 容器属性 - 单个属性修改

alignSelf 其属性同 alignItems 侧轴对齐方式。允许单个 view 有与其他 view 不一样的对齐方式，可覆盖 alignItems（侧轴对齐方式）属性。默认为 auto，表示继承父元素的 alignItems 属性，如果没有父元素，则等同于 stretch。举个例子：其他 view 都是居中对齐，第一个 view 要求顶端对齐，可用此属性。