## 概要
帮助构建 Clean MVP 应用，`ViewState` 和 `retaining Presenters` 能帮助控制屏幕翻转。
仅仅是一个库，不是框架。虽然有提供MvpFragment等可以继承的类，但本质上是基于代理的轻型库。根据需求使用 MvpFragment 或者自定义代理，不会有框架的边界的限制。

## 依赖关系

Mosby 是模块独立的。可以自由选择依赖的模块：

``` groovy
dependencies {
	compile 'com.hannesdorfmann.mosby:mvp:2.0.1'
	compile 'com.hannesdorfmann.mosby:viewstate:2.0.1'
}
```

## 开始

