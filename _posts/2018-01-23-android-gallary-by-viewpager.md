---
layout: post
title:  手把手带你用viewpager实现gallary效果，外加无限循环，自动轮播
date:   2018-01-23 18:31:59 +0800
categories: demo
tag: [demo]
---

* content
{:toc}



## 手把手带你用viewpager实现gallary效果，外加无限循环，自动轮播
效果图：图很丑，各位看官且按需更改。

<img src="http://img.blog.csdn.net/20180123181419412?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast"  height="600" alt="效果图"/>

主要功能：

①Gallary样式

②无限轮播

③自动轮播和手势操作间冲突解决

提前说明：可以先不看提供的连接，文章末尾会提供整体代码。

### 1、大致思路：
经过查阅网上资料，个人感觉viewPager实现最为靠谱，省时省力。下面就将上述的需求一个个解决：

①Gallary样式：使用ViewPager.PageTransformer解决滑动动画。

参照：

[https://developer.android.com/training/animation/screen-slide.html](https://developer.android.com/training/animation/screen-slide.html)

[https://www.jianshu.com/p/722ece163629](https://www.jianshu.com/p/722ece163629)

②无限轮播：在PagerAdapter里面进行操作，让getCount()方法返回的值足够大。

参照：
方法二
[http://blog.csdn.net/Just_Sanpark/article/details/17436037](http://blog.csdn.net/Just_Sanpark/article/details/17436037)

③自动轮播和收拾操作间冲突解决：

自动轮播使用handler发送延时消息；在viewpager的dispatchTouchEvent方法里面来进行ViewPager接收到Touch事件的操作。

### 2、接下来下面是代码实现了，先创建个普通的ViewPager：
①activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.tongtong.tiny.loopviewpagergallary.MainActivity">

    <!-- 这里的ViewPager最好包一层父布局，并给它设置android:clipChildren="false"属性-->
    <FrameLayout
        android:id="@+id/fl_vp_parent"
        android:layout_width="match_parent"
        android:layout_height="160dp"
        android:layout_centerInParent="true"
        android:background="#aadc71ff"
        android:clipChildren="false"
        >

        <!-- 宽度不要占满屏幕
        设置android:clipChildren="false"属性 -->
        <android.support.v4.view.ViewPager
            android:id="@+id/id_viewpager"
            android:layout_width="250dp"
            android:layout_height="120dp"
            android:layout_gravity="center"
            android:clipChildren="false"
            >
        </android.support.v4.view.ViewPager>

    </FrameLayout>

</android.support.constraint.ConstraintLayout>
```
ViewPager：

```
public class PageTransformerAdapter extends PagerAdapter {
    private Context context;
    private List<TextView> list;

    public PageTransformerAdapter(Context context, List<TextView> list) {
        this.context = context;
        this.list = list;
    }

    @Override
    public int getCount() {
        return list.size();
    }

    @Override
    public boolean isViewFromObject(View view, Object object) {
        return view == object;
    }

    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        container.addView(list.get(position));
        return list.get(position);
    }

    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        TextView iv = (TextView) object;
        container.removeView(iv);
    }
}
```
MainActivity代码：

```
public class MainActivity extends AppCompatActivity {
    public static final String TAG = MainActivity.class.getSimpleName();
    //测试数据个数
    public static final int LIST_SIZE = 10;

    FrameLayout flVpParent;
    ViewPager mViewPager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        flVpParent = findViewById(R.id.fl_vp_parent);
        mViewPager = findViewById(R.id.id_viewpager);

        List<TextView> viewList = getViewLists();
        PageTransformerAdapter mAdapter = new PageTransformerAdapter(MainActivity.this, viewList);

        mViewPager.setAdapter(mAdapter);
    }

    /**
     * 生成List<TextView>数据
     *
     * @return
     */
    private List<TextView> getViewLists() {
        List<TextView> list = new ArrayList<>();
        for (int j = 0; j < LIST_SIZE; j++) {
            TextView tv = generateTextView(j);
            list.add(tv);
        }
        return list;
    }

    /**
     * 生成单个的TextView对象
     *
     * @param j
     * @return
     */
    private TextView generateTextView(final int j) {
        TextView tv = new TextView(this);
        tv.setBackgroundColor(Color.parseColor("#2371e9"));
        String str = "这个是第" + (j + 1) + "个View";
        tv.setTag(str);
        tv.setText(str);
        tv.setTextColor(Color.parseColor("#f0523c"));
        tv.setTextSize(30);
        tv.setGravity(Gravity.CENTER);
        tv.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.e(TAG, "第" + (j + 1) + "个View被点击了");
            }
        });
        return tv;
    }
}
```
效果如下图：

<img src="http://img.blog.csdn.net/20180123181944489?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast"  height="600" alt="效果平淡无奇，甚至还有点丑。"/>

好了，架子搭建好了，可以开始一步步实现需求了。

### 3、Gallary视图效果：利用ViewPager.PageTransformer实现
①viewpager的布局文件设置之前已经设置好了，这里直接使用

②新建ScaleDownPageTransformer，实现ViewPager.PageTransformer接口

```
/**
 * Desc:    左右缩放的PageTransformer
 * Created by tiny on 2018/1/19.
 * Time: 17:04
 * Version:
 */

