---
layout: post  #这个不变
title: "Android的三个小技能" #标题
date: 2017-10-14 20:30 #时间
description: "知晓当前是在哪一个活动、随时随地地退出程序、启动活动的最佳写法"  #说明
tag: Android #这是分类标签
---

今天所记录的，是我在《第一行代码 Android》中看见的，感觉十分有用，在此想分享给大家。(～￣▽￣)～

1.知晓当前是哪一个活动：

  根据程序当前的界面判断出这是哪一个活动。有人肯定会问，为什么这个很有用呢？原因很简单，当你真正进入到企业之后，你很可能会接手一份儿别人写的代码
，而读别人的代码是很痛苦的一件事情，如果你想修改某个界面上的东西，会花很多时间去找该界面对应的活动是哪一个。而学会这个技巧后，这对你来说就都不是什么难事儿了ヾ(*´▽‘*)ﾉ

  首先，我们在已有项目中新建一个BaseActivity类（它只是一个普通的java类，不需要在AnroidManifest.xml中注册），然后让BaseActivity继承自AppCompatActivity,并重写onCreate()方法，我们在onCreate()方法中获取到了当前实例的类名，并通过Log打印了出来，如下代码：
```
public class BaseActivity extends AppCompatActivity{

  @Override
  protected void onCreate(Bundle savedInstanceState){
    super.onCreate(savedInstanceState);
    Log.d("BaseActivity",getClass().getSimpleName());
  }
}
```
  接下来我们需要让BaseActivity成为你项目中所有活动的父类，让所有活动不再继承自AppCompatActivity，而是继承自BaseActivity。由于BaseActivity又是继承自AppCompatActivity的，所以项目中所有活动的现有功能并不会受影响，仍然完全继承了Activity中的所有特性。

  写完之后，每当你进入到一个活动的界面，该活动的类名就会被打印出来，这样就能知晓当前界面所对应的是哪一个活动了!(～￣▽￣)～ 惊不惊喜？意不意外？先别着急，这BaseActivity我们还要接着往里加东西呢！我们紧接着来看看第二个小技能。

2.随时随地地退出程序:

  当我们使用Intent进行页面跳转的时候，比如从1->2->3,当我们要退出程序时则需要连按3次Back键，十分不方便。此时我们的程序就十分需要一个注销或者退出的功能。

  解决思路：用一个专门的集合类对所有的活动进行管理。接下来我们就来实现一下，新建一个ActivityCollector类作为活动管理器，代码如下：
```
public class ActivityCollector{

  private static List<Activity> activities = new ArrayList<>();

  /*向List中添加一个活动*/
  public static void addActivity(Activity activity){
    activities.add(activity);
  }

  /*从List中移除活动*/
  public static void removeActivity(Activity activity){
    activities.remove(activity);
  }

  /*将List中存储的活动全部销毁掉*/
  public static void finishAll(){
    for(Activity activity : activities){
      if(!activity.isFinishing()){
        activity.finish();
      }
    }
    activities.clear();
  }

}
```
  在活动管理器中，我们通过一个List来暂存活动，然后提供一个addActivity()方法用于向List中添加一个活动，提供了一个removeActivity()方法用于从List中移除活动，最后提供了一个finishAll()方法用于将List中存储的活动全部销毁掉。

  现在我们就要对BaseActivity的onCreate()方法中调用ActivityCollector的addActivity()方法，表明将当前正在创建的活动添加到活动管理器里。然后在BaseActivity中重写onDestory()方法，并调用ActivityCollector的removeActivity()方法,表明将一个马上要销毁的活动从活动管理器里移除。

  从此之后，不管你想在什么地方退出程序，只需要在所需要的活动中调用ActivityCollector.finishAll()方法，就可以直接退出程序了。 岂不美滋滋！ヾ(ﾟ∀ﾟゞ)
为了保证程序完全退出，也可以在销毁所有活动的代码后加入:android.os.Process.killProcess(android.os.Process.myPid());用于杀掉当前进程。其中，killProcess()方法用于杀掉一个进程，它接受一个进程id参数，我们通过myPid()方法来获取当前程序的进程id。注意：killProcess()方法只能用于杀掉当前程序的进程。

3.启动活动的最佳写法：

  在项目开发中，你会经常遇到对接的问题。比如一个活动(SecActivity)不是由你开发的，但现在你负责的部分需要有启动该Activiy这个功能，而启动这个活动需要传递哪些数据我们却不清楚。为了将该Activity所需要的数据全部体现出来，为了不去询问负责编写该Activity的人员，为了不用阅读该Activity中的代码，我们就需要写一个启动活动的最佳写法。

  首先，在SecActivity中添加一个actionStart()方法,代码如下：
  ```
  public static void actionStart(Context context,String data1,String data2){
    Intent intent = new Intent(context,SecActivity.class);
    intent.putExtra("param1",data1);
    intent.putExtra("param2",data2);
    context.startActivity(intent);
  }
  ```
  通过actionStart()方法，我们完成了Intent的构建，另外所有SecActivity中需要的数据都是通过actionStart()方法的参数传递过来的，然后存储到Intent中，最后调用startActivity()方法启动SecActivity，并且在启动SecActivity时，只需要一行代码即可（从firActivity跳到SecActivity）：
  ```
  SecActivity.actionStart(firActivity.this,"data1","data2");
  ```
至此，Android的三个小技能就介绍完毕拉，希望对你有帮助哟! (～￣▽￣)～...
