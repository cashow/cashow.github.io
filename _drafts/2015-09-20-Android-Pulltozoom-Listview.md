---
layout: post
title: "Android动画效果 - 实现Listview下拉时图片缩放的功能"
date: 2015-10-25 23:49:14 +0800
tags: Android Android动画效果 Listview
---

社交类app的个人资料页布局一般是：在一张背景图上展示用户的信息，在用户信息的下方则是用户最近发布的内容列表。部分app还会实现下拉时背景图片放大的效果。这种效果看起来很复杂，原理其实很简单。  
先看效果图：
![效果图](http://7xjvhq.com1.z0.glb.clouddn.com/pulltozoom.gif)  
***
###实现方法：  
####布局文件：
ListView顶部的用户信息栏分为两部分，一个是显示背景图片的image_background，一个是展示用户信息的layout_userinfo。两个控件放在FrameLayout里面。  
####图片缩放：
还记得ImageView的centerCrop有什么用吗？在将ScaleType属性设置成centerCrop后，ImageView里面的图片会等比例缩放，使得图片填充满整个控件，并显示图片中间部分。比如在100x200的imageview里显示400x400的图片，会将图片缩放成200x200并显示图片中间部分。  
假如展示用户信息的layout_userinfo宽度是100像素，高度是200像素，那么我们可以生成一个100x200的bitmap，并放入设置了centerCrop的image_background里面。  
想象一下，在手指往下拉的过程中，不断修改image_background的高度，控件大小从100x200，拉伸到100x300，100x400，而对应的bitmap也要等比例拉伸到高度为200、300、400像素，并截取中间部分。于是在拉伸过程中，显示背景的bitmap被拉大，两边多余的内容被裁剪，最后的效果就如上图所示。  
####用户信息栏的处理：
图片缩放功能实现好了，现在还需要实现下拉过程中，用户信息一直在layout_userinfo的底部。  
我们可以把layout_userinfo放入LinearLayout里面，在其上方放一个初始高度为0的view，下拉过程中通过修改这个view的高度，使得用户信息栏一直处在父控件LinearLayout的底部。通过获取占位的view的高度，还可以判断现在是否在缩放状态。  
####缩放过程的处理：  
1.如果不在缩放状态，并且listview已经滑动到顶部，这时进入缩放状态，记录下手指触摸的位置，作为初始点的位置；  
2.在移动过程中，假如触摸点的高度大于初始点的高度，表示图片需要拉伸，这时进行拉伸操作并拦截触摸事件，否则不拦截触摸事件。通过对触摸事件的拦截，可以实现listview上滑的过程中，背景图片缩放状态先复位，然后listview再继续上移的效果；  
3.手指抬起或者触摸点离开listview时，将缩放状态重置。  
***
###具体实现代码如下：  
####Activity的代码：  
<pre class="mcode">
public class MainActivity extends ActionBarActivity {

    private ListView listview;
    private ListviewAdapter adapter;

    // 图片是否在缩放状态
    private boolean isZooming;
    // 图片开始缩放时的触摸位置
    private float oriPosition;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        listview = (ListView) findViewById(R.id.listview);

        adapter = new ListviewAdapter(this);
        listview.setAdapter(adapter);

        init();
    }

    private void init() {
        Drawable drawable = getResources().getDrawable(R.drawable.listview_header_background);
        Bitmap bitmap = ((BitmapDrawable) drawable).getBitmap();

        // 将图片缩放到宽度为640像素，避免占用太多内存
        int newWidth = 640;
        int newHeight = (int) (640 * bitmap.getHeight() * 1.0 / bitmap.getWidth());
        Bitmap avatarBitmap = getResizedBitmap(bitmap, newWidth, newHeight);

        adapter.setAvatarBitmap(avatarBitmap);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        // 如果移动过程中触摸点不在listview上，比如移动到了屏幕外侧或者系统的虚拟按键等
        // 这时需要将缩放状态重置
        if (!isInView(listview, event.getRawX(), event.getRawY())) {
            adapter.resetZoom();
            isZooming = false;
            return false;
        }

        // 如果图片不在缩放状态并且listview已经滑动到了顶部，图片进入缩放状态
        int top = listview.getChildAt(0).getTop();
        if (!isZooming && top == 0) {
            oriPosition = event.getRawY();
            isZooming = true;
        }

        // 如果不在缩放状态，不需要处理ACTION_MOVE和ACTION_UP事件
        if (!isZooming)
            return super.dispatchTouchEvent(event);
        switch (event.getActionMasked()) {
            case MotionEvent.ACTION_MOVE:
                // 计算触摸点与初始点的距离，根据这个距离计算图片缩放的大小
                float diff = event.getRawY() - oriPosition;
                if (diff > 0) {
                    // diff大于0表示正在往下拉，此时图片处于缩放状态，
                    // 需要拦截触摸事件并缩放图片
                    adapter.setZoomDiff(diff);
                    return true;
                }
                break;
            case MotionEvent.ACTION_UP:
                isZooming = false;
                return adapter.resetZoom();
            default:
                break;
        }
        return super.dispatchTouchEvent(event);
    }

    // 用来判断坐标(rawX, rawY)是否在view上
    private boolean isInView(View view, float rawX, float rawY) {
        int scrcoords[] = new int[2];
        view.getLocationOnScreen(scrcoords);
        float x = rawX + view.getLeft() - scrcoords[0];
        float y = rawY + view.getTop() - scrcoords[1];
        if (x >= view.getLeft() && x <= view.getRight() && y >= view.getTop() && y <= view.getBottom())
            return true;
        return false;
    }

    // 将初始的bitmap缩放成宽度为newWidth，高度为newHeight的bitmap
    private Bitmap getResizedBitmap(Bitmap bitmap, int newWidth, int newHeight) {
        int width = bitmap.getWidth();
        int height = bitmap.getHeight();
        float scaleWidth = ((float) newWidth) / width;
        float scaleHeight = ((float) newHeight) / height;
        Matrix matrix = new Matrix();
        matrix.postScale(scaleWidth, scaleHeight);
        Bitmap resizedBitmap = Bitmap.createBitmap(bitmap, 0, 0, width, height, matrix, true);
        return resizedBitmap;
    }
}
</pre>
####ListviewAdapter的代码：  
<pre class="mcode">
public class ListviewAdapter extends BaseAdapter {

