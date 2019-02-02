---
description: '作者:darrenzheng 转自:https://juejin.im/post/5aaa2943f265da239c7b15c4'
---

# 23个经典设计模式的Swift实现

## 前言

> 这是一篇主观的文章，文字解释也尽可能简单，写作目的是一次团队内的知识分享，想让不了解设计模式的同事迅速对这些生词混个脸熟。所以本文适合懂Swift语法，想快速了解23个设计模式大概在讲什么的同学。

前置知识 [UML那些线](https://juejin.im/post/5a55d97f6fb9a01cbc6e487f)

## 基本结构

* **比喻** 让我联想到的一些事物
* **官方定义** 原版定义
* **UML** 不是原版UML 只保留了我觉得核心的部分
* **代码** Swift实现，这个是本体
* **讲解** 假设已经看过代码的一些零散评注

## 目录

* Creational 创建型 5
  * Abstract Factory 抽象工厂模式
  * Builder 建造者模式
  * Factory Method 工厂方法模式
  * Prototype 原型模式
  * Singleton 单例模式
* Structural 结构型 7
  * Adapter 适配器模式
  * Bridge 桥接模式
  * Composite 组合模式
  * Decorator 装饰者模式
  * Facade 外观模式
  * Flyweight 享元模式
  * Proxy 代理模式
* Behavioral 行为型 11
  * Chain of responsibility 责任链模式
  * Command 命令模式
  * Interpreter 解释器模式
  * Iterator 迭代器模式
  * Mediator 中介模式
  * Memento 备忘录模式
  * Observer 观察者模式
  * State 状态模式
  * Strategy 策略模式
  * Template Method 模板方法模式
  * Visitor 访问者模式

## 工厂模式

> 工厂模式顾名思义，就像一个工厂生产你所需要的产品

![](https://user-gold-cdn.xitu.io/2018/3/15/16228b44ad4cca44?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) _配图：http://cdn1.alphr.com/sites/alphr/files/2016/02/tesla\_factory\_tour\_1.jpg_

### 无工厂 Non-Factory

也就是工厂问题想解决的原始问题。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228b674df37c35?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```text
protocol Product {}
class ConcreteProductA: Product {}
class ConcreteProductB: Product {}

class Client {
    func createProduct(type: Int) -> Product {
        if type == 0 {
            return ConcreteProductA()
        } else {
            return ConcreteProductB()
        }
    }
}

let c = Client()
c.createProduct(type: 0) // get ConcreteProductA
复制代码
```

从代码和UML可以看出，为了得到产品A，调用者`Client`要同时依赖`Product`, `ConcreteProductA`和`ConcreteProductB`，并亲自写一个创建产品的方法。

每当需求新增一个产品，就要改动到调用方`Client`。如果这一堆创建代码如果可以抽离出去就好了，于是简单工厂出现了。

### 简单工厂 Simple Factory

简单工厂就做了一件事，**把Client要做的创建工作，挪到了另一个类里**。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228b6c46c23e40?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```text
protocol Product {}
class ConcreteProductA: Product {}
class ConcreteProductB: Product {}

class Client {
    let s = Factory()
}

class Factory {
    func createProduct(type: Int) -> Product {
        if type == 0 {
            return ConcreteProductA()
        } else {
            return ConcreteProductB()
        }
    }
}

let c = Client()
c.s.createProduct(type: 0) // get ConcreteProductA
复制代码
```

`Factory`代替了`Client`对具体`Product`的依赖，那么当需求变化的时候，我们不再需要改动调用方。这固然有所进步，但无法避免的是，**每次变动都要在createProduct的方法内部新增一个if-else分支**，这显然违背了[开闭原则](https://link.juejin.im?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FOpen%2Fclosed_principle)。

为了解决这个问题，我们引入另一个模式。

### 工厂方法 Factory Method

**官方定义**

> 定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228b7414ea9071?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```text
protocol Product {}
class ConcreteProductA: Product {}
class ConcreteProductB: Product {}

class Client {
    let f = Factory()
}

class Factory {
    func createProduct() -> Product? { return nil } // 用于继承
    func createProduct(type: Int) -> Product? { // 用于调用
        if type == 0 {
            return ConcreteFactoryA().createProduct()
        } else {
            return ConcreteFactoryB().createProduct()
        }
    }
}

class ConcreteFactoryA: Factory {
    override func createProduct() -> Product? {
        // ... 产品加工过程
        return ConcreteProductA()
    }
}

class ConcreteFactoryB: Factory {
    override func createProduct() -> Product? {
        // ... 产品加工过程
        return ConcreteProductB()
    }
}

let c = Client()
c.f.createProduct(type: 0) // get ConcreteProductA
复制代码
```

对于工厂方法的实现，有众多不同的解法，比如`Factory`只保留一个`createProduct`让子类实现，让`Client`来选择生成哪个具体工厂实例；或是引入一个FactoryMaker的中间层，作为生产工厂的“简单工厂”。我这里采用的方式是Factory既作为工厂父类，让具体工厂决定生产生么产品，又作为接口类，让`Client`可以通过[依赖注入](https://link.juejin.im?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FDependency_injection)选择特定工厂。我这样做的目的是，在不引入新的中间层的情况下，最小化Client的依赖。

工厂方法在简单工厂的基础上做了两件事：

* 多了一层抽象，把生产产品的工作延迟到子类执行。
* 把“选择如何生产产品的工作”转化为“选择让哪个具体工厂生产”。

工厂方法的贡献在于，这样做虽然不能完美避免对一个if-else的扩展，但是这个扩展规模被极大限制住了（只需要new一个类）。

工厂方法着重点是解决了单一产品线的派生问题。那如果有多个相关产品线呢？

### 抽象工厂 Abstract Factory

**官方定义**

> 提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228b775f6ff3af?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```text
protocol ProductA {}
class ConcreteProductA1: ProductA {}
class ConcreteProductA2: ProductA {}

protocol ProductB {}
class ConcreteProductB1: ProductB {}
class ConcreteProductB2: ProductB {}

class Client {
    let f = Factory()
}

class Factory {
    func createProductA() -> ProductA? { return nil } // 用于继承
    func createProductB() -> ProductB? { return nil } // 用于继承
    func createProductA(type: Int) -> ProductA? { // 用于调用
        if type == 0 {
            return ConcreteFactory1().createProductA()
        } else {
            return ConcreteFactory2().createProductA()
        }
    }
    func createProductB(type: Int) -> ProductB? { // 用于调用
        if type == 0 {
            return ConcreteFactory1().createProductB()
        } else {
            return ConcreteFactory2().createProductB()
        }
    }
}

class ConcreteFactory1: Factory {
    override func createProductA() -> ProductA? {
        // ... 产品加工过程
        return ConcreteProductA1()
    }
    override func createProductB() -> ProductB? {
        // ... 产品加工过程
        return ConcreteProductB1()
    }
}

class ConcreteFactory2: Factory {
    override func createProductA() -> ProductA? {
        // ... 产品加工过程
        return ConcreteProductA2()
    }
    override func createProductB() -> ProductB? {
        // ... 产品加工过程
        return ConcreteProductB2()
    }
}

let c = Client()
c.f.createProductA(type: 0) // get ConcreteProductA1
c.f.createProductA(type: 1) // get ConcreteProductA2
c.f.createProductB(type: 0) // get ConcreteProductB1
c.f.createProductB(type: 1) // get ConcreteProductB2
复制代码
```

图很吓人，其实很简单。

当我们有两个相关的产品线`ProductA`和`ProductB`, 例如螺丝和螺母，他们派生出`ProductA1`，`ProductB1` 和 `ProductA2`，`ProductB2`，前者我们由工厂`ConcreteFactory1`来制作，后者由 `ConcreteFactory2`来制作。

对于`Client`来说，他只需要知道有一个抽象的工厂能**同时**生产`ProductA`和`ProductB`就行了，那就是图中的Factory。

**重点来了**，这个抽象的`Factory`是通过“工厂方法”模式把构造过程延迟到子类执行的，也就是说，抽象工厂是建立在工厂方法的基础上的模式。

所以抽象工厂，换句话说，就是多个产品线需要绑定在一起，形成一个抽象的综合工厂，由具体的综合工厂来_批量_实现“工厂方法”的一种更“高级”的模式。

#### 总结

有点绕，说完这些感觉我已经中文十级了。总之，我想表达的观点是：**这些工厂模式并不是割裂的存在，而是一个递进的思想**。

## Builder 建造者模式

> 建造者模式就像你委托一个室内设计师装修你的新家

![](https://user-gold-cdn.xitu.io/2018/3/15/16228bcaa20fed51?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) _配图: http://tse2.mm.bing.net/th?id=OIP.N3hIhcOq32Bh6Ezi6q6kGwHaFj&pid=Api_

**官方定义**

> 将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。

如果说之前的谈到的工厂模式是把创建产品\(对象\)的工作抽离出去的话，这次要聊的建造者模式，就是把**产品内部**的组件生产工作抽离出去。这样做的场景，适用于那些有着复杂、规则模块的对象生成流程。换句话说，工厂模式是一个类\(工厂\)创建另一个类\(产品\)，而建造者是一个类\(产品\)自身的属性\(组件\)构造过程。

对于建造者模式的实现网上也是版本不一：复杂点的版本会引入一个Director的角色，做一个整体上下文，组装更傻瓜化的builder和product。或是抽象一层Builder协议，用不同的具体Builder来构造不同的产品。但我认为这些都模糊了这个模式要传达的焦点，对理解没有帮助，所以这里我选择一个极简的模型。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228bd251c56c84?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```text
struct Builder {
    var partA: String
    var partB: String
}

struct Product {
    var partA: String
    var partB: String
    init(builder: Builder) {
        partA = builder.partA
        partB = builder.partB
    }
}

// 通过builder完成产品创建工作
let b = Builder(partA: "A", partB: "B")
// 这样产品只需要一个builder就可以完成制作
let p = Product(builder: b)
复制代码
```

我们让Product的生成由自己发起，但是它的组件\(属性\)全都委托给Builder来实现，而它只需要依赖一个Builder就完成了自身的生产工作。

## Prototype 原型模式

> 原型模式让你有了一个可以源源不断自我赋值的类。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228be4cf498c4f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) _配图: http://thumbs.dreamstime.com/z/cell-division-two-cells-divide-osmosis-background-other-cells-48181492.jpg_

**官方定义**

> 用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

原型模式很简单，你只要实现一个**返回你自己的新对象**的方法即可。这里我采用的实现还不是最简单的，这个interface并不是必须的。

原型模式实现了深拷贝。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228be892b4b350?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```text
protocol Prototype {
    func clone() -> Prototype
}

struct Product: Prototype {
    var title: String
    func clone() -> Prototype {
        return Product(title: title)
    }
}

let p1 = Product(title: "p1")
let p2 = p1.clone()
(p2 as? Product)?.title // OUTPUT: p1
复制代码
```

## Singleton 单例模式

> 单例就像一个公司的IT部门，他们是唯一的存在，并且被所有人直接访问。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228bef47904c11?imageslim) _配图：The IT Crowd_

**官方定义**

> 保证一个类仅有一个实例，并提供一个访问它的全局访问点。

因为第二点常常被忽视，所以过度使用的危害极大，你无从知道调用从何而来，这种`goto`一般的存在会变成维护的噩梦。

单例比较常见的应用是例如数据库，网络框架的全局访问点。

单例其实就是**变种**的原型模式，只不过原型每次返回的是一个拷贝。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228bf3071227f0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Swift的简单实现：

```text
class Singleton {
    static let sharedInstance = Singleton()
    private init() {
        // 用private防止被new
    }
}
let s  = Singleton.sharedInstance
let s2 = Singleton() // ERROR: initializer is inaccessible due to 'private' protection level
复制代码
```

Swift的完整实现：

```text
class Singleton {
    static var singleton: Singleton? = nil
    private init() {}
    static func sharedInstance() -> Singleton {
        if singleton == nil {
            singleton = Singleton()
        }
        return singleton!
    }
}

let s = Singleton.sharedInstance()
let s2 = Singleton() // ERROR: initializer is inaccessible due to 'private' protection level
复制代码
```

## Adapter 适配器模式

> 适配器就像一个电源转换插头。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228c2e165fe1fa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) _配图: http://www.wap135.com/270/timg08/uploaded/i8/TB2452gtpXXXXaLXXXXXXXXXXXX\_!!622114048.jpg_

**官方定义**

> 将一个类的接口转换成客户希望的另外一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

一个类能称之为适配器，就是指它能把另一个类进行某种变形，让其能和实际需求对接。 比如一个底层数据模型，可以通过不同的UI适配器，对应不同的展现需要。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="264"%20height="247"></svg>)

```text
protocol Target {
    var value: String { get }
}

struct Adapter: Target {
    let adaptee: Adaptee
    var value: String {
        return "\(adaptee.value)"
    }
    init(_ adaptee: Adaptee) {
        self.adaptee = adaptee
    }
}

struct Adaptee {
    var value: Int
}

Adapter(Adaptee(value: 1)).value // "1"
复制代码
```

## Bridge 桥接模式

> 桥接模式就是这么一座桥，它矗立在具体和抽象之间，当你调用的时候只看到了抽象，但是它内部实现时“桥接”到了具体。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228c4c4b5fc92c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) _配图: https://refactoring.guru/design-patterns/bridge_

