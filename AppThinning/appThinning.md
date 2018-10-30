## App Thinning
据Apple官方文档的介绍，App Thinning主要有三个机制

## 1. Slicing

开发者把App安装包上传到AppStore后，Apple服务会自动对安装包切割为不同的应用变体(App variant)，当用户下载安装包时，系统会根据设备型号下载安装对应的单个应用变体。（你不需要做什么，iOS9.0.2以上就支持）
![](https://github.com/lupeihong/Wiki/blob/master/AppThinning/848953-20160509112307155-139525806.png)

 ## 2. Bitcode

开启Bitcode编译后，可以使得开发者上传App时只需上传Intermediate Representation(中间件)，为二进制数据表示的格式的中间码，而非最终的可执行二进制文件。 在用户下载App之前，AppStore会自动编译中间件，产生设备所需的执行文件供用户下载安装。也就是当我们提交程序到 App Store上时， Xcode 会将程序编译为一个中间表现形式( bitcode )。然后 App store 会再将这个 Bitcode 编译为可执行的64位或32位程序。苹果会根据下载应用的用户的手机指令集类型生成只有该指令集的二进制，进行下发
![](https://github.com/lupeihong/Wiki/blob/master/AppThinning/848953-20160509112343234-1404242667.jpg)

所以，通过这个方式，我们可以做到架构级别的App Slicing。

然而，一个很常见的误区是认为使用 bitcode 能优化包大小，其实启用 bitcode 作用并不大。实际上 bitcode 和包大小半毛钱关系都没有，它仅仅是把编译的最后一步留给苹果，这样苹果就可以在优化编译器后，再次将我们的应用打包，从而让历史应用也能享受到新技术
https://www.appcoda.com/app-thinning/
在文档里可看到
In fact, app slicing handles the majority of the app thinning process. ‘App Slicing’ feature finally switched on in iOS 9.0.2

**说明slicing才是主要处理 app thinning的而且该功能需要在iOS9.0.2以上才支持（iOS9.0中被关闭了，因为一个iCloud的bug）。实际上Bitcode，做的事情是指令集优化。根据你设备的状态去做编译优化，进而提升性能。所以Bitcode对包的大小优化起不到什么本质上的作用。**

这就好比饭店原来是把菜做好了，等顾客来了以后直接上菜。现在厨师长说：“大家买好原材料”，万一哪天我们有了新的菜谱，同样的原材料就能做出更好吃的菜，用户就经常光顾我们这里了

**注意点**
- 开启 Bitcode 编译后，编译产生的 .app 体积会变大(中间代码，不是用户下载的包)，且 .dSYM 文件不能用来崩溃日志的符号化(用户下载的包是 Apple 服务重新编译产生的，有产生新的符号文件) 
- 通过 Archive 方式上传 AppStore 的包，可以在Xcode的Organizer工具中下载对应安装包的新的dSYM符号文件。或者iTunes Connect上下载对应构建包的dSYM（需消除混淆）
详情查看：https://developer.apple.com/library/archive/technotes/tn2151/_index.html

**若开启Bitcode，对于手Y来讲，将面临的问题：**
- 意味着crash反馈系统将解不出符号
- 对体积的优化可能并没有预想的大
- 基本上各库都未支持bitcode，需要多方推动


## 3. On-Demand Resources

ORD(随需资源)是指开发者对资源添加标签上传后，系统会根据App运行的情况，动态下载并加载所需资源，而在存储空间不足时，自动删除这类资源。 
这可能在游戏中应用场景会多一些。你可以用 tag 来组织像图像或者声音这样的资源，比如把它们标记为 level1，level2 这样。然后一开始只需要下载 level1 的内容，在玩的过程中再去下载 level2。或者也可以通过这个来推后下载那些需要内购才能获得的资源文件。
![](https://github.com/lupeihong/Wiki/blob/master/AppThinning/848953-20160509112524062-1295651576.png)
看起来更像是按需加载网络图片，并作缓存处理。而On-Demand Resources只是将这个服务交由苹果来处理