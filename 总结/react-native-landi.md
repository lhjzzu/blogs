### 导出常量

+ 通过`constantsToExport`方法导出常量

  ```objc
  #import "ClassPlatform_Landi-Swift.h"
  @implementation EnvModule
  
  RCT_EXPORT_MODULE();
  
  - (NSDictionary *)constantsToExport {
      NSMutableDictionary *env = [NSMutableDictionary dictionary];
  #ifdef DEBUG
      [env setObject:@(1) forKey:@"DEBUG"];
  #else
      [env setObject:@(0) forKey:@"DEBUG"];
  #endif
      NSString *os = [EnvModuleForRN getOS];
      NSString *version = [EnvModuleForRN getAppVersion];
      NSString *baseURL = [EnvModuleForRN getBaseURL];
  
      [env setObject:os forKey:@"OS"];
      [env setObject:version forKey:@"VERSION"];
      [env setObject:baseURL forKey:@"API_URL"];
  
      return  env;
  }
  
  @end
  
  ```

  ```js
  const GetOSVersion = ()=>{
      let os = 'ios';
      let version = '1.0.0';
      const source = Platform.OS === 'ios' ? 20 : 18;
      const EnvModule = NativeModules.EnvModule;
      if(EnvModule){
          os = EnvModule.OS;
          version = EnvModule.VERSION;
      }
      
      return{
          os,
          version,
          source
      }
  }
  ```

### 定义原生模块，js进行调用

+ 创建模块，遵守`RCTBridgeModule`即可
+ 通过 `RCT_EXPORT_MODULE()`宏声明模块名
+ 通过`RCT_EXPORT_METHOD()`宏,导出方法供js使用

#### 模块定义和使用

##### 模块定义

```javascript
@interface MineModule : NSObject <RCTBridgeModule>

@end


#import "MineModule.h"
#import "ClassPlatform_Landi-Swift.h"
#import <React/RCTLog.h>

@implementation MineModule

RCT_EXPORT_MODULE();
RCT_EXPORT_METHOD(purchase) {
    [[MineModuleForRN shared] purchase];
}

RCT_EXPORT_METHOD(comment) {
    [[MineModuleForRN shared] comment];
}

RCT_EXPORT_METHOD(call:(NSString *)phone) {
    
    dispatch_async(dispatch_get_main_queue(), ^{
        NSString * str=[[NSString alloc] initWithFormat:@"%@%@:%@",@"tel",@"prompt",phone];
        [[UIApplication sharedApplication] openURL:[NSURL URLWithString:str]];
    });
}


RCT_EXPORT_METHOD(cacheSize:(RCTResponseSenderBlock)callback) {
    NSString *path = [[MineModuleForRN shared] cacheDir];
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSString *cacheSize = [self getCacheSizeWithPath:path];
        dispatch_async(dispatch_get_main_queue(), ^{
            if (callback) {
                callback(@[cacheSize]);
            }
        });
    });
}
...
...
@end
```

##### js使用

```js
import {
    NativeModules
} from 'react-native';

onItemPress = (item, index) => {
        const MineModule = NativeModules.MineModule
        if (item.img == 'clear') { //清除
            MineModule.clearCache((results) => {
              Tips.show('清除成功')
              this.setCacheSize()
            })
        } else if (item.img == 'comment') { //评论
            if (Platform.OS == 'ios') {
                MineModule.comment()
            }
        } else if (item.img == 'purchase') { //购买
            if (Platform.OS == 'ios') {
                MineModule.purchase()
            }
        } else if (item.img == 'setting') { //设置
            this.props.navigation.navigate('Setting')
        }
    }
```

#### js调用时，向原生代码传递参数

```javascript
const MineModule = NativeModules.MineModule
MineModule.call(phoneNum)
```

#### 原生代码给js回调结果

```objective-c
@implementation MineModule
RCT_EXPORT_MODULE();

//typedef void (^RCTResponseSenderBlock)(NSArray *response);
//RCTResponseSenderBlock只调用一次，多次调用会直接crash
RCT_EXPORT_METHOD(cacheSize:(RCTResponseSenderBlock)callback) {
    NSString *path = [[MineModuleForRN shared] cacheDir];
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSString *cacheSize = [self getCacheSizeWithPath:path];
        dispatch_async(dispatch_get_main_queue(), ^{
            if (callback) {
                callback(@[cacheSize]);
            }
        });
    });
}
@end
```

```js
//js调用
const MineModule = NativeModules.MineModule
        MineModule.cacheSize((result) => {
            var cacheSize = result
            console.log('cacheSize', cacheSize)
            var listData = this.state.list
            var firstData = listData[0]
            firstData.cacheSize = cacheSize
            this.setState({
                list: listData
            })
   })
```





