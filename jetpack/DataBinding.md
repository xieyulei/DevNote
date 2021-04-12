### **Jetpack 之 DataBinding**

------



[TOC]

------



##### **1.开启DataBinding**

```java
build.gradle(app)--->

android {
    defaultConfig {
			......       
    }

    buildFeatures {
        dataBinding true
    }

}


```



##### **2.使用SavedStateHandle依赖添加**

```java
implementation 'androidx.lifecycle:lifecycle-extensions:2.2.0'
```



##### **3.ActivityMainBinding类无法生成**

```java
首先，检查DataBinding 是否开启
其次，检查布局文件，是否由原始状态，切换为<layout></layout>标签模式

 同步、Make 、Rebuild尝试
```



##### **4.ViewModel获取**

```
mainViewModel = new ViewModelProvider(this, new SavedStateViewModelFactory(getApplication(), this)).get(MainViewModel.class);
```



##### **5.继承AndroidViewModel**

```java
继承AndroidViewModel 可以无需往构造参数中传入context，在ViewModel中可以自由的使用getApplication()方法，且不会因为Context导致内存泄漏的问题。
```

