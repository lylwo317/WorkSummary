# 解决死锁

## 原因
主线程阻塞，因为无法获取A锁。原因是另外两个子线程存在死锁，一个线程获取了A锁，而在等待B锁，另外一个线程在获取B锁，而在等待A锁。

## 解决方案
通过分析ANR日志来分析死锁在哪里产生的。最后一次的未响应的异常，都会被写在/data/anr目录的trace.txt文件里面。如果要找到之前的未响应异常，可以到/data/system/dropbox目录下查找。命名是data_app_anr@xxxxxxx.txt.gz，xxxx是时间戳。需要解压缩才能查看。

导出之后，里面就会有ANR时相应的线程信息

### 分析trace.txt

```
----- pid 1906 at 2016-08-30 23:22:08 -----
Cmd line: com.vyou.vcameraclient
Build fingerprint: 'htc/himauhl_htc_europe/htc_himauhl:6.0/MRA58K/671758.12:user/release-keys'
ABI: 'arm'
Build type: optimized
Zygote loaded classes=4196 post zygote classes=2616
Intern table: 61610 strong; 217 weak
JNI: CheckJNI is on; globals=660 (plus 9250 weak)
Libraries: /data/app/com.vyou.vcameraclient-4/lib/arm/libBaiduMapSDK_base_v4_0_0.so /data/app/com.vyou.vcameraclient-4/lib/arm/libBaiduMapSDK_map_v4_0_0.so /data/app/com.vyou.vcameraclient-4/lib/arm/liblocSDK6a.so /data/app/com.vyou.vcameraclient-4/lib/arm/libplayerjni.so /data/app/com.vyou.vcameraclient-4/lib/arm/libvvediojni.so /data/user/0/com.vyou.vcameraclient/files/libsmssdk.so.22 /system/lib/libandroid.so /system/lib/libcompiler_rt.so /system/lib/libhtcflag-jni.so /system/lib/libjavacrypto.so /system/lib/libjnigraphics.so /system/lib/libmedia_jni.so /system/lib/libwebviewchromium_loader.so libjavacore.so (14)
Heap: 12% free, 82MB/93MB; 299034 objects
Dumping cumulative Gc timings
Start Dumping histograms for 3 iterations for concurrent mark sweep


"main" prio=5 tid=1 Blocked
  | group="main" sCount=1 dsCount=0 obj=0x74e21ba8 self=0xf0c18d30
  | sysTid=1906 nice=0 cgrp=default sched=0/0 handle=0xf7120b34
  | state=S schedstat=( 0 0 0 ) utm=1506 stm=309 core=1 HZ=100
  | stack=0xff463000-0xff465000 stackSize=8MB
  | held mutexes=
  at com.vyou.app.sdk.bz.gpsmgr.service.TrackService.findTrackByTime(TrackService.java:735)
  - waiting to lock <0x070b3961> (a java.util.ArrayList) held by thread 19
  at com.vyou.app.ui.activity.LivePlayerActivity.onMsg(LivePlayerActivity.java:1215)
  at com.vyou.app.ui.player.VyLiveMediaCtrller.onPlayTimeChange(VyLiveMediaCtrller.java:1968)
  at com.vyou.app.ui.player.VMediaController.stopHeartbeatTimer(VMediaController.java:1167)
  at com.vyou.app.ui.player.VMediaController.setPlaytime(VMediaController.java:1176)
  at com.vyou.app.ui.activity.LivePlayerActivity.onStop(LivePlayerActivity.java:943)
  at android.app.Instrumentation.callActivityOnStop(Instrumentation.java:-1)
  at android.app.Activity.performStop(Activity.java:-1)
  at android.app.ActivityThread.performDestroyActivity(ActivityThread.java:-1)
  at android.app.ActivityThread.handleDestroyActivity(ActivityThread.java:-1)
  at android.app.ActivityThread.access$1500(ActivityThread.java:-1)
  at android.app.ActivityThread$H.handleMessage(ActivityThread.java:-1)
  at android.os.Handler.dispatchMessage(Handler.java:-1)
  at android.os.Looper.loop(Looper.java:-1)
  at android.app.ActivityThread.main(ActivityThread.java:-1)
  at java.lang.reflect.Method.invoke!(Native method)
  at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:-1)
  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:-1)
  at de.robv.android.xposed.XposedBridge.main(XposedBridge.java:117)
  
"timer_mailbox" prio=5 tid=19 Blocked
  | group="main" sCount=1 dsCount=0 obj=0x13a5d940 self=0xe680ad30
  | sysTid=12907 nice=0 cgrp=default sched=0/0 handle=0xd1f5e930
  | state=S schedstat=( 0 0 0 ) utm=21 stm=7 core=1 HZ=100
  | stack=0xd1e5c000-0xd1e5e000 stackSize=1038KB
  | held mutexes=
  at com.vyou.app.sdk.bz.gpsmgr.service.TrackDownMgr.startDownloadTrack(TrackDownMgr.java:117)
  - waiting to lock <0x01868ec3> (a java.lang.Object) held by thread 69
  at com.vyou.app.sdk.bz.gpsmgr.service.TrackService.updateTrackList(TrackService.java:464)
  - locked <0x070b3961> (a java.util.ArrayList)
  at com.vyou.app.sdk.bz.gpsmgr.handler.TrackMsgHandlerHelper.mailAddGsxFile(TrackMsgHandlerHelper.java:359)
  at com.vyou.app.sdk.bz.gpsmgr.handler.TrackMsgHandler.handleMailMsg(TrackMsgHandler.java:166)
  at com.vyou.app.sdk.api.mail.MailMsgHandler.handleMsgOut(MailMsgHandler.java:104)
  at com.vyou.app.sdk.api.mail.MailMsgHandler.handleSynMsg(MailMsgHandler.java:83)
  at com.vyou.app.sdk.api.RemoteOptor.synSendCtrlCmd(RemoteOptor.java:86)
  at com.vyou.app.sdk.api.RemoteOptor.synSendCtrlCmd(RemoteOptor.java:38)
  at com.vyou.app.ui.service.ShakeHandsService.receiveMailBox(ShakeHandsService.java:293)
  at com.vyou.app.ui.service.ShakeHandsService$3.run(ShakeHandsService.java:255)
  at java.util.Timer$TimerImpl.run(Timer.java:284)
  
"pool-4-thread-16" prio=5 tid=69 Blocked
  | group="main" sCount=1 dsCount=0 obj=0x12c2c760 self=0xd6354d30
  | sysTid=2323 nice=0 cgrp=default sched=0/0 handle=0xd4b75930
  | state=S schedstat=( 0 0 0 ) utm=9 stm=2 core=3 HZ=100
  | stack=0xd4a73000-0xd4a75000 stackSize=1038KB
  | held mutexes=
  at com.vyou.app.sdk.bz.gpsmgr.service.TrackDownMgr.startDownloadTrack(TrackDownMgr.java:120)
  - waiting to lock <0x070b3961> (a java.util.ArrayList) held by thread 19
  - locked <0x01868ec3> (a java.lang.Object)
  at com.vyou.app.sdk.bz.gpsmgr.service.TrackDownMgr$1.vrun(TrackDownMgr.java:162)
  at com.vyou.app.sdk.utils.VRunnable.run(VRunnable.java:25)
  at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:423)
  at java.util.concurrent.FutureTask.run(FutureTask.java:237)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1113)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:588)
  at java.lang.Thread.run(Thread.java:818)

```
上面只截取主要部分