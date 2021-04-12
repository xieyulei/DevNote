## **Jetpack 之 ViewModel** 

[TOC]

------

#### **1. 使用ViewModel的意义**

```java
ViewModel可以帮助我们更好地将页面与数据从代码层面上分离开来，更重要的是，依赖于ViewModel的生命周期特性，我们不再需要关心屏幕旋转带来的数据丢失问题，进而也不需要重新获取数据.
```

#### **2. ViewModel是什么**

```java
ViewModel是专门用于存放应用程序页面所需的数据,是View(视图)和Model（模型）之间的桥梁，可以使视图和数据既能够分离开，也能够保持通信。
```

#### **3. 屏幕旋转对ViewModel的影响**

```java
ViewModel独立与配置变化，屏幕旋转所导致的 Activity重建，并不会影响ViewModel的生命周期。
```

#### **4. 依赖添加**

```java
dependencies {
	implementation "androidx.lifecycle:lifecycle-viewmodel:2.2.0"
}
```

#### **5. 实例的获取**

```java
DemoViewModel demoViewModel = new ViewModelProvider(this).get(DemoViewModel.class);
```

#### **6. ViewModel中使用Context**

```java
在使用ViewModel时，不能将任何类型的Context或含有Context引用的对象传入ViewModel,因为这可能导致内存泄漏。

如果想要在ViewModel中使用Context，可以使用AndroidViewModel类，它继承自ViewModel,并接收Application作为Context,这意味着它的生命周期和Application是一样的，那么这就不算是一个内存泄漏了.
    
// 建议：
不要向ViewModel中传入Context或带有Context引用的对象，如果必须要使用Context,就让ViewModel类继承AndroidViewModel,传入ApplicationContext   
    
```

#### **7. ViewModel的原理**

```java
通过ViewModel的实例构造，我们可以看出ViewModelProvider接收一个ViewModelStoreOwner对象作为参数，
在上面的实例获取中，this代表当前Activity，这是因为我们的Activity继承FragmentActivity,
而在androidx的依赖包中，FragmentActivity默认实现了ViewModelStoreOwner接口;

从ViewModelStore的源码可以看出，ViewModel实际上是以HashMap<String,ViewModel>的形式被缓存起来了;
ViewModel与页面之间没有直接的关联，它们通过ViewModelProvider索要，ViewModelProvider检查该ViewModel是否已经存在于缓存中，
若存在则直接返回，若不存在则实例化一个;
因此，Activity由于配置变化导致的销毁重建并不会影响ViewModel，ViewModel是独立于页面而存在的;（所以不要向ViewModel中传递任何类型的Context）
```

#### **8. Fragment中是否可以使用ViewModel ？**

```java
androidx依赖包中的Fragment也默认实现了ViewModelStoreOwner接口，因此，我们也可以在Fragment 中正常使用ViewModel .
```

#### **9. ViewModel 和 OnSaveInstanceState()方法**

```java
// 区别1：保存数据的多少
OnSaveInstanceState()方法只能保存少量且能支持序列化的数据，而ViewModel没有这个限制，ViewModel能支持页面中所有的数据;

// 区别2：是否可以支持数据持久化
ViewModel不支持数据的持久化，当界面被彻底的销毁时，ViewModel及其持有的数据就不复存在了，但是OnSaveInstanceState()方法没有这个限制，它可以持久化页面的数据;
```