**官方定义**

> 将抽象部分与实现部分分离，使它们都可以独立的变化。

桥接模式是继工厂模式之后另一个有点绕的概念，为了不一言不合就抛概念把大家绕晕，我们直接来看一下这个模式最终成型的样子。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="302"%20height="355"></svg>)

首先假设我们封装了一个“开关能力”的接口。

```text
protocol 开关能力 {
    func turnOn(_ on: Bool)
}
复制代码
```

这个接口只有一个方法，实现者需要提供打开/关闭开关会触发什么操作。

然后我们抽象一个叫“设备”的类出来，这个类就有意思了：

* 首先他有一个实现了`开关能力`的实例变量，并通过初始化赋好值。
* 他有一个方法的实现就是直接提供了`开关能力`里的实现。

```text
class 设备 {
    let obj: 开关能力
    func turnOn(_ on: Bool) {
        obj.turnOn(on)
    }
    init(_ obj: 开关能力) {
        self.obj = obj
    }
}
复制代码
```

这样，其实一个桥接模式就已经搭建完了。再讲解之前，我们直接看一下如何应用它：

```text
class 电视: 开关能力 {
    func turnOn(_ on: Bool) {
        if on {
            // 打开电视
        } else {
            // 关闭电视
        }
    }
}

class 空调: 开关能力 {
    func turnOn(_ on: Bool) {
        if on {
            // 打开空调
        } else {
            // 关闭空调
        }
    }
}

let tv = 设备(电视())
tv.turnOn(true) // 打开电视

let aircon = 设备(空调())
aircon.turnOn(false) // 关闭空调
复制代码
```

