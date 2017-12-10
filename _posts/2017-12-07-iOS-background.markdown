---
layout: post
title: iOS background 执行详解
date: 2017-12-07 00:00:00 +0300
description: iOS background 执行详解
img:
---

大家都知道，当用户点击 iPhone 的 Home 键时，iOS 会把当前的 app 转移到 background 状态，在 background 状态停留短暂的时间再转移到 suspended，这样做 iOS 能节省系统资源，提高电池的使用周期。但是如果我们需要在后台执行一些任务呢？需要保持 app 在background状态？别担心，iOS 已经为我们提供了三种策略：

1. 当 app 需要在后台执行有限长度的任务时
2. 当 app 需要支持在后台下载文件时
3. app 需要在 background 执行长时间特定类型的人去

## app 需要在后台执行有限长度的任务
当 app 从 background 进入到 suspended ，需要执行耗时不长的任务，比如保存数据之类的，可以调用 UIApplication 对象的 [beginBackgroundTaskWithName:expirationHandler:](https://developer.apple.com/documentation/uikit/uiapplication/1623051-beginbackgroundtaskwithname) 或者 [beginBackgroundTaskWithExpirationHandler:](https://developer.apple.com/documentation/uikit/uiapplication/1623031-beginbackgroundtask) 方法，请求额外的时间执行任务，下面看 demo：

``` swift
//this is Swift
func applicationDidEnterBackground(_ application: UIApplication) 
{
     bgTask = application.beginBackgroundTask(withName: "myTask") {                        
          print("超过执行最长时间...")
          application.endBackgroundTask(self.bgTask!)
          self.bgTask = UIBackgroundTaskInvalid
      }
      DispatchQueue.global().async {
          print("后台执行任务.....")
          DispatchQueue.global().asyncAfter(deadline: .now() + 12) {
              print("任务完成")
              application.endBackgroundTask(self.bgTask!)
              self.bgTask = UIBackgroundTaskInvalid
          }
     }
}
```
```
//执行结果
后台执行任务.....
任务完成
```

每一次调用 beginBackgroundTaskWithName:expirationHandler:  或 beginBackgroundTaskWithExpirationHandler: 方法都会生成独特的 token 关联这个任务，当完成任务，必须使用相应的 token 调用endBackgroundTask: 方法，让系统知道这个任务执行完成，如果在一定的时间没有调用 endBackgroundTask: ，系统将调用expirationHandler 给 app 最后一次机会结束任务

## 在Background状态下载内容

要支持在后台下载内容，app 必须用 NSURLSession 对象下载，这样系统才能在 app 被 suspended 或 terminated 时，控制下载进程。app 使用 NSURLSession 并配置了后台下载，当 app 被终止，系统将接管下载任务，在下载完成时或需要 app 关注时，系统将在后台唤醒 app 。
为了支持 Background 传输，需要对 NSURLSession 对象进行配置，demo 如下：

```swift
//配置session支持后台下载
let configuration = URLSessionConfiguration.background(withIdentifier: "background downloading")
configuration.sessionSendsLaunchEvents = true
configuration.isDiscretionary = true
var session = URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
```

配置好后，URLSession 对象会在合适的时间把上传下载任务转交给系统。
- 如果任务在 foreground 或者 background 完成，会正常的调起相应的 delegate ；
- 如果系统在终止 app 前，还没有下载完成，系统会自动在后台继续完成任务，当完成后系统会重新打开 app（假设不是用户强制退出 app ），调用 app delegate’s 的 [application:handleEventsForBackgroundURLSession:completionHandler:](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622941-application) 方法，app 需要在方法里实现，用提供的 identifier 创建新的 URLSessionConfiguration 和 NSURLSession 对象，系统将新的 session 对象重新连接到先前的任务，并调用相应的 delegate 

``` swift
func application(_ application: UIApplication, handleEventsForBackgroundURLSession identifier: String, completionHandler: @escaping () -> Void) {
     let configuration = URLSessionConfiguration.background(withIdentifier: identifier)
     configuration.sessionSendsLaunchEvents = true
     configuration.isDiscretionary = true
     var session = URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
}
```

## 实现长时间执行任务

如果需要更长执行时间，app 需要请求特殊的权限在 background 执行而不进入 suspended 状态，在 iOS，只有几种特殊类型的 app 才允许运行在后台：

- 在后台播放音视频类的 app，比如音乐播放器
- 在后台录音类的 app
- 需要实时通知用户坐标的 app，比如导航类 app
- 支持 VoIP 的 app
- 需要定期下载和处理新内容的应用程序
- 从外部配件接收定期更新的应用程序

这些类型的 app 必须声明其支持的服务，并使用系统框架实现这些服务

#### 声明app支持的背景任务

在TARGETS选中项目，点击 Capabilities 页，开启 Background Modes，选择相应的 model ，如下图选择了 Audio and AirPlay 
![选择相应的类型服务](http://upload-images.jianshu.io/upload_images/7951141-597ac17a9073b5b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
因为篇幅有限，这里只给大家举一个例子，有兴趣的同学可自行研究

#### 追踪用户的位置信息

一款跑步软件，在跑步时需要记录用户的位置信息，并实时的更新时长和公里数，就需要app能在后台长时间的运行，demo如下：

```swift
locationManager = CLLocationManager()
locationManager.desiredAccuracy = kCLLocationAccuracyBest
locationManager.activityType = .fitness
if #available(iOS 9.0, *) {
    //允许app在后台执行位置更新
    locationManager.allowsBackgroundLocationUpdates = true
 }
 locationManager.distanceFilter = 10.0
 locationManager.startUpdatingLocation()
 locationManager.delegate = self
```

只有在 allowsBackgroundLocationUpdates 为 true 和 Capabilities 页开启 Background Modes，选择了 Location updates ，才能在后台运行，并在 delegate 执行相应的逻辑

```swift
func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        
        for location in locations {
            let howRecent = location.timestamp.timeIntervalSinceNow
            
            //筛选loaction（10秒前的坐标位置不要）
            if abs(howRecent) >= 10 {
                continue;
            }
            
            //筛选loaction（精度不达标的位置信息不要）
            if location.horizontalAccuracy < kCLLocationAccuracyNearestTenMeters * 3 && location.horizontalAccuracy > 0   {
                if !self.locations.isEmpty{
                    //计算跑步的距离
                    distance += location.distance(from: self.locations.last!)
                }
                self.locations.append(location)
            } 
        }
    }
```