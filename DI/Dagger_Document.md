
# Daager
Dagger 是一个完全静态地，编译时依赖注入框架，适用于 Java 和 Android。Dagger 改编自 Square 公司创建早期版本，目前由 Google 公司维护。

Dagger 旨在解决基于反射的解决方案引发开发性能的问题。更多细节可以参考[演讲](https://www.youtube.com/watch?v=oK_XtfXPkqw)[幻灯片](https://docs.google.com/presentation/d/1fby5VeGU9CN8zjw4lAb2QPPsKRxx6mSwCe9q7ECNSJQ/pub?start=false&loop=false&delayms=3000)

> DI 简史：Spring->Guice->Dagger1->Dagger2  XML->annotation + reflaction ->compile-time + annitation->compile-time

# 文档
* [用户文档](https://google.github.io/dagger//users-guide)
* [Api 文档](http://google.github.io/dagger/api/latest/)

# 源码
* [Dagger](https://github.com/google/dagger)

# User Guide

应用中能做事的类就是最好的类。比如：`BarcodeDecoder`，`KoopaPhysicsEngine`和`AudioStreamer`。这些类可能依赖于,`BarcodeCameraFinder`，`DefaultPhysicsEngine` 和 `HttpStreamer`。

相反地，应用中最差的类是那些白白占用空间却不做事的类。比如：`BarcodeDecoderFactory`，`CameraServiceLoader`和 `MutableContextWrapper`。那些有意思的东西被这些类粗鲁地绑在一起。

Dagger旨在替换这些工厂的工厂类，避免重复写枯燥无聊的代码模板来实现依赖注入。开发人员能专注在有意义的类上。声明依赖关系，指定满足条件，掌控应用。

建立在 `javax.inject` 注解(JSR 330)的标准之上，每个类都能容易被测试。你不需要一大堆模板就能轻易地讲 `RpcCreditCardService` 替换成 `FaceCreditCardService`。

依赖注入不仅仅是为了方便测试。它还能使创建易用，可移植模块变得容易。你能把同样的 `AuthenticationModule` 分享给你所有的应用。你能在开发中使用`DevLogginModule`，在生产环境中使用 `ProdLoggingModule`，在不同的情境下选择正确的实例。

## 为什么 Dagger 2 是与众不同的

依赖注入框架由来已久，配置和注入存在各式各样的API。为什么要重复造轮子？Dagger 2 是第一个实现全栈使用代码生成的。代码生成的知道原则是：模拟用户手写代码，确保依赖注入简单，易追踪高性能。

## 使用 Dagger 2
我们将通过创建一个 Coffee Maker 来演示 Dagger 2 的依赖注入。完整的例子代码[example](https://github.com/google/dagger/tree/master/examples/simple/src/main/java/coffee)

## 声明依赖关系
*Dagger*可以为你应用中的各种类创建实例，同时满足其依赖关系。*Dagger*通过[javax.inject.Inject](http://docs.oracle.com/javaee/7/api/javax/inject/Inject.html)注解来标记所需要的构造函数和字段。

使用`@Inject`标注构造函数，表明着*Dagger*可以通过它创建实例。当需要一个新实例时，*Dagger*将获取所需的参数然后调用该构造函数。

```java
class Thermosiphon implements Pump {
  private final Heater heater;

  @Inject
  Thermosiphon(Heater heater) {
    this.heater = heater;
  }

  ...
}
```
*Dagger*能直接注入成员。示例中，*Dagger*分别为 *Heater* 和 *Pump* 提供了实例。

```java
class CoffeeMaker {
  @Inject Heater heater;
  @Inject Pump pump;

  ...
}
```

如果你的类中有被@Inject注解的成员，没有被注解的构造函数，那么*Dagger*将在需要的时候注入那些字段，但将不会创建该类的新的实例。创建一个被@Inject注解的无参构造函数，表明*Dagger*也能创建其实例。

*Dagger*也提供了方法注入，但构造函数和字段注入通常是首选。

那些没有被@Inject标记的类无法被*Dagger*构建。

## 满足依赖关系
默认情况下，Dagger能通过构建所需类型的实例来满足所有依赖关系。如示例中，当你需要一个CoffeeMaker实例，Dagger会通过调用 *new CoffeeMaker()* ，将实例注入到所需字段。。

但@Inject并非万能：
* 接口无法被构建
* 第三方类无法被注解
* 可配置对象必须已配置

因此，在@Inject无法满足情况下，使用@Provides注解的方法来满足依赖。这些方法将返回满足依赖关系的类型。

例如，无论何时,当需要一个Heater实例，*provideHeater()*就会被调用。

```Java
@Provides static Heater provideHeater() {
  return new ElectricHeater();
}
```

@Provides 注解的方法也可以依赖自身。如下，当需要一个Pump实例时，将返回一个Thermosiphon实例。

``` java
@Provides static Pump providePump(Thermosiphon pump) {
  return pump;
}
```

所有 @Provides 方法必须属于一个模块——被`@Module`注解的类。

```java
@Module
class DripCoffeeModule {
  @Provides static Heater provideHeater() {
    return new ElectricHeater();
  }

  @Provides static Pump providePump(Thermosiphon pump) {
    return pump;
  }
}
```

按照惯例，@Provides 方法命名带有provide前缀，模块类名带有Module后缀。

## 构建图
对象图中被@Inject和@Provides注解的类，被它们的依赖对象关联着。在应用的主函数或者 Android Application 中调用代码从根节点集合出发访问对象图。在Dagger2中，该集合会被定义在一个接口的无参方法中，该方法返回期望的类型。通过给一个接口使用@Component注解，传入具体的 `Module`类型给`modules`作为参数，之后Dagger2就能按照约定自动生成实现。

```java
@Component(modules = DripCoffeeModule.class)
interface CoffeeShop {
  CoffeeMaker maker();
}
```

接口的实现类带有Dagger前缀。调用*builder()*方法获得建造者实例，设置依赖关系后调用*builder()*获得新实例。

>注：如果@Component不是顶级类，实现的
组件的名字将包含有所有封闭类型名，中间下划线分割。例如：

```java
class Foo {
  static class Bar {
    @Component
    interface BazComponent {}
  }
}
```

生成的组件叫做 `DaggerFoo_Bar_BazComponent`。

任何具有默认构造器的 `Module` 都能被隐藏，因为如果没有设置，构建起将自动构建一个实例。对任何使用了 `@Provides` 方法的 `Module` 都应是 `static` 静态的，这个实现类不需要任何实例。如果所有的依赖都用户不用自己创建依赖实例，那么这实现将会有一个 `create()` 方法用来获取信得实例，不用再使用 `builder`。

现在图已经构建好，入口已被注入，我们能运行我们的 coffee maker app。Fun。

```
$ java -cp ... coffee.CoffeeApp
~ ~ ~ heating ~ ~ ~
=> => pumping => =>
 [_]P coffee! [_]P
 ```

### 图中的绑定关系

上面的例子展示了怎样构建一个典型的绑定组件，但还有多种的方式来帮助绑定图。以下几条可用于依赖关系，生成一个良好的组件：

* Those declared by `@Provides` methods within a @Module referenced directly by `@Component.modules` or transitively via `@Module.includes`
* 任何一个带有 `@Inject` 标记的构造函数要么是无范围的，或者是与组件相同 `@Scope` 范围的。
* The [component provision methods](https://google.github.io/dagger/api/latest/dagger/Component.html#provision-methods) of the [component dependencies](https://google.github.io/dagger/api/latest/dagger/Component.html#dependencies--)
* The component itself
* Unqualified builders for any included subcomponent
* Provider or Lazy wrappers for any of the above bindings
* A Provider of a Lazy of any of the above bindings (e.g., Provider<Lazy<CoffeeMaker>>)
* A MembersInjector for any type

## 单例 和 范围绑定

使用 `@Singleton` 注解一个 `@Provides` 方法或注入类。这注入图将会为所有的客户端提供同一个实例。

``` java
@Provides @Singleton static Heater provideHeater() {
  return new ElectricHeater();
}
```

注入类的`@Singleton` 注解也有文档的作用。提醒用户注意这个类可能会被多个线程共享。

``` java
@Singleton
class CoffeeMaker {
  ...
}
```

 由于 Dagger 2 图中有范围的实例与组件实现的实例关联了起来，所以组件本身需要声明他们将要作用的范围。例如，同一组件同时绑定 `@Singleton` 和 `@RequestScoped` 是没有意义的，因为这些范围有不同的生命周期，因此必须存在不同生命周期的组件中。 要声明一个关联范围的组件，直接地给组件接口添加范围注解即可。

 ``` java
@Component(modules = DripCoffeeModule.class)
@Singleton
interface CoffeeShop {
  CoffeeMaker maker();
}
 ```

 组件可能会应用于多个范围注解。This declares that they are all aliases to the same scope, and so that component may include scoped bindings with any of the scopes it declares.

 ## 可重用范围

 有时，你想限制 `@Inject`类构建的实例个数或限制 `@Provides`方法调用的次数，但你不需要保证特定的组件和子组件再生命周期中精确地使用同一个实例。在 Android 这种分配内存很昂贵的环境中，这会很有用。

 >> to continue