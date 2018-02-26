---
layout: post
title: 常用的设计模式
date: 2017-12-30 00:00:00 +0300
description: 常用的设计模式
img:
---

前一段时间，看完了 Robert C. Martin 写的敏捷软件开发——原则、模式与实践，对其中涉及的设计模式有了进一步的认识，这篇 blog 就说说常用的几种设计模式
## 目录
- Command 模式
- Template Method 模式
- Strategy 模式
- Facade 模式
- Mediator 模式
- Singleton 模式

## Command 模式
command 模式，该模式提供了一种封装方法调用，隐藏方法细节的机制，调用组件只需调用 command 模式的指定方法，而无需知道此方法的实现细节，从而使调用组件和命令对象之间实现了业务上的解耦和时间上的解耦。

最常用这种模式是实现撤销功能，比如画图软件或者计算器、我们进行了一系列操作，想撤销上一步，就可以用 command 模式实现。我们在这里用计算总和的 demo ，来实现此模式，先看看普通版本。


```swift
class Calculate{
    var total = 0

    func add(x: Int) {
        total += x
    }

    func subtract(x: Int) {
        total -= x
    }

    func multiply(x: Int) {
        total *= x
    }

    func divide(x: Int) {
        total /= x
    }
}

let calculate = Calculate()
calculate.add(x: 5) // total = 5
calculate.multiply(x: 4) // total = 20
calculate.subtract(x: 10) // total = 10
calculate.divide(x: 2) // total = 5

```

这是很普通的 demo，如果想实现撤销的功能，一种方式，让调用组件记住其执行的操作，但是调用组件并不知道其他调用组件的操作，所以撤销会造成严重的错误，如果让调用组件协调完成，又会让组件之间造成的耦合。而命令模式可以解决这一问题。

```swift
protocol Command {
    func execute()
}

class UodoCommand<T>: Command {
    var reveiver: T
    let uodo: (T) -> Void

    init(reveiver: T, uodo:@escaping (T) -> Void) {
        self.reveiver = reveiver
        self.uodo = uodo
    }

    func execute() {
        uodo(reveiver)
    }
}

class Calculate{
    var total = 0
    var uodoCommand: [Command] = []

    func add(x: Int) {
        addUodoCommand(method: Calculate.subtract, value: x)
        total += x
    }

    func subtract(x: Int) {
        addUodoCommand(method: Calculate.add, value: x)
        total -= x
    }

    func multiply(x: Int) {
        addUodoCommand(method: Calculate.divide, value: x)
        total *= x
    }

    func divide(x: Int) {
        addUodoCommand(method: Calculate.multiply, value: x)
        total /= x
    }

    func addUodoCommand(method: @escaping (Calculate) -> (Int) -> Void, value: Int) {
        let command = UodoCommand(reveiver: self) { (calculate) in
            method(calculate)(value)
        }
        uodoCommand.append(command)
    }

    func uodo() {
        if !uodoCommand.isEmpty {
            let command = uodoCommand.removeLast()
            command.execute()
            uodoCommand.removeLast()
        }
    }
}

let calculate = Calculate()
calculate.add(x: 5) // total = 5
calculate.multiply(x: 4) // total = 20
calculate.subtract(x: 10) // total = 10
calculate.divide(x: 2) // total = 5

calculate.uodo()   // total = 10

```
这样，就实现了计算的撤销功能，而且把撤销和计算对象绑定，实现了调用组件的业务解耦和时间解耦，我们可以在任何时间去撤销上一步的操作。

## Template Method 模式

该模式是通过继承的方式，分离出通用的算法和具体的实现，也可以在父类提供默认实现，而在子类重写方法进行修改。但是这个模式，有点违背依赖倒置原则，我们用冒泡排序来作为例子，
一般实现方式：

```swift
class BubbleSorter<T: Comparable >{

    static func sort(array:inout [T]) {
        if array.count <= 1 {
            return
        }

        var i = array.count - 2
        while i >= 0 {
            var j = 0
            while j <= i {
                BubbleSorter.compareAndSwap(array: &array, index: j)
                j += 1
            }
            i -= 1
        }
    }

    private static func compareAndSwap(array: inout [T], index: Int) {
        if array[index] > array[index + 1] {
            BubbleSorter.swap(array: &array, index: index)
        }
    }

    private static func swap(array: inout [T], index: Int) {
        let temp = array[index]
        array[index] = array[index + 1]
        array[index + 1] = temp
    }
}

var array = [2.0,2.1,6,3,9.4,6.6]
BubbleSorter.sort(array: &array) // array = [2, 2.1, 3, 6, 6.6, 9.4]
```

