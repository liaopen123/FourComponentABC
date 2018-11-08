# FourComponentABC
Android Base Component APP
## Service
长期在后台运行，没有界面，简单的理解就是：一个没有界面长期在后台运行的Service
### 生命周期
>onCreate()
>>onStartCommand()
>>>onDestory()

如果服务已经开启(startService(intent))，就不会再调用onCreate()方法，**服务只会被创建一次。**
#### 使用
开启服务：
```java
startService(this,MyService.class);
```
关闭服务：
```java
stopService();
```
##### 服务开启的2种方法：
1. startService():直接开启服务，服务一旦启动，就跟调用者(开启者没有关系)。
>调用者的activity退出，服务还是会继续活的好好的，调用者activity，没发访问服务里的方法。

2.bindService();绑定服务，服务和开启者(调用者)有密切的关系。
>不求同时生，但求同时死，只要activity挂了，服务就跟着挂了，调用者activity，可以调用服务里的方法。

##### 用绑定的方法开启服务，调用服务方法的流程(重要)
生命周期：
>onCreate()
>>onBind()
>>>onDestory()
不会调用 onStart()和onStartCommand()的方法

### IntentService和Service的区别
IntentService是处理**异步请求的一个类**。
启动方式和Service的启动相同。
不同的是当任务执行完后，IntentService会自动停止，走onDestoryed().
IntentService可以启动多次。执行完第一个接着再执行第二个。以此类推
每一个耗时操作会以工作队列的方式在IntentService的**onHandleIntent**回调方法中执行。
代码不同处：
```java
   public MIntentService(){
        super("MIntentService");
    }

    /*自动生成有参构造 在无参数构造中调用该方法 传入的name 在app设置中显示。
     */
    public MIntentService(String name) {
        super(name);
    }
   @Override
    protected void onHandleIntent(Intent intent) {
        Log.e("MIntentService--", Thread.currentThread().getName() + "--" + intent.getStringExtra("info") );
        for(int i = 0; i < 100; i++){ //耗时操作
            Log.i("onHandleIntent--",  i + "--" + Thread.currentThread().getName());
        }
    }

```

### 绑定Service(bindService)
#### 生命周期
>onCreate()
>>onBind()
>>>onUnbind()
>>>onRebind()
>>>>onDestory()

startService和bindService的区别:
区别:startService是调用者退出  服务仍然存在  调用stopService方可.调用stopService后,会自动调用OnDestroy方法
而bindService方法是  调用者退出 服务自动退出


## ContentProvider内容提供者
### 作用
进程间进行**数据交换&共享**即跨进程间通信。
为存储和读取数据提供统一接口。android内置许多数据都是使用ContentProvider的形式，供开发者调用的(视频，音频，图片，通讯录)。
调用关系：
![d97481283d64538effc724ac31c59b30.png](evernotecid://AD864F29-6AE2-4F05-B617-3946CD33D099/appyinxiangcom/709762/ENNote/p1719?hash=d97481283d64538effc724ac31c59b30)
一句话总结这个图就是：
调用者不能直接通过ContentProvider去调用接口函数，需要通过ContentResolver，通过URI间接调用ContentProvider。

## 广播接收者
### 作用
用于组件与组件之间进行同行。全局监听器。用于监听APP发出的广播消息，并作出响应。
### 实现原理
观察者模式
模型中3个角色：
1. 消息订阅者(广播接收者)
2. 消息发布者(广播发布者)
3. 消息中心(Activity Message Center)

### 代码实现 
#### 注册方式
广播接收者可以静态注册 也可以动态注册
##### 静态注册

```xml
<receiver 
    android:enabled=["true" | "false"]
//此broadcastReceiver能否接收其他App的发出的广播
//默认值是由receiver中有无intent-filter决定的：如果有intent-filter，默认值为true，否则为false
    android:exported=["true" | "false"]
    android:icon="drawable resource"
    android:label="string resource"
//继承BroadcastReceiver子类的类名
    android:name=".mBroadcastReceiver"
//具有相应权限的广播发送者发送的广播才能被此BroadcastReceiver所接收；
    android:permission="string"
//BroadcastReceiver运行所处的进程
//默认为app的进程，可以指定独立的进程
//注：Android四大基本组件都可以通过此属性指定自己的独立进程
    android:process="string" >

//用于指定此广播接收器将接收的广播类型
//本示例中给出的是用于接收网络状态改变时发出的广播
 <intent-filter>
<action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
    </intent-filter>
</receiver>
```

##### 动态注册
注册方式：在代码中调用Context.registerReceiver（）方法
```java
// 1. 实例化BroadcastReceiver子类 &  IntentFilter
     mBroadcastReceiver mBroadcastReceiver = new mBroadcastReceiver();
     IntentFilter intentFilter = new IntentFilter();
    // 2. 设置接收广播的类型
    intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);
    // 3. 动态注册：调用Context的registerReceiver（）方法
     registerReceiver(mBroadcastReceiver, intentFilter);

//销毁
    unregisterReceiver(mBroadcastReceiver);

```
**特别注意：** 动态广播最好在Activity 的 onResume()注册、onPause()注销。
### 广播分类(发广播)：
1. 普通广播
2. 系统广播
3. 有序广播
4. 粘性广播
5. App引用内广播

#### 普通广播：
开发者自己定义的广播最为常用。
```java
Intent intent = new Intent();
//对应BroadcastReceiver中intentFilter的action
intent.setAction(BROADCAST_ACTION);
//发送广播
sendBroadcast(intent);
```
如果注册的广播和上述的IntentFilter action匹配，则会回调onReceive()方法。

#### 系统广播
Android中内置了多个系统广播：只要涉及到手机的基本操作（如开机、网络状态变化、拍照等等），都会发出相应的广播

####  有序广播
发送出去的广播被广播接收者按照先后顺序接收。
按照Priority属性值从大--小排序
如果Priority相同，动态注册的广播优先。
**特点**
1. 优先接收到的广播可以对广播进行**拦截**(abortBroadcast();//广播被拦截终止了。)。
2. 优先接收到的广播可以对广播进行**修改**(setResultData();//修改广播数据)。
于广播的发送方式：
```java
sendOrderedBroadcast(intent);
```