### 定义js监听，原生模块调用js

即使没有被 JavaScript 调用，原生模块也可以给 JavaScript 发送事件通知。最好的方法是继承`RCTEventEmitter`，实现`suppportEvents`方法并调用`self  sendEventWithName`

#### js代码

```js
componentDidMount = () => {
  const CourseModule  = NativeModules.CourseModule
  const courseModuleEmitter = new NativeEventEmitter(CourseModule);
  this.subscription = courseModuleEmitter.addListener('JSDownloadProgress',(reminder) => {
      let reminderJson = {};
      if (Platform.OS != 'ios') {
        reminderJson = JSON.parse(reminder);
      }else{
        reminderJson = reminder;
      }
      const { item } = reminderJson
      //下载失败
      if(item.cacheStatus == 3){
        if(Platform.OS !== "ios"){
          const ToastUtilModule =  NativeModules.ToastUtilModule
          ToastUtilModule.showToast('下载失败，请重新再试')
        }else{
          PopToast.show('下载失败，请重新再试')
        }
      }
      const index = this.state.listData.findIndex(data => data.id === item.id)
      if(index != -1){
        item['progress']=reminderJson.progress
        const { listData } = this.state
        listData[index] = item
        this.setState({listData:listData})
      }
    });
}

componentWillUnmount = ()=> {
    this.subscription.remove()
  }
```



#### 原生模块调用js

```objc
@interface CourseModule : RCTEventEmitter <RCTBridgeModule>

@end

#import "CourseModule.h"
#import "ClassPlatform_Landi-Swift.h"
#import <React/RCTLog.h>
#import "LERNTempViewController.h"
@interface CourseModule()
@property (nonatomic, strong) ReplayDownloadManager *downloadManager;

@end
@implementation CourseModule

RCT_EXPORT_MODULE();
//requiresMainQueueSetup:返回值为yes时，代表Module实例要在主线程进行创建和初始化
// 因为初始化可能会访问UIKit框架等
+ (BOOL)requiresMainQueueSetup {
    return YES;
}
- (NSArray<NSString *> *)supportedEvents
{
    return @[@"JSDownloadProgress",@"backAction"];
}

- (void)jsDownloadProgressEventReminder:(CGFloat)progress item:(NSDictionary *)itemDic
{
    [self sendEventWithName:@"JSDownloadProgress" body:@{@"progress":@(progress),@"item":itemDic}];
    RCTLogInfo(@"JSDownloadProgress");
}

- (void)downloadResource:(NSArray <NSURL *> *)resources classid:(NSString *)classId itemDic:(NSDictionary *)itemDic  callback:(RCTResponseSenderBlock)callback {
    [[UIApplication sharedApplication] setIdleTimerDisabled:YES];
    
    [self.downloadManager download:resources :classId progressHandler:^(float progresss) {
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            dispatch_async(dispatch_get_main_queue(), ^{
                RCTLog(@"progresss = %@", @(progresss));
                if (progresss < 1.0) {
                    //调用js函数，触发js监听
                    [self jsDownloadProgressEventReminder:progresss item:itemDic];
                }
            });
        });
    } completion:^(BOOL isCompletion, NSArray<NSURL *> * _Nonnull cachePath, NSString * _Nonnull classID) {
        [[UIApplication sharedApplication] setIdleTimerDisabled:NO];

        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            dispatch_async(dispatch_get_main_queue(), ^{
                NSMutableDictionary *mutableItemDic = [itemDic mutableCopy];
                if (isCompletion) {
                   //调用js函数，触发js监听
                    [mutableItemDic setObject:@(2) forKey:@"cacheStatus"];
                } else {
                    //调用js函数，触发js监听
                    [mutableItemDic setObject:@(3) forKey:@"cacheStatus"];
                }
                [self jsDownloadProgressEventReminder:1 item:mutableItemDic];
            });
        });
    }];
}
@end
```

### 加载的资源位置

```objective-c
- (RCTRootView *)getRctRootView {
    NSURL *jsCodeLocation;
#ifdef DEBUG
//   真机调试时，要放在同一个wifi下，并且ip修改为手机的ip
//    jsCodeLocation = [NSURL URLWithString:@"http://192.168.215.62:8081/index.bundle?platform=ios"];
//    模拟器时，使用localhost本机地址即可
//    jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.bundle?platform=ios"];
    NSString *path = [[NSBundle mainBundle] pathForResource:@"bundle/index.ios" ofType:@"jsbundle"];
   jsCodeLocation = [NSURL URLWithString:path];
#else
//    加载打包在本地的资源
    NSString *path = [[NSBundle mainBundle] pathForResource:@"bundle/index.ios" ofType:@"jsbundle"];  
    jsCodeLocation = [NSURL URLWithString:path];
#endif
    if (!self.rootView) {
        RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                            moduleName: @"UbaseNative"
                                                     initialProperties: nil                             launchOptions: nil];
        self.rootView = rootView;
        
    }
    return self.rootView;
}
```



