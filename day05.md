# 安全 #
## 组件理解 ##
1. Zygote：Android应用的孵化器，一切Android程序由此进程fork而来。
2. Binder：Android的进程间通信机制，它是Android平台最核心的功能组件。
3. Package Manager Service：应用安装包管理服务，不仅负责包的安装和卸载，更重要的是负责Android应用信息的查询和控制，例如Android权限管理。
4. WMS
5. Activity Manager Service：管理Android框架层的进程，也包含了Android应用四大组件的逻辑实现。
6. Dalvik虚拟机：虽然即将被ART取代，但Dalvik依然是帮助我们理解虚拟机和Android可执行程序文件格式的好教材。

## android安全机制 ##
![](imgs\46.png)  
### 进程沙箱隔离机制 ###
Android应用程序在安装时被赋予独特的用户标识（UID），并永久保持；应用程序及其运行的Dalvik虚拟机运行于独立的Linux进程空间，与UID不同的应用程序完全隔离。  
![](imgs\47.png)  
在特殊情况下，进程间还可以存在相互信任关系。如源自同一开发者或同一开发机构的应用程序，通过Android提供的共享UID（Shared UserId）机制，使得具备信任关系的应用程序可以运行在同一进程空间。  

### 应用程序签名机制 ###
应用程序包（.apk文件）必须被开发者数字签名；同一开发者可指定不同的应用程序共享UID，进而运行于同一进程空间，共享资源。

    签名的过程：
       • 生成私有、公共密钥和公共密钥证书
       • 对应用进行签名
       • 优化应用程序
    
    签名的作用：
       • 识别代码的作者。
       • 检测应用程序是否发生了改变。
       • 在应用程序之间建立信任，以便于应用程序可以安全地共享代码和数据。

在安装应用程序APK时，系统安装程序首先检查APK是否被签名，有签名才能安装。当应用程序升级时，需要检查新版应用的数字签名与已安装的应用程序的签名是否相同，否则，会被当做一个新的应用程序。Android开发者有可能把安装包命名为相同的名字，通过不同的签名可以把他们区分开来，也保证签名不同的包不被替换，同时防止恶意软件替换安装的应用。  

### 权限声明机制 ###

动态权限：

默认有权限--用户给权限

要想获得在对象上进行操作，就需要把权限和此对象的操作进行绑定。不同级别要求应用程序行使权限的认证方式也不一样，Normal级申请就可以使用，Dangerous级需要安装时由用户确认，Signature和Signatureorsystem级则必须是系统用户才可用。  

 • 通过manifest文件中声明以下属性

    <uses-permissionandroid:name="string" />

      请求android:name对应的权限。
•  通过以下属性添加自定义权限

    <permission

       xmlns:android="http://schemas.android.com/apk/res/android"

       android:name="com.test.android.ACCESS_FRIENDS_LIST"

       android:description="@string/permission_description"

       android:label="@string/permission_label"

       android:protectionLevel="normal" />
•  系统组件权限，如activity组件

    <activity      android:permission="com.test.android.ACCESS_FRIENDS_LIST"

### 访问控制机制 ###
确保系统文件和用户数据不受非法访问。  
#### Linux用户与权限 ####
    • 超级用户（root），具有最高的系统权限，UID为0。
    • 系统伪用户，Linux操作系统出于系统管理的需要，但又不愿赋予超级用户的权限，需要将某些关键系统应用
      文件所有权赋予某些系统伪用户，其UID范围为1～ 499，系统的伪用户不能登录系统。
    
    • 普通用户，只具备有限的访问权限，UID 为 500 ～ 6000，可以登录系统获得 shell
    
    在Linux权限模型下，每个文件属于一个用户和一个组，由UID与GID标识其所有权。针对于文件的具体访问权限
    
    定义为可读(r)、可写(w)与可执行(x)，并由三组读、写、执行组成的权限三元组来描述相关权限。
    
    第一组定义文件所有者（用户）的权限，第二组定义同组用户（GID相同但UID不同的用户）的权限，第三组定义其他用户的权限（GID与UID都不同的用户）。