public class ScaleDownPageTransformer implements ViewPager.PageTransformer {
    private static final float DEFAULT_MIN_SCALE = 0.9f;
    private float mMinScale = DEFAULT_MIN_SCALE;

    /**
     * Apply a property transformation to the given page.
     *
     * @param view     Apply the transformation to this page
     * @param position Position of page relative to the current front-and-center
     *                 position of the pager. 0 is front and center. 1 is one full
     *                 page position to the right, and -1 is one page position to the left.
     */
    @Override
    public void transformPage(View view, float position) {
        float limitLow = -1;
        float limitMid = 0;
        float limitHigh = 1;

        if (position < limitLow) {
            view.setScaleX(mMinScale);
            view.setScaleY(mMinScale);
        } else if (position <= limitHigh) { // [-1,1]

            if (position < limitMid) //[-1,0)
            {
                float factor = mMinScale + (1 - mMinScale) * (1 + position);
                view.setScaleX(factor);
                view.setScaleY(factor);
            } else//[0,1]
            {
                float factor = mMinScale + (1 - mMinScale) * (1 - position);
                view.setScaleX(factor);
                view.setScaleY(factor);
            }
        } else { // (1,+Infinity]
            view.setScaleX(mMinScale);
            view.setScaleY(mMinScale);
        }
    }
}
```
③将ScaleDownPageTransformer设置给viewPager对象。

```
//设置Page间间距
mViewPager.setPageMargin(20);
//设置缓存的页面数量
//这个如果设置为1，或者使用默认值1时，在向左滑动时，右面新加载出的view第一次加载时的缩放动画不显示。。
mViewPager.setOffscreenPageLimit(2);
mViewPager.setPageTransformer(true, new ScaleDownPageTransformer());
mViewPager.setAdapter(mAdapter);
```
参考：

[官网demo](https://developer.android.com/training/animation/screen-slide.html)

[https://www.jianshu.com/p/722ece163629](https://www.jianshu.com/p/722ece163629)

效果如下：

<img src="http://img.blog.csdn.net/20180123182134676?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast"  height="600" alt="有点模样了"/>

### 4、无限循环
接下来看下无线循环的需求，具体细节和原理的说明网上有好多，我选择的是这篇文章的第二种方法。

[传送门](http://blog.csdn.net/Just_Sanpark/article/details/17436037)

下面直接上代码：
PagerAdapter做了些修改：

```
public class PageTransformerAdapter extends PagerAdapter {
    private Context context;
    private List<TextView> list;

    public PageTransformerAdapter(Context context, List<TextView> list) {
        this.context = context;
        this.list = list;
    }

    @Override
    public int getCount() {
        return Integer.MAX_VALUE;
    }

