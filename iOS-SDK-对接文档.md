# iOS 广告 SDK 接入说明


| 文档版本   | 修订日期       | 修订说明   |
| ------ | ---------- | ------ |
| v2.9.0 | 2026-05-20 | 优化广告展示 |


# 前置说明

使用本SDK前必须满足以下条件。具体详情请联系官方指定负责人或到指定平台申请！

## 支持的广告类型

开屏、插屏、激励视频、信息流、横幅广告

## 平台配置

## SDK包体影响


| 接入模块                   | 包体影响(M) |
| ---------------------- | ------- |
| FLGAAdSaas.xcframework | 0.4     |


# SDK接入

## 展示广告接入

### 1.1 申请应用的应用ID 和 广告位ID

开发者需在平台创建应用和广告位，生成对应的应用ID和广告位ID。

### 1.2 导入framework

#### 1.2.1  SDK集成

##### 方法1 pod接入

# FunlinkGlobal主包

```
  pod 'FunlinkGlobal'
  
```

##### 方法2 工程设置导入framework

获取相应版本的framework库，导入项目工程即可。

- FLGAAdSaas.xcframework

拖入时请按以下方式选择：

image.png

#### 1.2.2 Xcode编译选项设置

#### 1.2.2.1 添加权限

 **注意要添加的系统库**

- 工程plist文件设置，点击右边的information Property List后边的 "+" 展开

添加 App Transport Security Settings，先点击左侧展开箭头，再点右侧加号，Allow Arbitrary Loads 选项自动加入，修改值为 YES。 SDK API 已经全部支持HTTPS，但是广告主素材存在非HTTPS情况。

```json
<key>NSAppTransportSecurity</key>
    <dict>
         <key>NSAllowsArbitraryLoads</key>
         <true/>
    </dict>
```

具体操作如图：

image.png

- Build Settings中Other Linker Flags **增加参数-ObjC**

具体操作如图：

image.png

- SDK中包含获取IDFA的权限,所以需要在info.plist中添加IDFA权限,如图所示:

```objective-c
<key>NSUserTrackingUsageDescription</key>
<string>该标识符将用于向您投放个性化广告</string>
```

image.jpg

#### 1.2.2.2 运行环境配置

- 支持系统 iOS 11.0 及以上;
- SDK编译环境 Xcode 15.1;
- 支持架构： x86-64,  arm64

#### 1.2.2.3 添加依赖库(pod接入可忽略此操作)

`以下依赖库已整合所有联盟广告主所需的依赖库`
工程需要在TARGETS -> Build Phases中找到Link Binary With Libraries，点击“+”，依次添加下列依赖库	

- libresolv.9.tbd
- libc++.tbd
- libc++abi.tbd
- libz.tbd
- libbz2.tbd 
- libxml2.tbd 
- libiconv.tbd
- libsqlite3.tbd
- StoreKit.framework
- MobileCoreServices.framework
- WebKit.framework
- MediaPlayer.framework
- CoreMedia.framework
- AVFoundation.framework
- CoreLocation.framework
- CoreTelephony.framework
- SystemConfiguration.framework
- AdSupport.framework
- CoreMotion.framework
- Security.framework  
- QuartzCore.framework 
- CoreGraphics.framework
- SafariServices.framework
- UIKit.framework
- Foundation.framework 
- JavaScriptCore.framework 
- MapKit.framework
- AssetsLibrary.framework
- AppTrackingTransparency.framework
- MessageUI.framework
- DeviceCheck.framework
- CoreML.framework

具体操作如图所示：

image

#### 1.2.3. 获取广告标识IDFA

