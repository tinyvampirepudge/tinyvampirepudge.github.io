---
layout: post
title:  Android在子线程中更新UI的方法汇总(共七种)
date:   2017-03-23 20:50:58 +0800
categories: Android
tag: [子线程更新UI]
---

* content
{:toc}



### Android在子线程中更新UI的方法汇总(共七种)
#### 1、常规写法：new Handler()的handleMessage()和handler.sendMessage(msg)

```
    Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
        }
    };
```

```
        new Thread(new Runnable() {
            @Override
            public void run() {
                Message msg = Message.obtain();
                msg.what = 1000;
                msg.arg1 = 10;
                handler.sendMessage(msg);
            }
        }).start();
```

#### 2、handler的另一种用法：

```
    private Handler.Callback callback = new Handler.Callback() {
        @Override
        public boolean handleMessage(Message msg) {
            return true;
        }
    };
    
	Handler handler1 = new Handler(callback);
```

```
        new Thread(new Runnable() {
            @Override
            public void run() {
                Message msg = Message.obtain();
                msg.what = 1001;
                msg.arg1 = 11;
                handler1.sendMessage(msg);
            }
        }).start();
```

#### 3、handler.post(runnable)
```
    Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
        }
    };
```

```
        new Thread(new Runnable() {
            @Override
            public void run() {
                handler.post(new Runnable() {
                    @Override
                    public void run() {

                    }
                });
            }
        }).start();
```

#### 4、handler.postDelayed(runnable, milliseconds)
```
    Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
        }
    };
```

```
        new Thread(new Runnable() {
            @Override
            public void run() {
                handler.postDelayed(new Runnable() {
                    @Override
                    public void run() {

                    }
                }, 3000);
            }
        }).start();
```

#### 5、activity.runOnUiThread(runnable)

```
        new Thread(new Runnable() {
            @Override
            public void run() {
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {

                    }
                });
            }
        }).start();
```

#### 6、view.post(runnable)

```
        new Thread(new Runnable() {
            @Override
            public void run() {
                button.post(new Runnable() {
                    @Override
                    public void run() {

                    }
                });
            }
        }).start();
```

#### 7、view.postDelayed(runnable, milliseconds)
```
        new Thread(new Runnable() {
            @Override
            public void run() {
                button.postDelayed(new Runnable() {
                    @Override
                    public void run() {

                    }
                }, 3000);
            }
        }).start();
```