### 进程通信机制 ###
基于共享内存的Binder实现，提供轻量级的远程进程调用（RPC）。通过接口描述语言（AIDL）定义接口与交换数据的类型，确保进程间通信的数据不会溢出越界,污染进程空间。    
### 内存管理机制 ###
基于标准 Linux的低内存管理机制（OOM），设计实现了独特的低内存清理（LMK）机制，将进程按重要性分级、分组，当内存不足时，自动清理最低级别进程所占用的内存空间；同时，引入不同于传统Linux共享内存机制的Android共享内存机制Ashmem，具备清理不再使用共享内存区域的能力。  
### SE Android ###
root与内核  
在Android 2.x时代，往往利用一些用户层程序的漏洞即可将手机root，现在则主要依赖内核漏洞。Android为Linux内核引入了新的内核模块，以及不同厂商的驱动方案。这就为系统内核引入了新的安全隐患，无论是高通、MTK还是三星猎户座，或者华为海思的芯片，多少都出现过一些内核漏洞，这是Android平台内核的一个主要攻击点。  

Android是一个基于Linux内核的系统，像传统的Linux系统一样，Android也有用户的概念。只不过这些用户不需要登录，也可以使用Android系统。Android系统将每一个安装在系统的APK都映射为一个不同的Linux用户。也就是每一个APK都有一个对应的UID和GID，这些UID和GID在APK安装的时候由系统安装服务PackageManagerService分配。Android沙箱隔离机制就是建立在Linux的UID和GID基础上。

　　这种基于Linux UID/GID的安全机制存在什么样的问题呢？

　　Linux将文件的权限划分为读、写和执行三种，分别用字母r、w和x表示。每一个文件有三组读、写和执行权限，分别针对文件的所有者、文件所有者所属的组以及除了所有者以及在所有者所属组的用户之外所有其它用户。这样，如果一个用户想要将一个自己创建的文件交给另外一个用户访问，那么只需要相应地设置一下这个文件的其它用户权限位就可以了。所以，在Linux系统中，文件的权限控制在所有者的手中。因此，这种权限控制方式就称为自主式的，正式的英文名称为Discretionary Access Control，简称为DAC。

　　在理想情况下，DAC机制是没有问题的。然而，一个用户可能会不小心将自己创建的文件的权限位错误地修改为允许其它用户访问。如果这个用户是一个特权用户，并且它错误操作的文件是一个敏感的文件，那么就会产生严重的安全问题。这种误操作的产生方式有三种：

    用户执行了错误的命令
    负责执行用户命令的程序有Bug
    负责执行用户命令的程序受到攻击

　　后来，Linux内核采用了必要的访问控制机制：SE Linux（Security-Enhanced Linux），它采用了一种强制存取控制MAC（Mandatory Access Control）策略的实现方式，目的在于通过限制系统中的任何进程以及用户对资源的访问，保护内核安全。而SE Android（Security-Enhanced Android）是Android与SE Linux的结合，由美国NSA在2012年推出的Android操作系统安全强化套件，以支持在Android平台上使用SE Linux。

　　目前SE Android系统中的策略机制主要有三种：

    安装时MAC（install-time MAC）
    权限取消（permission revocation）
    权限标签传播（tag propagation）

　　安装时MAC通过查找MAC策略配置来检查应用程序的权限。权限取消可以为已安装的应用取消权限，该机制在应用程序运行的权限检查时通过查找权限取消列表来取消应用的某些权限。权限标签传播是一种污点跟踪方式的应用，Android系统的权限作为抽象的标签映射到MAC策略配置文件中。

## andoird app的安全现状   
### 病毒 ###

病毒扫描

不再有本地病毒数据库，云端。

