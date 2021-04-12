### **Jetpack 之 LiveData** 

[TOC]

------

##### **1. 为什么使用LiveData？**

```java
ViewModel用于存放页面数据，当数据发生变化时，通知页面进行更新，而LiveData大部分时候是在ViewModel中使用的，也可以搭配Room使用;
使用LiveData对ViewModel中的我们关心的数据进行包装，并在页面中对其进行监听，当数据发生变化时，页面就能收到通知，进而更新UI;
LiveData的本质是观察者模式，并能感知页面的生命周期，只有在页面存活的时候才会进行通知，从而避免内存泄漏。
```

##### **2. LiveData出现**

```java
LiveData通常被放在ViewModel中使用，用于包装ViewModel中那些需要被外界观察的数据。通常境况下，在数据发生变化的时候，经常采用接口的方式对页面进行通知以及更新，但是如果观察的数据很多，则需要定义大量的接口，代码会显得十分冗余，因此Jetpack提供了LiveData组件.
```

##### **3. LiveData简介**

```java
LiveData是一个可被观察的数据容器类，具体来说，可以将LiveData理解为一个数据的容器，它将数据包装起来，使数据成为被观察者，当该数据发生变化时，观察者能够获得通知，并且我们不需要自己去实现观察者模式，LiveData内部已经默认实现好了，我们只需使用即可.
```

##### **4. LiveData与ViewModel的关系**

```java
ViewModel用于存放页面所需要的数据，还可以在其中放置一些与数据相关的业务逻辑，例如数据的获取和加工等操作;

LiveData的作用就是在ViewModel中数据发生变化时通知页面，因此LiveData被放在ViewModel中使用，用于包装ViewModel中那些需要被外界观察的数据;
```

##### **5. LiveData的基本用法**

```java

LiveData是一个抽象类，不能直接使用，我们一般使用它的直接子类MutableLiveData

// 例如，声明一个可供外部观察的int类型变量：currentScore
public class DemoViewModel extends ViewModel{    
	private MutableLiveData<Integer> currentScore ;
	public LiveData<Integer> getCurrentScore(){
		if(currentSocre == null){
        	currentScore = new MutableLiveData<>();
    	}
    	return currentScore;
	}
}

// 与页面之间建立通信
public class DemoActivity extends AppcompatActivity{
    @Override
    protected void onCreate(Bundle savedInstanceState){
        // 获取ViewModel
        DemoViewModel viewModel = new ViewModelProvider(this).get(DemoViewModel.class);
        
        // 得到ViewModel中的LiveData
        final MutableLiveData<Integer> liveData = (MutableLiveData<Integer>)viewModel.getCurrentScore();
        
        //通过LiveData.observe()观察ViewModel中数据的变化
        liveData.observe(this,new Observer<Integer>){
            @Override
            public void onChanged(@Nullable Integer second){
                //数据发生变化后，在这里进行处理，更新页面
            }
        }
    }
}

```

##### **6. LiveData的原理**

```java
LiveData中observe()方法接收的第一个参数玩是LifecycleOwner对象，第二个参数是Observer对象;
LiveData能够感知页面的生命周期，它可以检测页面当前的状态是否为激活状态，或者页面是否被销毁;
只有在页面处于激活状态（START或RESUME）时，页面才能够收到来自LiveData的通知，若页面被销毁（ONDESTORY）,
那么LiveData会自动清除与页面的关联，从而避免可能引发的内存泄漏问题；
```

##### **7. LiveData.oberverForever方法**

```java
使用方法和LiveData没有太大差别，主要区别在于：当LiveData所包装的数据发生变化时，无论页面处于什么状态，observeForever()都能收到通知;
因此用完observeForever后，一定要记得调用removeObserver()方法来停止对LiveData的观察，否则LiveData会一直处于激活状态，Activity则永远不会被系统自动回收，这就造成了内存泄漏。
```