    @Override
    public boolean isViewFromObject(View view, Object object) {
        return view == object;
    }

    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        int actualPos = position % list.size();
        if (list.get(actualPos).getParent() != null) {
            ((ViewPager) list.get(actualPos).getParent()).removeView(list.get(actualPos));
        }
        container.addView(list.get(actualPos));
        return list.get(actualPos);
    }

    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        TextView iv = (TextView) object;
        container.removeView(iv);
    }
}
```
MainActivity中增加了代码：

```
//这儿的1000，可以随便改，只要用户滑不出边界即可
int startPos = 1000 * viewList.size() + 1;
mViewPager.setCurrentItem(startPos);
```
效果如下图：

<img src="http://img.blog.csdn.net/20180123182248297?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast"  height="600" alt="来，滑动两下试试，看看是不是无限循环"/>

### 5、自动轮播，同时支持手势滑动
①自动轮播的功能决定使用handler发送消息实现

②当用户对轮播进行操作时，需要停止自动轮播的消息，这里决定在ViewPager的dispatchTouchEvent方法里面操作。

ps: 之所以不在onTouchEvent里面进行操作，是因为ViewPager对分发下来的事件进行了处理，在onTouchEvent中拦截不到ACTION_DOWN事件，长按事件也拦截不到。

代码如下：
①自定义LoopViewPager，继承自ViewPager，

```
/**
 * Desc:    自动切换的ViewPager，手势冲突已处理
 * Created by tiny on 2018/1/23.
 * Time: 14:38
 * Version:
 */

public class LoopViewPager extends ViewPager {
    public static final String TAG = LoopViewPager.class.getSimpleName();

    public LoopViewPager(Context context) {
        super(context);
        init();
    }

    public LoopViewPager(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    private void init() {
        autoPollTask = new AutoPollTask(this);
    }

    private static final long TIME_AUTO_POLL = 2000;
    private static AutoPollTask autoPollTask;
    private static boolean running; //标示是否正在自动轮询
    private static boolean canRun;//标示是否可以自动轮询,可在不需要的是否置false

    static class AutoPollTask implements Runnable {
        private final WeakReference<ViewPager> mReference;

        //使用弱引用持有外部类引用->防止内存泄漏
        public AutoPollTask(ViewPager reference) {
            this.mReference = new WeakReference<>(reference);
        }

        @Override
        public void run() {
            ViewPager viewPager = mReference.get();
            if (viewPager != null && running && canRun) {
                int currPos = viewPager.getCurrentItem();
                viewPager.setCurrentItem(currPos + 1, true);
                viewPager.postDelayed(autoPollTask, TIME_AUTO_POLL);
            }
        }
    }

    //开启:如果正在运行,先停止->再开启
    public void start() {
        if (running)
            stop();
        canRun = true;
        running = true;
        postDelayed(autoPollTask, TIME_AUTO_POLL);
    }

    public void stop() {
        running = false;
        removeCallbacks(autoPollTask);
    }

    /**
     * onTouchEvent中的事件被viewPager处理过了，有些拦截不到
     * 所以在这里拦截
     *
     * @param ev
     * @return
     */
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        //只要接收到事件，就停止滚动。
        if (running)
            stop();
        switch (ev.getAction()) {
            //当手指抬起或者划出页面时
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_OUTSIDE:
                if (canRun)
                    start();
                break;
        }
        return super.dispatchTouchEvent(ev);
    }
}
```
②接下来将activity_main.xml中的ViewPager替换为LoopViewPager，并在MainActivity中添加如下代码：

```
mViewPager.start();
```
最后，为了节省资源，在MainActivity中加上下面代码。

```
@Override
protected void onStart() {
    super.onStart();
    if (mViewPager != null) {
        mViewPager.start();
    }
}

@Override
protected void onStop() {
    super.onStop();
    if (mViewPager != null) {
        mViewPager.stop();
    }
}
```
最后附上福利：源码地址，求star。
[https://github.com/tinyvampirepudge/LoopViewPagerGallary](https://github.com/tinyvampirepudge/LoopViewPagerGallary)