Android病毒就是手机木马，主要是一些恶意的应用程序。通过钓鱼方式获取用户输入的信息上传到服务器。手机木马有的独立存在，有的则伪装成图片文件的方式附在正版App上，隐蔽性极强，部分病毒还会出现变种，并且一代比一代更强大。  
### 关键信息泄露 ###
虽然Java代码一般要做混淆，但是Android的几大组件的创建方式是依赖注入的方式，因此不能被混淆，而且目前常用的一些反编译工具比如apktool等能够毫不费劲的还原java里的明文信息，native里的库信息也可以通过objdump或IDA获取。因此一旦java或native代码里存在明文敏感信息，基本上就是毫无安全而言的。  
### APP重打包 ###

对app进行签名认证

即反编译后重新加入恶意的代码逻辑，从新打包一个APK文件。  
### 进程被劫持 ###
一般通过进程注入或者调试进程的方式来hook进程（Hook API），改变程序运行的逻辑和顺序，获取程序运行的内存信息，也就是用户所有的行为都被监控起来，这也是盗取帐号密码最常用的一种方式。  
### 数据在传输过程中遭劫持 ###  
### 键盘输入安全隐患 ###

自定义键盘：

使用三方输入法、截屏、getevent读取驱动层event信息。  
### Webview漏洞 ###
js注入漏洞和webkit xss漏洞    
### 服务器 ###
未做处理，遭到渗透攻击：重放攻击和注入攻击  

## app安全性如何保证？   

一个app的安全，应该是由服务器做。

代码安全。

怎么做安全，其实都是安全和逆向的战争。

### Android APP安全体系架构 ###
数据安全：需要将数据进行加密，网络请求使用https，校验证书，检查host。  

app， 用fiddler 抓包。

### 敏感信息安全 ###
比如支付密码安全，一般使用RSA和ASE进行加密。比如，将AES对称密码使用公钥加密，服务器使用私钥进行解密。  
HTTPS安全中，客户端和服务端的工作流程：  
1. 客户端生成随机数A，传递给服务端索要证书，证书是用来校验身份的。  
2. 服务端返回一个证书，证书中有公钥，带着一个随机数B。  
3. 客户端使用公钥加密一个随机数C，传递给服务端。  
4. 服务端可以利用私钥解密随机数C。  
5. 客户端和服务端都拥有这三个随机数，按照事先约定的加密方式，就可以生成会话的秘钥。  

为什么要采用对称的AES进行数据流通呢？  
因为非对称RSA有一定的缺陷：加密工作量大，效率不高、加密数据长度有限制。

### java加密技术 ###

1单向加密：也就是不可逆的加密，例如MD5,SHA,HMAC（信息完整的校验）

2对称加密：也就是加密方和解密方利用同一个秘钥对数据进行加密和解密，例如DES，PBE等等

3非对称加密：非对称加密分为公钥和秘钥，二者是非对称的，例如用私钥加密的内容需要使用公钥来解密，使用公钥加密的内容需要用私钥来解密，DSA，RSA...

keyGenerator：秘钥生成器，也就是更具算法类型随机生成一个秘钥，例如HMAC，所以这个大部分用在非可逆的算法中

SecretKeyFactory：秘密秘钥工厂，言外之意就是需要根据一个秘密（password）去生成一个秘钥,例如DES，PBE，所以大部分使用在对称加密中

KeyPairGenerator:秘钥对生成器，也就是可以生成一对秘钥，也就是公钥和私钥，所以大部分使用在非对称加密中
### url防篡改 ###

不让别人改，改了不认。

https://www.520it.com/home/list?uid=1234&aid=abcd&account=1234&pwd=123

防止篡改：

1.uid1234aidabcdaccount1234pwd123:base64加密。

计算最后的HMAC:一个秘钥+散列函数  ax7y

https://www.520it.com/home/list?uid=1234&aid=abcd&account=1234&pwd=123&sign=ax7y

