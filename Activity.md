


# 【Android练级之路】Activity



## 大纲
![大纲](https://github.com/meetleong/AndroidBookshelf/blob/master/resources/Activity.png#pic_center)

---

##   生命周期
![生命周期](https://img-blog.csdnimg.cn/img_convert/d8d5a0b496b7999aaae45cb33f45facc.png#pic_center)
+ onCreate()：不可见
+ onStart()：可见，但没有焦点，不可交互
+ onResume()：可见，有焦点，可以交互
+ onPause()：可见，有焦点，可以交互
+ onStop()：可见，没有焦点，不可交互
+ onDestroy()：不可见

---
## 启动模式
+ Standard：系统默认，每次启动都会创建一个新的Activity实例
+ SingleTop：栈顶复用，如果处于栈顶，直接调用，非栈顶则创建新的Activity实例
+ SingleTask：栈内复用，如果存在栈内，则在其上所有Activity全部出栈，使得其位于栈顶，生命周期和SingleTop一样，app首页基本是用这
+ SingleInstance：这个是SingleTask加强本，系统会为要启动的Activity单独开一个栈，这个栈里只有它

---
## 页面状态保存与复制
![页面状态保存与复制](https://img-blog.csdnimg.cn/img_convert/63609b720c04ef1a754997b70ada8d64.png#pic_center)

当系统“未经你许可”时销毁了你的activity，则onSaveInstanceState会被系统调用。用来保存状态，调用在onStop之前，但和onPause没有时序关系。onSaveInstanceState()适用于对临时性状态的保存，而onPause()适用于对数据的持久化保存

<br>

### onSaveInstanceState()方法
+ 什么时候调用
	+ 	 当用户按下HOME键时。
    + 长按HOME键，选择运行其他的程序时。
    + 按下电源按键（关闭屏幕显示）时。
    + 从activity A中启动一个新的activity时
    + 屏幕方向切换时，例如从竖屏切换到横屏时(==与android:configChanges属性相关联==)

```java
@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
    // 保存数据
    savedInstanceState.putString("data", "这是保存的数据");
    super.onSaveInstanceState(savedInstanceState);
}
```
### onRestoreInstanceState()方法
只有上述第4种情况会调用 onRestoreInstanceState()，onSaveInstanceState 和 onRestoreInstanceState 并不是一定成双出现的，终于当 Activity 真正的被销毁的时候才会执行 onRestoreInstanceState()。
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // 恢复数据
    if (savedInstanceState != null) {
        String data = savedInstanceState.getInt("data");
    }
}

@Override
public void onRestoreInstanceState(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // 也可以在 onRestoreInstanceState() 方法中恢复数据
    if (savedInstanceState != null) {
        String data = savedInstanceState.getInt("data");
    }
}
```

---
## 页面跳转
### startActivity()
### startActivityForResult()

 AActivity → BActivity

	AActivity:

```java
	@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    //...
    int resultCode = 1;
    startActivityForResult(new Intent(this, BActivity.class), resultCode);
}

/**
 * 接收 OtherActivity 返回的数据
 */
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    String result = data.getExtras().getString("result");
}
```

	BActivity:
```java
public void feekBack(){
    Intent intent = new Intent();
    // 把返回数据存入 Intent
    intent.putExtra("result", "我是返回数据");
    // 设置返回数据
    setResult(RESULT_OK, intent);
    // 关闭当前 Activity
    finish();
}
```

### URL Scheme
Android 中的 Scheme 是一种页面内跳转协议，是一种非常好的实现机制。通过定义自己的 Scheme 协议，可以非常方便跳转 App 中的各个页面。

使用场景：
+ 通过小程序，利用 Scheme 协议打开原生 App。
+ H5 页面点击锚点，根据锚点具体跳转路径 App 端跳转具体的页面。
+ App 端收到服务器端下发的 Push 通知栏消息，根据消息的点击跳转路径跳转相关页面。
+ App 根据URL跳转到另外一个 App 指定页面。
+ 通过短信息中的 URL 打开原生 App。

Scheme 路径的规则：

	<scheme> :// <host> : <port> [<path>|<pathPrefix>|<pathPattern>]

设置 Scheme
在 AndroidManifest.xml 中对标签增加设置 Scheme。


```xml
<activity
    android:name=".ui.activity.SchemeActivity"
    android:screenOrientation="portrait">
    <!--Android 接收外部跳转过滤器-->
    <!--要想在别的 App 上能成功调起 App，必须添加 intent 过滤器-->
    <intent-filter>
        <!--协议部分配置，注意需要跟 web 配置相同-->
        <!--协议部分，随便设置 aa://bb:1024/from?type=jeanboy-->
        <data android:scheme="aa"
            android:host="bb"
            android:port="1024"
            android:path="/from"/>
        <!--下面这几行也必须得设置-->
        <category android:name="android.intent.category.DEFAULT" />
        <!--表示 Activity 允许通过网络浏览器启动，以显示链接方式引用，如图像或电子邮件-->
        <category android:name="android.intent.category.BROWSABLE" />
        <action android:name="android.intent.action.VIEW" />
    </intent-filter>
</activity>
```
原生调用：
```java
Uri uri = Uri.parse("aa://bb:1024/from?type=jeanboy");
Intent intent = new Intent(Intent.ACTION_VIEW, uri);
startActivity(intent);
```
网页调用：
```xml
<a href="aa://bb:1024/from?type=jeanboy">打开 App</a>
```
在 SchemeActivity 中可以处理 Scheme 跳转的参数：
```java
public class SchemeActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Uri uri = getIntent().getData();
        if (uri != null) {
            //获取指定参数值
            String type = uri.getQueryParameter("type");
            Log.e("SchemeActivity", "type:" + type);

            if(type.equals("jeanboy")){
                ActivityUtils.startActivity(XXXActivity.class);
            }else if(type.equals("main")){
                ActivityUtils.startActivity(MainActivity.class);
            }
        }
        finish();
    }
}
```
如何判断一个 Scheme 是否有效：
```java
PackageManager packageManager = getPackageManager();
Uri uri = Uri.parse("aa://bb:1024/from?type=jeanboy");
Intent intent = new Intent(Intent.ACTION_VIEW, uri);
List<ResolveInfo> activities = packageManager.queryIntentActivities(intent, 0);
boolean isValid = !activities.isEmpty();
if (isValid) {
    startActivity(intent);
}
```



---