通过这段代码可以看出：

* 在把抽象的`设备`应用到具体的业务的时候，这个模式采用的是组合了一个实现了`开关能力`接口的实例，而没用继承。
* 最终调用的时候，是由统一的`设备`作为接入点的，而不是`电视`，`空调`这些具体类，具体的实现是通过组合的方式**注入**到`设备`里的。

了解了这个流程后，一个事情就明朗了：

> 这不就是在用组合代替继承吗？

没错，它把需要变化的代码通过接口代理出去，而避免了继承。

但是，桥接模式的**桥**在哪里？

这时，要搬出他的概念了： [Bridge pattern - Wikipedia](https://link.juejin.im?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FBridge_pattern)：

> The bridge pattern is a design pattern used in software engineering which is meant to "decouple an abstraction from its implementation so that the two can vary independently". 桥接模式解耦了抽象和具体，让它们可以独立变化。

从代码去寻找具体和抽象，就可以发现：

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="317"%20height="444"></svg>)

**这座桥能走通的关键就是这个组合在抽象类里的变量。**

最后，你会发现，这不就是Adapter和Adaptee吗？ 没错，设计模式其实是连续剧来着。

不同在于适配器的关注点是**如何让两个不兼容的类对接**， 而桥接模式关注点是**解耦**。

