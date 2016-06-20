###iOS现有项目集成React Native
----

####前提
已经配置好React Native命令行环境

####通过CocoaPods集成React Native
参考官方文档：
http://reactnative.cn/docs/0.27/embedded-app-ios.html#content

使用CocoaPods集成会出现很多问题，下面重点讲一下如何在原有项目中手工快速集成React Native。

#### 手工集成React Native

##### 第一步：初始化React Native环境

在我们要集成的项目中，进入到`*.xcodeproj`文件的上级目录，运行React Native初始化命令`react-native init [Project Name]`
会出现`prompt`, 输入`yes`,这样会在`ios`目录下生成一个同名工程。`init`过程会需要一点时间，耐心等待。

![](http://ww3.sinaimg.cn/large/8bd0decfgw1f51fs6b9vyj20n708j0vy.jpg)

完成后项目文件目录会变成这样：

![](http://ww1.sinaimg.cn/large/8bd0decfgw1f51fsxdrffj205t07oaai.jpg)

* `node_modules`:react native依赖包
* `ios`:iOS项目相关代码，Xcode工程文件
* `android`:Android项目相关代码
* `index.ios.js`:iOS平台加载的JS脚本
* `index.android.js`：Android平台加载的JS脚本
* `package.json`：当前项目的npm package的配置文件，内容如下：
```
{
  "private": true,
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "name": "Project",
  "dependencies": {
    "react": "^15.1.0",
    "react-native": "^0.27.2"
  },
  "version": "0.0.1"
}
```
接下来，我们可以先把Android相关的文件给干掉：
* `android`
* `index.android.js`
* `node_modules/react-native/android`
* `node_modules/react-native/ReactAndroid`

##### 第二步：添加React Native的Libraries

打开`ios`目录下的那个同名工程，可以看到这个工程引用的React Native库如下：

![](http://ww4.sinaimg.cn/large/8bd0decfgw1f4uk13ff3ij207y069js2.jpg)

打开我们原有的项目，右键工程目录，选择`New Group`,新建一个名为`Libraries`的目录

然后把前面项目`Libraries`下的`*.xcodeproj`文件全部拖到这个项目的`Libraries`下，效果跟上图一样。

##### 第三步：添加React Native shell脚本

React Native的iOS项目在编译时会先运行一个shell脚本，其作用有两个：
* 从Terminal启动了一个Node.js的server用于React Native调试
* 将React Native的资源文件打包放在编译目录下

具体添加方式：
* 选择`TARGETS`的`Build Phases`界面,点击左上角的`+`号
![](http://ww2.sinaimg.cn/large/8bd0decfgw1f51g3d62raj20p707cq45.jpg)
* 选择`New Run Script Phase`添加一个脚本，并命名为`Bundle React Native code and images`
* 把`ios`下工程中的脚本引用代码复制过来，如下图。注意路径，因为我们的原有的`*.xcodeproj`文件跟`node_modules`是在同一个目录下的，所以我们要去掉两个点中的一个点。
![](http://ww3.sinaimg.cn/large/8bd0decfgw1f4ukyiv2xfj20j2059jse.jpg)

##### 第四步：链接.a文件和添加搜索头文件的地址
到这一步，我们就可以干掉`ios`目录了，然后关闭原有的工程，再重新打开。
在工程的`Build Phases`界面的` Link Binary With Libraries`,点击
最下面的`+`号，可以看到在`Workspace`下多了很多`.a`文件，如下图，把它们全部添加进去。
![](http://ww4.sinaimg.cn/large/8bd0decfgw1f4ulgll4i7j20dp08dt9o.jpg)
* 接下来还需要添加`React`的搜索头文件，在`TARGETS->Build Settings->Header Search Paths`中添加一条`$(SRCROOT)/node_modules/react-native/React`,选择`recursive`,如下图所示。

![](http://ww3.sinaimg.cn/large/8bd0decfgw1f51g6722ebj20r807v0uf.jpg)

##### 第五步：编译

我们先编译一下，会发现编译不通过,一堆报错。
原因是我们还没有导入`JavaScriptCore`库。因为React Native与原生的交互需要依赖于`JavaScriptCore.framework`。
再次编译，发现可以编译成功了。

#### 测试运行React Native界面

在`AppDelegate.m`中的`didFinishLaunchingWithOptions:`方法中添加如下代码：

```
RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                      moduleName:@"Project"
                                               initialProperties:nil
                                                   launchOptions:launchOptions];
  rootView.backgroundColor = [[UIColor alloc] initWithRed:1.0f green:1.0f blue:1.0f alpha:1];

  self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
  UIViewController *rootViewController = [UIViewController new];
  rootViewController.view = rootView;
  self.window.rootViewController = rootViewController;
  [self.window makeKeyAndVisible];
```
`CMD+R`运行，会看到如下界面：

![](http://ww2.sinaimg.cn/large/8bd0decfgw1f4us1odirnj207y0eh0sx.jpg)

至此，原有项目集成React Native大功告成。祝在React Native的世界中玩得开心！

