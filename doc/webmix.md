# WebView页面嵌入原生控件


诉求：在WebView加载的web页面特定位置展示原生控件的内容,并跟随web元素滑动。

实现思路：
1. html文档流中插入占位元素初始高度为0.
2. 当页面加载完成后获取元素的位置，把原生控件移动到此位置.
3. 设置占位元素高度为原生控件的高度.

效果：

黄色背景为`TextView`，其他为html内容

<img src="https://img-blog.csdnimg.cn/20190328142941333.gif" width="300px"/><img src="https://img-blog.csdnimg.cn/20190329113502879.gif" width="300px"/>

---
## html中插入占位元素
其中id为的`advertisement`元素 为需要插入原生控件的的位置.

```html
<div id="section1"></div>
<div id="advertisement"></div>
<div id="section2"></div>
```

##  获取元素的位置，移动原生控件.
这里在`onPageFinished(WebView view, String url)`回调中,获取元素的位置，调用js方法`getAdPosition`获取位置(js传回的位置单位为dp),
然后通过`setTranslationY`移动原生控件到id为`advertisement`的html元素位置处.

```java
       mWebview.setWebViewClient(new WebViewClient(){
            @Override
            public void onPageFinished(WebView view, String url) {
                super.onPageFinished(view, url);
                L.e("mWebview:",mWebview.getChildCount()+"");
                TextView textView=new TextView(getApplication());
                textView.setTextColor(Color.GRAY);
                textView.setTextSize(20f);
                textView.setBackgroundColor(Color.YELLOW);
                textView.setText("WebActivity TextView ");
                textView.setGravity(Gravity.CENTER);
                textView.setLayoutParams(new LayoutParams(LayoutParams.MATCH_PARENT,DensityUtil.dp2px(AndroidApplication.getContext(),TV_HEIGHT)));

                mWebview .evaluateJavascript("javaScript:getAdPosition()",
                    new ValueCallback<String>() {
                        @Override
                        public void onReceiveValue(String value) {
                            Log.i("getAdPosition:",value);
                            textView.setTranslationY(DensityUtil.dp2px(WebActivity.this,Float.parseFloat(value)));
                            mWebview.addView(textView);
                            mWebview.loadUrl("javaScript:setAdHeight("+TV_HEIGHT+")");
                        }
                    });
            }
        });
 ```

## 设置占位元素高度
为了避免覆盖html的内容，还需要调用js方法`setAdHeight`把`advertisement`这个div设置为和原生控件同样的高度

js中的两个方法

```javascript
<script type="text/javascript">
    function getAdPosition() {
        var advertisement = document.getElementById("advertisement");
        return advertisement.offsetTop;
    }
    function setAdHeight(height) {
        var advertisement = document.getElementById("advertisement");
        advertisement.style.height=height+"px";
    }
</script>

```
当然如果web端不提供js方法供调用也是可以的，那就把js方法也在本地代码里定义并调用.如下:
```java
      mWebview.evaluateJavascript("javaScript: function getAdPosition() {\n"
                            + "        var advertisement = document.getElementById(\"advertisement\");\n"
                            + "        return advertisement.offsetTop;\n"
                            + "    };getAdPosition()",
                        new ValueCallback<String>() {
                            @Override
                            public void onReceiveValue(String value) {
                                Log.i("getAdPosition:", value);
                                textView.setTranslationY(DensityUtil.dp2px(WebActivity.this, Float.parseFloat(value)));
                                mWebview.addView(textView);
                                mWebview.loadUrl("javaScript: function setAdHeight(height) {\n"
                                    + "        var advertisement = document.getElementById(\"advertisement\");\n"
                                    + "        advertisement.style.height=height+\"px\";\n"
                                    + "    };javaScript:setAdHeight("+TV_HEIGHT+");");
                            }
                        });
```

## 线上页面
以上介绍了本地html的操作方法，那么加载在线页面，或者是第三方的网站还能这样操作吗?
也是可以的，方法基本一致。

这里以百度首页为例，目的是在搜索框上面插入自己的原生组件，这里找到搜索框的id为`index-form`,获取位置的操作同上，
出于简单起见这里不再插入元素占位，而是直接更改搜索框的样式，当插入原生组件后修改搜索框的`marginTop`值。

<img src="https://img-blog.csdnimg.cn/20190329114425926.jpg" />

```java
      mWebView.evaluateJavascript("javaScript: function getAdPosition() {\n"
                              + "        var advertisement = document.getElementById(\"index-form\");\n"
                              + "        return advertisement.offsetTop;\n"
                              + "    };getAdPosition()",
                          new ValueCallback<String>() {
                              @Override
                              public void onReceiveValue(String value) {
                                  Log.e("getAdPosition:", value);
                                  textView.setTranslationY(DensityUtil.dp2px(WebActivity.this, Float.parseFloat(value)));
                                  mWebView.addView(textView);
                                  mWebView.loadUrl("javaScript: function setAdHeight(height) {\n"
                                      + "        var advertisement = document.getElementById(\"index-form\");\n"
                                      + "        advertisement.style.marginTop=height+\"px\";\n"
                                      + "    }; setAdHeight("+(TV_HEIGHT+16)+");");
                              }
                          });
```
[获取源码:https://github.com/qkxyjren/blog/tree/master/webmix](https://github.com/qkxyjren/blog/tree/master/webmix)