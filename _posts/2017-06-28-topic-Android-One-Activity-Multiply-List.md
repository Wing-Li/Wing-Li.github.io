---
title: 单个Acticity显示多个列表
excerpt: 仿内涵段子详情页的热门评论、全部评论
categories:
  - topics
tag: Android  
---

> 如果本文帮助到你，本人不胜荣幸，如果浪费了你的时间，本人深感抱歉。
希望用最简单的大白话来帮助那些像我一样的人。如果有什么错误，请一定指出，以免误导大家、也误导我。
本文来自：http://www.jianshu.com/u/320f9e8f7fc9
感谢您的关注。


首先看实现效果图：

![](http://upload-images.jianshu.io/upload_images/1689895-f65e25587da79ff6.gif?imageMogr2/auto-orient/strip)

类似的这种需求在实际的项目中还是挺多的。 <br />
说说的详情页，顶部显示内容，然后显示一个热门评论，最后显示全部评论。 <br />
两个评论列表数量都是动态的，并且全部评论还可以下拉刷新。 <br />

先放项目地址：https://github.com/Wing-Li/DoubleList <br />
建议把项目下载下来看看，项目非常简单，此文主要是流程。

我们来实现这个效果。
<hr />
##### 1. 布局
最关键的是 NestedScrollView 控件，看这张图

![](http://upload-images.jianshu.io/upload_images/1689895-94faf57e724fb2e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


    NestedScrollView  // 滚动页
     - LinearLayout  // NestedScrollView 只能包含一个 LinearLayout
       - LinearLayout // 说说详情
       - LinearLayout // 热门评论
         - RecyclerView
       - LinearLayout // 全部评论
         - RecyclerView

##### 2. 重写 LinearLayoutManager
初始化 RecyclerView 时需要设置 setLayoutManager(),我们需要重写它来计算列表的高度。<br />
代码如下：

    package com.lyl.doublelist;

    import android.content.Context;
    import android.support.v7.widget.LinearLayoutManager;
    import android.support.v7.widget.RecyclerView;
    import android.view.View;
    import android.view.ViewGroup;

    /**
     * Created by lyl on 2017/6/6.
     */
    public class WrappingLinearLayoutManager extends LinearLayoutManager
    {

        public WrappingLinearLayoutManager(Context context) {
            super(context);
        }

        private int[] mMeasuredDimension = new int[2];

        @Override
        public boolean canScrollVertically() {
            return false;
        }

        @Override
        public void onMeasure(RecyclerView.Recycler recycler, RecyclerView.State state,
                              int widthSpec, int heightSpec) {
            final int widthMode = View.MeasureSpec.getMode(widthSpec);
            final int heightMode = View.MeasureSpec.getMode(heightSpec);

            final int widthSize = View.MeasureSpec.getSize(widthSpec);
            final int heightSize = View.MeasureSpec.getSize(heightSpec);

            int width = 0;
            int height = 0;
            for (int i = 0; i < getItemCount(); i++) {
                if (getOrientation() == HORIZONTAL) {
                    measureScrapChild(recycler, i,
                            View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED),
                            heightSpec,
                            mMeasuredDimension);

                    width = width + mMeasuredDimension[0];
                    if (i == 0) {
                        height = mMeasuredDimension[1];
                    }
                } else {
                    measureScrapChild(recycler, i,
                            widthSpec,
                            View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED),
                            mMeasuredDimension);

                    height = height + mMeasuredDimension[1];
                    if (i == 0) {
                        width = mMeasuredDimension[0];
                    }
                }
            }

            switch (widthMode) {
                case View.MeasureSpec.EXACTLY:
                    width = widthSize;
                case View.MeasureSpec.AT_MOST:
                case View.MeasureSpec.UNSPECIFIED:
            }

            switch (heightMode) {
                case View.MeasureSpec.EXACTLY:
                    height = heightSize;
                case View.MeasureSpec.AT_MOST:
                case View.MeasureSpec.UNSPECIFIED:
            }

            setMeasuredDimension(width, height);
        }

        private void measureScrapChild(RecyclerView.Recycler recycler, int position, int widthSpec,
                                       int heightSpec, int[] measuredDimension) {

            View view = recycler.getViewForPosition(position);
            if (view.getVisibility() == View.GONE) {
                measuredDimension[0] = 0;
                measuredDimension[1] = 0;
                return;
            }
            // For adding Item Decor Insets to view
            super.measureChildWithMargins(view, 0, 0);
            RecyclerView.LayoutParams p = (RecyclerView.LayoutParams) view.getLayoutParams();
            int childWidthSpec = ViewGroup.getChildMeasureSpec(
                    widthSpec,
                    getPaddingLeft() + getPaddingRight() + getDecoratedLeft(view) + getDecoratedRight(view),
                    p.width);
            int childHeightSpec = ViewGroup.getChildMeasureSpec(
                    heightSpec,
                    getPaddingTop() + getPaddingBottom() + getDecoratedTop(view) + getDecoratedBottom(view),
                    p.height);
            view.measure(childWidthSpec, childHeightSpec);

            // Get decorated measurements
            measuredDimension[0] = getDecoratedMeasuredWidth(view) + p.leftMargin + p.rightMargin;
            measuredDimension[1] = getDecoratedMeasuredHeight(view) + p.bottomMargin + p.topMargin;
            recycler.recycleView(view);
        }
    }

