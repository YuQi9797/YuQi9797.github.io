---
layout: post
title: "Android 6.0系统中的运行时权限"
date: 2017-10-16 17:17
description: "详细介绍Android 6.0系统中引入的运行时权限这个功能"
tag: Android
---

Android开发团队在Android 6.0系统中引用了运行时权限这个功能，顾名思义，用户不需要在安装软件的时候一次性授权所有申请的权限，而是可以在软件的使用过程中对某一项权限申请进行授权，即便是拒绝了这个权限，用户仍然能够使用这个应用的其他功能，而不是直接无法安装它。

当然并不是所有的权限需要在运行时申请，而需要运行时申请权限的仅有危险权限而已，那危险权限有些什么呢？ヾ(。￣□￣)ﾂ゜゜゜往下看!!!

这里列出了Android中所有的危险权限，一共是9组24个权限。
<div align="left">
  <img src = "/images/image/permissionList.png" />
</div>
每当要使用一个权限时，可以先到这张表中查一查，如果是属于这张表中的权限，那么就需要进行运行时权限处理。说了那么多，接下来我们就来学习一下如果在程序运行时申请权限。
本次我们使用CALL_PHONE这个权限，来实现拨打电话功能。

首先我们在AndroidManifest.xml中添加权限：
<uses-permission android:name="android.permission.CALL_PHONE"/>

其次是我们对activity_main.xml布局文件进行修改，添加一个button控件，代码如下：
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:id="@+id/btn_startCall"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:textAllCaps="false"
        android:text="Start Call"/>

</RelativeLayout>

```
效果如下：
<div>
  <img src="/images/image/activity_main.png" width="270" height="480"/>
</div>
接下来我们修改MainActivity中的代码，为button注册监听事件，代码如下：
```
public class MainActivity extends AppCompatActivity {

    //要申请的权限
    private String[] permissions = {Manifest.permission.CALL_PHONE};
    private AlertDialog dialog;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Button startCall = (Button) findViewById(R.id.btn_startCall);
        startCall.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (ContextCompat.checkSelfPermission(MainActivity.this, permissions[0]) != PackageManager.PERMISSION_GRANTED) {
                    ActivityCompat.requestPermissions(MainActivity.this, permissions, 1);
                } else {
                    call();
                }

            }
        });
    }

    private void call() {
        try {
            Intent intent = new Intent(Intent.ACTION_CALL);
            intent.setData(Uri.parse("tel:10000"));
            startActivity(intent);
        } catch (SecurityException e) {
            e.printStackTrace();
        }
    }
}
```
在这里，我们用String[] permissions来保存我们需要申请的权限，同时定义了一个弹框以备我们后面的使用。在button的onClick()方法中，我们首先判断用户是不是已经给过我们授权了，借助的是ContextCompat.checkSelfPermission()方法。checkSelfPermission()方法接受2个参数，第一个参数是Context，第二个参数是具体的权限名，比如打电话的权限名就是{Manifest.permission.CALL_PHONE，然后我们使用方法的返回值和PackageManager.PERMISSION_GRANTED做比较，如果两者相等就说明用户已经授权，不等则表示用户没有授权。

  如果授权了，就直接调用自定义方法call()，去执行拨打电话，如果没有授权就调用ActivityCompat.requestPermissions()方法来向用户申请授权。requestPermissions()方法接受3个参数，
  第一个参数要求是Activity实例，第二个参数是一个string数组，我们把要申请的权限名放在数组即可(本代码中直接传入变量permissions)，第三个参数是请求码，只要是唯一值就可以了，这里传入1。

  调用完了requestPermissions()方法后，系统会弹出一个权限申请的对话框，然后用户可以选择同意或拒绝我们的申请。
<div>
  <img src="/images/image/dialog.png"/>
</div>
  不论哪种结果，最终都要回调到onRequestPermissionsResult()方法中。所以我们继续修改MainActivity中的代码，添加onRequestPermissionsResult()方法：
```
public class MainActivity extends AppCompatActivity {

