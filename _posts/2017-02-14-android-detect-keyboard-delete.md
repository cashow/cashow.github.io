---
layout: post
title: Android - 监听软键盘删除事件
date: 2017-02-14 22:35:05 +0800
tags: Android 软键盘
---

我们项目要做一个类似微博头条文章的图文混排编辑器，在考虑了各种实现方案后，最终定下来的做法就是把每段文字当做一个EditText，每个图片当做一个ImageView，文字和图片放在LinearLayout里。当EditText内容是空的时候，如果用户在软键盘里按删除按钮，要把这个EditText删掉，并将焦点转移到上一个EditText。android官方没有提供监听删除事件的接口，所以只能自己实现。

首先来看效果图：

![android delete detect](http://7xjvhq.com1.z0.glb.clouddn.com/android_delete_detect.gif)

-----------------------

### 实现原理：

先不考虑edittext内容是空的情况。假如现在edittext有内容，那么我们只需要监听edittext的内容变化（<span style="color:red">addTextChangedListener</span>）。如果edittext内容变少了，就说明有数据被删除了。

那有没有办法确保edittext永远有内容？这个实现起来很简单，只需要在edittext是空的时候，<span style="color:red">自动添加一个空格</span>到edittext，并将edittext的<span style="color:red">焦点移到空格之后。</span>

试想下，当你把edittext所有数据都删掉后，edittext会自动补充一个空格进去。在这之后，你每次按下删除键，都会将这个空格删掉，触发edittext的回调，并且在edittext里重新补充一个空格。如此循环往复，就能保证edittext永远有内容，并且每次删除事件都能捕获到。

但还有个问题，空格占的空间较大，如果用户手动拖动光标，或者edittext的内容有多行，很容易就能发现这个空格的存在。那有没有什么字符，用户看不到，宽度还要比空格小？

有的！！！接下来为大家科普一个Unicode字符：<span style="color:red">零宽空格</span>（Zero Width Space）。

wiki链接：<https://en.wikipedia.org/wiki/Zero-width_space>。

顾名思义，零宽空格的意思就是，宽度是0的空格。宽度为0，就意味着所有人都看不见。

\u200b 和 \uFEFF 都是零宽空格，这俩个字符的区别是：

在一行内容放不下所有文字时，\uFEFF 不会导致该字符后面的词换行，\u200b 会导致字符后面的词换行。比如在 Hello 和 World中间插入 \uFEFF，当这一行不够显示所有的文字时，最终显示的效果是：

<pre>
HelloWor
ld
</pre>

而如果在Hello 和 World中间插入 \u200b，最终显示的效果是：

<pre>
Hello
World
</pre>

\uFEFF 没有宽度，而且不导致换行的，这2点特性完全就是我们想要的！！！只要将上文中edittext的空格改成 \uFEFF，我们就既能实现监听删除事件的功能，又不会改变界面显示、影响用户的使用。

----------------------------

### 代码如下：

<pre class='mcode'>
public class MainActivity extends AppCompatActivity {
    /*
     * android官方没有提供监听删除事件的接口，所以需要自己实现。
     * 一个最简单的实现方法就是监听edittext的文本变化（addTextChangedListener），
     * 如果edittext内容变少了，就说明进行了删除操作。
     * 但这个方法有个缺陷，在edittext没有内容的时候，监听不到删除事件。
     * 解决办法就是，在edittext的内容最前面加一个看不见且不占空间的字符，
     * 如果检测到这个字符被删掉，就说明进行了删除操作。
     * 这个特殊字符就是"零宽空格"。
     * wiki链接：https://en.wikipedia.org/wiki/Zero-width_space
     */

    private EditText edittext;
    private TextView textview;

    // 记录edittext在状态改变之前的字数
    // 假如edittext在afterTextChanged之后，字数比preLength少，说明进行了删除操作
    private int preLength;

    // 删除按钮的点击次数
    private int deleteCount;

    // "\uFEFF"是零宽空格，不会引起换行
    // 需要注意的是，"\u200b"也是零宽空格，但是这个字符会导致换行
    // 也就是说，如果"\u200b"字符后面的单词超出了该行的剩余长度，
    // 会导致这个单词换行，在下一行显示
    // 使用"\uFEFF"就不会出现这种情况
    private static final String ZERO_WIDTH_SPACE = "\uFEFF";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initData();
        initView();
    }

    private void initData() {
        preLength = 0;
        deleteCount = 0;
    }

    private void initView() {
        edittext = (EditText) findViewById(R.id.edittext);
        textview = (TextView) findViewById(R.id.textview);

        // edittext初始化时，加上零宽空格，这样的话即使用户没有输入任何内容，
        // 在软键盘按下删除键后，还是能捕获到删除事件
        edittext.setText(ZERO_WIDTH_SPACE);
        preLength += 1;

        // 如果edittext删除了零宽空格，那说明用户按下了删除按钮，
        // 并且这个时候edittext已经没有内容。
        // 为了能检测到下一次删除事件，这时需要再往edittext里补一个零宽空格。
        edittext.addTextChangedListener(new TextWatcher() {
            @Override
            public void beforeTextChanged(CharSequence s, int start,
                  int count, int after) {
            }

            @Override
            public void onTextChanged(CharSequence s, int start, int before,
                  int count) {
            }

            @Override
            public void afterTextChanged(Editable editable) {
                // edittext 现在的内容
                String text = editable.toString();

                // edittext 现在的字数
                int length = text.length();

                if (length &lt; preLength) {
                    // edittext 的字数比之前记录的字数要少，说明进行了删除操作
                    deleteCount += 1;
                    textview.setText(String.format("delete count: %s", deleteCount));
                }
                preLength = length;

                // 【注意】为了保证每次删除事件都能检测到，
                // 零宽空格必须要在所有输入内容的前面，并且要在光标前面。
                // 有2种情况需要特别处理：
                // 1.用户把零宽空格删除掉了：
                // 这个时候需要在 edittext 最前面手动加上零宽空格。
                // 2.edittext 的光标出现在了零宽空格之前：
                // 这个时候用户所有输入的内容都会出现在零宽空格之前。
                // 为了保证零宽空格出现在所有内容之前，
                // 需要把 edittext 里位置不对的零宽空格移除掉，
                // 并在 edittext 最前面加上零宽空格。
                if (!text.startsWith(ZERO_WIDTH_SPACE)) {
                    text = text.replace(ZERO_WIDTH_SPACE, "");
                    text = ZERO_WIDTH_SPACE + text;
                    edittext.setText(text);
                }

                // 如果现在 edittext 里只有零宽空格
                // 将光标移到零宽空格之后
                if (text.equals(ZERO_WIDTH_SPACE)) {
                    moveSelectionToLast(edittext);
                }
            }
        });
    }

    // 将光标移到末尾
    private void moveSelectionToLast(EditText edittext) {
        int length = edittext.getText().length();
        edittext.setSelection(length);
    }
}
</pre>
