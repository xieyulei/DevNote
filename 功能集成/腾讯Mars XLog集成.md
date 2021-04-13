### Tencent Mars XLog集成到Android项目中



**学习背景：**

```java
查看官网教程，有点蒙，搜集各个大牛博客以及维基百科整理一下，在此记录方便以后查看
    
资源整理：    
    
Mars源码下载：
Github链接：（有时候网真的不行啊）https://github.com/Tencent/mars
网盘链接: https://pan.baidu.com/s/1tNKH937ncLArsHrjwWTkgg 提取码: z5aq

pyelliptic-百度网盘链接: https://pan.baidu.com/s/1BqFbFyyZMWv3jjCF8ywGxg 提取码: tgen
```



**环境配置: JDK + NDK + Python + CMake**

```java
运行环境：
1. 系统：Ubuntu20.04 LTS 64位
2. Android Studio 4.0

配置环境：
Ubuntu配置环境主要操作 bashrc 文件，常用的两个命令：
    打开bashrc文件：sudo gedit ~/.bashrc
    使bashrc文件改动立即生效：source ~/.bashrc
   
1. 配置jdk环境
    
2. 配置ndk环境：
   下载android-ndk-r20b(其它版本会出现编译错误的问题),下载网址：https://developer.android.com/ndk/downloads/older_releases
   配置环境变量：
     命令1：Ctrl + Alt + T 打开终端，输入命令：sudo gedit ~/.bashrc
        在文本最下方加入ndk环境配置：
        #ndk
		export NDK_ROOT=/../../android-ndk-r20b
		export PATH=$PATH:$NDK_ROOT
	 命令2：验证ndk环境是否配置成功
        输入命令：ndk-build -v 
            
3. 配置CMake环境
	 命令1：安装Cmake
            sudo apt-get intsall CMake
     命令2：验证CMake是否安装成功
            cmake --version
            
4. 配置Python环境             
	 命令1：安装Python
            sudo apt-get intsall python
     命令2：验证CMake是否安装成功
            python
```



**编译指定架构的库**

```java
1. 下载mars源码，git仓库地址为：https://github.com/Tencent/mars

2. 进入源码中：mars/mars目录，执行脚本：
	生成armeabi-v7a架构：python build_android.py (默认生成)
	生成arm64-v8a架构：python build_android.py (修改此文件中的arch参数为arm64-v8a)
    
    选择3，然后Enter,只生成xlog模块的库；
    
	其它架构生成方式也是如此；

3. 生成后的库保存在：sample/mars-master/mars/libraries/mars_xlog_sdk/libs目录下；
```



**项目配置：**

```java
1. build.gradle(app)--->android标签下加入：

	externalNativeBuild {
		cmake {
        	cppFlags "-std=c++11"
        	abiFilters 'armeabi-v7a', 'arm64-v8a'
    	}
	}

2. 创建与java目录平级别的目录jniLibs,分别将刚生成的两个架构so文件复制进去
    
3. 将xlog的完整路径复制到项目中，与包名同级

4. 项目Application中进行XLog初始化（封装XLog，代替Log进行打印）
   
   //保存路径 
   final String SDCARD = Environment.getExternalStorageDirectory().getAbsolutePath();
   final String logPath = SDCARD + "/marssample/log";
    
   MarsLog.init(this, "app name", logPath);

    /**
     * 初始化操作
     * 
     * true : 是否加载so库
     * Xlog.LEVEL_DEBUG 或  Xlog.LEVEL_INFO ：打印日志的等级
     * Xlog.AppednerModeSync: 同步
     * Xlog.AppednerModeAsync: 异步
     * cachePath: 这个参数必传，而且要data下的私有文件目录，例如 /data/data/packagename/files/xlog， 
     * mmap文件会放在这个目录，如果传空串，可能会发生 SIGBUS 的crash。	
     * logPath : xlog文件保存的路径
     * namePrefix: 一般填写项目名称
     * pub_key : 加/解密 用到的key （此处未加密）
     */
   public class MarsLog {

    public static void init(Context context, String namePrefix, String logPath) {
        String cacheDir = context.getFilesDir() + "/xlog";
        if (BuildConfig.DEBUG) {
            Xlog.open(true, Xlog.LEVEL_DEBUG, Xlog.AppednerModeAsync, cacheDir, logPah, namePrefix, "");
            Xlog.setConsoleLogOpen(true);
        } else {
            Xlog.open(true, Xlog.LEVEL_INFO, Xlog.AppednerModeAsync, cacheDir, logPah, namePrefix, "");
            Xlog.setConsoleLogOpen(false);
        }

        Log.setLogImp(new Xlog());
    }
    
       此处也可以重新Log中打印的方法，使用MarsLog.v的形式进行日志打印，皆可
       
 }   

5. 在文件需要导出之前，手动调用以下代码，将缓存区的日志刷新到文件存储里面，确保日志完整
	 Log.appenderFlush();
	 
6. 在程序完全退出时，进行初始化，加入代码：
     Log.appenderClose();
```