## Composite 组合模式

> 组合模式就像一个公司的组织架构，存在基层员工\(Leaf\)和管理者\(Composite\)，他们都实现了组件\(Component\)的work方法，这种树状结构的每一级都是一个功能完备的个体。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228c5ed2200c04?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) _配图：https://realtimeboard.com/static/images/page/examples/detail/organizational-chart.png_

**官方定义**

> 将对象组合成树形结构以表示"部分-整体"的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。

这个组合模式不是“组合优于继承”的那种“组合”，这里是狭义的指代一种特定场景，就是如配图描述的一种树状结构。

理解这个模式要知道三个设定：

* **Component**：一个有功能性的组件。
* **Leaf**：它实现了组件。
* **Composite**：它既实现了组件，他又包含多个组件。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228c624d5b8e2d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```text
protocol Component {
    func someMethod()
}

class Leaf: Component {
    func someMethod() {
        // Leaf
    }
}

class Composite: Component {
    var components = [Component]()
    func someMethod() {
        // Composite
    }
}

let leaf = Leaf()
let composite = Composite()
composite.components += [leaf]

composite.someMethod()
leaf.someMethod()
复制代码
```

这个模式的精髓就是`Composite`这个角色，事实上`Leaf`可以看做一个特殊的`Compostie`。由于他即可以是一个功能执行者，又可以包含其它节点，这个特性可以派生出泛用度很高的树状结构。

## Decorator 装饰者模式

> 如果点咖啡时价格的计算是`牛奶(糖(咖啡(价格: 19元)))`就好了

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="300"%20height="199"></svg>) _配图：http://cdn.marksdailyapple.com/wordpress/wp-content/themes/Marks-Daily-Apple-Responsive/images/blog2/coffee.jpg_

**官方定义**

> 动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活。

下面我们就用装饰者模式的思想来设计这个点咖啡程序:

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="844"%20height="814"></svg>)

### 代码

```text
protocol Component {
    var cost: Int { get }
}

protocol Decorator: Component {
    var component: Component { get }
    init(_ component: Component)
}

struct Coffee: Component {
    var cost: Int
}

struct Sugar: Decorator {
    var cost: Int {
        return component.cost + 1
    }
    var component: Component
    init(_ component: Component) {
        self.component = component
    }
}

struct Milk: Decorator {
    var cost: Int {
        return component.cost + 2
    }
    var component: Component
    init(_ component: Component) {
        self.component = component
    }
}

Milk(Sugar(Coffee(cost: 19))).cost
复制代码
```

当你的需求是零散的不断给“主菜加点佐料”的时候，并且这些佐料会经常变化，那么这个模式就可以有效的解决排列组合产生的类爆炸

理解的一个关键点就是区分**组件Component**和**装饰者Decorator**两个角色，单纯组件的实现者（咖啡）是被装饰的对象，他不再能装饰别人。

这个模式没有一个集中的**计算器**，每一个装饰者都参与了部分计算并输出当下的结果。

## Facade 外观模式

> 外观模式就是化繁为简。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228c845ed667c4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) _配图：http://www.3dmgame.com/news/201709/3684484.html_

**官方定义**

> 为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。

值得一提的是，这个单词读音很怪。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228c87db8d9efa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

在谈起外观模式的时候，常常是指对一个复杂的\(旧\)代码库在不改变其内在的情况下，包装一层易于调用的表层API。

外观模式不会采用继承，而是用接口和组合。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228c8a0ba5d3bd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```text
protocol Facade {
    func simpleMethod()
}

class LegacyCode {
    func someMethod1() { }
    func someMethod2() { }
}

extension LegacyCode: Facade {
    func simpleMethod() {
        someMethod1()
        someMethod2()
    }
}

class Client {
    let f: Facade = LegacyCode()
}

let c = Client()
c.f.simpleMethod()
复制代码
```

