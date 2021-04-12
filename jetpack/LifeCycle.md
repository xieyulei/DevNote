## **Jetpack 之 LifeCycle** 

[TOC]

------

**使用关键点在于：**

```java
实现接口LifecycleObserver，添加具体实现类的监听addObserver，重写对应的接口方法@LifeCycle.Event.ON_REUMSE
```



#### **1. 使用意义**

```java
Lifecycle完美解决了组件对页面生命周期的依赖问题，使组件能够自己管理其生命周期，而无须对页面中对其进行管理；
大大的降低了代码的耦合度，提高了组件的复用程度，也杜绝了由于对页面生命周期管理的疏忽而引发的内存泄漏问题.
```

#### **2. LifeCycle诞生的背景**

```java
在Android应用程序开发中，解耦很大程度上表现为系统组件的生命周期与普通组件之间的解耦;
普通组件在使用的过程中要依赖于系统组件的生命周期，有时候不得不在系统组件的生命周期回调方法中，主动对普通组件进行调用或控制，因为普通组件无法主动获知系统组件的生命周期;

// 举例说明
我们经常需要在onCreate()方法中对组件进行初始化，在onPause()方法中停止组件，而在页面的onDestory()方法这种对组件进行资源回收工作，这样的工作非常繁琐，会让页面与组件之间的耦合度更高。但这些工作又不得不做，因为这可能会引发内存泄漏。

所以，我们希望我们对自定义组件的管理，不依赖于页面生命周期的回调方法，同时，在页面看生命周期发生变化时，也能够及时收到通知，为此Google提供了LifeCycle作为解决方案
```

#### **3. LifeCycle可以做什么**

```java
LifeCycle可以帮助开发者创建可感知生命周期的组件; 
这样，组件便能够在其内部管理自己的生命周期，从而降低模块间的耦合度，并降低内存泄漏发生的可能性;
```

#### **4. 使用场景**

```java
LifeCycle不只对Activity/Fragment有用，在Service和Application中也可以使用.
```

#### **5. 原理**

```java
1. Jetpack为我们提供了两个类：LifecycleOwner--被观察者 和 LifecycleObserver--观察者，即通过观察者模式，实现对页面生命周期的监听;

2. AppCompatActivity 继承了 SupportActivity, SupportActivity又继承了Activity，在新版的SDK包中，Activity已经默认实现了LifecycleOwner接口，LifecycleOwner接口中只有一个getLifecycle(LifecycleObserver observer)方法，LifecycleOwner正是通过该方法实现观察者模式的;

3. 对于组件中那些需要在页面生命周期发生变化时得到通知的方法，我们需要在这些方法上使用@onLifecycleEvent(Lifecycle.Event.ON_XXX)标签进行标识，这样当页面生命周期发生变化时，这些被标识过的方法便会被自动调用.
```

#### **6. Activity 与 Fragment中的使用**

```java
//Activity中（继承自AppCompatActivity的活动可以使用下面代码，
//继承Activity的活动需使用另外方式：https://blog.csdn.net/u011810352/article/details/81203095）
在Activity中要做的只是通过 getLifecycle().addObserver()方法，将观察者与被观察者绑定起来
例如：
getLifecycle().observer(new TestLifeCycle()); 

//此处的TestLifeCycle是生命周期监听类
public class TestLifeCycle implements LifecycleObserver {
    private static final String TAG = "test";

    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    public void onCreate() {
        Log.d(TAG, "onCreate: ");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    public void onStart() {
        Log.d(TAG, "onStart: ");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void onResume() {
        Log.d(TAG, "onResume: ");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void onPause() {
        Log.d(TAG, "onPause: ");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    public void onStop() {
        Log.d(TAG, "onStop: ");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    public void onDestroy() {
        Log.d(TAG, "onDestroy: ");
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_ANY)
    public void onAny() {
        Log.d(TAG, "onAny: ");
    }
}


//Fragment中
LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);
getLifecycle().observer(new TestLifeCycle()); 

public Lifecycle getLifecycle(){
	return this.mLifecycleRegistry;
}
```

