---
title: 关于防抖动(Debounce)和节流(Throttle)
date: 2018-01-08 23:06:26
categories:
  - Javascript
tags: 
  - Javascript
---

## 了解背景

某项目中，需要实现一个及时搜索框，当用户输入关键字即去搜索相关数据。项目框架为React，使用onChange来监听input的输入事件。初步完成之后发现了问题，用户输入一个中文，会多次调用change事件，因每一次拼音的输入都会触发change事件。随即我决定采用延迟调用的方式去处理，沿着这个思路，发现了[throttle-debounce](https://github.com/niksy/throttle-debounce)，同时对debounce有了初步了解。

## 什么是Debounce

百度百科：按键去抖动，机械按键在按下时，并非按下就接触的很好，尤其是有弹簧片的机械开关，会在接触的瞬间反复开合多次。为了消除这种情况，会在断开闭合后执行一个延时程序，5ms～10ms的延时，让前沿抖动消失后再一次检测键的状态，如果仍保持相同状态电平，则确认为真正有键按下。

```
Creates a debounced function that delays invoking `func` until after `wait`
milliseconds have elapsed since the last time the debounced function was
invoked, or until the next browser frame is drawn.
```

以上来源于: [lodash/debounce.js](https://github.com/lodash/lodash/blob/master/debounce.js)

个人理解：当函数调用后，指定时间内没有再次被调用，则执行。若在指定时间内再次调用，则重新计算时间。

在某些情况下，由于事件被频繁调用(例如：mousemove, keyup)，会造成很多重复性的动作，这些动作可能会造成占用内存，重复调用等资源浪费行为。

这个时候，debounce的作用就是限制这些事件以一定的频率调用，例如限制搜索框的change事件500ms调用一次，这样我们可以避免多次的服务器请求。

## 什么是Throttle

```
Creates a throttled function that only invokes `func` at most once per 
every `wait` milliseconds (or once per browser frame).
```

以上来源于: [lodash/throttle.js](https://github.com/lodash/lodash/blob/master/throttle.js)

个人理解： 函数在预定义的执行周期内，最多执行一次

例如onscroll事件，在一个无限滚动的内容区域内，我们必须检测滚动条距离底部的位置，如果使用debounce则会造成用户滚动底部停止动作了才去请求数据的情况。

而使用throttle则可以保障我们不断的去检查滚动条的位置，及时获取数据。

## 如何使用

debounce和throttle的轮子已经非常多了，没有必要自己去实现一套。这里推荐一种方式，使用Lodash的自定义库，生成的代码仅2kb。

```
npm install -g lodash-cli
lodash include=debounce,throttle
```

## 结论

debounce和throttle都可以帮助我们延时执行函数，但是具体该用哪一个就需要按照自己的实际需求去选择了。