2.企业中，更多的是使用短地址：通过你的参数，去访问另外一个接口，直接返回一个sorturl

https://www.520it.com/s/t/1upt   

防界面劫持

假的app，只有一个服务，开机启动。

四大组件每一个都可以通过隐式的Intent方式打开，所以这些组件只要不是对外公开的必须在AndroidManifest里面注明exported为false，禁止其它程序访问我们的组件。对于要和外部交互的组件，应当添加访问权限的控制，还需要要对传递的数据进行安全的校验。

一般的做法：在界面onstop时候判断。如果我们自己的界面不再顶层，给用户提示。

这样有一个问题：误判。  

### 代码混淆 ###

把取名的地方变成了花指令。一般不会影响我们的代码执行。

如果使用了第三方框架，不知道他里面的实现，如果他们中间有些不能被混淆，我们做了混淆，就可能报错。

gitHub，混淆如何做。

如果这个框架没有介绍如何混淆，就最好不要混淆。

代码混淆(Obfuscated code)亦称花指令，是将计算机程序的代码，转换成一种功能上等价，但是难于阅读和理解的形式的行为。  
对于对安全性要求很高的场合，仅仅使用代码混淆并不能保证源代码的安全。但是可以在一定程度上保护自己的劳动成果。  

实践中，代码混淆主要的好处：  
1. 缩小包体，一般情况，可以缩小两成包体  
2. 保护代码不被竞争对手轻易使用  
3. 增加破解难度，提高一定的安全性  

缺点：人工成本，大概在两到三个工作日（可能有很多代码不允许混淆，其他三方框架）。  
1. 混淆不能阻止反编译  
2. 再如何混淆加固，最后http一些明文或者数据保存到内存卡，也不安全  
3. 一些人认为，代码混淆没有意义，当然，这个见仁见智，并不是所有的项目都需要混淆  
4. 大神认为，真正的安全，应该是服务端保证，而不是将难题留给客户端  
5. 考虑因素：业务场景，取得的收益，付出的成本。不混淆的危害  
6. 混淆不会带来额外开销，混淆不同于加固，加固的原理是运行时解密相关资源，比较好性能  
7. 反射相关的需求点，比如热修复，不建议混淆，可能导致反射失败  
8. android产品大多数不是靠软件来盈利，而是运营，所以android不是特别流行混淆  
9. 混淆，加固加壳，仅仅是增加逆向的成本  
10. 代码烂，自带混淆  
11. 代码最不值钱  
12. 关键逻辑在服务端或者so文件  
13. 如果把系统的安全性托付给终端，一定是设计的架构问题  

说了这么多，混淆还得做！

gradle配置：  

			// 混淆
	        minifyEnabled true
	        // Zipalign优化
	        zipAlignEnabled true
	        // 移除无用的resource文件
	        shrinkResources true
	        // 前一部分代表系统默认的android程序的混淆文件，该文件已经包含了基本的混淆声明，后一个文件是自己的定义混淆文件
	        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'