    ...省略前面的代码...

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        switch (requestCode) {
            case 1:
                //版本判断。当手机系统大于 23 时，才有必要去判断权限是否获取
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                    //判断用户是否点击了不再提醒。（检测该权限是否还可以申请）
                    if (grantResults[0] != PackageManager.PERMISSION_GRANTED) {
                        boolean flag = shouldShowRequestPermissionRationale(permissions[0]);
                        //用户还是想用我的App的,提示用户去应用设置界面手动开启权限
                        if (!flag) {
                            showSettingDialog();
                        } else {
                            Toast.makeText(this, "You denied the permission", Toast.LENGTH_SHORT).show();
                        }
                    } else {
                        call();
                        Toast.makeText(this, "You accepted the permission", Toast.LENGTH_SHORT).show();
                    }
                }
                break;
            default:
                break;
        }
    }
```
onRequestPermissionsResult()方法中，授权的结果会封装在grantResults参数中。我们只需要判断一下最后的授权结果，如果同意就调用call()方法来拨打电话。如果用户拒绝授权，下一次弹框，用户会有一个“Never ask again”的选项来防止app以后继续请求授权。
<div>
  <img src="/images/image/denied.png"/>
</div>
如果这个选项在拒绝授权前呗用户勾选了。下次对这个权限请求requestPermissions时，对话框就不再弹出来了，系统会直接调用onRequestPermissionsResult函数，回调结果为最后一次用户的选择。所以为了应对这种情况，系统提供了一个shouldShowRequestPermissionRationale()函数，当用户选择了“Never ask again”似，shouldShowRequestPermissionRationale()返回false。所以通过这个函数我们可以弹出一个dialog，告诉用户该功能为什么需要这个权限，并让用户手动开启权限。即自定义方法showSettingDialog().
```
public class MainActivity extends AppCompatActivity {

    ...省略前面的代码...

    // 提示用户该请求权限的弹出框
    private void showSettingDialog() {
        dialog = new AlertDialog.Builder(this)
                .setTitle("You denied the permission")
                .setMessage("请在设置-权限中，进行权限修改。")
                .setPositiveButton("accept", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        Intent intent = new Intent();

                        //根据包名跳转到系统自带的应用程序信息界面(需要参数)
                        intent.setAction(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
                        intent.setData(Uri.fromParts("package", getPackageName(), null));
                        startActivityForResult(intent, 2);
                    }
                })
                .setNegativeButton("cancel", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        Toast.makeText(MainActivity.this, "You denied the permission", Toast.LENGTH_SHORT).show();
                    }
                })
                .setCancelable(false).show();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode) {
            case 2:
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                    //检查该权限是否已经获取
                    int i = ContextCompat.checkSelfPermission(this, permissions[0]);
                    //判断权限是否已经授权
                    if (i != PackageManager.PERMISSION_GRANTED) {
                        //提示用户应该去设置界面手动开启
                        showSettingDialog();
                    } else {
                        if (dialog != null && dialog.isShowing()) {
                            dialog.dismiss();
                        }
                        Toast.makeText(this, "You accepted the permission", Toast.LENGTH_SHORT).show();
                    }
                }
                break;
        }
    }
}
```

<div>
  <img src="/images/image/dialog2.png" />
</div>
<div>
  <img src="/images/image/setting.png" width="270" height="480"/>
</div>
我们在showSettingDialog()中，使用Intent跳转到应用设置界面，并在onActivityResult()中进行判断，看权限是否授权，并执行相应的操作。至此我们的功能就已经实现拉！ヾ(×× ) ﾂ
<div>
  <img src="/images/image/accept.png" width="270" height="480"/>
</div>

代码地址：[点击这里ヾ(･ω･`｡)](https://github.com/Shanks07/androidSources/tree/master/PermissionTest)