## Flyweight 享元模式

> 享元模式就像CPU的Cache Memory，它通过对缓存的命中来实现速度的提升和内存的降低。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228c91362e10ff?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) _配图：http://www.computerhope.com/issues/pictures/cpu\_cache\_die.jpg_

**官方定义**

> 运用共享技术有效地支持大量细粒度的对象。

享元模式其实就是指一套缓存系统。 显然他是一种复合模式，使用工厂模式来创造实例。 适用场景是系统中存在重复的对象创建过程。 好处是节省了内存加快了速度。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228c958468b9bc?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```text
struct TargetObject {
    var title: String?
    func printTitle() {
        print(title)
    }
}

class Cache {
    var targetObjects = [String: TargetObject]()
    func lookup(key: String) -> TargetObject {
        if targetObjects.index(forKey: key) == nil {
            return TargetObject()
        }
        return targetObjects[key]!
    }
}

let c = Cache()
c.targetObjects["Test"] = TargetObject(title: "Test")
c.lookup(key: "123").printTitle() // nil
c.lookup(key: "Test").printTitle() // Test
复制代码
```

## Proxy 代理模式

> 代理模式让一个类成为了另一个类的实际接口。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228c9c8ce18ba2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) _配图：http://tse2.mm.bing.net/th?id=OIP.igizh16RdxH-Xq9jCR7KLgHaCx&w=300&h=112&c=7&o=5&dpr=2&pid=1.7_

**官方定义**

> 为其他对象提供一种代理以控制对这个对象的访问。

有两种常见的代理场景：

* **Protection proxy**： 出于安全考虑，通过一个表层的类间接调用底层的类。
* **Virtual proxy**：出于性能考虑，通过一个低耗的类延迟调用一个高耗的类。

但他们的实现是类似的：

