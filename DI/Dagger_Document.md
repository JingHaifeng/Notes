
## 声明依赖关系
*Dagger*可以为你应用中的各种类创建实例，同时满足其依赖关系。*Dagger*通过[javax.inject.Inject](http://docs.oracle.com/javaee/7/api/javax/inject/Inject.html)注解来标记所需要的构造函数和字段。

使用`@Inject`标注构造函数意味着*Dagger*可以通过它创建实例。当需要一个新实例时，*Dagger*将获取需要的参数然后调用该构造函数。
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
*Dagger*能直接注入字段。示例中，*Dagger*分别为 *Heater* 和 *Pump* 获取了实例 heater 已经 pump。
```java
class CoffeeMaker {
  @Inject Heater heater;
  @Inject Pump pump;

  ...
}
```
如果你的类中有被@Inject注解的字段，没有被注解的构造函数，那么*Dagger*将在需要的时候注入所需字段，但不会创建该类的新的实例。创建一个被@Inject注解的无参构造函数，表明*Dagger*也可以创建其实例。

尽管构造函数和字段注入通常是首选，但*Dagger*也提供了方法注入。

那些没有被@Inject标记的类无法被*Dagger*构建。

## 满足依赖关系
默认情况下，Dagger能通过构建所需类型的实例来满足所有依赖关系。如示例中，当你需要一个CoffeeMaker实例，Dagger会通过调用 *new CoffeeMaker()* 并设置它注射过的字段，来获取一个实例。

但@Inject并非万能：
* 接口无法被构建
* 第三方类无法被注解
* 可配置对象必须被配置好

因此，在@Inject无法满足情况下，使用@Provides注解的方法来满足依赖。这些方法将返回满足依赖关系的类型。

例如，无论何时,当需要一个Heater实例，*provideHeater()*就会被调用。

```Java
@Provides static Heater provideHeater() {
  return new ElectricHeater();
}
```

@Provides 注解的方法也可以依赖自身。如下，当需要一个Pump实例时，将返回一个Thermosiphon实例。

所有 @Provides 方法必须属于一个模块——被@Module注解的类。

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

按照惯例，@Provides 方法命名带有provide前缀，模块类带有Module后缀。

## 构建图
对象图中@Inject和@Provides注解的类，被它们的依赖关系联系着。在应用的主函数或者 Android Application 中调用代码从根节点集合出发访问图。在Dagger2中，该集合会被定义在一个接口的无参方法中，该方法返回期望的类型。通过使用@Component注解给一个接口，传入模块类型给modules参数，Dagger2然后能自动实现。

```java
@Component(modules = DripCoffeeModule.class)
interface CoffeeShop {
  CoffeeMaker maker();
}
```

接口的实现类带有Dagger前缀。调用*builder()*方法获得建造着实例，设置依赖关系后调用*builder()*获得新实例。

>注：如果@Component不是顶级类，实现的
组件的名字将包含有所有关闭的类型名，中间下划线分割。例如：

```java
class Foo {
  static class Bar {
    @Component
    interface BazComponent {}
  }
}
```

生成的组件叫做 DaggerFoo_Bar_BazComponent。

