# 加入write_setting权限检测和获取
## 现象
友盟错误报告

``` java
java.lang.RuntimeException: Unable to pause activity {com.vyou.vcameraclient/com.vyou.app.ui.activity.MainActivity}: java.lang.SecurityException: com.vyou.vcameraclient was not granted  this permission: android.permission.WRITE_SETTINGS.
	at android.app.ActivityThread.performPauseActivity(ActivityThread.java:3405)
```
在6.0.1的一些手机上，没有开启改权限。这里主要做的是检测并提醒用户开启

## 解决
加入判断逻辑，并提醒和跳转到相应的页面，让用户开启该权限

```java

@Override
    protected void onPause()
    {
        VLog.v(TAG, "onPause");
        super.onPause();

        if (GlobalConfig.appMode != GlobalConfig.APP_MODE.Custom_roadeyes && GlobalConfig.isSupportUMeng())
        {
            MobclickAgent.onPause(this);
        }
        // 如果系统没有打开，则app打开了，此处关闭，
        if (!oldGrivateSwitch && isAutoGrivate)
        {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                if (!Settings.System.canWrite(this)) {
                    if (!confirmDlg.isShowing() && !isInOpenWriteSettingPermission) {
                        showSystemWriteSettingDialog(UiConst.REQUEST_CODE_WRITE_SETTING_TO_CLOSE_ROTATION);
                    }
                }else{
                    Settings.System.putInt(getContentResolver(), Settings.System.ACCELEROMETER_ROTATION, 0);
                }
            } else {
                Settings.System.putInt(getContentResolver(), Settings.System.ACCELEROMETER_ROTATION, 0);
            }
        }

    }
    
private void showSystemWriteSettingDialog(final int requestCode) {
        isInOpenWriteSettingPermission = true;
        confirmDlg.setConfirmListener(new View.OnClickListener()
        {
            public void onClick(View v)
            {
                // 消掉弹框
                confirmDlg.dismiss();
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                    Intent intent = new Intent(Settings.ACTION_MANAGE_WRITE_SETTINGS,
                            Uri.parse("package:" + getPackageName()));
                    intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP|Intent.FLAG_ACTIVITY_SINGLE_TOP);
                    startActivityForResult(intent, requestCode);
                }
            }
        });
        confirmDlg.show();
    }
    
@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        isInOpenWriteSettingPermission = false;
        switch (requestCode) {
            case UiConst.REQUEST_CODE_WRITE_SETTING_TO_CLOSE_ROTATION:
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                    if (Settings.System.canWrite(this)) {
                        Settings.System.putInt(getContentResolver(), Settings.System.ACCELEROMETER_ROTATION, 0);
                    }
                }
                break;
            case UiConst.REQUEST_CODE_WRITE_SETTING_TO_OPEN_ROTATION:
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                    if (Settings.System.canWrite(this)) {
                        Settings.System.putInt(getContentResolver(), Settings.System.ACCELEROMETER_ROTATION, 1);
                    }
                }
                break;
        }
    }

```