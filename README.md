# PPSPrivateStaticLibrary
用于演示创建静态库的例子，通过cocoapod集成，不公开源码，只是暴露一些头文件

pod环境我就不讲怎么集成了，如果还没集成的同学，请移步

http://ppsheep.com/2015/11/20/CocoaPods的安装/

### pod lib create

这里我们通过pod官方提供的模板来创建，我目前也是这样创建的，当然xcode也提供了创建静态库的方式，这个我就不讲了，感兴趣的同学可以Google一下

首先进入到我们想要存放工程的地方 通过命令：

```
pod lib create PPSPrivateStaticLibrary
```

这里我将我的静态库取名为**PPSPrivateStaticLibrary**

![](http://o8bxt3lx0.bkt.clouddn.com/ppsheep/mfycu.jpg)

这里开始创建的时候我们会填几个问题：

> 1. What language do you want to use?? [ Swift / ObjC ] //选择你要创建的静态库的语言
>
> 2. Would you like to include a demo application with your library? [ Yes / No ] //是否需要创建一个demo用来测试你的静态库
> 
> 3. Which testing frameworks will you use? [ Specta / Kiwi / None ] //使用哪种测试框架来进行测试
> 
> 4. Would you like to do view based testing? [ Yes / No ] //是否要做基础的视图测试
> 
> 5. What is your class prefix? //文件前缀
> 

下面是我选择的

![](http://o8bxt3lx0.bkt.clouddn.com/ppsheep/t5pg5.jpg)

创建完成过后，我们的工程会自动打开，创建完成后，工程的目录如下

![](http://o8bxt3lx0.bkt.clouddn.com/ppsheep/adgaj.jpg)

接下来我们主要编辑的是podspec文件，首先我们来看一下，我编写的SDK中的测试文件结构

![](http://o8bxt3lx0.bkt.clouddn.com/ppsheep/oqz3e.jpg)

我想要公开上面文件中的PPSPublic1和PPSPublic2的头文件，其他的全部不公开出来，那我们怎样编写podspec文件呢

```
Pod::Spec.new do |s|
  s.name             = 'PPSPrivateStaticLibrary'
  s.version          = '0.1.0'

#这里加上你的工程简介
  s.summary          = 'This is ppsheep‘s test'

#这里加上你的工程简介
  s.description      = <<-DESC
这是我的一个测试工程，用来演示怎样创建一个源码不公开的静态库
                       DESC

#项目主页，这里可以放上你的静态库的介绍网页
  s.homepage         = 'https://github.com/yangqian111/PPSPrivateStaticLibrary'

  s.license          = { :type => 'MIT', :file => 'LICENSE' }

  s.author           = { 'ppsheep' => 'ppsheep.qian@gmail.com' }

  s.source           = { :git => '/Users/yangqian/Desktop/demo/PPSPrivateStaticLibrary', :tag => s.version.to_s }

#最低iOS系统要求
  s.ios.deployment_target = '8.0'

#是否需要项目支持ARC
  s.requires_arc = true

#这个地方注意下，你在工程中要用到的framework,都需要在这里声明，我只是这里列举了几个
s.frameworks = 'SystemConfiguration', 'MobileCoreServices', 'CoreGraphics', 'Security', 'CoreTelephony'

#在项目中我们还会用到一些library，也需要在这里声明，比如sqllite等tbd结尾的
s.libraries = 'resolv'

#这里声明的存放源文件的地址，就是我们实际写的代码
  s.source_files = 'PPSPrivateStaticLibrary/Classes/**/*'

#这里可以用来存放你的资源文件，图片，证书等等
  # s.resource_bundles = {
  #   'PPSPrivateStaticLibrary' => ['PPSPrivateStaticLibrary/Assets/*.png']
  # }

#这里声明你需要公开的文件, 有几种声明方式，通配符也支持的，在这里我可以使用通配符PPSPrivateStaticLibrary/Classes/Public/*.h

  s.public_header_files = 'PPSPrivateStaticLibrary/Classes/Public/*.h'
#也可以一个一个写出来[]
#s.public_header_files = ['PPSPrivateStaticLibrary/Classes/Public/PPSPublic1.h',
#                          'PPSPrivateStaticLibrary/Classes/Public/PPSPublic2.h']

#这里可以声明你的静态库依赖的其他静态库
  # s.dependency 'AFNetworking', '~> 2.3'
end

```

上面的都写好了，接下来，我们测试一下，首先进入到刚才目录中创建的example文件夹install一下工程

然后，我们就可以在例子中引入两个公开的文件

```objc

#import "PPSAppDelegate.h"
#import <PPSPrivateStaticLibrary/PPSPublic1.h>
#import <PPSPrivateStaticLibrary/PPSPublic2.h>

@implementation PPSAppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    PPSPublic1 *p1 = [[PPSPublic1 alloc] init];
    [p1 public1];
    PPSPublic2 *p2 = [[PPSPublic2 alloc] init];
    [p2 public2];
    return YES;
}

```

能够正常调用，我们的例子程序已经能够正常调用SDK，然后我们还需要检查一下我们的pod库是否符合pod规则，也是通过命令行来检测，进入到我们的工程目录,使用命令：

```
pod lib lint
```

![](http://o8bxt3lx0.bkt.clouddn.com/ppsheep/ku2wn.jpg)

可能会遇到这个问题

```
[!] The validator for Swift projects uses Swift 3.0 by default, if you are using a different version of swift you can use a `.swift-version` file to set the version for your Pod. For example to use Swift 2.3, run: 
    `echo "2.3" > .swift-version`.
You can use the `--no-clean` option to inspect any issue.
```

因为我之前已经全局设置过了，所以这次没遇到这个问题，如果遇到了这个问题，在命令行执行以下

```
echo "3.0" > .swift-version
```

就可以解决了

其中可能还会遇到其他问题，这个就要具体情况具体分析了

验证我们通过了，接下来，就需要打包了，pod可以打成两种形式的包，一种framework，还有一种是.a的library，这里我使用的是framework

也是通过命令行来执行

```
pod package PPSPrivateStaticLibrary.podspec --force
```

![](http://o8bxt3lx0.bkt.clouddn.com/ppsheep/d233u.jpg)

这里，遇到了问题，因为我们还未将源码提交到git本地仓库，tag也未打，所以报错了

```
git add .
git commit -a -m '0.1.0'
git tag -a 0.1.0 -m '0.1.0'

```

![](http://o8bxt3lx0.bkt.clouddn.com/ppsheep/7vrfl.jpg)

再次执行

![](http://o8bxt3lx0.bkt.clouddn.com/ppsheep/bu6nz.jpg)

已经成功执行了

我们到工程目录下去找

![](http://o8bxt3lx0.bkt.clouddn.com/ppsheep/kgxd3.jpg)

好了已经有了我们想要的framework，如果不放心framework是否支持所有的架构，我们可以检测一下

```
lipo -info 
```
注意这里不是将framework文件夹拿来检查，而是文件夹下面的二进制文件

![](http://o8bxt3lx0.bkt.clouddn.com/ppsheep/lndwz.jpg)

可以看到，包含了所有的架构

### 上传到私有仓库

在我们生成的文件夹中，我们需要用到的两个文件，一个是framework，还有一个就是PPSPrivateStaticLibrary.podspec文件

在上传之前，我们还需要修改一下podspec文件其中的一个地方

```
//将  s.source修改成我们的私有仓库的地址
  s.source = { :git => 'https://github.com/yangqian111/PPSPrivateStaticLibrary.git', :tag => s.version.to_s}
//将s.ios.vendored_framework前的ios文件夹去掉
s.ios.vendored_framework   = 'PPSPrivateStaticLibrary.framework'
```

然后将framework和PPSPrivateStaticLibrary全部上传到我们的私有仓库

接下来我们就可以集成我们的pod库了

新建一个工程，在Podfile中

```
workspace ‘Test.xcworkspace'

project ‘Test.xcodeproj'

platform :ios, '8.0'

target 'Test' do

pod 'PPSPrivateStaticLibrary',:git=>'https://github.com/yangqian111/PPSPrivateStaticLibrary.git'

end
```

其中地址，写我们的私有仓库地址

这样就能顺利集成

```
#import <PPSPrivateStaticLibrary/PPSPublic1.h>
#import <PPSPrivateStaticLibrary/PPSPublic2.h>

@interface AppDelegate ()

@end

@implementation AppDelegate


- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    PPSPublic1 *p1 = [[PPSPublic1 alloc] init];
    [p1 public1];
    PPSPublic2 *p2 = [[PPSPublic2 alloc] init];
    [p2 public2];
    return YES;
}
```

这只能算是一个初步集成，还有很多能做，比如我还想要集成其他的第三方公开库，或者我还要集成其他的第三方私有库，这些，我在后面如果会用到，都会分享出来

源码这些我都放在了github上

https://github.com/yangqian111/PPSPrivateStaticLibrary

**欢迎关注微博：ppsheep_Qian**
 
http://weibo.com/ppsheep

**欢迎关注公众号**

![](http://ac-mhke0kuv.clouddn.com/830a4ead8294ceff5160.jpg)