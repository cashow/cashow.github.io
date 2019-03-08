---
layout: post
title: "Android - 记录一次关于 RecyclerView 自动下滑问题的 debug 过程"
date: 2017-11-09 00:53:06 +0800
tags: Android 学习笔记 RecyclerView
---

我在开发过程中遇到了一个很诡异的问题，某个由 RecyclerView 实现的页面偶尔会莫名其妙地自动下滑，在经过几个小时的 debug 后我终于解决掉了这个问题。我想通过这次事件分享下我平时的 debug 过程。

---------

### debug 的第一步：找到复现条件

我不喜欢在 bug 不能稳定复现的情况下盲目修改代码，因为如果修改代码后这个 bug 复现不了，这并不意味着 bug 解决掉了，说不定只是这次编译过后我完美地避开了 bug 的复现条件。所以为了修复这个 bug，我必须先找到这个 bug 的复现条件。

因此，我不停地打开关闭这个页面，不停地滑动 RecyclerView，不停地点击各个按钮，经过多次重复测试后，我最终确认了这个 bug 的复现条件：

将 RecyclerView 下滑到某个固定位置时打开一个新的页面，再返回这个页面，RecyclerView 会自动下滑。

--------

### debug 的第二步：找到问题来源

首先需要做的是，开个新分支，把这个页面不相关的代码去掉，只保留可能会影响到这个 bug 的代码。有时候一个复杂的页面经过处理后，会只剩几个函数，这样在 debug 过程中，可以让我们专注于这部分的代码，不至于在 debug 过程中纠结：下面这段看起来不相关的代码到底会不会影响这个 bug？

在精简代码后，就可以开始定位问题的来源了，具体做法就是注释掉某个控件后再运行一遍代码，直到发现某个控件注释掉后这个 bug 不会出现。在我注释掉一个横向的 RecyclerView 后，这个 bug 消失了，所以目前看来，是这个横向的 RecyclerView 的问题。

经过这一步后，这个问题总结如下：

1.在 fragment 中的竖向的 RecyclerView 嵌套一个横向的 RecyclerView；  
2.将竖向的 RecyclerView 下滑到一定位置，这时横向的 RecyclerView 显示在了屏幕中但并没有完全显示；  
3.开启一个新的 fragment 后再返回，竖向的 RecyclerView 会自动下滑。

是切换 fragment 导致的这个 bug 吗？为此我看了 fragment 切换的源码，这是 FragmentManager 里隐藏 fragment 的代码。

<pre class="mcode">
public void hideFragment(Fragment fragment, int transition, int transitionStyle) {
    if (DEBUG) Log.v(TAG, "hide: " + fragment);
    if (!fragment.mHidden) {
        fragment.mHidden = true;
        if (fragment.mView != null) {
            Animator anim = loadAnimator(fragment, transition, false,
                    transitionStyle);
            if (anim != null) {
                anim.setTarget(fragment.mView);
                // Delay the actual hide operation until the animation finishes, otherwise
                // the fragment will just immediately disappear
                final Fragment finalFragment = fragment;
                anim.addListener(new AnimatorListenerAdapter() {
                    @Override
                    public void onAnimationEnd(Animator animation) {
                        if (finalFragment.mView != null) {
                            finalFragment.mView.setVisibility(View.GONE);
                        }
                    }
                });
                setHWLayerAnimListenerIfAlpha(finalFragment.mView, anim);
                anim.start();
            } else {
                fragment.mView.setVisibility(View.GONE);
            }
        }
        if (fragment.mAdded && fragment.mHasMenu && fragment.mMenuVisible) {
            mNeedMenuInvalidate = true;
        }
        fragment.onHiddenChanged(true);
    }
}
</pre>

可以看到隐藏一个 fragment 是通过将 fragment 的 mView 设置成 View.GONE 实现的。那我直接把竖向的 RecyclerView 设置成 View.GONE 再设置成 View.VISIBLE 可以吗？经过测试后发现，竖向的 RecyclerView 隐藏后再显示，还是会有这个问题。

最终确定下的 bug 触发条件是：

1.竖向的 RecyclerView 嵌套一个横向的 RecyclerView；  
2.将竖向的 RecyclerView 下滑到一定位置，这时横向的 RecyclerView 显示在了屏幕中但并没有完全显示；  
3.隐藏并显示这个 RecyclerView，会出现自动下滑。

示意图：

![demo](https://cashow-github-io-1258334739.cos.ap-shanghai.myqcloud.com/nested_recycler_view_scroll_down.gif)

-------

### debug 的第三步：找到 bug 的关键词

好的，目前确定下来了，是嵌套 RecyclerView 隐藏并显示导致的竖向 RecyclerView 自动下滑的问题。接下来的 debug 就交给 Google 和 StackOverFlow 了。现在最重要的是，如何找到描述这个 bug 的正确关键词。

第一次尝试：  
搜索 RecyclerView 自动下滑：“recyclerview auto scroll down”。这几个关键词搜不到这次的 bug。

第二次尝试：  
搜索 RecyclerView 隐藏后显示，自动下滑：“recyclerview visible gone scroll down”。同样，还是没有用。

第三次尝试：  
搜索嵌套 RecyclerView 下滑：“nested recyclerview scroll down”，这时出现了一个可疑的 StackOverFlow 问题：

<https://stackoverflow.com/questions/38949034/nested-recyclerview-scrolls-by-itself>

点开一看，问题描述和我们的情况几乎一模一样！并且，这个问题也在 android 的 issue 里被提到了：

<https://issuetracker.google.com/issues/37120277>

通过上面的2个链接，可以确定下来这个 bug 的原因是：

嵌套 RecyclerView 显示时，子控件（横向的 RecyclerView）会要求获取焦点，从而导致父控件（竖向的RecyclerView）自动下滑。

这个问题的解决方案是，给竖向的 RecyclerView 加上以下代码：

<pre class="mcode">
android:descendantFocusability="blocksDescendants"
</pre>

给竖向的 RecyclerView 设置了 blocksDescendants 属性后，获取焦点时父控件会覆盖子控件，直接获取焦点。

我加上这段代码后，终于，RecyclerView 自动下滑的问题完美解决了。

-------

### 代码链接
<https://github.com/cashow/AndroidTricks/tree/master/NestedRecyclerView>
