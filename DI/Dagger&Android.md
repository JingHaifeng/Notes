

比起其他的DI框架，Dagger2 最主要的一个优势是自动生成实现代码（没有使用反射），这意味着 Dagger2 能被应用于 Android 应用。但在将 Dagger2 应用于 Android 之前，还需要理解一些注意事项。

## 哲学

由于 Android 使用 Java 语言写成，就代码风格而言，存在很多不同。通常，存在这些差异是为了适应移动平台的性能需求。

但许多 Java 常用的设计模式，对 Android 起了反作用。甚至许多 Effective Java 中的建议，在 Android 中都是不适用的。

为了得到既易读又易移植的代码，Dagger2 依赖 ProGuard 对编译后的字节码做了后续处理。这使得 Dagger2 生成的代码在服务器和 Android 端都很自然，同时使用不同的工具链生成的字节码在两端都能有效率地执行。此外，Dagger2 还确保生成的 Java 源码能够兼容 ProGuard 优化。

当然，不是所有问题都能这样解决，但这种思路能够提供相应的 Android 兼容能。

## tl;dr

Dagger 假设用户都会使用 ProGuard。

## 推荐的 ProGuard 设置

关注此处，了解使用 Dagger 后应用的 ProGruard 相关设置。

### dagger.android

在 Android 程序中使用 Dagger ，最核心的难点之一是：Android 大部分的框架类都是 OS 自身实例化的，例如 Activity，Fragment。 但如果 Dagger 能穿件所有注入的对象，那么它能很好的工作。因此，你必须再生命周期方法中执行成员注入。这意味着许多类最终将看起来像下面：

``` java
public class FrombulationActivity extends Activity {
  @Inject Frombulator frombulator;

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // DO THIS FIRST. Otherwise frombulator might be null!
    ((SomeApplicationBaseType) getContext().getApplicationContext())
        .getApplicationComponent()
        .newActivityComponentBuilder()
        .activity(this)
        .build()
        .inject(this);
    // ... now you can write the exciting code
  }
}
```

这会有以下问题：

复制粘贴的代码会使得它很难重构。随着越来越多了的开发者拷贝这个代码块，很少会有人知道它的实际作用。

更重要的是，它需要了解注入器的类型然后再请求注入。就算是通过接口而不是通过具体类型来完成注入，这也打破了 DI 的核心规则：一个类不能知道它是如何注入的。

`dagger.android` 提供了简化这一模式的类。

## 注入 Activity 对象

1. 在你的 Application 组件中安装 `AndroidInjectionModule` 确保所有绑定的基础类型是可用的。

2. 开始：写一个 `@Subcomponent` 实现 `AndroidInjector<YourActivity>`，包含一个 `@Subcomponent.Builder` 继承自 `AndroidInjector.Builder<YourActivity>`:

``` java
@Subcomponent(modules = ...)
public interface YourActivitySubcomponent extends AndroidInjector<YourActivity> {
  @Subcomponent.Builder
  public abstract class Builder extends AndroidInjector.Builder<YourActivity> {}
}
```
3. 定义了子组件后，通过定义一个`Module`绑定这些子组件，然后添加 `Module` 给 注入到 Application 中的组件中。

``` java
@Module(subcomponents = YourActivitySubcomponent.class)
abstract class YourActivityModule {
  @Binds
  @IntoMap
  @ActivityKey(YourActivity.class)
  abstract AndroidInjector.Factory<? extends Activity>
      bindYourActivityInjectorFactory(YourActivitySubcomponent.Builder builder);
}

@Component(modules = {..., YourActivityModule.class})
interface YourApplicationComponent {}
```

>Pro-tip: 如果你的`subcomponent`和它的 `builder` 与 #2 相比没有其他的方法或超类，你可以使用 `@ContributesAndroidInjector` 来生成其他代码 。添加一个 `abstract method` 返回你的 activity，添加注解 `@ContributesAndroidInjector`，指定这个 `module`给你想注入的 `subcomponent`。如果 `subcomponent` 需要范围，也可以使用范围注解。

```java
@ActivityScope
@ContributesAndroidInjector(modules = { /* modules to install into the subcomponent */ })
abstract YourActivity contributeYourActivityInjector();
```

4. 使你的 Application 实现 `HasActivityInjector` 接口，然后通过 `@Inject` 获得 `DispathingAndroidInjector<Activity>`实例， 然后再 `activityInjector()`方法返回。

```java
public class YourApplication extends Application implements HasActivityInjector {
  @Inject DispatchingAndroidInjector<Activity> dispatchingActivityInjector;

  @Override
  public void onCreate() {
    super.onCreate();
    DaggerYourApplicationComponent.create()
        .inject(this);
  }

  @Override
  public AndroidInjector<Activity> activityInjector() {
    return dispatchingActivityInjector;
  }
}
```

5. 最后，在你的`Activity.onCreate()`方法中，`super.onCreate()`前调用 `AndroidInjection.inject(this)`

``` java
public class YourActivity extends Activity {
  public void onCreate(Bundle savedInstanceState) {
    AndroidInjection.inject(this);
    super.onCreate(savedInstanceState);
  }
}
```

恭喜，配置成功！

它是如何生效的？

`AndroidInjection.inject()` 从Application中获得`DispatchingAndroidInjector<Activity>`实例，然后传入当前 Activity 给 inject(Activity)。`DispatchingAndroidInjector`会再 AndroidInjector.Factory 中查找这个Activity对应的类（`YourActivitySubcomponent.Builder`），创建一个 `AndroidInjector`（`YourActivitySubcomponent`）然后传递 `Activity` 给 `inject(YourActivity)`。

