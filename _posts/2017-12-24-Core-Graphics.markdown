---
layout: post
title: Core Graphics介绍
date: 2017-12-22 00:00:00 +0300
description: 简单介绍一下Core Graphics
img:
---

Core Graphics 也被称为 Quartz 2D ，是一个先进的二维绘图引擎。Core Graphics 是在 iOS 系统结构中的 Media Layer ，它提供了低水平、轻量级、高清晰度的 2D 渲染。

利用 Core Graphics 提供的 API ，我们能在应用程序画出各种不规则线条、形状，还有各种颜色、渐变色、离屏渲染，PDF文档创建，显示和解析等强大功能。

## 目录
- Core Graphics 介绍
- Graphics Contexts
- Creating Graphics Context
- 事例讲解
- 总结

## Core Graphics 介绍

上面说了，Core Graphics 也被称为 Quartz 2D，它是一个二维绘图引擎，可以在 iOS 环境中访问。使用 Quartz 2D 提供的 API 可以进行基于路径的绘图，变换，渐变，绘制阴影，透明度图层，颜色管理，消除锯齿渲染，PDF文档生成和PDF元数据访问。 而且Quartz 2D会尽可能利用图形硬件。

### 理解

可以把 Quartz 2D 绘图，理解为画家在画图，每个连续的绘画操作都将“绘画”层应用于输出“画布”，通常称为页面，通过在页面上增加绘图，来改变当前页面的内容，除了通过覆盖页面上的内容，在页面上绘制的内容不能被修改。所以绘画顺序是重要的。

### Graphics Context

Graphics Context 表示绘图目的地。它包含绘图参数和绘图系统执行任何后续绘图命令所需的所有设备特定信息。Graphics Context 定义了基本的绘图属性，例如绘图时使用的颜色，剪切区域，线宽和样式信息，字体信息，合成选项等等。

在绘制时，所有设备特定的特征都包含在当前特定类型的 Graphics Context 中。换句话说，可以简单地通过提供不同的 Graphics Context 来绘制相同的图像到不同的设备。您不需要执行任何设备特定的计算。 Quartz 2D 会帮助我们做好。

系统提供了五种 Graphics Context：
 
- Bitmap graphics context
- PDF graphics context
- Window graphics context
- Layer context
- PostScript graphics context

### 坐标系

由于不同的设备具有不同的底层成像功能，所以图形的位置和大小的定义必​​须和设备独立开，Quartz 使用了 CTM（current transformation matrix）来实现设备独立。通过一个单独的坐标系统 - user space，将其坐标系映射到输出设备 - device space 的坐标系上，这种映射的关系就是通过 CTM 来定义的。

user space 的坐标系的原点默认位于页面的左下角，x 轴从页面左侧向右侧移动时增加。y轴从页面的底部向顶部增加。但在 iOS 中，有两个特殊的 Graphics Context ，坐标系原点位于左上角（这样就和 UIKit 的坐标系一样）。

- 由 UIView 返回的绘图上下文
- 通过调用 UIGraphicsBeginImageContextWithOptions 函数创建绘图上下文

### 一些常用的特性

- path： 路径定义了一个或多个形状或子路径。子路径可以由直线，曲线或两者组成，它可以打开或关闭。子路径可以是简单的形状，如线条，圆形，矩形或星形，也可以是更复杂的形状，如山脉的轮廓或抽象的涂鸦。
- Transforms：CTM 改变 user space 坐标系到 device space 到映射。
- Patterns： 将一系列绘图操作，重复的绘制到图形上下文中。
- Shadows： 给图形绘制阴影。
- Gradients：给图形绘制渐变效果
- Transparency Layers：把多个图形，组合起来生成复合图形

## Creating Graphics Context

在 iOS 里，我们一般常用的获取 Graphics Context 的方式，有两种场景：

### 在 UIView 中获取

在 iOS 上绘制图形，我们需要一个 UIView ,在 UIView 的 drawRect：方法里调用 UIGraphicsGetCurrentContext() 方法，获取当前的 Graphics Context 。在调用 drawRect：方法之前，UIKit已经配置了其绘图环境，我们只需调用 UIGraphicsGetCurrentContext() 即可。

上面也说过，在UIKit创建并配置图形上下文进行绘制时，会调整该上下文的转换，以使其原点与视图的边界矩形的原点相匹配，也就是说，在 drawRect：方法里得到的 Graphics Context ，它的坐标系和 UIKit 的坐标系一样，都是原点位于左上角，x 轴从页面左侧向右侧移动时增加。y轴从页面的顶部向底部增加。

```swift
override func draw(_ rect: CGRect) {
     let context = UIGraphicsGetCurrentContext()
     //  ...绘图操作
}
```

### Creating a Bitmap Graphics Context

Bitmap Graphics Context 接受一个内存缓存区的指针，这个缓存区用于存储位图信息，当你在 Bitmap Graphics Context 绘图，缓存区就被更新，在你释放图形上下文之后，可以绘制的内容按指定的像素格式展示出来。

在 iOS 中，应该使用 UIGraphicsBeginImageContextWithOptions 获取 Bitmap Graphics Context ，和 drawRect：方法获取一样，获取的 Context ，坐标系和 UIKit 的坐标系一样，都是原点位于左上角。

