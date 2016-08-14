# 友盟上App频繁崩溃bug

6月份的时候，安卓端崩溃率较高，排前5个的崩溃没个都基本达到了400左右的错误次数。而这些错误其实都是由一个程序中的Bug间接造成的。出现的概率很大，用户使用体验很不好。

经过分析主要是由一个错误造成。在所有Activity共同父类的onCreate方法中，会判断启动页所作的初始化时候还有效完成（主要防止App后台被杀死后，用户从最近使用列表打开应用时，而此时app是直接恢复最后一个Activity，所以启动页的初始化流程没有走）。


## 错误代码

AbsActionbarActivity.java

```  java
	@Override
	protected void onCreate(Bundle savedInstanceState)
	{
		// 如果程序异常后，某些系统会自动加载上次的activity，让其调转到welcome
		if ((!VApplication.getApplication().isInited || !AppLib.getInstance().isInit())
				&& !(this instanceof SplashActivity))
		{
			VLog.v(TAG, "data reinit.");
			VApplication.getApplication().isInited = false;
			Intent intent = new Intent(this, SplashActivity.class);//跳转到启动页面，重新执行初始化。
			intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
			startActivity(intent);

			isNeedCreate = false;
			finish();//这里并不会马上结束Activity，即使return，也无法阻止子类的onCreate方法继续执行。
		}
        
		super.onCreate(savedInstanceState);
	}
    ```
 MainActivity.java   
 ``` java
 public MainActivity extend AbsActionbarActivity
 {
 
 ......
 
 @Override
	protected void onCreate(Bundle savedInstanceState)
	{
		super.onCreate(savedInstanceState);
        
        VCacheUtil.cacheObject.getSerialObjList....//这里的cacheObject是在splashActivity中初始化的
	}
    
    ......
    
 }
 ```
    
   上面是所有Activity父类的部分代码， 子类Activity会覆盖的父类的onCreate方法。
   
   父类这里有个逻辑是判断是否初始化成功（程序某些全局对象初始化操作是在SplashActivity(启动页）中完成的，例如`VCacheUtil.cacheObject`）。即使父类的onCreate方法提前结束执行，但是这里也无法停止子类的代码继续执行，所以造成了很多空指针异常