#### **7. Lifecycle在Service中的使用**

```java
拥有生命周期的组件除了Activity和Fragment,还有Service, 为了方便对Service生命周期的监听，达到解耦Service与组件的目的，Android提供了一个名为LifecycleService的类，该类继承自Service ,并实现了LifecycleOwner接口，与Activity与Fragment类似，它也提供了一个名为getLifecycle()的方法供我们使用。

// 源码：
public class LifecycleService extends Service implements LifecycleOwner {
	...
	
	private final ServiceLifecycleDispatcher mDispatcher = new ServiceLifecycleDispatcher(this);
	
	@override 
	public Lifecycle getLifecycle(){
		return mDispatcher.getLifecycle();
	}

}


//添加依赖
dependencies {
	implementation "androidx.lifecycle:lifecycle-extensions:2.2.0"
}



示例解析：
当Service的生命周期发生变化时，不再需要主动对组件进行通知，组件能够在其内部自行管理好生命周期所带来的变化，使得LifecycleService很好的实现了组件与Service之间的解耦;

public class MyService extends LifecycleService {
	private MyServiceObserver myServiceObserver;
	public MyService(){
		myServiceObserver = new MyServiceObserver();
		getLifecycle().addObserver(myServiceObserver)
	}
}


public class MyServiceObserver implements LifecycleObserver {

	@OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
	private void initData(){
	
	}
	
	@OnLifecycleEvent(Lifecycle.Event.ON_DESTORY)
	private void destoryData(){
	
	}
}

```

#### **8. Lifecycle在Application中的使用**

```java
LifeCycle提供了一个名为ProcessLifecycleOwner的类，以便我们知道整个应用--Application的生命周期情况,其本质也是观察者模式;
ProcessLifecycleOwner是针对整个应用的监听，用ProcessLifecycleOwner类，我们可以清晰知晓应用程序何时退到后台，何时进入前台，进而执行一些业务操作;

当程序从后台到前台，或者应用被首次打开时，会依次调用Lifecycle.Event.ON_START与Lifecycle.Event.ON_RESUME;
当程序从前台退到后台(用户按下Home键或任务菜单键)，会依次调用Lifecycle.Event.ON_PAUSE与Lifecycle.Event.ON_STOP;


// ProcessLifecycleOwner的使用方法
      
//1.依赖添加
dependencies {
    implementation "androidx.lifecycle:lifecycle-extensions:2.2.0"
}    

//2.在自定义的Application类中
public class MyApplication extends Application {
    @override
    public void onCreate(){
        super.onCreate();
        ProcessLifecycleOwner.get().getLifecycle().addObserver(new ApplicationObserver());
    }
}

//3.定义一个ApplicationObserver类，让该类实现LifecycleObserver接口，以负责对应用程序生命周期的监听
public class ApplicationObserver implementation LifecycleObserver {
    private String TAG = this.getClass().getName();
    
    //在应用程序的每个生命周期中只会调用一次
    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    public void onCreate(){
        Log.d(TAG,"Lifecycle.Event.ON_CREATE");
    }
    
    //当应用程序在前台出现时被调用
    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    public void onCreate(){
        Log.d(TAG,"Lifecycle.Event.ON_START");
    }
    
    //当应用程序在前台出现时被调用
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void onCreate(){
        Log.d(TAG,"Lifecycle.Event.ON_RESUME");
    }
    
    //当应用程序退出到后台时被调用
    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void onCreate(){
        Log.d(TAG,"Lifecycle.Event.ON_PAUSE");
    }
    
    //当应用程序退出到后台时被调用
    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    public void onCreate(){
        Log.d(TAG,"Lifecycle.Event.ON_STOP");
    }
    
     //永远不会被调用，系统不会分发调用ON_DESTORY事件
    @OnLifecycleEvent(Lifecycle.Event.ON_DESTORY)
    public void onCreate(){
        Log.d(TAG,"Lifecycle.Event.ON_DESTORY");
    }
}

```