注入 Fragment 对象

注入一个 Fragment 和注入一个 Activity 一样简单。相同的方法定义一个你的 `subcomponent`，替换参数`Activity`为`Fragment`，`@ActivityKey`为`@FragmentKey`，`HasActivityInjector` 为 `HasFragmentInjector`。 


在 `Fragment` 的 `onAttact()` 中注入，就像 `Activity` 中的 `onCreate()` 方法中一样。

与 `Acitivity` 定义的 `Module` 不同，你可以为 `Fragment` 选择性地安装 `Module`。你能创建你的 Fragment component 是 Fragment/Activity/Application 的子组件——它取决于你的 Fragment 需要绑定什么。决定 `component` 的位置后，实现 `HasFragmentInjector`。例如，如果你的 Fragment 需要绑定 `YourActivitySubcomponent`，你可以如下编码：

``` java
public class YourActivity extends Activity
    implements HasFragmentInjector {
  @Inject DispatchingAndroidInjector<Fragment> fragmentInjector;

  @Override
  public void onCreate(Bundle savedInstanceState) {
    AndroidInjection.inject(this);
    super.onCreate(savedInstanceState);
    // ...
  }

  @Override
  public AndroidInjector<Fragment> fragmentInjector() {
    return fragmentInjector;
  }
}

public class YourFragment extends Fragment {
  @Inject SomeDependency someDep;

  @Override
  public void onAttach(Activity activity) {
    AndroidInjection.inject(this);
    super.onAttach(activity);
    // ...
  }
}

@Subcomponent(modules = ...)
public interface YourFragmentSubcomponent extends AndroidInjector<YourFragment> {
  @Subcomponent.Builder
  public abstract class Builder extends AndroidInjector.Builder<YourFragment> {}
}

@Module(subcomponents = YourFragmentSubcomponent.class)
abstract class YourFragmentModule {
  @Binds
  @IntoMap
  @FragmentKey(YourFragment.class)
  abstract AndroidInjector.Factory<? extends Fragment>
      bindYourFragmentInjectorFactory(YourFragmentSubcomponent.Builder builder);
}

@Subcomponent(modules = { YourFragmentModule.class, ... }
public interface YourActivityOrYourApplicationComponent { ... }
```

## 基本框架类型

因为 `DispatchingAndroidInjector` 会在运行时查找 `AndroidInjector.Factory`。可以实现 `HasActivityInjector/HasFragmentInjector/etc`，然后调用 `AndroidInjection.inject()`。所有的自雷需要绑定对应的 `@Subcomponent`。Dagger 提供一些基本的类型来简化这些：`DaggerActivity/DaggerFragment/DaggerApplication`。

包含如下类型：

>DaggerService and DaggerIntentService
DaggerBroadcastReceiver
DaggerContentProvider

*注*：`DaggerBroadcastReceiver` 应该只能在 `AndroidManifest.xml` 中注册后才能使用。当在代码中动态生成时，最好使用构造器注入来替换。

Support libraries


使用 Android support library 。注意：使用 support Fragment 时，必须绑定 `AndroidInjector.Factory<? extends android.support.v4.app.Fragment>`，AppCompat 需要继续实现 `AndroidInjector.Factory<? extends Activity> and not <? extends AppCompatActivity> (or FragmentActivity)`。

How do I get it?

## 怎样得到它

Add the following to your build.gradle:

dependencies {
  compile 'com.google.dagger:dagger-android:2.x'
  compile 'com.google.dagger:dagger-android-support:2.x' // if you use the support libraries
  annotationProcessor 'com.google.dagger:dagger-android-processor:2.x'
}

何时注入

无论何时构造器注入是最好的方式，因为 javac 讲确保成员被引用，避免空指针错误。当成员需要成员注入时，越早注入越好。因此，`DaggerActivity` 调用 `AndroidInjection.inject()` 直接再 `OnCreate()` 中 `super.onCreate()` 前调用。DaggerFragment 同样地，再 `onAttach()` 注入，防止 Fragment reattached 出现前后不一致。

It is crucial to call AndroidInjection.inject() before super.onCreate() in an Activity, since the call to super attaches Fragments from the previous activity instance during configuration change, which in turn injects the Fragments. In order for the Fragment injection to succeed, the Activity must already be injected. For users of ErrorProne, it is a compiler error to call AndroidInjection.inject() after super.onCreate().

在 `super.onCreate()` 之前调用 `AndroidInjection.inject()` 是至关重要的。由于在 configuration change 时，会再 Activity 实例化前 调用 acttaches Fragments。为了确保 Fragment 注入成功，Activity 必须已经被注入。

FAQ

Scoping AndroidInjector.Factory

AndroidInjector.Factory is intended to be a stateless interface so that implementors don’t have to worry about managing state related to the object which will be injected. When DispatchingAndroidInjector requests a AndroidInjector.Factory, it does so through a Provider so that it doesn’t explicitly retain any instances of the factory. Because the AndroidInjector.Builder implementation that is generated by Dagger does retain an instance of the Activity/Fragment/etc that is being injected, it is a compile-time error to apply a scope to the methods which provide them. If you are positive that your AndroidInjector.Factory does not retain an instance to the injected object, you may suppress this error by applying @SuppressWarnings("dagger.android.ScopedInjectoryFactory") to your module method.