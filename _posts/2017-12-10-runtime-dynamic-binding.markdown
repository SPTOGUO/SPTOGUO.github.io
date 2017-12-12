---
layout: post
title: iOS runtime 动态绑定
date: 2017-12-07 00:00:00 +0300
description: 讲解 iOS runtime 动态绑定（关联对象和消息转发机制）
img:
---

我们在 iOS 开发时，经常会涉及到 Objective-C 的 runtime ，其中，基本的也是常用的是动态类型（Dynamic typing）和动态绑定（Dynamic binding），这次我介绍动态绑定（Dynamic binding）相关的知识。

## 什么是动态绑定

我理解的动态绑定，即是在实例所属类确定后，利用runtime的特性，将某些属性和相应的方法绑定到实例上。这里所指的属性和方法当然包括了原来没有在类中实现的，而是在运行时才需要的新加入的实现。

利用动态绑定，我们可以替换类中的方法，为类成员添加变量，动态改变方法实现等。下面我就一一介绍下怎么实现动态绑定。

## 替换类中的方法和动态改变方法实现

要想动态添加方法实现和替换类中已有方法等实现，首先我们要理解 Objective-C 的消息转发机制。

#### 消息转发机制

在我们给实例发送消息时，我们必须在类中实现相应的方法。但是，在编译期向类发送了其无法解读的消息并不报错，因为在编译时还无法知道类中有没有该方法的实现，而是推迟到运行时，在运行时接受到无法解读的消息，会启动消息转发机制，只有在消息转发机制后，还没有找到相应的实现，应用程序才会崩溃。

触发消息转发机制后，消息转发分为两个阶段：

1. 动态方法解析
2. 完整的消息转发机制

###### 动态方法解析

转发机制触发，首先会进行动态方法解析，看接收者所属的类，是否能动态添加方法，处理这个未知的消息。

动态方法解析，先调用所属类的该类方法：

```swift
// 实例方法
+ (BOOL)resolveInstanceMethod:(SEL)sel
// 类方法
+ (BOOL)resolveClassMethod:(SEL)sel
```

参数就是未知的消息，返回值bool，表示能否新增一个实例方法处理该消息，可以配合 class_addMethod 方法添加相应的方法。

```swift
class_addMethod(Class _Nullable cls, SEL _Nonnull name, 
IMP _Nonnull imp, const char * _Nullable types) 
```

到这一步，动态方法解析已经完成，如果还没有处理未知的消息，则需要进行第二步，完整的消息转发机制。

###### 完整的消息转发机制

完整的消息转发机制，是在第一步动态方法解析未处理的情况下，才会触发，这一步又细分为两小步

1. 备援接收者
2. 完整的消息转发

首先进行备援接收者，系统会调用该类的下列方法，运行时系统询问它，能不能把这条消息转给其他接收者处理。

```swift
- (id)forwardingTargetForSelector:(SEL)aSelector
```

参数代表未知的消息，如果有转发对象，就返回该对象，如果没有，返回nil。

如果备援接收者返回nil，则接着进行第二小步，完整的消息转发，系统会创建 NSInvocation 对象，把与尚未处理的那条消息有关的全部细节都封装在其中。此步骤会调用下列方法来转发消息：

```swift
- (void)forwardInvocation:(NSInvocation *)anInvocation
```

这个方法实现起来很简单，只需改变调用目标，比如初始化个新实例，让消息在新实例上调用即可，但这么用的话，和备援接收者是没有什么区别的。那这两者有什么区别呢？

1. forwardingTargetForSelector 仅支持一个对象的返回，也就是说消息只能被转发给一个对象, 而 forwardInvocation 可以将消息同时转发给任意多个对象。
2. forwardInvocation 能在触发前，以某种方式改变消息内容，比如追加另外一个参数等，而forwardingTargetForSelector无法操作转发的消息。

到此，整个消息转发机制就完成了，如果消息还没有处理，则应用程序就会崩溃。消息转发流程图如下：

