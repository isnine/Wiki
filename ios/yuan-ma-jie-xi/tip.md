---
description: 阅读源码时觉得一些很有意思的写法
---

# tip

加锁

```swift
internal final class SpinLock {
    private let lock = NSRecursiveLock()

    func sync<T>(action: () -> T) -> T {
        lock.lock()
        defer { lock.unlock() }
        return action()
    }
}
```

Swinject这个库中，需要在register的时候把类的构造方法传进去，使用的时候再传参调用这个构造方法，那么势必有些类的构造方法是需要多个参数的。

```swift
  // 自己实现一个基本的Container
class MyContainer {
    var services = [ServiceKey: Any]()
    
    fileprivate func _resolve<Service, Arguments>(
        invoker: @escaping ((Arguments) -> Service) -> Any
        ) -> Service? {
        var resolvedInstance: Service?
        let key = ServiceKey(serviceType: Service.self)
        
        if let entry = services[key]  {
            resolvedInstance = resolve(entry: entry, invoker: invoker)
        }
        
        return resolvedInstance
    }
    fileprivate func resolve<Service, Factory>(
        entry: Any,
        invoker: (Factory) -> Any
        ) -> Service? {
        let resolvedInstance = invoker(entry as! Factory)
        return resolvedInstance as? Service
    }
    fileprivate func _register<Service, Arguments>(
        _ serviceType: Service.Type,
        factory: @escaping (Arguments) -> Service)
    {
        let key = ServiceKey(serviceType: serviceType)
        services[key] = factory
    }
}

extension MyContainer: Register {
    public func register<Service>(
        _ serviceType: Service.Type,
        factory: @escaping (Resolver) -> Service) {
        _register(serviceType, factory: factory)
    }
    
    public func register<Service, Arg1>(
        _ serviceType: Service.Type,
        factory: @escaping (Resolver, Arg1) -> Service)
    {
        _register(serviceType, factory: factory)
    }
}

extension MyContainer: Resolver {
    public func resolve<Service>(_ serviceType: Service.Type) -> Service? {
        return _resolve() { (factory: (Resolver) -> Any) in factory(self) }
    }
    
    func resolve<Service, Arg1>(_ serviceType: Service.Type, argument: Arg1) -> Service? {
        typealias FactoryType = ((Resolver, Arg1)) -> Service
        return _resolve(){ (factory: FactoryType) -> Any in return factory((self, argument)) }
    }
}

public protocol Resolver {
    func resolve<Service>(_ serviceType: Service.Type) -> Service?
    
    func resolve<Service, Arg1>(
        _ serviceType: Service.Type,
        argument: Arg1) -> Service?
}

public protocol Register {
    func register<Service>(
        _ serviceType: Service.Type,
        factory: @escaping (Resolver) -> Service)
    
    func register<Service, Arg1>(
        _ serviceType: Service.Type,
        factory: @escaping (Resolver, Arg1) -> Service)
}


```

