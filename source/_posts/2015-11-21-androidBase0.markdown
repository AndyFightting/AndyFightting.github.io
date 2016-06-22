---
layout: post
title: "Android 基础知识"
date: 2015-11-21 09:26:15 +0800
comments: true
categories: Android
---
### * Activity启动模式

在AndroidMainfest.xml里的activity里设置android:launchMode = “xxx”

**standard:** 默认模式，每次都会创建一个新的放在栈顶，即使栈里已经有了
 	
**singleTop:** 已经在**栈顶**了就不创建，否则还是会创建，即使栈里已经有了

**singleTask:** 在活动栈中有了就不创建，直接推到栈顶，**在它之前的全部会被挤出栈**，若活动栈中没有就创建

**singleInstance:** 会创建一个**新的活动栈**把自己放进去

### * 随时随地退出程序

在任何页面都可以被踢下线的实现技巧

创建一个收集Activity的管理类，在BaseActivity的`onCreate()`方法中添加`ActivityManager.addActivity(this)`,在`onDestroy`方法中添加`ActivityManager.removeActivity(this)`。然后在任何要强退的地方调用`ActivityManager.finishAllActivity()`。<!--more-->

	public class ActivityManager {

    public static List<Activity> activityList = new ArrayList<>();

    public static void addActivity(Activity activity){
        activityList.add(activity);
      }

    public static void removeActivity(Activity activity){
        activityList.remove(activity);
      }
      
    public static void finishAllActivity(){
        for (Activity activity : activityList){
            if (!activity.isFinishing()){
                activity.finish();
            }
        }
        activityList.clear();
      }
	}
### * Activity跳转

一般我们是用如下的方式跳转，需要知道下一个Activity要用哪些参数，多人开发可能有些不便。

	Intent intent = new Intent(OneActivity.thi,TwoActivity.class);
	intent.putExtra("key","value");
	startActivity(intent);
	
可以在下一个Activity中把需要的参数暴露出来。

	public static void actionStart(Activity activity,String param){
	  Intent intent = new Intent(activity,TwoActivity.class);
	  intent.putExtra("key","param");
	  activity.startActivity(intent);
	  activity.finish();
	}

### * ListView的正确使用方式

实现了item的复用，这样效率最好，ArrayAdapter源码中就是这样弄的。ArrayAdapter把BaseAdapter包装了一遍，实现了`getCount`,`getItem`,`getItemId`，所以这些我们就可以不用再写了，只重写个`getView`就可以。

	public class UserAdapter extends ArrayAdapter<User> {

    private int layoutId;
    public UserAdapter(Context context, int resourceId, List<User> objects){
          super(context,resourceId,objects);
          layoutId = resourceId;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent){

        User user = getItem(position);
        ViewHolder viewHolder;
        View layoutView;

        if (convertView == null){
            layoutView = LayoutInflater.from(getContext()).inflate(layoutId,null);
            viewHolder = new ViewHolder();
            viewHolder.headImage = (ImageView)layoutView.findViewById(R.id.head_img);
            viewHolder.nameTv = (TextView)layoutView.findViewById(R.id.name_tv);
            layoutView.setTag(viewHolder);
        }else {
           layoutView = convertView;
           viewHolder = (ViewHolder)layoutView.getTag();
        }
        //--------在下面赋值 ----------------
        viewHolder.headImage.setImageResource(user.getHeadImgId());
        viewHolder.nameTv.setText(user.getName());

        return layoutView;
    }

    class ViewHolder{
        ImageView headImage;
        TextView nameTv;
      }
	}
	
### * px,dpi,dp,density

Android规定：如果手机一英寸长度上有160个px，那么手机的dpi就是160,此时1dp==1px，即density=1。所以如果一英寸长度上有320个px的话，那么手机的dpi就是320，此时1dp==2px,即density=2。

结论：即相同大小的手机上的dp个数是不变的，如果相同大小的手机像素不同的话，只会影响dip和density的值,所以我们适配手机的时候用的长度单位要用 ---- `dp`

	public static int getScreenWidthPx(Context context){
        return  context.getResources().getDisplayMetrics().widthPixels;
    }

    public static int getScreenHeightPx(Context context){
        return  context.getResources().getDisplayMetrics().heightPixels;
    }

    public static float getXdpi(Context context){
        return  context.getResources().getDisplayMetrics().xdpi;
    }

    public static float getYdpi(Context context){
        return  context.getResources().getDisplayMetrics().ydpi;
    }

    public static float getDensity(Context context){
       return context.getResources().getDisplayMetrics().density;
    }
    
### * 9-patch图片

1，在`上`边绘制的`垂直区域`会被`水平拉伸`，在`左`边绘制的`横向区域`会被`垂直拉伸`。

2，在`下`边和`右`边绘制的`交叉区域`是内容放置的区域。

![img](/myimg/android/9patch.jpg)

