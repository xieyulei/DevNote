## **Jetpack 之 Navication** 

[TOC]

------

#### **1. Navigation 诞生的背景**

```java
大多数Android工程师，目前采用单个Activity嵌套多个Fragment的UI架构模式，但是对于Fragment的管理一直是比较麻烦的，例如：
1.使用FragmentManager和FragmentTranslation来管理Fragment之间的切换
2.应用程序Appbar的管理
3.动画的切换
4.参数的传递
纯代码的方式使用起来不是特别友好，并且App bar在管理和使用的过程中显得很混乱;
因此，Jetpack提供了一个名为Navigation的组件，旨在方便我们管理页面和App bar.

```

#### **2. Navigation的优势**

```java
1.可视化的页面导航图，方便我们理清页面之间的关系
2.通过destination和action完成页面间的导航
3.方便添加页面切换动画
4.页面间类型安全的参数传递
5.通过Navigation UI类，对菜单/底部导航/抽屉蓝菜单导航进行统一的管理
6.支持深层链接DeepLink
```

#### **3. Navigation 的主要元素**

```java
如果想切换Fragment时，使用NavController对象，告诉它你想要去的Navigation Graph中的哪个Fragment，NavController会将你想去的Fragment展示在NavHostFragment中。

Navigation Graph：这是一种新型的XML资源文件，其中包含应用程序所有的页面，以及页面之间的关系;
NavHostFragment:这是一个特殊的Fragment,是Fragment的容器，Navigation Graph中的Fragment是通过NavHostFragment展示的;
NavController:这是一个Java/Kotlin对象，用于在代码中完成Navigation Graph中具体的页面切换工作。
```

#### **4. Navigation的基本使用**

```java
// 1.创建Navigation Graph
新建Android项目, res --> New --> Android Resource File,新建一个Navigation Graph文件;
将File Name设置为nav_graph, Resource type设置为 Navigation ,点击OK，然后按照提示添加对应的依赖即可;

// 2.添加NavHostFragment
在Activity对应的布局文件中，Design --> Containers --> NavHostFragment,拖动其到布局文件中，表示添加

属性分析：    

	//name 表示告知系统，这是一个特殊的Fragment    
	android：name="androidx.navigation.fragment.NavHostFragment" 
    
	//defaultHost 属性为true,则该Fragment会自动处理系统返回键，即当用户按下手机的返回按钮时，系统将当前展示的Fragment退出;
	app:defaultNavHost="true"    
    
	//navGraph 用于设置该Fragment对应的导航图
	app:navGraph="@navigation/nav_graph"        
    
// 3.创建destination

	在Design页面依次 点击加号，"Create new destination"按钮，创建一个destination;
	
	destination:表示目的地，代表想去的页面，可以是Activity或Fragment,但常见的是Fragment,因为Navigation组件的作用是方便开发者在一个Activity中管理多个Fragment;
        
	// startDestination:表示当前Activity的起始页面
	app:startDestination="@id/mainFragment"

// 4.完成Fragment页面切换
        
    第一步：Design页面拖拽当前Frgament的右边框，拖着线条连接目标Fragment的左边框，建立Fragment之间的链接      //action标签 包含id 和 目的地Fragment
         
// 5.使用NavController完成导航
	点击View切换Fragment
        
	方法1：
        view.findViewById(R.btn1).setOnClickListener(new View.OnClickListener()
			@override
			public void onClick(View v){
                //此处的id就是上一步建立Fragment链接后，action标签中的id
                Navigation.findNavController(v).
                    navgate(R.id.action_mainFragment_to_SecondFragment);
            }                                                     
        );
        
	方法2：        
        view.findViewById(R.id.btn1).setOnClickListener(Navigation.
        	createNavigateOnClickListener(R.id.action_mainFragment_to_SecondFragment));

// 6.添加页面切换动画效果
	Design页面，点击 两个Fragment之间链接的线条，在右边的菜单栏中，点击Animation，添加对应的页面切换动画;

```