混淆文件  

		#############################################
		#
		# 对于一些基本指令的添加
		#
		#############################################
		# 代码混淆压缩比，在0~7之间，默认为5，一般不做修改
		-optimizationpasses 5
		
		# 混合时不使用大小写混合，混合后的类名为小写
		-dontusemixedcaseclassnames
		
		# 指定不去忽略非公共库的类
		-dontskipnonpubliclibraryclasses
		
		# 这句话能够使我们的项目混淆后产生映射文件
		# 包含有类名->混淆后类名的映射关系
		-verbose
		
		# 指定不去忽略非公共库的类成员
		-dontskipnonpubliclibraryclassmembers
		
		# 不做预校验，preverify是proguard的四个步骤之一，Android不需要preverify，去掉这一步能够加快混淆速度。
		-dontpreverify
		
		# 保留Annotation不混淆
		-keepattributes *Annotation*,InnerClasses
		
		# 避免混淆泛型
		-keepattributes Signature
		
		# 抛出异常时保留代码行号
		-keepattributes SourceFile,LineNumberTable
		
		# 指定混淆是采用的算法，后面的参数是一个过滤器
		# 这个过滤器是谷歌推荐的算法，一般不做更改
		-optimizations !code/simplification/cast,!field/*,!class/merging/*


		#############################################
		#
		# Android开发中一些需要保留的公共部分
		#
		#############################################
		
		# 保留我们使用的四大组件，自定义的Application等等这些类不被混淆
		# 因为这些子类都有可能被外部调用
		-keep public class * extends android.app.Activity
		-keep public class * extends android.app.Appliction
		-keep public class * extends android.app.Service
		-keep public class * extends android.content.BroadcastReceiver
		-keep public class * extends android.content.ContentProvider
		-keep public class * extends android.app.backup.BackupAgentHelper
		-keep public class * extends android.preference.Preference
		-keep public class * extends android.view.View
		-keep public class com.android.vending.licensing.ILicensingService


		# 保留support下的所有类及其内部类
		-keep class android.support.** {*;}
		
		# 保留继承的
		-keep public class * extends android.support.v4.**
		-keep public class * extends android.support.v7.**
		-keep public class * extends android.support.annotation.**
		
		# 保留R下面的资源
		-keep class **.R$* {*;}
		
		# 保留本地native方法不被混淆，因为c代码中生成的方法名字是根据我们java包名+类名+方法名
		-keepclasseswithmembernames class * {
		    native <methods>;
		}
		
		# 保留在Activity中的方法参数是view的方法，
		# 这样以来我们在layout中写的onClick就不会被影响
		-keepclassmembers class * extends android.app.Activity{
		    public void *(android.view.View);
		}
		
		# 保留枚举类不被混淆
		-keepclassmembers enum * {
		    public static **[] values();
		    public static ** valueOf(java.lang.String);
		}
		
		# 保留我们自定义控件（继承自View）不被混淆
		-keep public class * extends android.view.View{
		    *** get*();
		    void set*(***);
		    public <init>(android.content.Context);
		    public <init>(android.content.Context, android.util.AttributeSet);
		    public <init>(android.content.Context, android.util.AttributeSet, int);
		}
		
		# 保留Parcelable序列化类不被混淆
		-keep class * implements android.os.Parcelable {
		    public static final android.os.Parcelable$Creator *;
		}
		
		# 保留Serializable序列化的类不被混淆
		-keepclassmembers class * implements java.io.Serializable {
		    static final long serialVersionUID;
		    private static final java.io.ObjectStreamField[] serialPersistentFields;
		    !static !transient <fields>;
		    !private <fields>;
		    !private <methods>;
		    private void writeObject(java.io.ObjectOutputStream);
		    private void readObject(java.io.ObjectInputStream);
		    java.lang.Object writeReplace();
		    java.lang.Object readResolve();
		}
		
		# 对于带有回调函数的onXXEvent、**On*Listener的，不能被混淆
		-keepclassmembers class * {
		    void *(**On*Event);
		    void *(**On*Listener);
		}
		
		# webView处理，项目中没有使用到webView忽略即可
		-keepclassmembers class fqcn.of.javascript.interface.for.webview {
		    public *;
		}
		-keepclassmembers class * extends android.webkit.webViewClient {
		    public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
		    public boolean *(android.webkit.WebView, java.lang.String);
		}
		-keepclassmembers class * extends android.webkit.webViewClient {
		    public void *(android.webkit.webView, jav.lang.String);
		}
		
		# 移除Log类打印各个等级日志的代码，打正式包的时候可以做为禁log使用，这里可以作为禁止log打印的功能使用
		# 记得proguard-android.txt中一定不要加-dontoptimize才起作用
		# 另外的一种实现方案是通过BuildConfig.DEBUG的变量来控制
		#-assumenosideeffects class android.util.Log {
		#    public static int v(...);
		#    public static int i(...);
		#    public static int w(...);
		#    public static int d(...);
		#    public static int e(...);
		#}
		
		#############################################
		#
		# 项目中特殊处理部分
		#
		#############################################
		
		#-----------处理反射类---------------


​		
		#-----------处理js交互---------------

​		
​		
		#-----------处理实体类---------------
		# 在开发的时候我们可以将所有的实体类放在一个包内，这样我们写一次混淆就行了。
		#-keep public class com.ljd.example.entity.** {
		#    public void set*(***);
		#    public *** get*();
		#    public *** is*();
		#}


		#-----------处理第三方依赖库---------

		#如果引用了v4或者v7包
		-dontwarn android.support.**
	
		-dontwarn com.alibaba.fastjson.**
		-keep class com.alibaba.fastjson.** { *; }
		-keep interface com.alibaba.fastjson.** { *; }
		-dontwarn com.squareup.okhttp.**
		-dontwarn okio.**
		-keep class com.squareup.okhttp.** { *; }
		-keepattributes Exceptions,InnerClasses,Signature,Deprecated,SourceFile,LineNumberTable,LocalVariable*Table,*Annotation*,Synthetic,EnclosingMethod
	
		#apk 包内所有 class 的内部结构
		-dump class_files.txt
		#未混淆的类和成员
		-printseeds seeds.txt
		#列出从 apk 中删除的代码
		-printusage unused.txt
		#混淆前后的映射
		-printmapping mapping.txt

不能混淆的代码

顾名思义，不能混淆代码如果被混淆了，就会出现错误。

	四大组件依赖注入
	需要反射的代码
	系统接口
	Jni接口
	需要序列号和反序列化的代码（即实现Serializable接口的JavaBean）
	与服务端进行元数据交互的JavaBean（JSON、XML中对应的类）

实例：

		-keepclassmembers class fqcn.of.javascript.interface.for.webview {
		   public *;
		}
		
		#如果引用了v4或者v7包
		-dontwarn android.support.**
		-keep class com.m520it.tounaer.net.HttpsUtil
		-keep class com.m520it.tounaer.model.**{*;}


		-dontwarn com.alibaba.fastjson.**
		-keep class com.alibaba.fastjson.** { *; }
		-keep interface com.alibaba.fastjson.** { *; }
		-dontwarn com.squareup.okhttp.**
		-dontwarn okio.**
		-keep class com.squareup.okhttp.** { *; }
		-keepattributes Exceptions,InnerClasses,Signature,Deprecated,SourceFile,LineNumberTable,LocalVariable*Table,*Annotation*,Synthetic,EnclosingMethod
		-ignorewarning
		
		#apk 包内所有 class 的内部结构
		-dump class_files.txt
		#未混淆的类和成员
		-printseeds seeds.txt
		#列出从 apk 中删除的代码
		-printusage unused.txt
		#混淆前后的映射
		-printmapping mapping.txt


![](imgs\48.png)  

fastjson，在混淆的时候，如果添加了对fastjson不混淆，其实已经没什么问题了。但是。注意，fastjson需要将一个json串解析成一个bean对象。

1.对于所有的要解析的bean不能混淆。

2.对于fastjson要解析的bean对象，需要全部在一个包中。



所有的混淆，仅仅是让逆向工作增加成本，不能让代码变得安全。

不会改变代码的逻辑，反而有可能给你带来bug

工作量：半天- 三天。

但是，1，代码量多。2，版本迭代（上线了出现问题）

混淆的代码出现了问题，咋改。

com.a.b.c.x() 出问题了。

Tink

# app加固 #

不能更改我们的代码逻辑（界面劫持），只能增加逆向的成本。

梆梆加固

360加固宝

腾讯

百度

爱加密

网易云

免费 60%

给钱的 99%

51job

智联

拉钩

猎聘

boss



![](imgs\50.bmp)

    http://bbs.51cto.com/thread-1159902-1.html 