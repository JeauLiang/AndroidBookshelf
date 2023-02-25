# 【Android练级之路】那些隐藏在角落的bug

<br>

## 动画repeatCount无效
### 前情提要
在`res`文件夹创建了个`anim_scale_in.xml`的动画文件，代码如下：
```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
	<scale
		android:duration="500"
		android:fromXScale="0"
		android:fromYScale="0"
		android:pivotX="50%"
		android:pivotY="50%"
		android:toXScale="1.0"
		android:toYScale="1.0" >
	</scale>
</set>
```
在`Activity`加载动画，代码如下：
```
val anim = AnimationUtils.loadAnimation(this,R.anim.anim_scale_in)
anim.repeatMode = Animation.REVERSE
anim.repeatMode = 10
view.animation = anim
```
运行之后就会发现动画只会执行一次！！
### 解决方案
`anim_scale_in.xml`最外层的`<set>`标签改为`<scale>`就可以了
