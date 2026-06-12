# iOS Ad SDK Integration Guide


| Document Version | Revision Date | Revision Notes       |
| ---------------- | ------------- | -------------------- |
| v2.9.0           | 2026-05-20    | Optimized ad display |


# Prerequisites

The following conditions must be met before using this SDK. For details, please contact the designated official representative or apply on the designated platform.

## Supported Ad Types

Splash, interstitial, rewarded video, native (feed), and banner ads.

## Platform Configuration

## SDK Package Size Impact


| Integration Module     | Package Size Impact (MB) |
| ---------------------- | ------------------------ |
| FLGAAdSaas.xcframework | 0.4                      |


# SDK Integration

## Ad Display Integration

### 1.1 Apply for App ID and Ad Placement ID

Developers must create an app and ad placements on the platform to obtain the corresponding App ID and ad placement ID.

### 1.2 Import the Framework

#### 1.2.1 SDK Integration

##### Method 1: CocoaPods

# FunlinkGlobal main package

```
  pod 'FunlinkGlobal'
  
```

##### Method 2: Import framework via project settings

Obtain the framework library for the corresponding version and import it into your project.

- FLGAAdSaas.xcframework

When dragging the framework into the project, select as shown below:

![image.png](http://esimapp.oss-us-west-1.aliyuncs.com/img/img_add.png)

#### 1.2.2 Xcode Build Settings

#### 1.2.2.1 Add Permissions

 **Note: Required system libraries**

- In the project's plist file, click the "+" next to Information Property List to expand it.

Add **App Transport Security Settings**. Expand the left arrow, then click the "+" on the right. The **Allow Arbitrary Loads** option will be added automatically; set its value to **YES**. All SDK APIs support HTTPS, but some advertiser creatives may use non-HTTPS URLs.

```json
<key>NSAppTransportSecurity</key>
    <dict>
         <key>NSAllowsArbitraryLoads</key>
         <true/>
    </dict>
```

Steps are shown below:

![image.png](https://upload-images.jianshu.io/upload_images/12555132-8c31bfc7ec42bc36.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- In **Build Settings**, add **-ObjC** to **Other Linker Flags**.

Steps are shown below:

![image.png](https://upload-images.jianshu.io/upload_images/12555132-c098362e75f2e0d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- The SDK includes IDFA access. Add the IDFA permission to `info.plist` as shown:

```objective-c
<key>NSUserTrackingUsageDescription</key>
<string>This identifier will be used to deliver personalized ads to you.</string>
```

![image.jpg](https://upload-images.jianshu.io/upload_images/12555132-5ea3a64f792b34fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1.2.2.2 Runtime Environment

- Supported OS: iOS 11.0 and above
- SDK build environment: Xcode 15.1
- Supported architectures: x86-64, arm64

#### 1.2.2.3 Add Dependency Libraries (can be skipped if using CocoaPods)

`The following dependency libraries include all dependencies required by ad network partners.`

In your project, go to **TARGETS -> Build Phases -> Link Binary With Libraries**, click "+", and add the following libraries one by one:

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

Steps are shown below:

![image](https://upload-images.jianshu.io/upload_images/12555132-d88d8026c9a74532.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 1.2.3 Obtain the Advertising Identifier (IDFA)

Starting with iOS 14, apps may access the user's IDFA and deliver targeted ads only after obtaining explicit user consent. Before your app calls the [App Tracking Transparency](https://developer.apple.com/documentation/apptrackingtransparency?language=objc) framework to request tracking authorization, IDFA will be unavailable. If your app does not request authorization, the returned IDFA will be a string of all zeros.

##### 1.2.3.1 Add tracking permission description in info.plist

Below are sample descriptions in Chinese and English. You may add either one:

- Chinese version:

```
<key>NSUserTrackingUsageDescription</key>
<string>该标识符将用于向您投放个性化广告。</string>
```

![image.png](https://upload-images.jianshu.io/upload_images/12555132-1e8e46e672c05029.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- English version:

```
<key>NSUserTrackingUsageDescription</key>
<string>This identifier will be used to deliver personalized ads to you.</string>
```

![img](https://upload-images.jianshu.io/upload_images/12555132-36e6996efa498ce5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 1.2.3.2 Request App Tracking Transparency Authorization

To obtain authorization, use [requestTrackingAuthorizationWithCompletionHandler:](https://developer.apple.com/documentation/apptrackingtransparency/attrackingmanager/3547037-requesttrackingauthorization). We recommend requesting authorization before initializing FunlinkSDK so that if the user grants **Allow Tracking**, FunlinkSDK can use IDFA in ad requests.

```
#import <AppTrackingTransparency/AppTrackingTransparency.h>

if (@available(iOS 14, *)) {
    // iOS 14
    [ATTrackingManager requestTrackingAuthorizationWithCompletionHandler:^(ATTrackingManagerAuthorizationStatus status) {
        // to do something, like preloading
        // load AD
    }];
} else {
    // load AD
}
```

The description will appear in the App Tracking Transparency authorization dialog, as shown below:

![img](https://upload-images.jianshu.io/upload_images/4651038-462ef8db8e336964.png?imageMogr2/auto-orient/strip|imageView2/2/w/338/format/webp)

**Notes:**

1. If you need to preload ads, we strongly recommend placing preloading logic inside the completion block of `requestTrackingAuthorizationWithCompletionHandler`.
2. This permission is available only in Xcode 12 and above. Please update to Xcode 12 or later for testing.

##### 1.2.3.3 App Tracking Transparency on iOS 15

iOS 15 made adjustments to the **AppTrackingTransparency** framework. The following two conditions must be met:

- The app is in `UIApplicationStateActive`.
- No other authorization dialog is currently being shown.

After both conditions are met, check `ATTrackingManagerAuthorizationStatus` and request authorization. You may choose one of the following approaches:

- After app launch, delay 1–2 seconds before checking and requesting authorization.
- Check and request authorization in the main view controller's `viewDidAppear` method.

Reference: [https://developer.apple.com/forums/thread/690607](https://developer.apple.com/forums/thread/690607)

#### 1.2.5 Ad Personalization Settings

##### Overseas ad personalization settings

We recommend the following integration flow:

1. After app launch, determine whether the user is in the EU (implement your own EU detection logic).

- **Not in the EU**: skip to step 4
- **In the EU**: continue to the next step

2. Check whether `[FLGAAdSDKManager defaultManager].dataConsentSet` is `FLGAGDPRConsentSetUnknown`.

- **If `FLGAGDPRConsentSetUnknown`** (GDPR level not set): continue to the next step
- **If not `FLGAGDPRConsentSetUnknown`** (GDPR level already set): skip to step 4

3. Call `[FLGAAdSDKManager defaultManager] presentDataConsentDialogInViewController:dismissalCallback:` (let the user set the GDPR level).

Use the following code to configure GDPR:

```
if ([FLGAAdSDKManager defaultManager].dataConsentSet == FLGAGDPRConsentSetUnknown) {
    [[FLGAAdSDKManager defaultManager] presentDataConsentDialogInViewController:self dismissalCallback:^(FLGAGDPRConsentSet dataConsentSet) {
        // Request ads
    }];
}
```

The dialog appears as shown below:

![img](https://upload-images.jianshu.io/upload_images/12555132-cd510d16498234fb.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. Initialize the SDK

**Important:**

If your app also shows the **App Tracking Transparency authorization dialog**, we strongly recommend following the flow below to handle ATT and GDPR dialogs. Otherwise, your app may be rejected during App Store review.

1. After app launch, request App Tracking Transparency authorization first.

- If the user grants authorization, show the GDPR dialog next, then enter the app.
- If the user denies authorization, enter the app directly without showing the GDPR dialog.

ATT and GDPR authorization flow

![ATT and GDPR authorization flow](https://upload-images.jianshu.io/upload_images/4651038-f5c876c3461e7bfa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.3 SDK API Overview and Ad Integration

#### 1.3.0 Global Settings

For splash ads, we recommend initializing the SDK in AppDelegate's `- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions`.

**Initialize as early as possible after the window's root view controller is set and displayed.**

```
// Overseas environment initialization
[FLGAAdSDKManager registerAppId:@"App ID from the dashboard"];
```

#### 1.3.1 Splash Ads

- **Description:** Splash ads are full-screen ad views shown when the app launches. Follow the integration guide to display the designed view. **Note: Splash ad integration code must be placed after the window's rootViewController is set.** See the `LaunchScreenViewController` example in the Demo.

1. Import

```objective-c
#import <FLGAAdSaas/FLGAAdSaas.h>
```

1. Conform to the delegate

```objective-c
<FLGASplashDelegate>
```

1. Request a splash ad

- Full-screen example:

```objective-c
_manager = [FLGASplashManager new];
_manager.delegate = self;
_manager.mediaId = @"Your ad placement ID";
_manager.showAdController = self.window.rootViewController;
[_manager loadAdData];
```

- Non-full-screen example:

```objective-c
// bottom is a view containing a logo
UILabel *bottom = [[UILabel alloc]initWithFrame:CGRectMake(0, 0, [UIScreen mainScreen].bounds.size.width, [UIScreen mainScreen].bounds.size.height * 0.2)];
bottom.text = @"AD Demo";
bottom.textAlignment = NSTextAlignmentCenter;
bottom.font = [UIFont systemFontOfSize:35];
bottom.userInteractionEnabled = YES;
bottom.backgroundColor = [UIColor whiteColor];

_manager = [FLGASplashManager new];
_manager.delegate = self;
_manager.mediaId = @"Your ad placement ID";
_manager.bottomView = bottom;
_manager.showAdController = self.window.rootViewController;
[_manager loadAdData];
```

- Ad delegate callbacks

```objective-c
/**
 * Ad data loaded successfully
 */
- (void)splashAdDidLoad;
/**
 * Ad data failed to load
 * @param error : Error information
 */
- (void)splashAdDidFailed:(NSError *)error;
/**
 * Ad displayed successfully
 */
- (void)splashAdDidVisible;
/**
 * Ad view clicked
 * @param urlStr Landing page URL returned for custom media ads
 */
- (void)splashAdDidClickedWithUrlStr:(NSString *_Nullable)urlStr;
/**
 * Ad view closed
 */
- (void)splashAdDidShowFinish;
/**
 * Ad rendered successfully
 */
- (void)splashAdDidRender;
```

1. Display the splash ad

Before calling show, confirm that you have received the load success callback (`splashAdDidLoad`), and use `[self.manager getCurrentBaseEcpmInfo].isAdValid` to verify the ad is valid. Only call the show method when the ad is valid.

- Display the ad in the success callback:

```objective-c
[self.manager showSplashAdWithWindow:[UIApplication sharedApplication].keyWindow];
```

We recommend setting the wait timeout to 5 seconds and the display duration to 5 seconds.
When the app returns to the foreground after being in the background for 5 minutes or more, we also recommend showing a splash ad.

#### 1.3.2 Interstitial Ads

- **Description:** Interstitial ads are designed to meet diverse media needs. The SDK returns a fully rendered ad view; you only need to display it, which greatly reduces integration effort.
- **Usage:** Use `FLGAInterstitialManager` to call `loadAdData` to request an ad, and `showInterstitialAd` to display it. Set `FLInterstitialDelegate` to receive callbacks for load success, load failure, click, return from landing page, close, and more. See the `InterstitialViewController` example in the Demo.
- Before calling show, use `[self.manager getCurrentBaseEcpmInfo].isAdValid` to verify the ad is valid.

1. Import

```objective-c
#import <FLGAAdSaas/FLGAAdSaas.h>
```

1. Conform to the delegate

```objective-c
<FLGAInterstitialDelegate>
```

1. Request an ad

```objective-c
self.interstitialAd = [FLGAInterstitialManager new];
self.interstitialAd.mediaId = @"Your ad placement ID";
self.interstitialAd.showAdController = self;
self.interstitialAd.delegate = self;
[self.interstitialAd loadAdData];
```

- Ad delegate callbacks

```objective-c
/**
 * Ad data loaded successfully
 */
- (void)interstitialAdDidLoad;
/**
 * Ad data failed to load
 * @param error : Error information
 */
- (void)interstitialAdDidFailed:(NSError *)error;
/**
 * Ad view displayed
 */
- (void)interstitialAdDidVisible;
/**
 * Ad view clicked
 */
- (void)interstitialAdDidClick;
/**
 * Returned from landing page or App Store
 */
- (void)interstitialAdDidCloseOtherController;
/**
 * Ad view closed
 */
- (void)interstitialAdDidAutoClose:(BOOL)autoClose;
```

- Display the ad in the success callback:

Before calling show, confirm that you have received the load success callback (`interstitialAdDidLoad`), and use `[self.interstitialAd getCurrentBaseEcpmInfo].isAdValid` to verify the ad is valid. Only call the show method when the ad is valid.

```
[self.interstitialAd showInterstitialAd];
```

#### 1.3.3 Rewarded Video

- **Description:** Rewarded video ads let users choose to watch a video in exchange for in-app rewards such as virtual currency, items, or exclusive content. These ads are typically 5–60 seconds long and end with a closing screen that guides the user to the next action. Set `FLGARewardVideoDelegate` to receive callbacks for load success, load failure, click, reward completion, close, and more. See the `MotivationVideoViewController` example in the Demo.

1. Import

```objective-c
#import <FLGAAdSaas/FLGAAdSaas.h> 
```

1. Conform to the delegate

```objective-c
<FLGARewardVideoDelegate>
```

1. Create a property

```objective-c
@property (nonatomic, strong) FLGARewardVideoManager *motivationVideo;
```

1. Usage example

```objective-c
self.motivationVideo = [FLGARewardVideoManager new];
self.motivationVideo.mediaId = @"Your ad placement ID";
self.motivationVideo.delegate = self;
[self.motivationVideo loadAdDataWithExtra:nil];
```

- Ad delegate callbacks

```objective-c
/**
 * Rewarded video ad loaded successfully
 */
- (void)rewardedVideoDidLoad;

/**
 * Rewarded video ad failed to load
 * @param error Error object
 */
- (void)rewardedVideoDidFailWithError:(NSError *)error;

/**
 * Rewarded video ad displayed successfully
 */
- (void)rewardedVideoDidVisible;

/**
 * Rewarded video ad clicked
 */
- (void)rewardedVideoDidClick;

/**
 * Rewarded video ad reached the reward condition
 * @param extra Extra parameters passed during initialization
 */
- (void)rewardedVideoDidRewardEffectiveWithExtra:(NSDictionary*)extra;

/**
 * Rewarded video ad closed
 */
- (void)rewardedVideoDidClose;
```

- Display the ad in the success callback:

Before calling show, confirm that you have received the load success callback (`rewardedVideoDidLoad`), and use `[self.motivationVideo getCurrentBaseEcpmInfo].isAdValid` to verify the ad is valid. Only call the show method when the ad is valid.

```objective-c
[self.motivationVideo showRewardVideoAdWithController:self];
```

#### 1.3.4 Banner Ads

- **Description:** Banner ads are designed to meet diverse media needs. The SDK returns a fully rendered ad view; you only need to display it, which greatly reduces integration effort.
- **Usage:** Use `FLGABannerManager` to call `loadAdData` to request an ad, and `showBannerAdWithView` to display it. Set `FLGABannerDelegate` to receive callbacks for load success, load failure, click, return from landing page, close, and more. See the `BannerViewController` example in the Demo.

1. Import

```objective-c
#import <FLGAAdSaas/FLGAAdSaas.h>
```

1. Conform to the delegate

```objective-c
<FLGABannerDelegate>
```

1. Request an ad

```objective-c
self.banner =  [FLGABannerManager new];
self.banner.size = self.bannerView.bounds.size;
self.banner.delegate = self;
self.banner.mediaId = @"Your ad placement ID";
self.banner.showAdController = self;
[self.banner loadAdData];
```

- Ad delegate callbacks

```objective-c
/**
 * Ad data loaded successfully
 */
- (void)bannerAdDidLoad;
/**
 * Ad data failed to load
 * @param error : Error information
 */
- (void)bannerAdDidFailed:(NSError *)error;
/**
 * Ad view displayed
 */
- (void)bannerAdDidVisible;
/**
 * Ad view clicked
 */
- (void)bannerAdDidClick;
/**
 * Returned from landing page or App Store
 */
-(void)bannerAdDidCloseOtherController;
/**
 * Ad view closed
 */
- (void)bannerAdDidClose;
```

- Display the ad in the success callback:

Before calling show, confirm that you have received the load success callback (`bannerAdDidLoad`), and use `[self.banner getCurrentBaseEcpmInfo].isAdValid` to verify the ad is valid. Only call the show method when the ad is valid.

```
[self.banner showBannerAdWithView:self.bannerView];
```

#### 1.3.5 Native (Mixed) Ads

- **Description:** Native mixed ads include self-rendered ads and template ads.

1. Import

```objective-c
#import <FLGAAdSaas/FLGAAdSaas.h> 
```

1. Conform to the delegate

```objective-c
<FLGANativeDelegate>
```

1. Create a property

```objective-c
@property (nonatomic, strong) FLGANativeManager *nativeManager;
```

1. Usage example

```objective-c
FLGANativeManager *manager = [[FLGANativeManager alloc] init];
manager.mediaId = @"Your ad placement ID";
manager.adCount = 1;
manager.size = CGSizeMake(FLGA_ScreenW, 0);
manager.showAdController = self;
manager.delegate = self;
[manager loadAdData];
self.nativeManager = manager;
```

- Ad delegate callbacks

```objective-c
/**
 * Ad data loaded successfully
 */
- (void)nativeAdDidLoadDatas:(NSArray<__kindof FLGAFeedAdData *> *)datas;
/**
 * Ad data failed to load
 * @param error : Error information
 */
- (void)nativeAdDidFailed:(NSError *)error;
/**
 * Ad view displayed
 */
- (void)nativeAdDidVisible;
/**
 * Ad view clicked
 */
- (void)nativeAdDidClicked;
/**
 * Returned from landing page or App Store
 */
- (void)nativeAdDidCloseOtherController;
/**
 * Video ad playback status changed
 * @param status Video ad playback status
 */
- (void)nativeAdVideoPlayerStatusChanged:(FLGAMediaPlayerStatus)status;

// The following callbacks apply to template ads
/**
 * Ad view rendered successfully
 */
- (void)nativeAdDidRenderSuccessWithADView:(UIView *)nativeAdView;
/**
 * Ad view closed
 */
- (void)nativeAdDidCloseWithADView:(UIView *)nativeAdView;
```

In the load success callback (`- (void)nativeAdDidLoadDatas:(NSArray<__kindof FLGAFeedAdData *> *)datas;`), determine whether the ad is a template ad:

```objective-c
for (FLGAFeedAdData *adData in datas) {
    if (adData.adView) {
    		// Template ad
    } else {
    		// Self-rendered ad
    }
}
```

Before calling show, confirm that you have received the load success callback (`nativeAdDidLoadDatas:`), and use `[self.manager getCurrentBaseEcpmInfo].isAdValid` to verify the ad is valid. Only call the show method when the ad is valid.

### **1.4** Bidding Win and Loss Reporting

Each ad type includes the following two methods. When FunlinkSDK is integrated as a bidding source, you must report them:

/// **Must be reported when the media displays an ad through bidding. Call before invoking the ad show method.**

/**
 ======= Report second price after winning the auction =======
 @param secondPrice Second price from the media (unit: cents)
 */

```
- (void)sendWinNotificationWithPrice:(CGFloat)secondPrice;
```

/**

- ======= Report highest price and loss reason after losing the auction =======
- @param firstPrice First price from the media (unit: cents)
- @param reason Loss reason
 */

- (void)sendLossNotificationWithPrice:(CGFloat)firstPrice andLossReason:(FLGAADBidLossReason)reason;

```


## Appendix

### Error Codes

Below are the values for various error codes.

#### SDK Error Codes

| Code | Description |
| --- | --- |
| 20400 | Ad configuration request returned empty |
| 20401 | Parsed data contains no ads |
| 20402 | App ID not registered. Register the App ID during app initialization |
| 20403 | Please check the ad placement ID |
| 20404 | Ad request exceeded the platform-configured timeout |
| 20405 | Ad configuration request error. Contact ad operations to review ad configuration |
| 20406 | Creative rendering failed |
| 20407 | Ad request failed. Possible causes include missing ad configuration, ad placement ID not matching App ID, no ad fill, or network request failure. See error details for the specific reason |
| 20408 | Main ad flow has ended |
| 20409 | This ad placement has a display interval limit. The time since the last display has not reached the configured minimum. Please try again later |
| 20410 | This ad placement has an hourly display frequency limit. The maximum number of displays for the current hour has been reached. Please try again later |
| 20411 | This ad placement has a daily display frequency limit. The maximum number of displays for today has been reached. Please try again tomorrow |


### FAQ

1. Why does the demo run fine but my project fails after integration?

```

Answer: SDK integration requires many configuration steps. Please follow the documentation completely and ensure nothing is missing.

```

2. Where do I get ad placement IDs?

```

Answer: Contact ad operations.

```

3. Integration succeeded but there is no revenue. Why?

```

Answer: Ad revenue is usually shown in the dashboard the next day (delayed on holidays). If you still see no revenue, verify that the ad placement ID is correct. Test ad placements do not generate revenue.

```

4. What is the package size after iOS integration?

```

Answer: Based on our demo build, the FunlinkSDK main package is about 0.4 MB. The actual size may vary depending on which features you import. Use the size of your integrated build as the reference.

```
