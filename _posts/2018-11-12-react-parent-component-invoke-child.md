---
layout: post
title:  react中父组件调用子组件的方法
date:   2018-11-12 17:08:52 +0800
categories: React
tag: [React]
---

* content
{:toc}



### react中父组件调用子组件的方法
最近项目中用到了react，需要在父组件中调用子组件的某个方法，那么如何获取到子组件的实例呢？
这里使用了回调，简单实用，兼容低版本。
```
class Parent extends Component {

componentDidMount() {
...
}

forceRefresh() {
// 调用子组件的refresh()方法刷新。
this.xxxChildView.refresh();
}
...
render() {
return (
<ChildView ref={(ref) => this.xxxChildView = ref} >
...
</ChildView>
);
}
```
子组件：
```
class ChildView extends Component {

componentDidMount() {
...
}

// 对外提供的刷新方法
refresh() {
...
}
...
render() {
return (
...
);
}
```


#### 参考
1、[https://react-cn.github.io/react/docs/more-about-refs.html#](https://react-cn.github.io/react/docs/more-about-refs.html#)