##### 3. 初始化 RecyclerView
就是按照正常的流程初始化 RecyclerView，只不过在 setLayoutManager() 时，使用我们自定的 WrappingLinearLayoutManager。<br />
至于其中的 DetailCommentAdapter 就是正常的 RecyclerView.Adapter。

    RecyclerView RecyclerHot;
    RecyclerView RecyclerAll;

    private DetailCommentAdapter mHotCommentAdapter;
    private DetailCommentAdapter mAllCommentAdapter;

    private List<CommentsBean> mHotCommentsList = new ArrayList<>();
    private List<CommentsBean> mAllCommentsList = new ArrayList<>();

    private void setView() {
        // 设置热门评论列表
        WrappingLinearLayoutManager wrappingLinearLayoutManager = new WrappingLinearLayoutManager(mContext);
        wrappingLinearLayoutManager.setAutoMeasureEnabled(false);// 如果导入的包是  Android Support Library 23.2.0  以上的，需要加这句
        RecyclerHot.setLayoutManager(wrappingLinearLayoutManager);

        mHotCommentAdapter = new DetailCommentAdapter(mContext, mHotCommentsList, DetailCommentAdapter.COMMENT_TYPE_HOT);
        RecyclerHot.setAdapter(mHotCommentAdapter);
        RecyclerHot.setNestedScrollingEnabled(false);


        // 设置全部评论列表
        WrappingLinearLayoutManager wrappingLinearLayoutManager2 = new WrappingLinearLayoutManager(mContext);
        wrappingLinearLayoutManager2.setAutoMeasureEnabled(false);// 如果导入的包是  Android Support Library 23.2.0  以上的，需要加这句
        RecyclerAll.setLayoutManager(wrappingLinearLayoutManager2);

        mAllCommentAdapter = new DetailCommentAdapter(mContext, mAllCommentsList, DetailCommentAdapter.COMMENT_TYPE_ALL);
        RecyclerAll.setAdapter(mAllCommentAdapter);
        RecyclerAll.setNestedScrollingEnabled(true);
    }

<br />
**注意：**
1. wrappingLinearLayoutManager.setAutoMeasureEnabled(false);<br /><br />
如果导入的包是  Android Support Library 23.2.0  以上的，需要加这句。<br />
2. RecyclerHot.setNestedScrollingEnabled(false);<br />
在这里setNestedScrollingEnabled（false）禁用滚动为RecyclerView，它不会拦截从NestedScrollView滚动事件。<br />
3. setHasFixedSize（false） （默认false）<br />
确定适配器内容中的更改可以更改RecyclerView的大小。

<br />
至此就可以实现想要达到的效果。<br />
其中，主要的核心就是 NestedScrollView 的使用 和 WrappingLinearLayoutManager 的自定义。
<hr />

<br />
项目地址：https://github.com/Wing-Li/DoubleList<br />
建议把项目下载下来看看，项目非常简单，此文主要是流程。