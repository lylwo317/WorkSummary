# 修复数据库操作中的Cursor泄露

## 原因
友盟上有好几个崩溃错误是因为Cursor泄露导致

## 分析
内存泄露的原因很简单，就是在使用完相应的Cursor时，没有调用它的close方法，关闭资源

人眼来找是很不现实的，也是不可能完成的任务。所以需要一个能检测这种泄露的方法或工具。

## 解决方案
这里我使用了Android的一个叫做StrictMod（严格模式）的功能。在App中加入相应代码打开，然后使用App，触发数据库操作，当某个数据库操作没有调用close方法时，就会在手机或者终端上打印相应的日志，甚至可以设置成，使app进程关闭。这样每当检测到未close的Cursor操作，程序就会被强制关闭。

如此反复，就会收集到了有关错误代码方法调用栈的信息，之后修正相应的代码就好。