![消息转发流程图](http://p0iv8hbe9.bkt.clouddn.com/messageForwarding.jpeg)

#### 黑魔法：method swizzling

我们知道了消息的转发机制，我们可以在类中没有实现某方法时，动态的添加实现，或者是把该消息转移到别的实例上。但这些都是在类中没有实现要执行的方法，那我们是否可以在运行时，动态的改变已有方法的实现，答案是肯定的，这个关键就是 method swizzling。

这样，我们就可以不需要源代码，也不需要继承父类去覆写方法，就能改变这个方法实现的功能，而且新功能将在本类的所有实例中生效，而不是仅限于覆写了相关方法等那些子类实例。

首先，介绍一下 method swizzling 相关的方法

```swift
void method_exchangeImplementations(Method _Nonnull m1, Method _Nonnull m2)

Method _Nullable class_getInstanceMethod(Class _Nullable cls, SEL _Nonnull name)
```

每个类都维护着一个方法表，这个表里会把选择子的名称映射到相关的方法实现上，而 method_exchangeImplementations 是把两个方法等映射关系交换，如下图：

![](http://p0iv8hbe9.bkt.clouddn.com/WechatIMG30.jpeg)

我们来看demo:

```swift
@interface UIControl (CustomControl)

- (void)swizzledAction:(SEL)action to:(id)target forEvent:(UIEvent *)event;

@end

@implementation UIControl (CustomControl)

- (void)swizzledAction:(SEL)action to:(id)target forEvent:(UIEvent *)event {
    NSLog(@"swizzledAction  target = %@", target);
    [self swizzledAction:action to:target forEvent:event];
}

@end
```
我们为UIControl扩展一个方法swizzledAction，内部是打印信息和调自己，大家可能很疑惑，这不就成了无限递归了，不要着急接着往下看。

```swift
@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    Method originalMethod = class_getInstanceMethod([UIControl class], @selector(sendAction:to:forEvent:));
    Method swizzledMethod = class_getInstanceMethod([UIControl class], @selector(swizzledAction:to:forEvent:));
    method_exchangeImplementations(originalMethod, swizzledMethod);
    
    UIButton *control = [[UIButton alloc] initWithFrame:CGRectMake(0, 0, 300, 300)];
    control.backgroundColor = [UIColor redColor];
    [control addTarget:self action:@selector(tapControl:) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:control];
}

- (void)tapControl:(id)sender {
    NSLog(@"tapImageControl");
}

@end

```
在这里，我们用到了上面讲的两个方法，把 UIControl 的sendAction:to:forEvent: 和 swizzledAction:to:forEvent: 方法交换，然后初始化了 UIButton ，点击这个 button ，会打印信息。

```swift
swizzledAction  target = <ViewController: 0x7fcbe24271e0>
tapImageControl
```
当我们点击 Button 时，button 会调用 sendAction:to:forEvent: ，但我们在这里把选择子的映射关系交换了，现在 sendAction:to:forEvent: 指向的是 swizzledAction:to:forEvent: 的实现，而 swizzledAction:to:forEvent: 指向了 sendAction:to:forEvent: 实现，所以 button 实际调用的是swizzledAction 的实现，现在再看 swizzledAction 的实现，它在内部其实调用的不是自己而是 sendAction:to:forEvent: 的实现，就不会造成递归，只是在 sendAction:to:forEvent: 实现上加了一个打印信息。

虽然这个domo没有什么意义，但它实现了自定义方法和系统定义的方法交换，


## 关联对象（类添加属性）

有时我们想为某个类添加属性，但是该类的实例是由某种机制所创建，而我们无法让这种机制创建自己所写的子类实例，这种情况，我们就要用到关联对象。关联对象有三个方法：

```swift
objc_setAssociatedObject(id _Nonnull object, const void
* _Nonnull key,id _Nullable value,
objc_AssociationPolicy policy)
// 此方法以给定的键和策略为某对象设置关联对象值

objc_getAssociatedObject(id _Nonnull object, const void * _Nonnull key)
// 此方法根据给定的键从某对象中获取相应的关联对象值

objc_removeAssociatedObjects(id _Nonnull object)
// 此方法移除指定对象的全部关联对象
```

用这些方法可以关联其他对象，这些对象通过键来区分，还可以指明存储策略，这个存储策略和 @property 属性相对应，如果关联对象成为属性，那么就会具备对应的语义。

| 关联类型        | 等效的 @property 属性          | 
| ---------------------------------- |:-------------:| 
| OBJC_ASSOCIATION_ASSIGN            | assign | 
| OBJC_ASSOCIATION_RETAIN_NONATOMIC  | strong,nonatomic| 
| OBJC_ASSOCIATION_COPY_NONATOMIC    | copy,nonatomic|  
| OBJC_ASSOCIATION_RETAIN            | strong,atomic | 
| OBJC_ASSOCIATION_COPY              | copy, atomic  | 