```swift
UIGraphicsBeginImageContextWithOptions(wrappedValue.size, false, 0.0)
let ctx = UIGraphicsGetCurrentContext()
//  ...绘图操作
UIGraphicsEndImageContext()
```

其他的 PDF Graphics Context 和在 Mac OS X 的 Window Graphics Context ，还有 Paths 、Transforms 、Shadows 、Patterns 和 Gradients ，就不一一介绍了，有兴趣的可以看[这里](https://developer.apple.com/library/content/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/dq_context/dq_context.html#//apple_ref/doc/uid/TP30001066-CH203-TPXREF118)。

## 事例讲解

### 离屏渲染

我们经常给 UIView 或 UIImageView 同时设置 cornerRadius 和 masksToBounds 属性，这样会触发离屏渲染，如果在一个 tableView 页面没个 cell 都有这种 view 就会遇到性能问题，会造成卡顿，那要怎么解决离屏渲染的问题呢？在这我们就通过两种 Core Graphics  的方式解决。

#### 在 UIView 上加圆角

使用 Core Graphics 为 UIView 加圆角，自己来画出一个圆角图形。可以重写 draw(_ rect: CGRect) 方法，在这个方法里绘制有圆角的图形。代码如下，通过 radius 参数改变圆角的弧度，setFillColor 改变颜色。

```swift
override func draw(_ rect: CGRect) {
        let context = UIGraphicsGetCurrentContext()
        context?.move(to: CGPoint(x: 0, y: 0))
        context?.addArc(tangent1End: CGPoint(x: rect.size.width, y: 0),
         tangent2End: CGPoint(x: rect.size.width , y: rect.size.height), radius: 60)
        context?.addArc(tangent1End: CGPoint(x: rect.size.width , y: rect.size.height),
         tangent2End: CGPoint(x: 0 , y: rect.size.height), radius: 60)
        context?.addArc(tangent1End: CGPoint(x: 0 , y: rect.size.height), 
        tangent2End: CGPoint(x: 0, y: 0), radius: 60)
        context?.addArc(tangent1End: CGPoint(x: 0, y: 0), 
        tangent2End: CGPoint(x: rect.size.width, y: 0), radius: 60)
        context?.setFillColor(UIColor.red.cgColor)
        context?.setStrokeColor(UIColor.clear.cgColor)
        context?.drawPath(using: .fillStroke)
    }
```
我们可以在这个 view 上绘制任意弧度的图形（当然，通过 Core Graphics 各种形状的图形都能绘制出来），在调用时 只需要像这样写：

```swift
let cornersView = CornersView(frame: CGRect(x: 100, y: 100, width: 200, height: 200));
view.addSubview(cornersView)
```

效果图如下：

![](http://p0iv8hbe9.bkt.clouddn.com/2FC2FCCF-7430-4B70-8C95-A3A41578D2A1.png)

#### 在 UIImageView 上加圆角

UIImageView 需要通过 Bitmap Graphics Context 来给 image 加圆角，其实 UIView 也能通过这种方式加圆角，在这还是用 UIImageView 作为例子。首先给 UIImage 加个扩展，代码如下

```swift
extension UIImage {  
     func drawRoundedCorner(radius: CGFloat, size: CGSize) -> UIImage? {
        let rect = CGRect(origin: CGPoint(x: 0,y:0), size: size)
        
        UIGraphicsBeginImageContextWithOptions(rect.size, false, UIScreen.main.scale)
        let context = UIGraphicsGetCurrentContext()
        context?.addPath(UIBezierPath(roundedRect: rect, byRoundingCorners: .allCorners,
                                      cornerRadii: CGSize(width: radius, height: radius)).cgPath)
        context?.clip()
        draw(in: rect)
        context?.drawPath(using: .fillStroke)
        let image = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        
        return image
    }
}
```
利用 UIBezierPath 函数，为生成的 Bitmap Graphics Context 绘制一个圆角图形，再把图形之外的 clip 掉，最后在当前 Context ，绘制一个新的 image 。我们还需给  UIImageView  加个扩展方便使用。如下：

```swift

extension UIImageView { 

	func  addCorner(radius: CGFloat) {
       self.image = 
       self.image?.drawRoundedCorner(radius: radius, size: self.bounds.size)
    }
}

```
这种加圆角的方式，必须要 UIImageView 已经设置了 image 才行。使用起来很简单：

```swift
let imageView = UIImageView(image: #imageLiteral(resourceName: "blog_test"))
imageView.frame.origin = CGPoint(x: 50, y: 100)
imageView.addCorner(radius: 40)
view.addSubview(imageView)
```

效果如下：

![](http://p0iv8hbe9.bkt.clouddn.com/E717D387-626D-49F1-874B-20DD17EB5DF4.png)

## 总结

我们一般什么情况，用 Core Graphics 呢，比如需要各种不规则的图形、解决离屏渲染的问题、需要重绘 image 等等，基本上任何图形， Core Graphics 都可以绘制出来，这篇博客只是讲了皮毛，有兴趣的同学，可以自己去看苹果文档。

## 参考

[ Core Graphics 文档](https://developer.apple.com/library/content/documentation/GraphicsImaging/Conceptual/drawingwithquartz2d/dq_context/dq_context.html#//apple_ref/doc/uid/TP30001066-CH203-TPXREF118)

[离屏渲染](https://hit-alibaba.github.io/interview/iOS/Cocoa-Touch/Performance.html)。










