# Flutter_Boost 框架

Flutter_Boost是基于单引擎的Flutter混合开发框架

## Flutter基本原理

创建一个单引擎纯Flutter项目时，AS编辑器默认创建了一个FlutterActivity作为MainActivity，并且整个Flutter APP中
只有一个Activity。
	
FlutterActivity内部默认创建了一个继承自FrameLayout的FlutterView。FlutterView 就是Android原生层
提供给Flutter的一个绘制容器。也就是说Flutter页面内容最终是显示在Android原生层的FlutterView上面的
	

## Android 端的机制

FlutterBoostActivity 继承自 FlutterActivity，通过重写一些方法来处理 FlutterView和Flutter Engine
绑定和解绑逻辑，来支持多Activity的flutter混合开发。
   
FlutterBoost把FlutterBoostActivity抽象成了一个FlutterViewContainer容器，映射到Flutter端也有一个BoostContainer容器。一个BoostContainer容器
对应一个外部路由，里面可以包含多个Page，BoostContainer容器里面的Page管理是通过其内部路由来管理的。所以Flutter_Boost框架的Flutter端是基于外部路由+内部路由，
双路由管理Page
   
1.在onCreate中，创建一个FlutterViewContainer容器并且保存到map中，便于后面通过管理FlutterViewContainer容器来通知Flutter端渲染页面
以及处理生命周期回调事件
   
   
2.重写onResum方法，在该方法中，先解绑上一个FlutterView和FlutterEngine的关系，然后绑定当前Activity的FlutterView的FlutterEngine。
	
这么做的目的是为了在Activity切换时能做到FlutterView和FlutterEngine无缝绑定，也就是任何时刻都有一个FlutterView和FlutterEngine绑定
	
处理完FlutterView和FlutterEngine关系之后，通知Flutter端，push一个新的Flutter页面，同时通知Flutter端发生页面生命周期改变事件
	
绑定过程主要做两件事
-  FlutterEngine相关的插件绑定到当前Activity：确保Plugin通信正常
	
-  当前Activity的FlutterView绑定到FlutterEngine上：确保Flutter页面能正常渲染
	
3.在onPause中，通知Flutter端发生页面生命周期改变事件
	
4.在onBackPressed中处理back返回事件，通知Flutter端 pop页面
	
4.在onDestroy中，通知Flutter端，移除Flutter端的BoostContainer容器，同时移除Android端的FlutterViewContainer容器
	
## Flutter 端的机制

在Flutter端，FlutterBoost抽象了一个BoostContainer容器，来对应Navite端的FlutterBoostActivity。
	
一个Flutter App 中有一个FlutterBoostApp，继承自StatefulWidget，用于管理多个BoostContainer。一个FlutterBoostApp包含一个Navigator，用于管理外部路由
	
每个BoostContainer容器包含一个Navigator，每个BoostContainer可以有多个Flutter Page，通过内部的Navigator管理多个内部Flutter Page，称内部路由

所以FlutterBoost在Flutter侧是基于双层路由管理。

贴一张架构图
- ![avatar](https://pic3.zhimg.com/80/v2-cec9a709ee743c3abd1a2b7949b24a7a_720w.jpg)
	
	
## push过程

BoostNavigator#push方法中有一个参数withContainer，
	
-  withContainer=true，表示push一个Flutter Page会在Native侧创建一个FlutterViewContainer容器，
Native侧会通知Flutter侧创建一个BoostContainer容器来装载Flutter Page。此时Flutter Page是由外部路由管理
	
-  withContainer=false，表示push一个Flutter Page不会在Native侧创建一个容器，Flutter侧也不会创建BoostContainer容器，此时路由是由内部路由管理

![avatar](https://github.com/xujinping/blog/blob/main/ac5wDF9Z6d.png)

## pop过程
- Flutter Navigator 尝试pop成功，则说明当前页面是一个纯Flutter Page
- Flutter Navigator 尝试pop失败，则说明当前页面是一个FlutterBoostActivity容器包含的 Flutter Page, 需要通知Native侧 finish FlutterBoostActivity。在finish的时候设置Flutter侧的pop结果参数，然后由FlutterBoostPlugin设置监听，在addActivityResultListener中拿到结果之后通知Flutter侧。在FlutterBoostActivity的onDestroy回调方法通知Flutter侧remove Page同时刷新Flutter页面

流程图如下：
![avatar](https://github.com/xujinping/blog/blob/main/vtmI4bdmEa.png)
	
