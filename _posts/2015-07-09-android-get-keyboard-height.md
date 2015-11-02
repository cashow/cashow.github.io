---
layout: post
title: Android - 监听软键盘状态以及获取软键盘的高度
date: 2015-07-09 00:51:05 +0800
tags: Android 软键盘
---

有时候在开发过程中需要监听软键盘的显示状态，在显示和隐藏键盘时对UI做出相应的调整。Android官方没有提供获取软键盘高度和状态的方法，但是可以通过监听当前应用的高度变化计算出软键盘的高度。  
![效果图](http://7xjvhq.com1.z0.glb.clouddn.com/keyboard-height.gif)  
首先获取到activity布局的最外层layout，通过调用getWindowVisibleDisplayFrame获取到整个应用可以显示的区域，这其中包括ActionBar和状态栏，但不含设备底部的虚拟按键。  
再通过调用getRootView().getHeight()，获取整个屏幕的高度。同样，这个高度也不含虚拟按键的高度。  
通过比较这两个高度，可以推断出软键盘是否显示，并且获取到软键盘的高度。  
<font color='red'>需要注意的是，这种方法前提是软键盘会影响界面布局，所以需要在AndroidManifest里给activity添加android:windowSoftInputMode="adjustResize"</font>  
具体实现代码如下：  
<pre class="mcode">
private LinearLayout layout_main;
private TextView text;

// 状态栏的高度
private int statusBarHeight;
// 软键盘的高度
private int keyboardHeight;
// 软键盘的显示状态
private boolean isShowKeyboard;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    statusBarHeight = getStatusBarHeight(getApplicationContext());

    text = (TextView) findViewById(R.id.text);
    layout_main = (LinearLayout) findViewById(R.id.layout_main);

    layout_main.getViewTreeObserver().addOnGlobalLayoutListener(globalLayoutListener);
}

private ViewTreeObserver.OnGlobalLayoutListener globalLayoutListener = new ViewTreeObserver.OnGlobalLayoutListener() {

    @Override
    public void onGlobalLayout() {
    	// 应用可以显示的区域。此处包括应用占用的区域，
        // 以及ActionBar和状态栏，但不含设备底部的虚拟按键。
        Rect r = new Rect();
        layout_main.getWindowVisibleDisplayFrame(r);

        // 屏幕高度。这个高度不含虚拟按键的高度
        int screenHeight = layout_main.getRootView().getHeight();

        int heightDiff = screenHeight - (r.bottom - r.top);

        // 在不显示软键盘时，heightDiff等于状态栏的高度
        // 在显示软键盘时，heightDiff会变大，等于软键盘加状态栏的高度。
        // 所以heightDiff大于状态栏高度时表示软键盘出现了，
        // 这时可算出软键盘的高度，即heightDiff减去状态栏的高度
        if(keyboardHeight == 0 && heightDiff > statusBarHeight){
            keyboardHeight = heightDiff - statusBarHeight;
        }

        if (isShowKeyboard) {
            // 如果软键盘是弹出的状态，并且heightDiff小于等于状态栏高度，
            // 说明这时软键盘已经收起
            if (heightDiff <= statusBarHeight) {
                isShowKeyboard = false;
                onHideKeyboard();
            }
        } else {
            // 如果软键盘是收起的状态，并且heightDiff大于状态栏高度，
            // 说明这时软键盘已经弹出
            if (heightDiff > statusBarHeight) {
                isShowKeyboard = true;
                onShowKeyboard();
            }
        }
    }
};

private void onShowKeyboard() {
    // 在这里处理软键盘弹出的回调
    text.setText("onShowKeyboard : keyboardHeight = " + keyboardHeight);
}

private void onHideKeyboard() {
    // 在这里处理软键盘收回的回调
    text.setText("onHideKeyboard");
}

@Override
protected void onDestroy() {
    super.onDestroy();
    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN) {
        layout_main.getViewTreeObserver().removeGlobalOnLayoutListener(globalLayoutListener);
    } else {
        layout_main.getViewTreeObserver().removeOnGlobalLayoutListener(globalLayoutListener);
    }
}

// 获取状态栏高度
public static int getStatusBarHeight(Context context) {
    try {
        Class&lt;?&gt; c = Class.forName("com.android.internal.R$dimen");
        Object obj = c.newInstance();
        Field field = c.getField("status_bar_height");
        int x = Integer.parseInt(field.get(obj).toString());
        return context.getResources().getDimensionPixelSize(x);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return 0;
}
</pre>