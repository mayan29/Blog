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
    backgroundColor : 'red',
  },
  innerContainer: {
    width : 200,
    height : 200,
    backgroundColor : 'green',
  }
});
```