    private LayoutInflater mLayoutInflater;

    private View headerView;
    private Bitmap avatarBitmap;

    // 图片目前的缩放比例
    private double currentRatio;
    // 图片接下来要缩放的比例
    private double nextRatio;

    // 判断有没有加载过背景图片
    private boolean isInit;

    // 背景图片初始的高度
    private int avatarOriHeight;

    public ListviewAdapter(Context context) {
        mLayoutInflater = LayoutInflater.from(context);;
    }

    public void setAvatarBitmap(Bitmap avatarBitmap) {
        this.avatarBitmap = avatarBitmap;
        notifyDataSetChanged();
    }

    @Override
    public int getCount() {
        return 50;
    }

    @Override
    public int getItemViewType(int position) {
        if (position == 0)
            return 0;
        return 1;
    }

    @Override
    public int getViewTypeCount() {
        return 2;
    }

    @Override
    public Object getItem(int position) {
        return null;
    }

    @Override
    public long getItemId(int position) {
        return 0;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        if (getItemViewType(position) == 0) {
            if (headerView == null) {
                headerView = mLayoutInflater.inflate(R.layout.listview_header,
                        parent, false);
            }
            // view_placeholder和headerView里面的内容（头像、关注、粉丝等）
            // 处在LinearLayout里。在图片缩放的过程中修改view_placeholder的高度，
            // 可以使headerView的内容一直处在底部，实现headerView的内容随着手指下拉
            // 而下移的效果
            LinearLayout layout_info = (LinearLayout)
                    headerView.findViewById(R.id.layout_info);
            View view_placeholder =
                    headerView.findViewById(R.id.view_placeholder);
            ImageView image_background = (ImageView)
                    headerView.findViewById(R.id.image_background);

            if (!isInit && avatarBitmap != null) {
                updateImageBackground(layout_info, view_placeholder,
                        image_background);
            }
            return headerView;
        }
        ViewHolder holder;
        View view = convertView;
        if (convertView == null) {
            view = mLayoutInflater.inflate(R.layout.listview_item, parent,
                    false);
            holder = new ViewHolder();
            holder.text = (TextView) view.findViewById(R.id.text);
            view.setTag(holder);
        } else {
            holder = (ViewHolder) view.getTag();
        }
        holder.text.setText("text " + position);
        return view;
    }

    private void updateImageBackground(final LinearLayout layout_info,
        final View view_placeholder, final ImageView image_background){
        isInit = true;
        // image_background的宽度和高度要和layout_info的一样，
        // 所以需要在获取到layout_info高度后设置image_background的高度
        final ViewTreeObserver viewTreeObserver =
                layout_info.getViewTreeObserver();
        viewTreeObserver.addOnGlobalLayoutListener(
            new ViewTreeObserver.OnGlobalLayoutListener() {
                @Override
                public void onGlobalLayout() {
                    // view_placeholder的高度大于0时表示正处于缩放状态，
                    // 不能使用这个时候layout_info的高度
                    if (view_placeholder.getHeight() > 0) {
                        return;
                    }
                    int layoutWidth = layout_info.getWidth();
                    int layoutHeight = layout_info.getHeight();

                    // 记录背景图片初始的高度
                    avatarOriHeight = layoutHeight;

                    // 设置image_background的宽度和高度
                    FrameLayout.LayoutParams params = (FrameLayout.LayoutParams) image_background.getLayoutParams();
                    params.width = layoutWidth;
                    params.height = layoutHeight;
                    image_background.setLayoutParams(params);

                    // 生成宽高比和image_background一样的图片
                    Bitmap cropAvatarBitmap = getCropAvatarBitmap(avatarBitmap
                            , layoutWidth * 1.0 / layoutHeight);
                    image_background.setImageBitmap(cropAvatarBitmap);

                    avatarBitmap = null;

                    if (android.os.Build.VERSION.SDK_INT >=
                            android.os.Build.VERSION_CODES.JELLY_BEAN) {
                        viewTreeObserver.removeOnGlobalLayoutListener(this);
                    } else {
                        viewTreeObserver.removeGlobalOnLayoutListener(this);
                    }
                }
            });
    }

