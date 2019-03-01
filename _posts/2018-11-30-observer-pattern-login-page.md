---
layout: post
title:  观察者模式实战——登录页面
date:   2018-11-30 15:31:52 +0800
categories: Design Pattern
tag: [Observer, 观察者模式]
---

* content
{:toc}




### 观察者模式实战——登录页面

Android开发中我们会遇到这样的需求，某个需要用户输入信息的页面，只有在用户输入了多条数据之后，下一步按钮才能可点击，用户才可以进行下一步操作。
如下图所示，用户只有在同时输入了手机号和密码之后，登录按钮才可以点击，否则登录按钮不可点击。

<img src="https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/174E7AA801E9C7CB806DC587BF75E3C3.jpg" width=256/> <img src="https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/B86D357A407E61E8B78A35BCE21B38FE.jpg" width=256/>

这种需求在开发中很常见，当且仅当多个条件同时被满足，某个步骤才会执行。

那么这个问题如何解决呢？第一眼看过去，这个是典型的观察者模式，那我们看下Java中观察者的相关实现。

Java中定义了两个类供我们使用，java.util.Observer和java.util.Observable，代码如下：
```
public interface Observer {
void update(Observable o, Object arg);
}
```
```
public class Observable {
private boolean changed = false;
private Vector<Observer> obs;

/** Construct an Observable with zero Observers. */

public Observable() {
obs = new Vector<>();
}

public synchronized void addObserver(Observer o) {
if (o == null)
throw new NullPointerException();
if (!obs.contains(o)) {
obs.addElement(o);
}
}

public synchronized void deleteObserver(Observer o) {
obs.removeElement(o);
}

public void notifyObservers() {
notifyObservers(null);
}

...
}
```

不过我们不能直接拿过来用，因为java.util.Observer和java.util.Observable中，观察者和被观察者是多对一的关系，一个Observable对应多个Observer，当Observable有变化时，会通知它持有的多个Observer。我们的需求是，多个被观察者对应一个观察者，当某个被观察者的数据变化时会通知观察者，此时观察者会检查所有被观察者的状态。这样，当所有被观察者的状态都满足条件时，观察者就会被通知到。

好了，接下来我的代码实现：
```
/**
* @Description: 被观察者，单个View的输入状态，true or false
* 参考 {@link java.util.Observable}
* @Author wangjianzhou@qding.me
* @Date 2018/11/29 2:06 PM
* @Version v4.4
*/
public class InputStatusObservable {
private boolean ready;
private InputStatusObserver observer;

public InputStatusObservable(boolean ready, InputStatusObserverImpl observer) {
this.ready = ready;
this.observer = observer;
}

public InputStatusObservable(InputStatusObserverImpl observer) {
this.observer = observer;
}

public boolean isReady() {
return ready;
}

public void setReady(boolean ready) {
this.ready = ready;
observer.update();
}
}
```
观察者接口：
```
/**
* @Description: 观察者
* 参考 {@link java.util.Observer}
* @Author wangjianzhou@qding.me
* @Date 2018/11/29 2:06 PM
* @Version v4.4
*/
public interface InputStatusObserver {
void update();
}
```
观察者实现：
```
/**
* @Description: 观察者实现，监听多个InputStatusObservable的状态。
*
* @Author wangjianzhou@qding.me
* @Date 2018/11/29 2:06 PM
* @Version v4.4
*/
public class InputStatusObserverImpl implements InputStatusObserver {
private ArrayList<InputStatusObservable> observableList = new ArrayList<>();
private OnObservablesReadyListener onObservablesReadyListener;

public InputStatusObserverImpl(OnObservablesReadyListener onObservablesReadyListener) {
this.onObservablesReadyListener = onObservablesReadyListener;
}

public void add(InputStatusObservable observable) {
observableList.add(observable);
}

public void remove(InputStatusObservable observable) {
observableList.remove(observable);
}

/**
* 所有InputStatusObservable的状态是否都已经Ok了
*
* @return
*/
public boolean isObservablesReady() {
for (InputStatusObservable observable : observableList) {
if (!observable.isReady()) {
return false;
}
}
return true;
}

@Override
public void update() {
if (onObservablesReadyListener != null) {
onObservablesReadyListener.ready(isObservablesReady());
}
}

public interface OnObservablesReadyListener {
void ready(boolean isReady);
}
}
```

下面介绍如何使用：
1、定义观察者和被观察者，并对其进行绑定
```
/**
* 登录按钮是否可点击，依赖于是否输入了手机号码和密码
*/
InputStatusObserverImpl.OnObservablesReadyListener bindReadyListener = new InputStatusObserverImpl.OnObservablesReadyListener() {
@Override
public void ready(boolean isReady) {
// isReady如果为true，表示所有被观察者的状态均为true。
finishBtn.setEnabled(isReady);
finishBtn.setOnClickListener(!isReady ? null : new View.OnClickListener() {
@Override
public void onClick(View v) {
login();
}
});
}
};
// 定义观察者
InputStatusObserverImpl observer = new InputStatusObserverImpl(bindReadyListener);
//定义两个被观察者, 并将它们与观察者绑定。
observablePhoneNum = new InputStatusObservable(observer);
observer.add(observablePhoneNum);
observablePwd = new InputStatusObservable(observer);
observer.add(observablePwd);
```
2、给EditText添加监听器。
```
/**
* 给手机号码、密码输入框添加监听器。当输入内容变化时，就更新被观察者的状态。
*/
private void setEditTextListener() {
// 给手机输入框添加监听
compPhoneEt.addTextChangedListener(new TextWatcher() {
@Override
public void beforeTextChanged(CharSequence s, int start, int count, int after) {

}

@Override
public void onTextChanged(CharSequence s, int start, int before, int count) {

}

@Override
public void afterTextChanged(Editable s) {
observablePhoneNum.setReady(s.length() > 0);
}
});
// 给密码输入框添加监听
compPwdEt.addTextChangedListener(new TextWatcher() {
@Override
public void beforeTextChanged(CharSequence s, int start, int count, int after) {

}

@Override
public void onTextChanged(CharSequence s, int start, int before, int count) {

}

@Override
public void afterTextChanged(Editable s) {
observablePwd.setReady(s.length() > 0);
}
});
}
```

当然了，这个用rxJava的操作符也可以很容易的实现，这里就不废话了。

[项目地址](https://github.com/tinyvampirepudge/Android_Base_Demo)，进入搜索ObserverLoginActivity即可。