**打印XLog与查看Xlog**

```java
1. 直接在项目中打印xlog，打印完毕，在logcat日志模块成功看到日志；
   使用studio自带Device File Explorer 工具，进入data/data/当前包名/files/log , 即可查看到xlog ,右键save as到本地
   
2. 找到之前下载好的mars源码，寻找目录   mars-master/mars/log/crypt目录下，其中有两个文件我们将要使用：
	decode_mars_crypt_log_file.py ：使用此脚本可以打开 加密 的xlog文件
	decode_mars_nocrypt_log_file.py ： 使用此脚本可以打开 未加密 的xlog文件
	
	当前我们生成的是没有加密的xlog文件，在mars-master/mars/log/crypt目录下，打开终端，命令行输入：
	python decode_mars_nocrypt_log_file.py + 导出到本地的xlog文件路径 ，
    
    此时会在xlog文件同级的目录下，生成一个与xlog文件名称相同的 .xlog文件，使用文本编辑器 即可查看刚打印的日志
    
   
```



**Xlog日志的加密与解密**

```java
1. 安装 openssl
	命令：sudo apt-get install openssl
	
2.下载 pyelliptic1.5.7

	pyelliptic1.5.7 不支持 openssl 1.1 版本，会报以下错误：
	AttributeError: /usr/lib/x86_64-linux-gnu/libcrypto.so.1.1: undefined symbol: ECDH_OpenSSL
	
	安装下面这个打补丁的版本
	pip install https://github.com/mfranciszkiewicz/pyelliptic/archive/1.5.10.tar.gz#egg=pyelliptic
	百度网盘链接: https://pan.baidu.com/s/1BqFbFyyZMWv3jjCF8ywGxg 提取码: tgen
	
3. 复制文件到指定位置
     openssl安装完毕 且 pyelliptic1.5.7补丁版下载完毕，进入重要一步：

	解压：pyelliptic1.5.7，进入其根目录下，将其所有文件复制到 mars源码的 mars-master/mars/log/crypt目录下，
	
	执行命令：python gen_key.py,会生成private_key 和 appender_open`s parameter 两个key
	
	使用文本保存好这两个key，接下来要用；

4. 将上一步得到的key设置到 mars-master/mars/log/crypt中的decode_mars_crypt_log_file.py脚本中，如：

	#PRIV_KEY : private_key
	#PUB_KEY : appender_open`s parameter
	
5. 把 pulic key作为appender_open 函数参数设置进去，private key务必保存在安全的位置，防止泄露
	
	public class MarsLog {
	 
	 final String pub_key = “”；
   	 public static void init(Context context, String namePrefix, String logPath) {
        String cacheDir = context.getFilesDir() + "/xlog";
        if (BuildConfig.DEBUG) {
            Xlog.open(true, Xlog.LEVEL_DEBUG, Xlog.AppednerModeAsync, cacheDir, logPah, namePrefix, pub_key);
            Xlog.setConsoleLogOpen(true);
        } else {
            Xlog.open(true, Xlog.LEVEL_INFO, Xlog.AppednerModeAsync, cacheDir, logPah, namePrefix, pub_key);
            Xlog.setConsoleLogOpen(false);
        }

        Log.setLogImp(new Xlog());
    } 
    
```



**验证加密，验证解密**

```java
1. 导出加密的日志：

	卸载之前运行的程序，加入pub_key参数运行程序，按照之前的方法，
	使用studio自带Device File Explorer 工具，进入data/data/当前包名/files/log , 即可查看到xlog ,右键save as到本地

2. 错误的解密方式：
	使用不加密decode_mars_nocrypt_log_file的脚本，对xlog进行解密，
		命令1 ：python decode_mars_nocrypt_log_file.py + 导出到本地的xlog文件路径 ，查看刚打印的日志
		提示：use wrong decode script ，这是因为xlog已经加密了，使用这个nocrypt的脚本错误导致的，需要使用另一个脚本
		
3. 正确的解密方式：（确保private_key和pub_key设置正确）		
		命令1：python decode_mars_crypt_log_file.py + 导出到本地的xlog文件路径 ，查看刚打印的日志 
		
```