    // 设置图片的缩放比例
    // zoomDiff是现在的触摸点和初始的触摸点之间的距离
    // 图片最大可以缩放的比例是2倍
    // 图片的缩放比例是：
    // nextRatio = 1.0 + zoomDiff / avatarOriHeight / (1.0 + zoomDiff / avatarOriHeight)
    // 这个计算方法可以实现移动得越远，图片缩放的速度越慢
    public void setZoomDiff(double zoomDiff) {
        double ratio = 1.0 + zoomDiff / avatarOriHeight;
        this.nextRatio = 1.0 + zoomDiff / avatarOriHeight / ratio;
        this.nextRatio = Math.min(this.nextRatio, 2.0);

        ImageView image_background = (ImageView)
                headerView.findViewById(R.id.image_background);
        View view_placeholder =
                headerView.findViewById(R.id.view_placeholder);

        // currentRatio不等于zoomRatio表示需要修改图片的高度
        if (Math.abs(currentRatio - nextRatio) > 1e-6) {
            zoomImage(image_background, view_placeholder, currentRatio,
                    nextRatio, 10);
            currentRatio = nextRatio;
        }
    }

    // 重置缩放状态。图片在200ms内从现在的缩放比例恢复到初始的比例
    // 如果图片处于缩放状态，需要拦截触摸事件的ACTION_UP
    // 返回值是true表示需要拦截触摸事件，false表示不需要拦截
    public boolean resetZoom() {
        if (this.currentRatio != 1.0) {
            ImageView image_background = (ImageView)
                    headerView.findViewById(R.id.image_background);
            View view_placeholder =
                    headerView.findViewById(R.id.view_placeholder);

            zoomImage(image_background, view_placeholder, currentRatio,
                    1.0, 200);
            currentRatio = 1.0;
            return true;
        }
        return false;
    }

    // 缩放图片，将图片的缩放比例从startRatio渐变到endRatio
    private void zoomImage(final ImageView image_background,
                           final View view_placeholder,
                           final double startRatio,
                           final double endRatio,
                           long time) {
        ValueAnimator valueAnimator = ValueAnimator.ofInt(1, 100);
        valueAnimator.addUpdateListener(
            new ValueAnimator.AnimatorUpdateListener() {

            private IntEvaluator mEvaluator = new IntEvaluator();

            @Override
            public void onAnimationUpdate(ValueAnimator animator) {
                //获得当前动画的进度值，整型，1-100之间
                int currentValue = (Integer) animator.getAnimatedValue();

                //计算当前进度占整个动画过程的比例，浮点型，0-1之间
                float fraction = currentValue / 100f;

                int imageHeight = mEvaluator.evaluate(
                        fraction,
                        (int) (avatarOriHeight * startRatio),
                        (int) (avatarOriHeight * endRatio));
                image_background.getLayoutParams().height = imageHeight;
                image_background.requestLayout();
                
                int viewHeight =  mEvaluator.evaluate(
                        fraction,
                        (int) (avatarOriHeight * (startRatio - 1.0)),
                        (int) (avatarOriHeight * (endRatio - 1.0))); 

                view_placeholder.getLayoutParams().height = viewHeight;
                view_placeholder.requestLayout();
            }
        });
        valueAnimator.setDuration(time).start();
    }

    // 将图片裁剪成宽高比为ratio的图片
    private Bitmap getCropAvatarBitmap(Bitmap bitmap, double ratio) {
        int bitmapWidth = bitmap.getWidth();
        int bitmapHeight = bitmap.getHeight();

        int newWidth;
        int newHeight;
        if (bitmapWidth / ratio > bitmapHeight) {
            newWidth = (int) (bitmapHeight * ratio);
            newHeight = bitmapHeight;
        } else {
            newWidth = bitmapWidth;
            newHeight = (int) (bitmapWidth / ratio);
        }

        Bitmap newBitmap = Bitmap.createBitmap(
                bitmap,
                (bitmapWidth - newWidth) / 2,
                (bitmapHeight - newHeight) / 2,
                newWidth,
                newHeight);
        return newBitmap;
    }

    class ViewHolder {
        TextView text;
    }
}
</pre>