因为 swift3.0 之后，移除了 for（；；）的语法，所以我是用 while 实现的，这种实现方式，可以让元素可比较的数组进行冒泡排序，但不遵守 Comparable 的数组就不行了。再看这个版本：

```swift
class BubbleSorter {
    var length = 0

    func outOfOrder(index: Int) -> Bool {
        //子类实现
        return false
    }

    func swap(index: Int) {
        //子类实现
    }

    func sort() {
        if length <= 1 {
            return
        }

        var i = length - 2
        while i >= 0 {
            var j = 0
            while j <= i {
                if outOfOrder(index: j) {
                    swap(index: j)
                }
                j += 1
            }
            i -= 1
        }
    }
}

class ComparableBubbleSorter<T: Comparable >: BubbleSorter {
    var array: [T]

    init(array: [T]) {
        self.array = array
        super.init()
        self.length = array.count
    }

    override func outOfOrder(index: Int) -> Bool {
        return array[index] > array[index + 1]
    }

    override func swap(index: Int) {
        let temp = array[index]
        array[index] = array[index + 1]
        array[index + 1] = temp
    }
}

var bs = ComparableBubbleSorter(array: [2,4,1,6,4,8])
bs.sort()  //[1, 2, 4, 4, 6, 8]
```

这就是 template method 模式，把通用算法放置在基类中，并且通过在不同的子类中实现该通用算法。这个比普通版本好的地方在与，我们可以为任何类型进行排序，只要它在子类中实现 swap 和 outOfOrder 方法即可。比如下面:

```swift
class Person {
    let name: String
    let age: Int
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
}

class PersonBubbleSorter: BubbleSorter {
    var array: [Person]

    init(array: [Person]) {
        self.array = array
        super.init()
        self.length = array.count
    }

    override func outOfOrder(index: Int) -> Bool {
        return array[index].age > array[index + 1].age
    }

    override func swap(index: Int) {
        let temp = array[index]
        array[index] = array[index + 1]
        array[index + 1] = temp
    }
}

let persons = [Person(name: "Amy", age: 40), Person(name: "Json", age: 15), Person(name: "Last", age: 20)]
let pbs = PersonBubbleSorter(array: persons)
pbs.sort() //[Person(name: "Json", age: 15), Person(name: "Last", age: 20), Person(name: "Amy", age: 40)]
```

但这个设计模式又代价的，继承是一种强关系，比如 ComparableBubbleSorter 类中的 swap 和 outOfOrder 方法在其他方式的排序算法中，也需要，但却没有办法在其他方式的排序算法中重用这两个方法。因为 ComparableBubbleSorter 继承自 BubbleSorter 。要解决这一问题，可以看下面这个模式。

## Strategy 模式

使用该模式可以在不修改，或者不继承一个类的情况下扩展功能。它对通用算法和具体实现的依赖倒置。我再来看看冒泡排序的算法：

```swift
protocol SortStrategy {
    func swap(index: Int)
    func outOfOrder(index: Int) -> Bool
    func setArray(array: inout [Any])
}

class BubbleSorter {
    let sortStrategy: SortStrategy
    init(strategy: SortStrategy) {
        self.sortStrategy = strategy
    }

    func sort(array: inout [Any]) {
        sortStrategy.setArray(array: &array)
        let length = array.count

        if length <= 1 {
            return
        }

        var i = length - 2
        while i >= 0 {
            var j = 0
            while j <= i {
                if sortStrategy.outOfOrder(index: j) {
                    sortStrategy.swap(index: j)
                }
                j += 1
            }
            i -= 1
        }
    }
}

class ComparableStrategy <T: Comparable >: SortStrategy {
    var array = [T]()

    func swap(index: Int) {
        guard !array.isEmpty else { return }

        let temp = array[index]
        array[index] = array[index + 1]
        array[index + 1] = temp
    }

    func outOfOrder(index: Int) -> Bool {
        return array[index] > array[index + 1]
    }

    func setArray(array: inout [Any]) {
        guard let array = array as? [T] else
        {
            fatalError()
        }

        self.array = array
    }
}

var array = [2,4,1,6,4,8] as [Any]
let cbs = ComparableStrategy<Int>()
let bs = BubbleSorter(strategy: cbs)
bs.sort(array: &array) // array = [1,2,4,4,6,8]
```

我们看到 strategy 比 template method 模式涉及更多的类和间接层，增加了代码的复杂度，但是 strategy 却更加灵活，它把通用代码和具体实现完全隔离，而且遵守开放关闭原则，我们可以通过实现 SortStrategy 协议，完成不同类型的数组的排序，还有解决 template method 最后提到问题，ComparableStrategy 中 swap 和 outOfOrder 的实现可以被其他方式排序重用。

