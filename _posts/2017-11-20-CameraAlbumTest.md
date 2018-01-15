---
layout: post  #这个不变
title: "调用摄像头和相册" #标题
date: 2017-11-20 20:57 #时间
description: "调用摄像头和相册来获取图片"  #说明
tag: Android #这是分类标签
---

# 前言
写代码时，遇到了调用摄像头和相册来上传头像的功能，然而自己对调用摄像头和相册的功能不是很熟悉，在网上也查了很多类似代码，感觉都比较老，同时不够详细，于是再次翻起我的神书《Android第一行代码》，找到了此功能的Demo，在此做个笔记，同时分享给大家ヾ(×× ) ﾂ ，俺真的是菜得一匹啊！
### 调用摄像头拍照
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
}

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

### 从相册中选择照片
修改activity_main.xml文件，添加一个按钮用于从相册中选择照片。
```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    .....
    <Button
        android:id="@+id/choose_from_album"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Choose From Album"/>
    ......

</LinearLayout>
```
修改MainActivity中的代码，加入从相册选择照片的逻辑。
```
public class MainActivity extends AppCompatActivity {

    ......

    public static final int CHOOSE_PHOTO = 2;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ....
        Button chooseFromAlbum = (Button)findViewById(R.id.choose_from_album);
        ....
        chooseFromAlbum.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
                    ActivityCompat.requestPermissions(MainActivity.this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, 1);
                } else {
                    openAlbum();
                }
            }
        });
}

private void openAlbum(){
  Intent intent = new Intent("android.intent.action.GET_CONTENT");
  intent.setType("image/*");
  startActivityForResult(intent,CHOOSE_PHOTO);//打开相册
}

@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    switch (requestCode) {
        case 1:
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                openAlbum();
            } else {
                Toast.makeText(this, "You denied the permission", Toast.LENGTH_SHORT).show();
            }
            break;
        default:
    }
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    switch (requestCode) {
      ...
        case CHOOSE_PHOTO:
            if (resultCode == RESULT_OK) {
                //判断手机系统版本号
                if(Build.VERSION.SDK_INT>=19){
                  //4.4及以上系统使用这个方法处理图片
                  handleImageOnKitKat(data);
                }else{
                  //4.4以下系统使用这个方法处理图片
                  handleImageBeforeKitKat(data);
                }
            }
            break;
        default:
            break;
    }
}

@TargetApi(19)
private void handleImageOnKitKat(Intent data){
  String imagePath = null;
        Uri uri = data.getData();
        if (DocumentsContract.isDocumentUri(this, uri)) {
            //如果是document类型的Uri，则通过document id处理
            String docId = DocumentsContract.getDocumentId(uri);
            if ("com.android.providers.media.document".equals(uri.getAuthority())) {
                String id = docId.split(":")[1];//解析出数字格式的id
                String selection = MediaStore.Images.Media._ID + "=" + id;
                imagePath = getImagePath(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, selection);
            } else if ("com.android.providers.downloads.documents".equals(uri.getAuthority())) {
                Uri contentUri = ContentUris.withAppendedId(Uri.parse("content://downloads/public_downloads"), Long.valueOf(docId));
                imagePath = getImagePath(contentUri, null);
            }
        } else if ("content".equalsIgnoreCase(uri.getScheme())) {
            //如果是content类型的Uri，则使用普通方式处理
            imagePath = getImagePath(uri, null);
        } else if ("file".equalsIgnoreCase(uri.getScheme())) {
            //如果是file类型的Uri，直接获取图片路径即可
            imagePath = uri.getPath();
        }
        displayImage(imagePath);
}

private void handleImageBeforeKitKat(Intent data) {
    Uri uri = data.getData();
    String imagePath = getImagePath(uri, null);
    displayImage(imagePath);
}

private String getImagePath(Uri uri, String selection) {
    String path = null;
    //通过Uri和Seleciton来获取真实的图片路径
    Cursor cursor = getContentResolver().query(uri, null, selection, null, null);
    if (cursor != null) {
        if (cursor.moveToFirst()) {
            path = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.DATA));
        }
        cursor.close();
    }
    return path;
}

private void displayImage(String imagePath) {
    if (imagePath != null) {
        Bitmap bitmap = BitmapFactory.decodeFile(imagePath);
        picture.setImageBitmap(bitmap);
    } else {
        Toast.makeText(this, "failed to get image", Toast.LENGTH_SHORT).show();
    }
}
```

在ChooseFromAlbum按钮中，我们先进行了一个运行时权限处理，动态申请WRITE_EXTERNAL_STORAGE这个危险权限，因为相册中的照片都存储在SD卡上，我们要从SD卡中读取照片就需要申请这个权限。

当权限申请后，会调用openAlbum()方法，这里我们首先构建了一个Intent对象，并将它的action指定为android.intent.action.GET_CONTENT。接着给这个Intent对象设置一些必要的参数，然后调用startActivityForResult()方法，打开相册程序选择照片。
**注意在调用startActivityForResult()方法时，我们给第二个参数传入的值变为了CHOOSE_PHOTO，这样当相册选择完图片回到onActivityResult()方法时，就会进入到CHOOSE_PHOTO的case来处理图片。**

首先为了兼容新老版本的手机，我们需要做一个判断，如果是4.4及以上系统的手机就调用handleImageOnKitKat()方法来处理图片，否则调用handleImageBeforeKitKat()方法。这样做的原因是Android系统4.4版本开始，选取相册中的图片 **不再返回图片真实的Uri** 了，而是一个 **封装过的Uri** ，因为如果是4.4版本以上的手机就需要对这个Uri进行解析了。

在handleImageOnKitKat()方法中，有好几种判断情况，如果返回的Uri是document类型的话，就取出document id进行处理，如果不是就使用普通的方式处理。**如果Uri的authority是media格式的话，document id还需要再进行一次解析，要通过字符串分割的方式取出后半部分才能得到真正的数字 id。** 取出的id用于构建新的Uri和条件语句，然后把这些值作为参数传入到getImagePath()方法当中，就可以获取到图片的真实路径了。拿到图片路径后，调用displayImage()方法将图片显示到界面上。

相比于handleImageOnKitKat()方法，handleImageBeforeKitKat()方法中的逻辑就要简单得多，因为它的Uri是没有封装过的，不需要任何解析，直接将Uri传入到getImagePath()方法当中就能获取到图片的真实路径了，然后调用displayImage()方法来将图片显示到界面上。

0.0 通过拍照或相册选择图片的功能就实现了!
