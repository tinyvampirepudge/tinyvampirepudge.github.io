---
layout: post
title:  RecyclerView常用方法总结
date:   2019-08-08 10:29:52 +0800
categories: RecyclerView
tag: [RecyclerView]
---

* content
{:toc}



### RecyclerView常用方法总结

1、获取recyclerView内容高度

```
// 获取recyclerView内容高度
int recyclerViewRealHeight = recyclerView.computeVerticalScrollRange();
```

我们通过recyclerView.getHeight方法获取到的高度是RecyclerView控件的高度，不是内容高度

2、获取adapter中的item总个数

```
int size = recyclerView.getAdapter().getItemCount();
```

3、获取recyclerView可见的item数量

```
int childCount = recyclerView.getChildCount();
```

4、获取某个Item的实际position

```
RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child.getLayoutParams();
//获取其在adapter中的位置
int position = params.getViewLayoutPosition();
```

```
// 这个方式也可以
int position = recyclerView.getChildAdapterPosition(view);
```

5、根据position获取对应的Item的View，需要注意的是，如果当前position对应的View不可见，获取到的View为null。

```
// llm为对应的LayoutManager
View itemView = llm.findViewByPosition(position);
```

6、获取第一个可见的Item的position

```
int firstPosition = ((LinearLayoutManager)recyclerView.getLayoutManager()).findFirstVisibleItemPosition();
```

```
// 这样也可以获取到
View childFirst = recyclerView.getChildAt(0);
RecyclerView.LayoutParams paramsFirst = (RecyclerView.LayoutParams) childFirst.getLayoutParams();
int firstPosition1 = paramsFirst.getViewLayoutPosition();
```

7、获取第一个完全可见的Item的position

```
int firstCompletelyVisibleItemPosition = ((LinearLayoutManager)recyclerView.getLayoutManager()).findFirstCompletelyVisibleItemPosition();
```


8、获取最后一个可见的Item的position

```
int lastPosition = ((LinearLayoutManager)recyclerView.getLayoutManager()).findLastVisibleItemPosition();
```

```
// 这样也可以获取到
int childCount = recyclerView.getChildCount();
View childLast = recyclerView.getChildAt(childCount - 1);
RecyclerView.LayoutParams paramsLast = (RecyclerView.LayoutParams) childLast.getLayoutParams();
int lastPosition1 = paramsLast.getViewLayoutPosition();
```

9、获取最后一个完全可见的Item的position

```
int lastCompletelyVisibleItemPosition = ((LinearLayoutManager)parent.getLayoutManager()).findLastCompletelyVisibleItemPosition();
```