其实，我们想想在 iOS 里，UITableView 的 UITableViewDataSource 就是通过策略模式实现的，controller 通过遵守 UITableViewDataSource 协议，去实现相关的策略提供数据。

## Facade 模式

Facade 模式一般用于简化一个或者多个类提供的 api ，让常见任务可以更简便的执行。当想为一组复杂且全面的接口对象提供一个简单且特定的接口时，就可以使用 Facade 模式。比如访问数据库，就可以在数据库和应用程序之间加 Facade 类，应用程序不需要知道数据库的内部细节，把数据库的所有接口都集中在 Facade 类中，Facade 类再向应用程序提供一些简洁和特定的接口。

如图：

![Facade](http://p0iv8hbe9.bkt.clouddn.com/facade.jpg)

像 database 类，知道如何初始化、连接、关闭数据库，知道如何把 model 的数据转化成数据库字段，或者数据库字段映射到 model ，知道如何查询、删除数据库，这些对 application 来说都是隐藏的， application 不知道数据库，只知道 database 类和相应的 model 


## Mediator 模式

Mediator 模式，用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。通过引入中介者的方式简化同类对象之间的通信，比如聊天软件中，用户 a 、用户 b 和用户 c ，就可以引入中介者模式，没个用户都关联中介者，通过中介者传递消息，而用户之间没有关联。

```swift
protocol User {
    func getName () -> String
    func receiveMessage(content: String)
    func sendMessage(name: String, content: String)
    func set(mediator: Mediator)
}

protocol Mediator {
    func register(user: User)
    func operation(userName: String, content: String)
}

class UserManager: User {
    let name: String
    var mediator: Mediator?
    init(name: String) {
        self.name = name
    }
    
    func getName() -> String {
        return name
    }
    
    func receiveMessage(content: String) {
        print(content)
    }
    
    func sendMessage(name: String, content: String) {
        guard let mediator = mediator else {
            print("send fail")
            return
        }
        
        mediator.operation(userName: name, content: content)
    }
    
    func set(mediator: Mediator) {
        self.mediator = mediator
    }
}

class UserMediator: Mediator {
    var dic = [String: User]()
    func register(user: User) {
        dic[user.getName()] = user
        user.set(mediator: self)
    }
    
    func operation(userName: String, content: String) {
        guard let user = dic[userName] else {
            print("没有该用户")
            return
        }
        user.receiveMessage(content: content)
    }
}

let userA = UserManager(name: "userA")
let userB = UserManager(name: "userB")
let mediator = UserMediator()
mediator.register(user: userA)
mediator.register(user: userB)
userA.sendMessage(name: "userB", content: "userB 你好！")
```


当用户多了，他们都是 Mediator 提供的中转作用，需要发送消息，直接给 Mediator 发送，由 Mediator 完成对应用户的发送，用户统一和 Mediator 交互，用户之间不存在耦合。

其实MVC模式中，controller 就提供了 Mediator 的作用，view 和 model 都是通过 controller 完成交互和交换信息。

但这个模式有个缺点，就是大量业务交互代码都放在 Mediator 中，会造成 Mediator 代码庞大，难以维护。

## Singleton 模式

单例模式大家都很熟悉，该模式的作用就是确保某个类型的对象在应用程序的生命周期内只存在一个实例，所有调用组件共享该单例的资源。在 Objective-C 中单例的公认的写法类似下面这样：

```swift
+ (id)sharedManager {
    static MyManager * staticInstance = nil;
    static dispatch_once_t onceToken;

    dispatch_once(&onceToken, ^{
        staticInstance = [[self alloc] init];
    });
    return staticInstance;
}
```

在 Swift 中，因为 let 可以保证线程安全，所以一般单例这样：

```swift
 static let sharedInstance = MyManager()
 private init() {}
```

单例模式，简单好用，但是它经常会并发陷阱，
因为该单例是在整个应用共享数据，所以当调用组件同时并发的往单例写数据，就需要开发者并发保护的措施。

在 iOS 系统中单例十分常用，如 NSParagraphStyle.default 、 UIScreen.main 、 UIApplication.shared 、 UserDefaults.standard 和 FileManager.default 等都是单例。

## 总结

其实，还有一些常用设计模式没有列举出来，比如工厂模式(Factory Method)、建造者模式（builder）、组合模式（Composite）、装饰器模式（Decorator）和享元模式（Flyweight）等等，有时间我会再开一篇 blog 说说这些设计模式。