### * Fragment
在fragment里可以用`getActivity()`得到与之关联的Activity。`FragmentTransaction`的实例可以调用`addToBackStack(null)`方法把这个事务添加到返回栈中，这样点击返回就是回到事务处理之前的状态,而不是退出Activity。fragment之间的通信可以通过与之共同关联的Activity来转达。

fragment的生命周期与Activity类似，只不过比Activity多了`onAttach()`,`onCreateView()`,`onActivityCreated()`,`onDestroyView()`,`onDetach()`。

由生到死依次是：`onAttach()`, `onCreate()`,`onCreateView()`,`onActivityCreated()`,`onStart()`,`onResume()`,`onPause()`,`onStop()`,`onDestroyView()`,`onDestroy()`,`onDetach()`。

从返回栈回来后执行的第一个方法是`onActivityCreated()`,而Activity的轮回后执行的第一个方法是`onRestart()`,然后再`onStart()`,`onResume()`。

同一个Activity怎样让它在不同大小的设备自动加载不同的布局文件呢？用`限定符`,即在`res`下建`layout-large`文件夹或者不同限定符的文件夹，在里面放`同名`的布局文件。限定符有：`small`,`normal`,`large`,`xlarge`以及分辨率限定符：`ldpi`:120dpi以下，`mdpi`:120-160dpi，`hdpi`:160-240dpi,`xhdpi`:240-320dpi，以及方向限定符：`land`:横屏,`port`:竖屏。

还有个问题，`large`，大，到底多大算大呢？可以自己定义这个边界值。如`layout-sm600`,就是`宽度`大于600`dp`的设备叫大，否则叫小。`sm`的意思是：`smallest width`。

### * Broadcast 广播

见名知意，即发一下消息，很多地方都可以收到。广播有两种：异步执行的广播 和 按顺序执行的同步广播。异步广播接收没有先后之分，可看做同时收到，这种广播发出之后不可被拦截。同步广播是按顺序一个一个执行，可以设置接收的优先级(设置`IntentFilter`的`priority`值,越大越优先)，这种广播可以被拦截(在`onReceive()`方法里调用`abortBroadcast()`废掉这条广播)。

注册接收广播有两种方式：一种是用代码注册，也叫动态注册，动态注册(在`onCreate()`里调用Activity的方法`registerReceiver()`)的广播接收一定要在`onDestroy`中移除注册(调用Activity的`unregisterReceiver()`)。另一种叫静态注册，是在xml中配置的，这种注册不用移除。

怎么写广播接收器？写个类继承`BroadcastReceiver`,它是个抽象类，然后重写`onReceive()`即可，可以通过`intent.getAction()`来区分不同广播，**在`onReceive()`方法中不能开启线程**。下面是一个监听网络变化的广播接收器。
	
	class NetWorkChangeReceiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context,Intent intent){
            ConnectivityManager connectivityManager = (ConnectivityManager)getSystemService(Context.CONNECTIVITY_SERVICE);
            NetworkInfo networkInfo = connectivityManager.getActiveNetworkInfo();
            if (networkInfo != null && networkInfo.isAvailable()){
                Toast.makeText(context,"网络可用",Toast.LENGTH_SHORT).show();
            }else {
                Toast.makeText(context,"网络不可用",Toast.LENGTH_SHORT).show();
            }
        }
    }

用Activity里的方法发送异步广播`sendBroadcaset(intent)`,和发送同步广播`sendOrderedBroadCaset(intent,null)`。这样发送的广播都是`跨应用`的，即A应用发出的广播，B应用也可以收到的！应为这些都是全局广播。

本地广播，即发出的广播只会被`本身应用`接收到,手机里的其他应用就收不到了。本地广播要用`LocalBroadcastManager`这个类来管理，即注册和移除的时候不能用`Activity`的`registerReceiver()`和`unregisterReceiver()`方法，而应该用`LocalBroadcastManager`实例的`registerReceiver()`和`unregisterReceiver()`方法，简单吧！**本地广播不会被静态注册的接收器收到！**要接收本地广播，接收器都得用动态注册的方式。

### * ContentProvider / ContentResolver

这个是用于在手机中`不同应用程序`间的数据传递，可以自己选择那些数据可暴露出来给其他应用获取。有些系统的应用程序已经提供这样的接口了，可以直接通过这种方式获取数据，如 短信应用，通讯录应用等，而我们自己写的应用程序则要自己写这个数据提供接口。

获取数据通过`ContentResolver`类，可以通过Context 的 `getContentResolver()`得到实例，然后数据处理就是 `insert()`,`update()`,`delete()`,`quert()`,和`SQLiteDatabase`很像，但不是传`表名`而是传`Uri`。

提供数据，要继承抽象类`ContentProvider`然后重写里面的六个方法：`onCreate()`,`quert()`,`insert()`,`update()`,`delete()`,`getType()`。

### * NotificationManager

Notification 在手机顶部提示用户的通知，











	






