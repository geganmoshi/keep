# keep
#### 安卓监控端软件，用来监控所需的app永远在最前端运行
#### 在需要运行在最前端的app中添加service不停发送自身状态的广播，然后keep会一直监听

```html
ActivityManager activityManager = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);
List<ActivityManager.RunningAppProcessInfo> infoList;
infoList = activityManager.getRunningAppProcesses();
ComponentName cn = activityManager.getRunningTasks(1).get(0).topActivity;
bs_front = cn.getClassName().contains("com.xxx.application");
for (ActivityManager.RunningAppProcessInfo info : infoList) {
    if (info.processName.contains("com.xxx.application")) {
        bs = true;
    }
}
if (!bs || !bs_front){
          System.out.println("keep need excute");
          Intent intent = new Intent("com.heyiweilai.keepservice_excute");
          sendBroadcast(intent);
      }else {
          System.out.println("keep dont need excute");
          Intent intent = new Intent("com.heyiweilai.keepservice_dont_excute");
          sendBroadcast(intent);
}

```
#### 以上方法可以判断当前app是否正在运行并且是否运行在设备最前端，如果是则发送不需要执行命令，否则发送需要执行命令

#### keep关键代码
```html
//开启handler持续接收命令
 @Override
public int onStartCommand(Intent intent, int flags, int startId) {
    handler.postDelayed(task,10 * 1000);
    handler.postDelayed(task_reload,1 * 1000);
    return super.onStartCommand(intent, flags, startId);
}

//如果app正在更新，keep会连续执行20次启动命令，否则判断是否需要执行启动，count为记录启动次数，app未更新状态下会执行20次，否则10次，每10分钟将count归0，防止app出现奔溃前端一直闪退情况，可以在超过启动次数后添加上报app奔溃状态方法
 if (is_update){
            count += 1;
            if (count > 20) {
                return;
            }
            List<String> commnandList = new ArrayList<String>();
            commnandList.add("am start -n com.heyiweilai.applicationtv/com.heyiweilai.applicationtv.LoadingActivity");
            ShellUtils.CommandResult rs = ShellUtils.execCommand(commnandList, true);
        }else {
            if (need_excute) {
                count += 1;
                if (count > 10) {
                    return;
                }
                List<String> commnandList = new ArrayList<String>();
                commnandList.add("am start -n com.heyiweilai.applicationtv/com.heyiweilai.applicationtv.LoadingActivity");
                ShellUtils.CommandResult rs = ShellUtils.execCommand(commnandList, true);
            }          
}
```