#### **5. 页面之间参数的传递**

```java
Fragment的切换经常伴随着参数的传递，为了配合Navigation组件在切换Fragment时传递参数，Android Studio为开发者提供了safe args插件.

// 1.常见的参数传递方式
    
    // 传递参数
	Bundle bundle = new Bundle();
	bundle.putString("user_name","Lucy");
	bundle.putInt("age",20);
	Navigation.findNavController(v).navigate(R.id.action_mainFragment_to_SecondFragment);

	// 接收参数
	Bundle bundle = new Bundle();
	if(bundle != null){
        String userName = bundle.getString("user_name");
        int age = bundle.getInt("age");
    }
    
// 2.使用safe args传递参数    
	
	// 安装插件：
        dependencies {
        	def nav = "2.3.0-alpha01"
			classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$nav"                
    	}
	
	// 引用插件：
        app plugin: 'androidx.navigation.safeargs'
            
	// 添加参数：在导航图中添加<argument/>标签，可以直接编写xml代码，也可以直接通过Design面板进行添加
            //name:传递的参数key   argType:参数的类型   defaultValue:默认值
		添加argument标签之后，便可以在app/generatedJava目录下看到safe args插件为我们生成的代码，
        这些代码文件中包含了参数所对应的Getter/Setter方法.            
            
	// 传递参数：
	Bundle bundle = new MainFragmentArags.Builder()
            .setUserName("Lucy")
            .setAge(30)
            .build().toBundle();
	Navigation.findNavController(v).navigate(R.id.action_mainFragment_to_SecondFragment);

	// 接收参数：
	Bundle bundle = getArguments();
	if(bundle != null){
		String userName = MainFragmentArgs.fromBundle(getArguments()).getUserName();
        int age = MainFragmentArgs.fromBundle(getArguments().getAge();
  	}                                              
```

#### **6. 使用NavigationUI切换导航以及App bar**

```java
导航图是Navigation组件中很重要的一部分，它可以帮助我们快速了解页面之间的关系，再通过NavController完成页面的切换工作，而在页面的切换过程中，往往伴随着App bar中菜单的变化。对于不同的页面，App bar中的menu菜单很可能是不一样的，App bar中的各种按钮和菜单，同样承担着页面切换的工作。

例如，当ActionBar左边的返回按钮被点击时，我们需要响应该事件，返回到上一个页面;既然Navigation和App bar都需要处理页面切换事件，那么为了方便管理，Jetpack引入了Navition UI组件，使 App bar中的按钮和菜单能够与导航图中的页面关联起来.

// 1.书写menu文件
    menu --> 书写item,注意item所对应要跳转的Fragment,item的id要与关系图中对应Fragment的id相同，表示单击此item后会跳转到id所对应的Fragment;

// 2.activity中实例化菜单
	@override
	public boolean onCreateOptionsMenu(Menu menu){
        super.onCreateOptionsMenu(menu);
        getMenuInflater().inflate(R.menu.main_menu,menu);
        return true;
    }

// 3.响应菜单点击事件
	public class MainActivity extends AppCompatActivity{
        private AppBarConfiguration appbarConfiguration;
        private NavController navController;
        
        @Override
        protected void onCreate(Bundle savedInstanceState){
            navController = Navigation.findNavController(this,R.id.nav_host_fragment);
            appBarConfiguration = new AppBarConfiguration.Builder(navController.getGraph()).build();
            
            //将App bar与 NavController绑定起来
            NavigationUI.setupActionBarWithNavController(this,navController,appBarConfiguration);
		}
        
        @Override
        public boolean onOptionsItemSelected(MenuItem item){
            return NavigationUI.onNavDestinationSelected(item,navController) || super.onOptionsItemSelected(item);
        }
        
        
        //覆盖onSupportNavigateUp方法，当点击返回按钮时，NavigationUI可以帮助我们返回上一个页面
        @Override
        public boolean onSupportNavigateUp(){
            return NavigationUI.navigateUp(navController,appBarConfiguration) || super.onSupportNavigateUp();
        }
        
    }

// 4.通过App bar跳转到下一个页面后,清除之前Fragment对应的menu
	public class SecondFragment extends Fragment{
        
        @Override
        public void onCreateOptionsMenu(Menu menu,MenuInflater inflater){
            menu.clear();
            super.onCreateOptionsMenu(menu,inflater);
        }
    }

// 添加页面切换监听
navController.addOnDestinationChangedListener(new NavController.OnDestinationChangedListener()
	@Override
	public void onDestinationChanged(NavController controller,
                                     NavDestination destination,Bundle bundle) {
        //收到页面切换的事件
    }                                             
                                                                                        
);

```

