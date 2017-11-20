---
layout: post  #这个不变
title: "调用摄像头和相册" #标题
date: 2017-11-20 20:57 #时间
description: "调用摄像头和相册来获取图片"  #说明
tag: Android #这是分类标签
---

博客更新中....
# 前言
写代码时，遇到了调用摄像头和相册来上传头像的功能，然而自己对调用摄像头和相册的功能不是很熟悉，在网上也查了很多类似代码，感觉都比较老，同时不够详细，于是再次翻起我的神书《Android第一行代码》，找到了此功能的Demo，在此做个笔记，同时分享给大家ヾ(×× ) ﾂ ，俺真的是菜得一匹啊！
## 调用摄像头拍照
首先修改activity_main.xml中的代码：

布局文件中只有两个控件，分别是一个Button和一个ImageView。其中Button用于打开摄像头进行拍照，而ImageView用于将拍到的图片显示出来。
```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:id="@+id/take_photo"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Take Photo"/>

    <ImageView
        android:id="@+id/picture"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"/>

</LinearLayout>
```
然后开始编写调用摄像头的具体逻辑，修改MainActivity中的代码：
```
public class MainActivity extends AppCompatActivity {

    public static final int TAKE_PHOTO = 1;

    private ImageView picture;

    private Uri imageUri;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button takePhoto = (Button) findViewById(R.id.take_photo);
        picture = (ImageView) findViewById(R.id.picture);
        takePhoto.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //创建File对象，用于存储拍照后的图片
                File ouputImage = new File(getExternalCacheDir(), "output_image.jpg");
                try {
                    if (ouputImage.exists()) {
                        ouputImage.delete();
                    }
                    ouputImage.createNewFile();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                if (Build.VERSION.SDK_INT >= 24) {
                    imageUri = FileProvider.getUriForFile(MainActivity.this, "com.example.cameraalbumtest.fileprovider", ouputImage);
                } else {
                    imageUri = Uri.fromFile(ouputImage);
                }
                //启动相机程序
                Intent intent = new Intent("android.media.action.IMAGE_CAPTURE");
                intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
                startActivityForResult(intent, TAKE_PHOTO);
            }
        });

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        switch (requestCode) {
            case TAKE_PHOTO:
                if (resultCode == RESULT_OK) {
                    //将拍摄的照片显示出来
                    try {
                        Bitmap bitmap = BitmapFactory.decodeStream(getContentResolver().openInputStream(imageUri));
                        picture.setImageBitmap(bitmap);
                    } catch (FileNotFoundException e) {
                        e.printStackTrace();
                    }
                }
                break;
            default:
                break;
        }
    }
}
```
代码从这里就开始复杂起来了ヾ(×× ) ﾂ ，我们首先获取到Button和ImageView的实例，并给Buton注册上点击事件。在Button的点击事件中处理调用摄像头的逻辑。

1.首先创建了一个File对象，用于存放摄像头拍下的图片，命名为output_image.jpg，并将它存放在手机SD卡的**应用关联缓存目录**下。

应用关联缓存目录：指SD卡中专门用于**存放当前应用缓存数据**的位置，调用getExternalCacheDir()方法可以得到这个目录，具体的路径是/sdcard/Android/data/<package name>/cache。那么为什么要使用应用关联缓存目录来存放图片呢？？？因为从Android6.0系统开始，读写SD卡被列为危险权限，如果图片存放在SD卡的任何其他目录，都要进行运行时权限处理才行，而使用应用关联目录则可以跳过这一步。

2.接着会进行一个判断，如果运行设备的系统版本低于Android7.0，就调用Uri的fromFile()方法将File对象转换成Uri对象，这个Uri对象标识着output_image.jpg这张图片的本地真实路径。否则，就调用FileProvider的getUriForFile()方法，将File对象转换成一个封装过的Uri对象。getUriForFile()方法接受3个参数，第一个参数要求传入Context对象，第二个参数可以是任意唯一的字符串，第三个参数则是我们刚刚创建的File对象。之所以要进行这样一层转换，是因为从Android7.0系统开始，直接使用本地真实路径的Uri被认为是不安全的，会抛出异常。而FileProvider则是一种特殊的内容提供器，它使用了和内容提供器类似的机制来对数据进行保护，可以选择性地将封装过的Uri共享给外部，从而提高了应用的安全性。

3.构建出一个Intent对象，将这个Intent的action指定为android.media.action.IMAGE_CAPTURE，在调用Intent的putExtra()方法指定图片的输出地址，这里填入刚得到的Uri对象，最后调用startActivityForResult()来启动活动，因此拍完照的结果会返回到onActivityResult()方法中。若拍照成功，调用BitmapFactory的decodeStream()方法，将output_image.jpg这张照片解析成Bitmap对象，然后把它设置到ImageView中显示出来。

4.刚提到内容提供器，那么我们需要在AndroidManifest.xml中对内容提供器进行注册了，代码如下：
```

<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.example.cameraalbumtest">
    <application
        ......
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.example.cameraalbumtest.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
    </application>
</manifest>
```
其中，android:name属性的值是固定的，android:authorities属性的值必须要和刚才FileProvider.getUriForFile()方法中的第二个参数一致。**另外**，我们在<provider>标签
的内部使用<meta-data>来指定Uri的共享路径，并引用了一个@xml/file_paths资源。由于这个资源现在还不存在，所以需要创建下：

在res目录下创建一个xml目录，在该目录下创建一个file_paths.xml文件，并修改其中内容：
```
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android:"http://schemas.android.com/apk/res/android">
  <external-path name="my_images" path=""/>
</paths>
```
其中，external-path就是用来指定Uri共享的，name属性的值可以随便填写，path属性的值表示共享的具体路径，设置**空值表示将整个SD卡进行共享**，当然也可以仅共享我们存放output_image.jpg这图片的路径。

注意：在Android4.4系统之前，访问SD卡的应用关联目录也是要声明权限的，从4.4系统开始不再需要使用权限声明，为了兼容老版本，我们在AndroidManifest.xml中声明下访问SD卡的权限：
```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
OK，调用摄像头拍照，并显示在ImageView上的功能就实现了!