![](https://user-gold-cdn.xitu.io/2018/3/15/16228c9e9eea16e6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```text
protocol Subject {
    mutating func operation()
}

struct SecretObject: Subject {
    func operation() {
        // real implementation
    }
}

struct PublicObject: Subject {
    private lazy var s = SecretObject()
    mutating func operation() {
        s.operation()
    }
}

var p = PublicObject()
p.operation()
复制代码
```

`SecretObject`既可以看做一个隐藏类，又可以看做一个高费的类。通过`PublicObject`对他的代理，可以实现信息隐藏和延迟加载两个特性。

## Chain of responsibility 责任链模式

> 责任链模式就像去取钱，每一面值都参与了计算和职责传递。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="474"%20height="227"></svg>) _配图：http://www.joezimjs.com/wp-content/uploads/chain\_of\_responsibility\_atm.png_

**官方定义**

> 避免请求发送者与接收者耦合在一起，让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止。

UIKit的touch事件就应用了责任链模式

这个模式的实现就是**既实现一个接口，又组合这个接口**，这样自己执行完毕后就可以调用下一个执行者。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="207"%20height="329"></svg>)

```text
protocol ChainTouchable {
    var next: ChainTouchable? { get }
    func touch()
}

class ViewA: ChainTouchable {
    var next: ChainTouchable? = ViewB()
    func touch() {
        next?.touch()
    }
}

class ViewB: ChainTouchable {
    var next: ChainTouchable? = ViewC()
    func touch() {
        next?.touch()
    }
}

class ViewC: ChainTouchable {
    var next: ChainTouchable? = nil
    func touch() {
        print("C")
    }
}

let a = ViewA()
a.touch() // OUTPUT: C
复制代码
```

## Command 命令模式

> 命令模式就是一种指令系统。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="914"%20height="609"></svg>) _配图：http://cdn.wikimg.net/strategywiki/images/thumb/e/e3/Light-bot\_2-3.jpg/914px-Light-bot\_2-3.jpg_

**官方定义**

> 将一个请求封装成一个对象，从而使您可以用不同的请求对客户进行参数化。

命令模式有两个特征：

* 命令逻辑对象化，这个命令对象可以从外界通过参数传入，也可内部实现。
* 支持undo，因为每个命令对象自己知道如何撤销（反命令），所以可以封装到命令对象内部。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="438"%20height="273"></svg>)

```text

protocol Command {
    var operation: () -> Void { get }
    var backup: String { get }
    func undo()
}

struct ConcreteCommand: Command {
    var backup: String
    var operation: () -> Void
    func undo() {
        print(backup)
    }
}

struct Invoker {
    var command: Command
    func execute() {
        command.operation()
    }
    func undo() {
        command.undo()
    }
}

let printA = ConcreteCommand(backup: "Default A") {
    print("A")
}
let i1 = Invoker(command: printA)
i1.execute() // OUTPUT: A

let printB = ConcreteCommand(backup: "Default B") {
    print("B")
}
let i2 = Invoker(command: printB)
i2.execute() // OUTPUT: B
i2.undo() // OUTPUT: Default B
复制代码
```

## Interpreter 解释器模式

> 有了解释器，Skyrim的龙语也不在话下。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="600"%20height="361"></svg>) _配图：https://staticdelivery.nexusmods.com/mods/110/images/thumbnails/32821-3-1362476170.jpg_

**官方定义**

> 给定一个语言，定义它的文法表示，并定义一个解释器，这个解释器使用该标识来解释语言中的句子。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="243"%20height="208"></svg>)

我们可以实现一个能计算文本“1加1”的解释器。

```text
protocol Expression {
    func evaluate(_ context: String) -> Int
}

struct MyAdditionExpression: Expression {
    func evaluate(_ context: String) -> Int {
        return context.components(separatedBy: "加")
            .flatMap { Int($0) }
            .reduce(0, +)
    }
}

let c = MyAdditionExpression()
c.evaluate("1加1") // OUTPUT: 2
复制代码
```

## Iterator 迭代器模式

> 迭代器就像回转寿司，保证每一道菜品都能得到展示。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="400"%20height="227"></svg>) _配图：http://www.sohu.com/a/207894434\_99978064_

**官方定义**

> 提供一种方法顺序访问一个聚合对象中各个元素, 而又无须暴露该对象的内部表示。

迭代器就是能用hasNext和Next来遍历的一种集合元素。

他很像责任链，但是责任链是一个随时可断的链条（有可能在某一个节点不再把责任下放），他不强调一次完整的遍历。迭代器更像一次次的循环，每次循环都强调完整性，所以更适合集合的场景。

还有一个区别是迭代器是提供元素，而责任链在每一个经手人那做业务。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="350"%20height="342"></svg>)

```text
protocol AbstractIterator {
    func hasNext() -> Bool
    func next() -> Int?
}

class ConcreteIterator: AbstractIterator {
    private var currentIndex = 0
    var elements = [Int]()
    

    func next() -> Int? {
        guard currentIndex < elements.count else { currentIndex = 0; return nil }
        defer { currentIndex += 1 }
        return elements[currentIndex]
    }
    
    func hasNext() -> Bool {
        guard currentIndex < elements.count else { currentIndex = 0; return false }
        return elements[currentIndex] != nil
    }
}

protocol AbstractCollection {
    func makeIterator() -> AbstractIterator
}

class ConcreteCollection: AbstractCollection {
    let iterator = ConcreteIterator()
    func add(_ e: Int) {
        iterator.elements.append(e)
    }
    func makeIterator() -> AbstractIterator {
        return iterator
    }
}

let c = ConcreteCollection()
c.add(1)
c.add(2)
c.add(3)

let iterator = c.makeIterator()
while iterator.hasNext() {
    print(iterator.next() as Any)
}
复制代码
```

## Mediator 中介模式

> 中介模式是对象间发消息的传话筒。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="500"%20height="333"></svg>) _配图：http://www.quanjing.com/imgbuy/top-959496.html_

**官方定义**

> 用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。

中介模式对于通信关系复杂的系统有很好的解耦效果

它和观察者模式很像，区别在于观察者是不关心接受方的广播，中介者是介入两个（或多个）对象之间的定点消息传递。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="434"%20height="342"></svg>)

```text
protocol Receiver {
    func receive(message: String)
}

protocol Mediator: class {
    func notify(message: String)
    func addReceiver(_ receiver: Receiver)
}

class ConcreteMediator: Mediator {
    var recipients = [Receiver]()
    func notify(message: String) {
        recipients.forEach { $0.receive(message: message) }
    }
    func addReceiver(_ receiver: Receiver) {
        recipients.append(receiver)
    }
}

protocol Component: Receiver {
    var mediator: Mediator? { get }
}

struct ConcreteComponent: Component {
    weak var mediator: Mediator?
    var name: String
    func receive(message: String) {
        print(name, " receive: ", message)
    }
}

var mediator = ConcreteMediator()

let c1 = ConcreteComponent(mediator: mediator, name: "c1")
let c2 = ConcreteComponent(mediator: mediator, name: "c2")
let c3 = ConcreteComponent(mediator: mediator, name: "c3")

mediator.addReceiver(c1)
mediator.addReceiver(c2)
mediator.addReceiver(c3)

//c1  receive:  hi
//c2  receive:  hi
//c3  receive:  hi
c1.mediator?.notify(message: "hi")
复制代码
```

## Memento 备忘录模式

> 备忘录就是游戏存档。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="1280"%20height="720"></svg>) _配图：http://gearnuke.com/wp-content/uploads/2015/05/witcher3-ps4-savedata-2.jpg_

**官方定义**

> 在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。

这个模式有两个角色，一个是要存储的类型本身\(Memento\)和执行存储操作的保管人\(Caretaker\)。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="524"%20height="221"></svg>)

```text
typealias Memento = [String: String] // chatper: level
protocol MementoConvertible {
    var memento: Memento { get }
    init?(memento: Memento)
}

class GameState: MementoConvertible {
    var memento: Memento {
        return [chapter: level]
    }
    var chapter: String
    var level: String
    
    required init?(memento: Memento) {
        self.chapter = memento.keys.first ?? ""
        self.level = memento.values.first ?? ""
    }
    init(chapter: String, level: String) {
        self.chapter = chapter
        self.level = level
    }
}

protocol CaretakerConvertible {
    static func save(memonto: Memento, for key: String)
    static func loadMemonto(for key: String) -> Memento?
}

class Caretaker: CaretakerConvertible {
    static func save(memonto: Memento, for key: String) {
        let defaults = UserDefaults.standard
        defaults.set(memonto, forKey: key)
        defaults.synchronize()
    }
    
    static func loadMemonto(for key: String) -> Memento? {
        let defaults = UserDefaults.standard
        return defaults.object(forKey: key) as? Memento
    }

}

let g = GameState(chapter: "Prologue", level: "0")
// after a while
g.chapter = "Second"
g.level = "20"
// want a break
Caretaker.save(memonto: g.memento, for: "gamename")
// load game
let gameState = Caretaker.loadMemonto(for: "gamename") // ["Second": "20"]
复制代码
```

## Observer 观察者模式

> 观察者模式就像预购，支付了就坐等收货，省心省力。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="600"%20height="337"></svg>) _配图：http://images.pushsquare.com/news/2015/01/what\_the\_hecks\_going\_on\_with\_playstation\_store\_pre-orders\_in\_europe/attachment/0/original.jpg_

**官方定义**

> 定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

**观察者模式**定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。

观察者模式适用在一个**被观察者（数据源）要通知多个**观察者的场景。

这个模式主要通过添加一层接口，来把观察者的具体类型擦除，从何实现松耦合。（针对接口编程，而非实现）

观察者模式一个最重要的特点就是它是一种推模型\(PUSH\)，在很多情况下比拉模型更高效和即时。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="1220"%20height="580"></svg>)

```text
protocol Observable {
    var observers: [Observer] { get }
    func add(observer: Observer)
    func remove(observer: Observer)
    func notifyObservers()
}

class ConcreteObservable: Observable {
    var observers = [Observer]()
    func add(observer: Observer) {
        observers.append(observer)
    }
    func remove(observer: Observer) {
        if let index = observers.index(where: { $0 === observer }) {
            observers.remove(at: index)
        }
    }
    func notifyObservers() {
        observers.forEach { $0.update() }
    }
}

protocol Observer: class {
    func update()
}

class ConcreteObserverA: Observer {
    func update() { print("A") }
}

class ConcreteObserverB: Observer {
    func update() { print("B") }
}

//////////////////////////////////

let observable = ConcreteObservable()
let a = ConcreteObserverA()
let b = ConcreteObserverB()
observable.add(observer: a)
observable.add(observer: b)
observable.notifyObservers() // output: A B

observable.remove(observer: b)
observable.notifyObservers() // output: A
复制代码
```

## State 状态模式

> 状态模式就像马里奥吃了各种蘑菇后的变身。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="500"%20height="500"></svg>) _配图：http://www.superluigibros.com/images/super\_mario\_bros\_3\_power\_ups.jpg_

**官方定义**

> 允许对象在内部状态发生改变时改变它的行为，对象看起来好像修改了它的类。

状态模式很像策略模式，但是他封装的不是算法，而是一个一个本体下的不同状态。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="451"%20height="221"></svg>)

```text
protocol State {
    func operation()
}

class ConcreteStateA: State {
    func operation() {
        print("A")
    }
}

class ConcreteStateB: State {
    func operation() {
        print("B")
    }
}

class Context {
    var state: State = ConcreteStateA()
    func someMethod() {
        state.operation()
    }
}

let c = Context()
c.someMethod() // OUTPUT: A
c.state = ConcreteStateB() // switch state
c.someMethod() // OUTPUT: B
复制代码
```

## Strategy 策略模式

> 策略模式就像不同的交通路线，可以相互替换，都能达到终点。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="1242"%20height="567"></svg>) _配图：高德地图iOS_

**官方定义**

> 定义一系列的算法,把它们一个个封装起来, 并且使它们可相互替换。

这个模式主要还是在解决**一个稳定的继承树**当遇到**需求变化**时的狼狈。他把**变化**的几种情况分门别类的封装好，然后把自己变化的部分隔离成一个接口实例\(interface/protocol\)，让子类继承后来做选择题。

一个需要转变的思维就是：建模一个类的时候不能只局限于物体（object），也可以是行为（behavior）的封装。

用继承来替换interface实现多态的behavior也是理论可行的。 这个模式还有一个强大之处：他是**动态的**。就是至于子类选择了什么行为，不见得是写代码的时候写死的；也可以是类似用户点击一下切换了算法。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="1014"%20height="960"></svg>)

```text

protocol WeaponBehavior {
    func use()
}

class SwordBehavior: WeaponBehavior {
    func use() { print("sword") }
}

class BowBehavior: WeaponBehavior {
    func use() { print("bow") }
}

class Character {
    var weapon: WeaponBehavior?
    func attack() {  weapon?.use() }
}

class Knight: Character {
    override init() {
        super.init()
        weapon = SwordBehavior()
    }
}

class Archer: Character {
    override init() {
        super.init()
        weapon = BowBehavior()
    }
}

///////////////////////////////////

Knight().attack() // output: sword
Archer().attack() // output: bow
复制代码
```

## Template Method 模板方法模式

> 模板方法就是抽象类衍生出来的继承树。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="500"%20height="400"></svg>) _配图：https://ss0.bdstatic.com/70cFuHSh\_Q1YnxGkpoWK1HF6hhy/it/u=3167143621,3882552175&fm=27&gp=0.jpg_

**官方定义**

> 定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

模板方法指的就是存在在抽象类中的方法声明，由子类具体实现。这样这套方法产生了**一套模具不同产品**的派生效果。

通过抽象类和子类实现多态。

很可惜Swift/Objective-C都没有抽象类, 所以用一般的class来实现

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="231"%20height="221"></svg>)

```text
class Soldier {
    func attack() {} // <-- Template Method
    private init() {} // <-- avoid creation
}

class Paladin: Soldier {
    override func attack() {
        print("hammer")
    }
}

class Archer: Soldier {
    override func attack() {
        print("bow")
    }
}
复制代码
```

## Visitor 访问者模式

> 访问者模式就像一个个酒店，他们都有接待你的接口，你入住后又会利用它的基础设施。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="1280"%20height="720"></svg>) _配图：https://marketingland.com/wp-content/ml-loads/2014/08/hotel-bell-customer-service-ss-1920.jpg_

**官方定义**

> 主要将数据结构与数据操作分离。

号称最复杂的设计模式，先看一下实现代码。

![](data:image/svg+xml;utf8,<?xml%20version="1.0"?><svg%20xmlns="http://www.w3.org/2000/svg"%20version="1.1"%20width="696"%20height="355"></svg>)

```text
protocol Visitor {
    func visit(_ c: ComponentA)
    func visit(_ c: ComponentB)
}

struct ConcreteVisitor: Visitor {
    func visit(_ c: ComponentA) {
        c.featureA()
    }
    func visit(_ c: ComponentB) {
        c.featureB()
    }
}

protocol Component {
    func accept(_ v: Visitor)
}

struct ComponentA: Component {
    func featureA() {
        print("Feature A")
    }
    func accept(_ v: Visitor) {
        v.visit(self)
    }
}

struct ComponentB: Component {
    func featureB() {
        print("Feature B")
    }
    func accept(_ v: Visitor) {
        v.visit(self)
    }
}

let components: [Component] = [ComponentA(), ComponentB()]
components.forEach {
    $0.accept(ConcreteVisitor())
}
复制代码
```

要理解这个模式需要了解一些概念：

通过这篇文章可以了解到， [其实设计模式（GoF\)中的Visitor模式就是Java解决Double Dispatch的一种应用。](https://link.juejin.im?target=http%3A%2F%2Fwww.blogjava.net%2Fchaocai%2Farchive%2F2009%2F02%2F19%2F255640.html)，

我们用Swift实现一下文中双重派发问题

```text
protocol Event {}
class BlueEvent: Event {}
class RedEvent: Event {}

class Handler {
    func handle(_ e: Event) {
        print("Event")
    }
    func handle(_ r: RedEvent) {
        print("RedEvent")
    }
    func handle(_ b: BlueEvent) {
        print("BlueEvent")
    }
}

let b = BlueEvent()
let r = RedEvent()
let c = Handler()

c.handle(b) // OUTPUT: BlueEvent
c.handle(r) // OUTPUT: RedEvent
复制代码
```

验证发现，Swift是支持双重派发的。

那什么是**双重派发（Double Dispatch）**？

要解释这个，要先搞清楚什么是**派发（Dispatch）**。

派发或者说Single Dispatch，Dynamic Dispatch，wiki是这样解释的：

> In most object-oriented systems, the concrete function that is called from a function call in the code depends on the dynamic type of a single object and therefore they are known as single dispatch calls, or simply virtual function calls. 抓要点说，就是多态的情况下，在运行时，虚方法是由哪个具体实现执行了，这样的绑定动作就是派发。

那什么**虚方法**？ 看一下wiki的解释：

> In short, a virtual function defines a target function to be executed, but the target might not be known at compile time. 虚方法就是编译期还没确定实现的方法，我们可以理解成接口中声明的方法。

好，我们现在知道了一次虚方法的绑定就是派发，双重派发就是两次绑定操作。

那双重派发的代码是如何触发派发的呢？

* **overload 重载** 重载产生的多态引起一次派发
* **override 重写** 接口实现的多态引起第二次派发

理解这些就知道这个模式在折腾什么了。

当然他也有另外的好处，就是可以让具体的业务类可以丢到集合里批量的执行accept visitor。  
作者：darrenzheng  
链接：https://juejin.im/post/5aaa2943f265da239c7b15c4  
来源：掘金  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