#### **7. 深层链接DeepLink**

常见的应用场景有两种：

```java
// 第一种: PendingIntent的方式

当应用程序接受到某个通知推送，你希望用户在点击该通知时，能够直接跳转到展示该通知内容的页面，那么可以通过PendingIntent来完成此操作.	
    
思路：通过sendNotification()方法向通知栏发送一条通知，模拟用户收到哦啊一条推送的情况，并在通知中构建一个PendingIntent对象，在其中进行设置，当通知被点击时，需要跳转到的目的地（destination）,以及传递的参数.

private void sendNotifcation(){
    if(getActivity()!=null){
        return;
    }
    
    if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
        int importance = NotificationManager.IMPORTANCE_DEFAULT;
        NotificationChannel channel = new NotificationChannel(CHANNEL_ID,"ChannelName",importance);
        channel.setDescription("description");
        
        NotificationManager notificationManager = getActivity().getSystemService(NotificationManager.class);
        notificationManager.createNotificationChannel(channel);
    }
    
    NotificationCompat.Builder builder = new NotificationCompat.Builder(getActivity(),CHANNEL_ID)
        .setSmallIcon(R.drawable.ic_lancher_foreground)
        .setContentTitle("DeepLink")
        .setContentText("Hello World")
        .setPriority(NotificationCompat.PRIORITY_DEFAULT)
        .setContentIntent(getPendingIntent)
        .setAutoCancel(true);
    
    NotificationManagerCompat notificationManager = NotificationManagerCompat.from(getActivity());
    notificationManager.notify(notificationId,builder.build());
}


private PendingIntent getPendingIntent(){
    if(getActivity() != null){
        Bundle bundle = new Bundle();
        bundle.putString("params","Params From Noticaiton");
        return Navigation.findNavController(getActivity(),R.id.sendNotification)
            .createDeepLink()
            .setGraph(R.navigation.graph_deep_link_activity)
            .setDestination(R.id.deepLinkSettingsFragment)
            .setArguments(bundle)
            .createPendingIntent();
    }
    return null;
}
    
    

// 第二种：URL的方式

当用户通过手机浏览器网站上的某个页面时，可以在网页上放置一个类似于“在应用内打开”的按钮，如果用户的手机安装有我们的的应用程序，那么通过deepLinke就能打开相应的页面如果没有安装那么网站可以导航到应用程序的下载页面，从而引导用户安装应用程序。
    
第一步：在导航图中为页面添加<deepLink/>标签，在app:uri属性中填入的是你的网站的相应Web页面地址，后面的参数会通过Bundle对象传递到页面中。

    例如：<deepLink app:uri="www.baidu.com/{params}">

第二步：为Activity设置<nav-graph/>标签，当用户在web页面中访问你的网址时，应用程序便能够得到监听.
        
第三步：使用adb工具进行测试
	键入：adb shell am start -a android.intent.action.VIEW -d "http://www.baidu.com/Parms from notification"
	        
第四步：在Fragment中获取显示
	Bundle bundle = getArguments();        
	if(bundle != null){
        String params = bundle.getString("params");
        TextView tvDesc = view.findViewById(R.id.tvDesc);
        if(TextUtils.isEmpty()){
            tvDesc.setText(params);
        }
    }
    
 
```