从iOS 14开始，只有在获得用户明确许可的前提下，应用才可以访问用户的IDFA数据并向用户投放定向广告。在应用程序调用 [App Tracking Transparency](https://developer.apple.com/documentation/apptrackingtransparency?language=objc) 框架向最终用户提出应用程序跟踪授权请求之前，IDFA将不可用。如果某个应用未提出此请求，则读取到的IDFA将返回全为0的字符串。 

##### 1.2.3.1 在info.plist文件里添加跟踪权限请求描述说明

以下提供中英文描述说明示例，开发者可参考添加其中一种即可：

- 中文版：

```
<key>NSUserTrackingUsageDescription</key>
<string>该标识符将用于向您投放个性化广告。</string>
```

image.png

- 英文版：

```
<key>NSUserTrackingUsageDescription</key>
<string>This identifier will be used to deliver personalized ads to you.</string>
```

img

##### 1.2.3.2 获取App Tracking Transparency权限

想要获取授权，需要使用[requestTrackingAuthorizationWithCompletionHandler:](https://developer.apple.com/documentation/apptrackingtransparency/attrackingmanager/3547037-requesttrackingauthorization)。我们建议您在初始化FunlinkSDK之前获取授权，以便如果用户授予**允许跟踪**权限，FunlinkSDK则可以在广告请求中使用IDFA。

```
#import <AppTrackingTransparency/AppTrackingTransparency.h>

if (@available(iOS 14, *)) {
    //iOS 14
    [ATTrackingManager requestTrackingAuthorizationWithCompletionHandler:^(ATTrackingManagerAuthorizationStatus status) {
        //to do something，like preloading
        // load AD
    }];
} else {
    // load AD
}
```

描述说明将会显示在App Tracking Transparency授权对话框中，如下：

img

**注意：**

1. 如果有预加载广告需求的，强烈建议把预加载写在requestTrackingAuthorizationWithCompletionHandler的block回调中。
2. 该权限只有Xcode 12及以上版本才有，需要大家更新Xcode 12版本来进行测试使用。

##### 1.2.3.3 iOS 15获取App Tracking Transparency权限说明

iOS 15对 **AppTrackingTransparency** 框架做了调整，需要满足以下 2 点：

- 应用当前的状态为 UIApplicationStateActive。
- 当前没有其他的授权弹窗。

当满足以上 2 点后，再检查 ATTrackingManagerAuthorizationStatus 和请求授权。因此，可以在以下的选项中选择一个作为调整方案：

- 启动应用后，延时 1~2 秒去检查和申请权限。
- 在主控制器的 “ViewDidAppear” 方法中检查和申请权限。

参考链接：[https://developer.apple.com/forums/thread/690607](https://developer.apple.com/forums/thread/690607)

#### 1.2.5  广告个性化开关设置

##### 海外广告个性化开关设置

建议开发者的集成流程如下

1、 APP启动后，开发者判断用户是否在欧盟(开发者自行实现是否在欧盟的判断方法）

- **不在欧盟**，跳到第4步
- **在欧盟**，下一步

2、判断**[FLGAAdSDKManager defaultManager]**的**dataConsentSet**是否为**FLGAGDPRConsentSetUnknown**

- **为FLGAGDPRConsentSetUnknown**（未设置过GDPR等级），下一步
- **不为FLGAGDPRConsentSetUnknown**（设置过GDPR等级），跳到第4步

3、调用**[FLGAAdSDKManager defaultManager]**的**presentDataConsentDialogInViewController:dismissalCallback**方法(由用户设置GDPR等级）

使用以下代码段设置GDPR:

```
if ([FLGAAdSDKManager defaultManager].dataConsentSet == FLGAGDPRConsentSetUnknown) {
    [[FLGAAdSDKManager defaultManager] presentDataConsentDialogInViewController:self dismissalCallback:^(FLGAGDPRConsentSet dataConsentSet) {
        // 请求广告
    }];
}
```

展示如图：

img

4、初始化SDK

**重要提示：**

如果你的应用同时获取了**App Tracking Transparency授权弹窗**，我们强烈建议参考如下图所示流程来获取App Tracking Transparency授权弹窗与GDPR弹窗，否则有提审被拒的风险！

1. 应用启动后，首先获取App Tracking Transparency授权弹窗

- 用户通过授权，接着获取GDPR弹窗，然后进入应用。
- 用户拒绝授权，直接进入应用，不再弹出GDPR弹窗。

ATT与GDPR授权流程

### 1.3 SDK接口类介绍与广告接入

#### 1.3.0 全局设置

SDK的开屏广告建议在 AppDelegate 的方法 `- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions` 里

**当window指定root控制器并显示之后最先进行初始化**

```
// 海外环境初始化设置
[FLGAAdSDKManager registerAppId:@"从后台获取的应用ID"];
```

#### 1.3.1 开屏广告

- **类型说明：开屏广告主要是 APP 启动时展示的全屏广告视图，开发只要按照接入标准就能够展示设计好的视图==(注意：开屏接入代码必须放在window指定rootViewController之后)==**。具体可参考Demo中LaunchScreenViewController部分示例代码

1. 导入

```objective-c
#import <FLGAAdSaas/FLGAAdSaas.h>
```

1. 遵循代理

```objective-c
<FLGASplashDelegate>
```

1. 请求开屏广告

- 全屏示例：

```objective-c
_manager = [FLGASplashManager new];
_manager.delegate = self;
_manager.mediaId = @"获取的广告位";
_manager.showAdController = self.window.rootViewController;
[_manager loadAdData];
```

- 非全屏示例：

```objective-c
//bottom 为含logo的view
UILabel *bottom = [[UILabel alloc]initWithFrame:CGRectMake(0, 0, [UIScreen mainScreen].bounds.size.width, [UIScreen mainScreen].bounds.size.height * 0.2)];
bottom.text = @"AD Demo";
bottom.textAlignment = NSTextAlignmentCenter;
bottom.font = [UIFont systemFontOfSize:35];
bottom.userInteractionEnabled = YES;
bottom.backgroundColor = [UIColor whiteColor];

_manager = [FLGASplashManager new];
_manager.delegate = self;
_manager.mediaId = @"获取的广告位";
_manager.bottomView = bottom;
_manager.showAdController = self.window.rootViewController;
[_manager loadAdData];
```

- 广告的代理回调

```objective-c
/**
 * 广告数据：加载成功
 */
- (void)splashAdDidLoad;
/**
 * 广告数据：加载失败
 * @param error : 错误信息
 */
- (void)splashAdDidFailed:(NSError *)error;
/**
 * 广告成功展示
 */
- (void)splashAdDidVisible;
/**
 * 广告视图：点击
 * @param urlStr 媒体自定义广告时，返回的落地页链接
 */
- (void)splashAdDidClickedWithUrlStr:(NSString *_Nullable)urlStr;
/**
 * 广告视图：关闭
 */
- (void)splashAdDidShowFinish;
/**
 * 广告成功渲染
 */
- (void)splashAdDidRender;
```

1. 展示开屏广告

调用前请确认收到加载成功回调(splashAdDidLoad),并调用[self.manager getCurrentBaseEcpmInfo].isAdValid 来判断广告是否有效，有效时再去调用广告的展示方法进行展示。

- 在广告成功的回调中展示广告：

```objective-c
[self.manager showSplashAdWithWindow:[UIApplication sharedApplication].keyWindow];
```

​    

​    建议等待时间设置为5秒，展示时间设置为5秒。
​    App在从后台5分钟后到前台时 建议也加上开屏广告。

#### 1.3.2 插屏广告

- **类型说明：**插屏广告是为满足媒体多元化需求而开发的一种广告。SDK会返回已渲染完成的广告视图,开发只需展示即可,避免了接入方的大量工作量。
- **使用说明：**插屏广告 使用 FLGAInterstitialManager 对象调用 loadAdData 请求广告，使用 showInterstitialAd 添加广告对象来进行广告展示，通过设置 FLInterstitialDelegate 代理，获取广告获取成功、广告获取失败、点击、从落地页返回、关闭等回调。具体可参考Demo中 InterstitialViewController 部分示例代码
- 调用前请调用[self.manager getCurrentBaseEcpmInfo].isAdValid 来判断广告是否有效，有效时再去调用广告的展示方法进行展示。

1. 导入

```objective-c
#import <FLGAAdSaas/FLGAAdSaas.h>
```

1. 遵循代理

```objective-c
<FLGAInterstitialDelegate>
```

1. 请求广告

```objective-c
self.interstitialAd = [FLGAInterstitialManager new];
self.interstitialAd.mediaId = @"获取的广告位";
self.interstitialAd.showAdController = self;
self.interstitialAd.delegate = self;
[self.interstitialAd loadAdData];
```

- 广告的代理回调

```objective-c
/**
 * 广告数据：加载成功
 */
- (void)interstitialAdDidLoad;
/**
 * 广告数据：加载失败
 * @param error : 错误信息
 */
- (void)interstitialAdDidFailed:(NSError *)error;
/**
 * 广告视图：展示
 */
- (void)interstitialAdDidVisible;
/**
 * 广告视图：点击
 */
- (void)interstitialAdDidClick;
/**
 * 落地页或者appstoe返回事件
 */
- (void)interstitialAdDidCloseOtherController;
/**
 * 广告视图：关闭
 */
- (void)interstitialAdDidAutoClose:(BOOL)autoClose;
```

- 在广告成功的回调中展示广告：
调用前请确认收到加载成功回调(interstitialAdDidLoad),并调用[self.interstitialAd getCurrentBaseEcpmInfo].isAdValid 来判断广告是否有效，有效时再去调用广告的展示方法进行展示。

```
[self.interstitialAd showInterstitialAd];
```

#### 1.3.3 激励视频

- **类型说明：**激励视频广告是一种全新的广告形式，用户可选择观看视频广告以换取有价物，例如虚拟货币、应用内物品和独家内容等等；这类广告的长度为 5-60 秒，且广告的结束画面会显示结束页面，引导用户进行后续动作。通过设置 FLGARewardVideoDelegate 代理，获取广告获取成功、广告获取失败、点击、播放达到激励条件、关闭等回调。具体可参考Demo中 MotivationVideoViewController 部分示例代码。

1. 导入

```objective-c
#import <FLGAAdSaas/FLGAAdSaas.h> 
```

1. 遵循代理

```objective-c
<FLGARewardVideoDelegate>
```

1. 创建属性对象

```objective-c
@property (nonatomic, strong) FLGARewardVideoManager *motivationVideo;
```

1. 使用示例

```objective-c
self.motivationVideo = [FLGARewardVideoManager new];
self.motivationVideo.mediaId = @"获取的广告位";
self.motivationVideo.delegate = self;
[self.motivationVideo loadAdDataWithExtra:nil];
```

- 广告的代理回调

```objective-c
/**
 * 激励视频广告-视频-加载成功
 */
- (void)rewardedVideoDidLoad;

/**
 * 激励视频广告素材加载失败
 * @param error 错误对象
 */
- (void)rewardedVideoDidFailWithError:(NSError *)error;

/**
 * 激励视频广告成功展示
 */
- (void)rewardedVideoDidVisible;

/**
 * 激励视频广告点击
 */
- (void)rewardedVideoDidClick;

/**
 * 激励视频广告播放达到激励条件
 * @param extra 额外参数，即初始化传入的extra
 */
- (void)rewardedVideoDidRewardEffectiveWithExtra:(NSDictionary*)extra;

/**
 * 激励视频广告已经关闭
 */
- (void)rewardedVideoDidClose;
```

- 在广告成功的回调中展示广告：
- 调用前请确认收到加载成功回调(rewardedVideoDidLoad),并请调用[self.motivationVideo getCurrentBaseEcpmInfo].isAdValid 来判断广告是否有效，有效时再去调用广告的展示方法进行展示。

```objective-c
[self.motivationVideo showRewardVideoAdWithController:self];
```

#### 1.3.4 横幅广告

- **类型说明：**横幅广告是为满足媒体多元化需求而开发的一种广告。SDK会返回已渲染完成的广告视图,开发只需展示即可,避免了接入方的大量工作量。
- **使用说明：**横幅广告 使用 FLGABannerManager 对象调用 loadAdData 请求广告，使用 showBannerAdWithView 添加广告对象来进行广告展示，通过设置 FLGABannerDelegate 代理，获取广告获取成功、广告获取失败、点击、从落地页返回、关闭等回调。具体可参考Demo中 BannerViewController 部分示例代码

1. 导入

```objective-c
#import <FLGAAdSaas/FLGAAdSaas.h>
```

1. 遵循代理

```objective-c
<FLGABannerDelegate>
```

1. 请求广告

```objective-c
self.banner =  [FLGABannerManager new];
self.banner.size = self.bannerView.bounds.size;
self.banner.delegate = self;
self.banner.mediaId = @"获取的广告位";
self.banner.showAdController = self;
[self.banner loadAdData];
```

- 广告的代理回调

```objective-c
/**
 * 广告数据：加载成功
 */
- (void)bannerAdDidLoad;
/**
 * 广告数据：加载失败
 * @param error : 错误信息
 */
- (void)bannerAdDidFailed:(NSError *)error;
/**
 * 广告视图：展示
 */
- (void)bannerAdDidVisible;
/**
 * 广告视图：点击
 */
- (void)bannerAdDidClick;
/**
 * 落地页或者appstoe返回事件
 */
-(void)bannerAdDidCloseOtherController;
/**
 * 广告视图：关闭
 */
- (void)bannerAdDidClose;
```

- 在广告成功的回调中展示广告：
调用前请确认收到加载成功回调(bannerAdDidLoad),并请调用[self.banner getCurrentBaseEcpmInfo].isAdValid 来判断广告是否有效，有效时再去调用广告的展示方法进行展示。

```
[self.banner showBannerAdWithView:self.bannerView];
```

#### 1.3.5 原生混出广告

- **类型说明：**原生混出广告包含自渲染广告和模板广告。

1. 导入

```objective-c
#import <FLGAAdSaas/FLGAAdSaas.h> 
```

1. 遵循代理

```objective-c
<FLGANativeDelegate>
```

1. 创建属性对象

```objective-c
@property (nonatomic, strong) FLGANativeManager *nativeManager;
```

1. 使用示例

```objective-c
FLGANativeManager *manager = [[FLGANativeManager alloc] init];
manager.mediaId = @"获取的广告位";
manager.adCount = 1;
manager.size = CGSizeMake(FLGA_ScreenW, 0);
manager.showAdController = self;
manager.delegate = self;
[manager loadAdData];
self.nativeManager = manager;
```

- 广告的代理回调

```objective-c
/**
 * 广告数据：加载成功
 */
- (void)nativeAdDidLoadDatas:(NSArray<__kindof FLGAFeedAdData *> *)datas;
/**
 * 广告数据：加载失败
 * @param error : 错误信息
 */
- (void)nativeAdDidFailed:(NSError *)error;
/**
 * 广告视图：展示
 */
- (void)nativeAdDidVisible;
/**
 * 广告视图：点击
 */
- (void)nativeAdDidClicked;
/**
 * 落地页或者appstoe返回事件
 */
- (void)nativeAdDidCloseOtherController;
/**
 * 视频广告播放状态更改回调
 * @param status 视频广告播放状态
 */
- (void)nativeAdVideoPlayerStatusChanged:(FLGAMediaPlayerStatus)status;

//当为模板广告时有以下回调
/**
 * 广告视图：渲染成功
 */
- (void)nativeAdDidRenderSuccessWithADView:(UIView *)nativeAdView;
/**
 * 广告视图：关闭
 */
- (void)nativeAdDidCloseWithADView:(UIView *)nativeAdView;
```

在加载成功的回调（`- (void)nativeAdDidLoadDatas:(NSArray<__kindof FLGAFeedAdData *> *)datas;`）中，判断是否是模板广告：

```objective-c
for (FLGAFeedAdData *adData in datas) {
    if (adData.adView) {
    		//模板广告
    } else {
    		//自渲染广告
    }
}
```

调用前请确认收到加载成功回调(nativeAdDidLoadDatas:),并请调用[self.manager getCurrentBaseEcpmInfo].isAdValid 来判断广告是否有效，有效时再去调用广告的展示方法进行展示。

### **1.4** 竞价成功与竞价失败上报

每类广告都包含如下两个方法，在FunlinkSDK作为竞价接入时，需上报:

/// ** 媒体竞价展示广告时需要上报，需要在调用广告 show 之前调用 **

/**
  ======= 我方竞胜后需要回传第二价 ======= * @param secondPrice 媒体二价 (单位: 分) */

```
- (void)sendWinNotificationWithPrice:(CGFloat)secondPrice;
```

/**

- ======= 我方竞败后需要回传最高价以及竞败原因 =======
- @param firstPrice 媒体一价  (单位: 分)
- @param reason  竞败原因
 */

- (void)sendLossNotificationWithPrice:(CGFloat)firstPrice andLossReason:(FLGAADBidLossReason)reason;

```


## 附录

### 错误码
下面是各种ErrorCode的值
#### SDK 错误码
| code | 说明 |
| --- | --- |
|20400 |请求广告配置为空|
|20401 |解析的数据没有广告|
|20402 |未注册APP ID,请在应用初始化时注册APP ID|
|20403 |请检查广告位ID|
|20404 |广告请求时间已超过平台配置时间|
|20405 |广告配置请求错误，请联系广告运营人员查看广告配置|
|20406 |素材渲染失败|
|20407 |广告请求失败，可能原因是有广告未配置，配置的广告位ID与APPID不匹配，广告未填充，网络请求失败等原因之一，具体错误原因，请查看错误详情！|
|20408 |广告的主流程已结束|
|20409 |此广告位设置了展示间隔限制，当前时间距离上次展示广告时间未达到配置的最低要求，请稍后再试|
|20410 |此广告位设置了一小时内广告展示频次限制，当前小时内广告展示已达设置的最大频次，请稍后再试|
|20411 |此广告位设置了一天内广告展示频次限制，今天广告展示已达设置的最大频次，请明天再试|


### FAQ

1. 为什么demo可以运行，接入项目会出错？
```

答：接入SDK需要很多的配置工作，请按照文档说明配置齐全，保证没有遗漏！

```
2. 广告位在哪获取？
```

答：联系广告运营获取。

```
3.	广告对接成功，但是没有收益是怎么回事？
```

答：广告收益一般在第二天会在后台系统显示，节假日顺延，如果还是没看到，请确保广告位是否使用正确，如果误用测试广告位，这个是没有广告收益的！

```
3. iOS集成的包大小是多少?
```

答：FunlinkSDK主包根据我们demo打包后的计算为0.4MB左右，具体大小会根据导入的功能有所差别，实际情况以集成后的包大小为主。

```

```