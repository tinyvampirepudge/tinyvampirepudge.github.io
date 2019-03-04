---
layout: post
title:  超简单的自定义ImageView，支持圆角和直角
date:   2018-01-25 15:19:25 +0800
categories: 自定义View
tag: [ImageView]
---

* content
{:toc}


## 超简单的自定义ImageView，支持圆角和直角
### 1、需求：ImageView显示的图片，上方的两个角是圆角，下方的两个角是直角。
![需求图](http://img.blog.csdn.net/20180125151146126?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 2、这篇文章推荐了三种方式，我选择第三种ClipPath方式，这种方式很精简。
参考：https://www.jianshu.com/p/626dbd93207d

### 3、先来总结下自定义ImageView需要实现的功能：
①四个角的度数均要支持自定义

②可以在xml布局当中添加自定义的度数

### 4、需求明确了，接下来就是实现了
①先自定义一个ImageView，继承自AppCompatImageView，实现它的构造方法；创建一个init()方法，保证构造方法里面都会调用。ps：这里没有继承自ImageView，是因为官方推荐使用AppCompatImageView。

②在onLayout方法里面获取到ImageView的width和height

③在onDraw()方法里面对Carvas进行裁剪。ps: 这里裁剪的弧度是12px.


```javascript
public class CustomRoundAngleImageView extends AppCompatImageView {
    float width, height;

    public CustomRoundAngleImageView(Context context) {
        this(context, null);
        init(context, null);
    }

    public CustomRoundAngleImageView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
        init(context, attrs);
    }

    public CustomRoundAngleImageView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context, attrs);
    }

    private void init(Context context, AttributeSet attrs) {
        if (Build.VERSION.SDK_INT < 18) {
            setLayerType(View.LAYER_TYPE_SOFTWARE, null);
        }
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
        width = getWidth();
        height = getHeight();
    }

    @Override
    protected void onDraw(Canvas canvas) {
        if (width >= 12 && height > 12) {
            Path path = new Path();
            //四个圆角
            path.moveTo(12, 0);
            path.lineTo(width - 12, 0);
            path.quadTo(width, 0, width, 12);
            path.lineTo(width, height - 12);
            path.quadTo(width, height, width - 12, height);
            path.lineTo(12, height);
            path.quadTo(0, height, 0, height - 12);
            path.lineTo(0, 12);
            path.quadTo(0, 0, 12, 0);

            canvas.clipPath(path);
        }
        super.onDraw(canvas);
    }

}
```

测试下效果：

```javascript
<com.xxx.view.CustomRoundAngleImageView
    android:id="@+id/iv_avatar"
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:layout_marginBottom="50dp"
    android:layout_marginTop="50dp"
    android:scaleType="centerCrop"
    tools:src="@mipmap/ic_launcher"/>
```


```javascript
String avatarUrl = "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1516644385815&di=c0552674db9f07a5f889d7c0980e33db&imgtype=0&src=http%3A%2F%2Fimg.mp.itc.cn%2Fupload%2F20170529%2F83d3ce719e9d4c0a8f1cd033ecac3692_th.jpg”;
Glide.with(this).load(avatarUrl).into(ivAvatar);
```

效果如下所示：
![圆角度数为12px的ImageView](http://img.blog.csdn.net/20180125151212028?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

④接下来，我们需要让四个角的度数支持自定义，并且支持在xml中设置
先将四个角度对应的自定义样式定义了：下方定义的值依次对应的是默认的度数、左上、右上、右下、左下。
如果用户没有设置左上、右上、右下、左下的值，那么四个角的值就去radius的。radius的值默认为0，也就是直角。
attrs.xml中：

```
<declare-styleable name="Custom_Round_Image_View">
    <attr name="radius" format="dimension"/>
    <attr name="left_top_radius" format="dimension"/>
    <attr name="right_top_radius" format="dimension"/>
    <attr name="right_bottom_radius" format="dimension"/>
    <attr name="left_bottom_radius" format="dimension"/>
</declare-styleable>
```

获取到用户定义在xml里面的数据，只需要修改刚才定义的init()方法即可

```
private int defaultRadius = 0;
private int radius;
private int leftTopRadius;
private int rightTopRadius;
private int rightBottomRadius;
private int leftBottomRadius;

private void init(Context context, AttributeSet attrs) {
    if (Build.VERSION.SDK_INT < 18) {
        setLayerType(View.LAYER_TYPE_SOFTWARE, null);
    }
    // 读取配置
    TypedArray array = context.obtainStyledAttributes(attrs, R.styleable.Custom_Round_Image_View);
    radius = array.getDimensionPixelOffset(R.styleable.Custom_Round_Image_View_radius, defaultRadius);
    leftTopRadius = array.getDimensionPixelOffset(R.styleable.Custom_Round_Image_View_left_top_radius, defaultRadius);
    rightTopRadius = array.getDimensionPixelOffset(R.styleable.Custom_Round_Image_View_right_top_radius, defaultRadius);
    rightBottomRadius = array.getDimensionPixelOffset(R.styleable.Custom_Round_Image_View_right_bottom_radius, defaultRadius);
    leftBottomRadius = array.getDimensionPixelOffset(R.styleable.Custom_Round_Image_View_left_bottom_radius, defaultRadius);

    LogUtils.e("radius --> " + radius);

    //如果四个角的值没有设置，那么就使用通用的radius的值。
    if (defaultRadius == leftTopRadius) {
        leftTopRadius = radius;
    }
    if (defaultRadius == rightTopRadius) {
        rightTopRadius = radius;
    }
    if (defaultRadius == rightBottomRadius) {
        rightBottomRadius = radius;
    }
    if (defaultRadius == leftBottomRadius) {
        leftBottomRadius = radius;
    }
    array.recycle();
}
```

上边这段代码较清晰，就是获取到用户设置的四个角上的值，如果没有，就使用radius的值代替。
接下来就是如何使用获取到的数据了。
在onDraw()方法中：


```
@Override
    protected void onDraw(Canvas canvas) {
        //这里做下判断，只有图片的宽高大于设置的圆角距离的时候才进行裁剪
        int maxLeft = Math.max(leftTopRadius, leftBottomRadius);
        int maxRight = Math.max(rightTopRadius, rightBottomRadius);
        int minWidth = maxLeft + maxRight;
        int maxTop = Math.max(leftTopRadius, rightTopRadius);
        int maxBottom = Math.max(leftBottomRadius, rightBottomRadius);
        int minHeight = maxTop + maxBottom;
        if (width >= minWidth && height > minHeight) {
            Path path = new Path();
            //四个角：右上，右下，左下，左上
            path.moveTo(leftTopRadius, 0);
            path.lineTo(width - rightTopRadius, 0);
            path.quadTo(width, 0, width, rightTopRadius);

            path.lineTo(width, height - rightBottomRadius);
            path.quadTo(width, height, width - rightBottomRadius, height);

            path.lineTo(leftBottomRadius, height);
            path.quadTo(0, height, 0, height - leftBottomRadius);

            path.lineTo(0, leftTopRadius);
            path.quadTo(0, 0, leftTopRadius, 0);

            canvas.clipPath(path);
        }
        super.onDraw(canvas);
    }
```

需要说明的是，在onDraw()方法中进行了宽度和高度判断，判断的依据是用户设置的角度值的和是小于等于ImageView的宽度或高度的，这样就可以避免很奇怪的体验。

###5、代码封装完了，接下来是测试效果了。

```
<com.xxx.view.CustomRoundAngleImageView
    android:id="@+id/iv_avatar6"
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:layout_marginTop="50dp"
    android:scaleType="centerCrop"
    tools:src="@mipmap/ic_launcher"/>

<com.xxx.view.CustomRoundAngleImageView
    android:id="@+id/iv_avatar7"
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:layout_marginTop="50dp"
    android:scaleType="centerCrop"
    roundiv:radius="10dp"
    tools:src="@mipmap/ic_launcher"/>

<com.xxx.view.CustomRoundAngleImageView
    android:id="@+id/iv_avatar8"
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:layout_marginTop="50dp"
    android:scaleType="centerCrop"
    roundiv:left_bottom_radius="10dp"
    roundiv:left_top_radius="10dp"
    roundiv:right_bottom_radius="10dp"
    roundiv:right_top_radius="10dp"
    tools:src="@mipmap/ic_launcher"/>

<com.xxx.view.CustomRoundAngleImageView
    android:id="@+id/iv_avatar9"
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:layout_marginTop="50dp"
    android:scaleType="centerCrop"
    roundiv:radius="20dp"
    roundiv:right_bottom_radius="10dp"
    roundiv:right_top_radius="10dp"
    tools:src="@mipmap/ic_launcher"/>

<com.xxx.view.CustomRoundAngleImageView
    android:id="@+id/iv_avatar10"
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:layout_marginBottom="10dp"
    android:layout_marginTop="50dp"
    android:scaleType="centerCrop"
    roundiv:left_top_radius="10dp"
    roundiv:right_top_radius="10dp"
    tools:src="@mipmap/ic_launcher"/>
```

```
String avatarUrl = "https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1516644385815&di=c0552674db9f07a5f889d7c0980e33db&imgtype=0&src=http%3A%2F%2Fimg.mp.itc.cn%2Fupload%2F20170529%2F83d3ce719e9d4c0a8f1cd033ecac3692_th.jpg";
        Glide.with(this).load(avatarUrl).into(ivAvatar6);
        Glide.with(this).load(avatarUrl).into(ivAvatar7);
        Glide.with(this).load(avatarUrl).into(ivAvatar8);
        Glide.with(this).load(avatarUrl).into(ivAvatar9);
        Glide.with(this).load(avatarUrl).into(ivAvatar10);
```
效果如下：
![效果图1-4](http://img.blog.csdn.net/20180125150804354?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![效果图5](http://img.blog.csdn.net/20180125150815574?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjYyODc0MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
从上往下看，第一个没设置任何有关角度的属性，所以就是直角；
第二个设置了radius=10dp，并且没有设置四个角单独的度数，所以四个角的度数都是radius的度数，就是图中的四个圆角
第三个单独设置了四个角的度数，均为10dp，跟单独设置radius=10dp的效果一样。
第四个设置了radius=20dp，并且还单独设置了右上、右下的度数为10dp，左上、左下的度数没有单独设置，会取radius的数值。
第五个这是了左上和右上的度数为10dp，没有设置radius的值，因此默认radius的值为0，显示就是直角。如图上面两个角是圆角，下方两个角是直角。

###最后，再附上一个使用说明：

```
/**
 * Desc:    可以自己设置角度的ImageView
 * 参考：https://www.jianshu.com/p/626dbd93207d
 * <com.xxx.xxx.common.view.CustomRoundAngleImageView
        android:id="@+id/iv_avatar"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_marginTop="50dp"
        android:scaleType="centerCrop"
        roundiv:left_bottom_radius="10dp"
        roundiv:radius="10dp"
        roundiv:left_top_radius="10dp"
        roundiv:right_bottom_radius="10dp"
        roundiv:right_top_radius="10dp"
        tools:src="@mipmap/ic_launcher"/>
 * 所有角度默认值均为0，即直角。
 * 左上、右上、右下、左下四个角的度数的值分别对应的是
 * 左上：left_top_radius
 * 右上：right_top_radius
 * 右下：right_bottom_radius
 * 左下：left_bottom_radius
 * 如果四个角度的度数一致，推荐使用radius，在不使用上述四个参数的时候，它的值会作为默认    值。
 * tips: 使用时尽量保证ImageView的宽高大于设置的圆角度数，避免不必要的错误。
 */
```

