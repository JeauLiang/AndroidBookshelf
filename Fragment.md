
# 【Android练级之路】Activity {ignore=true}

[TOC]



# 生命周期

• onAttach()：当Fragment和Activity建立关联时调用
• onCreate()
• onCreateView()：当Fragment创建视图时调用
• onActivityCreated()：当与Fragment相关联的Activity完成onCreate()之后调用
• onStart()
• onResume()
• onPause()
• onStop()
• onDestroyView()：在Fragment中的布局被移除时调用
• onDestroy()
• onDetach()：当Fragment和Activity解除关联时调用

# Fragment通信
## Fragment向Activity传递数据
首先，在 Fragment中 定义接口，并让 Activity 实现该接口。

```java
public interface OnFragmentCallback {
    void onCallback(String value);
}
```

在 Fragment 的 `onAttach()` 中，将参数 Context 强转为 OnFragmentCallback 对象：

```java
@Override
public void onAttach(Context context) {
    super.onAttach(context);
    if (context instanceof OnFragmentCallback) {
        callback = (OnFragmentCallback) context;
    } else {
        throw new RuntimeException(context.toString()
                                   + " must implement OnFragmentCallback");
    }
}
```


## Activity向Fragment传递数据

Activity 向 Fragment 传递数据比较简单，获取 Fragment 对象，并调用 Fragment 的方法即可。比如要将一个字符串传递给 Fragment，则在 Fragment 中定义方法：

```java
public void setString(String data) { 
    this.data = data;
}
```
并在 Activity 中调用 `fragment.setString("hello")` 即可。

## Fragment之间的通信
由于 Fragment 之间是没有任何依赖关系的，因此如果要进行 Fragment 之间的通信，建议通过 Activity 作为中介，不要 Fragment 之间直接通信。


# DialogFragment
DialogFragment 是 Android 3.0 提出的，代替了 Dialog，用于实现对话框。它的优点是：即使旋转屏幕，也能保留对话框状态。
如果要自定义对话框样式，只需要继承 DialogFragment，并重写 `onCreateView()`，该方法返回对话框 UI。这里我们举个例子，实现进度条样式的圆角对话框。

```java
public class ProgressDialogFragment extends DialogFragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        //消除Title区域
        getDialog().requestWindowFeature(Window.FEATURE_NO_TITLE);
        //将背景变为透明
        getDialog().getWindow()
            .setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
        //点击外部不可取消
        setCancelable(false);
        View root = inflater.inflate(R.layout.fragment_progress_dialog, container);
        return root;
    }
    public static ProgressDialogFragment newInstance() {
        return new ProgressDialogFragment();
    }
}
```

然后通过下面代码显示对话框：

```java
ProgressDialogFragment fragment = ProgressDialogFragment.newInstance();
fragment.show(getSupportFragmentManager(), "tag");//显示对话框
fragment.dismiss();//关闭对话框
```