### 多入口

```objective-c
#import <React/RCTBridge.h>
@interface TTBridgeDelegate : NSObject <RCTBridgeDelegate>
@end
#import "TTBridgeDelegate.h"
@implementation TTBridgeDelegate
- (NSURL *)sourceURLForBridge:(RCTBridge *)bridge {
  return [NSURL URLWithString:@"http://127.0.0.1:8081/index.bundle?platform=ios"];
}
@end
  

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  id<RCTBridgeDelegate> moduleInitialiser =  [[TTBridgeDelegate alloc] init];
  RCTBridge *bridge = [[RCTBridge alloc] initWithDelegate:moduleInitialiser launchOptions:nil];
 
  RCTRootView *rootView = [[RCTRootView alloc]
     initWithBridge:bridge
         moduleName:@"ReactNativeDemo"
  initialProperties:nil];
  
   self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
  UIViewController *rootViewController = [UIViewController new];
  rootViewController.view = rootView;
  self.window.rootViewController = rootViewController;
  [self.window makeKeyAndVisible];
  return YES;
}


```

### react-native打包

````shell
react-native bundle --entry-file index.js --platform ios --dev false --bundle-output ./ios/iPad_Landi/ClassPlatform_Landi/ClassPlatform_Landi/bundle/index.ios.jsbundle --assets-dest ./ios/iPad_Landi/ClassPlatform_Landi/ClassPlatform_Landi/bundle --sourcemap-output ./ios/iPad_Landi/ClassPlatform_Landi/ClassPlatform_Landi/bundle/index.ios.jsbundle.map
````

### Code Push

+ 在code push中注册app获取key

+ 获取key: `code-push deployment ls ClassPlatform_Landi -k`

+ #### release-react: 此命令用于一键发布，其实是将`react-native bundle`命令和`code-push release`命令结合起来使用。

  ```
  "release-react-staging":"code-push release-react ClassPlatform_Landi ios --plistFile ios/iPad_Landi/ClassPlatform_Landi/ClassPlatform_Landi/Info.plist --deploymentName Staging --targetBinaryVersion 1.3.7 --bundleName  index.ios.jsbundle --description 'this is test2 for 1.3.7'",
  
  "release-react-production":"code-push release-react ClassPlatform_Landi ios --plistFile ios/iPad_Landi/ClassPlatform_Landi/ClassPlatform_Landi/Info.plist --deploymentName Production --targetBinaryVersion 1.3.7 --bundleName  index.ios.jsbundle --description 'this is test2 for 1.3.7'"
   
  ```

+ **补丁更新(patch)**

  ```
  - 在发布更新之后，如果想要修改此次更新的参数可以使用`patch`命令，如：你想增加更新的首次展示百分比。
  
   "patch":"code-push patch ClassPlatform_Landi Production --rollout 100%"
  ```

+ ### 促进更新(promote)

  + 有一个场景， 当我们在线上的Staging环境下测试完毕后，我们可以执行`promote`命令将之推进到`Product`环境，而不是重新执行`release`命令，然后重新设置参数。我们只需执行`promote`命令进行一个拷贝即可。

    ```
    "promote":"code-push promote ClassPlatform_Landi Staging Production --description '来自于staging2' --rollout 10%",
    ```

+ ### 回滚更新(rollback)

  + 当某个版本出现重大问题时，需要将版本回滚到老的正常版本去，可以使用rollback命令

+ 总结

  - 在code push中创建应用后，会得到两个环境（staging  production)下的key
  - 第一次上传应用时，内测环境配置staging对应的key， release环境配置production对应的key ，使用react-native bundle 将js等各种资源打包到安装包中
  -  我们通过release-react进行打包并指定对应的二进制版本，上传js资源到staging环境中，提交测试包给测试进行测试。
  - 当测试通过之后，通过promote直接将测试通过的js版本推送到production环境,并通过`--rollout`指定灰度的范围进行灰度测试。然后观察bugly统计平台，和Code Push后台是否产生自动回滚，是否有新的crash产生。
  - 如果版本是稳定的，则通过patch修改`--rollout`为100%,将对应的js版本完全覆盖
  - 如果发现版本有严重的crash，则可进行rollback将版本回滚掉

### 原理

https://www.jianshu.com/p/8fc5015f4180