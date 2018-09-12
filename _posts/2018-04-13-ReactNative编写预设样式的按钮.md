---
layout:     post
title:      React Native编写预设样式的按钮
subtitle:   
date:       2018-04-13
author:     HelloMin
header-img: img/post-bg-debug.png
catalog: true
tags:
    - React Native
---

导语：
  > 按钮在应用开发中应该是很常见的了。本文主要总结了如何从开发一个简单的预设了固定样式的按钮，到开发一个预设了多个可选择的样式的按钮。文中的代码有部分省略，全部代码可以在[我的Github代码库](https://github.com/HelloMin/ReactNativeDemoLib)找到。


## 1. 预定义不可变样式的按钮

这应该是最简单的方式，我们采用无状态组件的方式实现它。在这里，我们实现了一个可以传入点击事件以及显示文字的按钮，并且预设了按钮和文字的样式。

```js
import React from 'react';
import {
  Text,
  StyleSheet,
  TouchableOpacity,
} from 'react-native';

const MyButton = ({onPress, title}) => {
  return (
    <TouchableOpacity
      activeOpacity={0.8}
      onPress={onPress}
      style={styles.defaultBtn}
    >
      <Text style={styles.defaultText} numberOfLines={1}>
        {title}
      </Text>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  defaultBtn: {
    height: 30,
    width: 80,
    borderRadius: 15,
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: 'pink',
  },
  defaultText: {
    fontSize: 13,
    color: 'white',
  },
});

export default MyButton;
```
我们可以这样使用这个按钮。

```js
<MyButton onPress={() => alert("Press!")} title="HelloMin"/>
```
如果我们希望像Text组件这样，可以将需要显示的文字放在按钮标签中间，而不是这样僵硬的用给title赋值的方式传进去，也是可以的。下面的例子展示了我们需要更改的地方。

```js
const MyButton = ({onPress, children}) => {
  return (
    <TouchableOpacity
      activeOpacity={0.8}
      onPress={onPress}
      style={styles.defaultBtn}
    >
      <Text style={styles.defaultText} numberOfLines={1}>
        {children}
      </Text>
    </TouchableOpacity>
  );
};
```
如上所述，我们将按钮标签下的children直接放进了Text组件下，使用这个按钮就可以像常见的组件一样方便了。

```js
<MyButton onPress={() => alert("Press!")}>
  HelloMin
</MyButton>
```
## 2.预定样式，并且可以外部更改部分样式的按钮
虽然不用多次给按钮写样式了，省了很多时间，但是，如果我们想要一个类似的，但是颜色改变了的按钮怎么办呢？
通过设置样式的优先级，我们可以像传递点击事件一样，在已有样式的基础上，把我们希望更改的样式传进去.

```js
const MyButton = ({onPress, btnstyle, textStyle, children}) => {
  return (
    <TouchableOpacity
      activeOpacity={0.8}
      onPress={onPress}
      style={[styles.defaultBtn, btnstyle]}
    >
      <Text style={[styles.defaultText, textStyle]} numberOfLines={1}>
        {children}
      </Text>
    </TouchableOpacity>
  );
};
```
这样一来，我们就可以通过传递btnstyle和textStyle，分别改变部分或者全部按钮以及文字的样式了。

```js
<MyButton
  textStyle={{color:'green'}}
  btnstyle={{backgroundColor:'yellow'}}
  onPress={() => alert("Press!")}>
  HelloMin
</MyButton>
```
## 3.一个有着多个预定义样式的按钮
很多时候，为了使应用显得更有整体性，项目中的UI风格是固定的。我们常用的按钮样式常常是固定的几种，如果每次都在一种的基础上做修改，就太浪费时间了。更好的方式是，把所需的几种样式全部作为按钮的预设样式，在使用的时候，通过传递一定的参数来控制按钮的样式就好了。具体修改如下。

```js
const MyButton = ({onPress, type = 'default', btnstyle, textStyle, children}) => {
  return (
    <TouchableOpacity
      activeOpacity={0.8}
      onPress={onPress}
      style={[styles.defaultBtn, styles[`${type}Btn`], btnstyle]}
    >
      <Text style={[styles.defaultText, styles[`${type}Text`], textStyle]} numberOfLines={1}>
        {children}
      </Text>
    </TouchableOpacity>
  );
};
```

此处，我们需要把想要的样式先写在按钮的样式表里。注意，为了方便传参的简洁性，样式的命名是遵循一定规律的，这样也有助于代码的可读性。
```js
const styles = StyleSheet.create({
  defaultBtn: {
    height: 30,
    width: 80,
    margin: 20,
    borderRadius: 15,
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: 'pink',
  },
  defaultText: {
    fontSize: 13,
    color: 'white',
  },
  // 新增的三种样式
  darkBtn: {
    backgroundColor: '#1DB0B8',
  },
  darkText: {
    color: '#FFF',
  },
  middleBtn: {
    borderWidth: 1,
    borderColor: '#0BBE06',
    backgroundColor: '#37C6C0',
  },
  middleText: {
    color: '#00343F',
  },
  lightBtn: {
    backgroundColor: '#D0E9FF',
  },
  lightText: {
    color: 'black',
  },
});
```
于是，使用按钮的方式也发生了变化。我们只需更改type的值，就可以简单的控制按钮的样式了。

```js
<MyButton
  type='dark'
  onPress={() => alert("Press!")}>
  dark
</MyButton>
<MyButton
  type='middle'
  onPress={() => alert("Press!")}>
  dark
</MyButton>
<MyButton
  type='light'
  onPress={() => alert("Press!")}>
  dark
</MyButton>
```
上面代码的效果如下:
<img src="/img/post_img/default-type-button.png" alt="四种预定义样式的按钮" width="250px" border="1"/>

如果你不希望其他的开发者在使用你的按钮的时候，传一些不是你预设样式名字的type值进来，可以在代码中对type的取值进行一定的限制。

```js
MyButton.propTypes = {
  // 你可以声明 prop 是特定的值，类似于枚举
  type: PropTypes.oneOf(
    ['default', 'dark', 'middle', 'light']
  ),
};
```
这样一来，如果用户写了type=‘blue’这样超出你预设的设置出来，处于debug状态的手机或模拟器上会出现一条黄色的warning，提醒他使用你预设好的几种样式。
