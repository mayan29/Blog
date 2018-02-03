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

### 核心机制

核心机制之一：虚拟 DOM，可以在内存中创建虚拟 DOM 元素。利用虚拟 DOM 来减少对实际 DOM 的操作从而提升性能


## 2. 基础语法

### 2.1. Hello World

```js
import React, { Component } from 'react';
import { StyleSheet, Text } from 'react-native';

export default class App extends Component<{}> {
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


