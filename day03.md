# 集成网络框架 #
## 集成okhttp ##

	compile 'com.squareup.okhttp:okhttp:2.0.0'  
1. 请求参数

    	RequestBody requsetBody=RequestBody.create(MediaType.parse("application/json"),json);

2. 封装请求信息  

   	// post
   	Request request = new Request.Builder()
   	         .url(url)
   	         .header("Content-Type","application/json")
   	         .post(requsetBody)
   	         .build();
   	// get
   	Request request = new Request.Builder()
   	         .url(url)
   	         .get()
   	         .build();

3. 创建工具类对象

       OkHttpClient client = new OkHttpClient();

4. 封装请求任务  

       Call call = okHttpClient.newCall(request);

5. 发起请求  

       //异步发起任务
       call.enqueue(callback);
       //同步发去任务
       Response r = call.execute();// 既然同步了，网络请求又不能再主线程。阻塞任务Callable-future
## 封装请求参数 ##

	public class RequestParams<T> {
	    private String apiVersion;
	    private T data;// 请求参数中包含一个返回的bean对象
	    private String sign;
	    private long timeStamp;
	}

## 集成ImageLoader ##

	compile 'com.nostra13.universalimageloader:universal-image-loader:1.9.5'

	ImageLoader loader = ImageLoader.getInstance();
	    DisplayImageOptions options = new DisplayImageOptions.Builder()
	            .showImageOnLoading(R.drawable.default_image_backage) // 设置图片下载期间显示的图片
	            .showImageForEmptyUri(R.drawable.default_image_backage) // 设置图片Uri为空或是错误的时候显示的图片
	            .showImageOnFail(R.drawable.default_image_backage) // 设置图片加载或解码过程中发生错误显示的图片
	            .resetViewBeforeLoading(false)  // default 设置图片在加载前是否重置、复位
	            .delayBeforeLoading(1000)  // 下载前的延迟时间
	            .cacheInMemory(true) // default  设置下载的图片是否缓存在内存中
	            .cacheOnDisk(true) // default  设置下载的图片是否缓存在SD卡中
	            .considerExifParams(false) // default
	            .imageScaleType(ImageScaleType.IN_SAMPLE_POWER_OF_2) // default 设置图片以如何的编码方式显示
	            .bitmapConfig(Bitmap.Config.ARGB_8888) // default 设置图片的解码类型
	            .displayer(new SimpleBitmapDisplayer()) // default  还可以设置圆角图片new RoundedBitmapDisplayer(20)
	            .build();
	    ImageLoaderConfiguration configuration=new ImageLoaderConfiguration.Builder(this).defaultDisplayImageOptions(options).build();
	    loader.init(configuration);

## 如何加载缓存 ##

只要在没网的情况下，为了不影响用户使用，都要做缓存。

缓存，是暂时存数据。

需要对缓存的图片有策略行的删除。

内存泄漏和内存溢出有啥区别？

怎么缓存？增删改查

![](imgs\33.png)  
# 集成json解析 #

	compile 'com.alibaba:fastjson:1.2.33'

	public class JSONHelper {
		private static final SerializerFeature[] CONFIG = new SerializerFeature[] {
				SerializerFeature.WriteNullBooleanAsFalse,// boolean为null时输出false
				// SerializerFeature.WriteMapNullValue, //输出空置的字段
				SerializerFeature.WriteNonStringKeyAsString,// 如果key不为String
															// 则转换为String
															// 比如Map的key为Integer
				SerializerFeature.WriteNullListAsEmpty,// list为null时输出[]
				SerializerFeature.WriteNullNumberAsZero,// number为null时输出0
				SerializerFeature.WriteNullStringAsEmpty, // String为null时输出""
				SerializerFeature.DisableCircularReferenceDetect
		};
	
		public static String toJson(Object object) {
			SerializeWriter out = new SerializeWriter();
			try {
				JSONSerializer serializer = new JSONSerializer(out);
	
				for (SerializerFeature feature : CONFIG) {
					serializer.config(feature, true);
				}
				serializer.config(SerializerFeature.WriteEnumUsingToString, false);
				serializer.write(object);
	
				return out.toString();
			} finally {
				out.close();
			}
		}
	
		public static <T> T parseObject(String jsonString, TypeReference<T> type) {
			return JSON.parseObject(jsonString, type);
		}
		
	}
![](imgs\40.png)  
## 封装Callback ##

	public interface RequestCallback<T> {

	    public T success();

	    public void fail();
	}
fastjson尽管有着比gosn，jackson高的效率，但是他仍然有一些未修复的bug：比如对于一些特殊字符的处理，比如，对于多重泛型的处理。  
多重泛型问题：在fastjson中存在一个bug，就是在使用自定义多重泛型的时候，不能正确解析。  
意思是说，我们不能通过参数，直接解析成一个需要的T类型。  

修改Callback  

	public interface RequestCallback {

	    public void success(String data);

	    public void fail();
	}

# https #

http  ip/tcp  socket:封装了TCP的一个api：

协议：双方达成的一个约定，如果不遵守这个规则，谁都不认识谁

http：封装数据，明文，不安全。

 一、HTTP和HTTPS的基本概念

HTTP：是互联网上应用最为广泛的一种网络协议，是一个客户端和服务器端请求和应答的标准（TCP），用于从WWW服务器传输超文本到本地浏览器的传输协议，它可以使浏览器更加高效，使网络传输减少。  

HTTPS：是以安全为目标的HTTP通道，简单讲是HTTP的安全版，即HTTP下加入SSL 3.0层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。 TLS 

HTTPS协议的主要作用可以分为两种：一种是建立一个信息安全通道，来保证数据传输的安全；另一种就是确认网站的真实性。  
 二、HTTP与HTTPS有什么区别？

url防篡改。

HTTP协议传输的数据都是未加密的，也就是明文的，因此使用HTTP协议传输隐私信息非常不安全，为了保证这些隐私数据能加密传输，于是网景公司设计了SSL（Secure Sockets Layer）协议用于对HTTP协议传输的数据进行加密，从而就诞生了HTTPS。  

 简单来说，HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，要比http协议安全。

HTTPS和HTTP的区别主要如下：

1、https协议需要到CA申请证书(验证身份，保证安全)，一般免费证书较少，因而需要一定费用。

2、http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议。

3、http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443，tomcat 8443。

4、http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

过程：  
1. 访问服务器，带上一个随机数  
2. 服务器返回信息，带上一个随机数（根据客户端生成的随机数，服务器用此来生成随机数）和公钥  
3. 客户端根据服务器的随机数，生成一个key 和 秘钥，将key传递给服务器  
4. 服务器收到key，因为知道了第一次握手的随机数，所以可以根据证书按照相同的计算方式生成秘钥  
   至此，得到秘钥之后，就可以进行沟通了。  

身份认证：  
1. 服务器认证客户端，只要客户端有证书，才能是合法的用户。  
2. 客户端认证服务器，就算服务器被劫持，有证书存在，也会提示用户，该访问的地址不是本来要访问的地址。

几款免费SSL证书，比如：CloudFlare SSL、StartSSL、Wosign沃通SSL、NameCheap等。  

三、https的缺点  
安全是优点，也是缺点，会导致一定的性能低和流量多，但是这个缺点远远低于https带来的安全保证。  

Https  原理： 
​	
	http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html

# 集成https #

	/**
	 * 设置证书
	 * @param certificates
	 */
	public static void setCertificates(InputStream... certificates)
	{
	    try
	    {
			// 创建一个证书工厂类，这个类用来读取证书信息，参数为证书标准
	        CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
			// 创建一个证书库，用来存储证书信息
	        KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
	        keyStore.load(null);
	        int index = 0;
	        for (InputStream certificate : certificates)
	        {
				
	            String certificateAlias = Integer.toString(index++);
				// 保存证书信息到证书库中
	            keyStore.setCertificateEntry(certificateAlias, 
				// 读取证书
				certificateFactory.generateCertificate(certificate));
	
	            try
	            {
	                if (certificate != null)
	                    certificate.close();
	            } catch (IOException e)
	            {
	            }
	        }
			// 创建一个安全上下文
	        SSLContext sslContext = SSLContext.getInstance("TLS");
			// 创建一个可信任的工厂
	        TrustManagerFactory trustManagerFactory =
	                TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
			// 通过证书库，初始化安全工厂
	        trustManagerFactory.init(keyStore);
			// 根据安全工厂，强随机数，初始化安全上下文
	        sslContext.init
	                (
	                        null,
	                        trustManagerFactory.getTrustManagers(),
	                        new SecureRandom()
	                );
			// 为httpsClient设置安全上下文，如果用原生HttpsUrlConnection
	        okHttpClient.setSslSocketFactory(sslContext.getSocketFactory());
			// 设置host校验
	        okHttpClient.setHostnameVerifier(new HostnameVerifier() {
	            @Override
	            public boolean verify(String hostname, SSLSession session) {
	                if("47.93.30.78".equals(hostname)){
	                    return true;
	                }else{
	                    return false;
	                }
					// 返回true，表示信任
	            }
	        });
	    } catch (Exception e)
	    {
	        e.printStackTrace();
	    }
	
	}	
思考：可不可以用http工具去访问https？可不可以用https工具去访问http？  
## 使用三方工具获取访问数据 ##

![](imgs\34.png)  

# 数据加密 #
## 信息安全 ##
机密性：为了防止信息被窃听  
完整性：为了防止信息被篡改  
认证：为了防止攻击者伪装成真正的发送者  
不可否认性：为了防止发送者事后否认自己没有做过  
![](imgs\35.png)  
## 加密方式 ##
java中使用了多种加密方式。常见的加密方式有两种：  
对称加密：  
  加密和解密使用同一个密钥：所有的数据，变成byte数组之后，每个数 + 1  
  算法：DES、DES3、AES、RC5、Blowfish等  
  分组密码，即以分组为单位进行处理的密码算法  
  初始化向量IV：与分组长度一致，加解密一致  
  存在密钥配送问题  
非对称加密  
  加密用公钥，解密用私钥  
  主要基于数学上困难的问题来保证机密性  
  算法：RSA、DSA、ElGamal、Rabin等  
  公钥密码处理速度远远低于对称密码  
混合密码  
  对称密码和公钥密码相结合  
  用对称密码来加密消息  
  用公钥密码来加密对称密码的密钥  
  对称密码的密钥是临时生成的会话密钥  
  ![](imgs\36.png)  
  ![](imgs\37.png)  
单向散列函数  
  MD4、MD5、SHA-1、SHA-256、SHA-384、SHA-512、SHA-3等等  
  ![](imgs\39.png)  
消息认证码

GUID  :UUID，不允许重复的场景，使用UUID
  消息认证码(message authentication code)是一种确认完整性并进行认证的技术，简称为 MAC  
  输入包括消息和共享密钥，输出固定长度的数据，称为MAC值  
  比单向散列函数多了一个共享密钥  
  普遍的实现算法：HMAC  
  ![](imgs\38.png)  

证书一般包含：主体公钥值、主体标识符信息、证书有效期、颁发者标识符信息、颁发者的数字签名.证书标准：X.509

通过私钥去权威机构认证，产生一个证书和公钥。私钥产生证书，公钥验证证书。  
但是如果是用私钥对消息进行签名，效率很低，所以一般，私钥是对消息的散列值进行签名。

## java加密API规范和参考架构 ##

	http://www.cis.upenn.edu/~bcpierce/courses/629/jdkdocs/guide/security/CryptoSpec.html

	http://docs.oracle.com/javase/1.5.0/docs/guide/security/CryptoSpec.html#AppA
 Appendix A:附录A	
# 自定义广告按钮 #
广告按钮使用默认图片  
​	
	http://h.hiphotos.baidu.com/image/pic/item/dc54564e9258d1093cf78e5cd558ccbf6d814dc3.jpg
	// 加载图片到imageView中
	displayImage(url, this, new ImageLoadingListener(){});	
	
	ImageLoadingListener 有默认实现，SimpleImageLoadingListener
## 判断手指位置 ##

	下压
				lastX = (int) event.getRawX();// 获取在屏幕中的位置
				lastY = (int) event.getRawY();
	
	移动
				int dx = (int) event.getRawX() - lastX;
	            int dy = (int) event.getRawY() - lastY;
	            int left = this.getLeft() + dx;
	            int top = this.getTop() + dy;
	            int right = this.getRight() + dx;
	            int bottom = this.getBottom() + dy;
	            // 设置不能出界
	            if (left < 0) {
	                left = 0;
	                right = left + this.getWidth();
	            }
	
	            if (right > ScreenUtil.getScreenWidth(this.getContext())) {
	                right = ScreenUtil.getScreenWidth(this.getContext());
	                left = right - this.getWidth();
	            }
	
	            if (top < 0) {
	                top = 0;
	                bottom = top + this.getHeight();
	            }
				// 判断图片bottom 的时候，最好是能够获取到我们tab的高度。
	            if (bottom > ScreenUtil.getScreenHeigth(this.getContext())) {
	                bottom = ScreenUtil.getScreenHeigth(this.getContext());
	                top = bottom - this.getHeight();
	            }
				// 布局参数，LayoutParams 到底是个啥。是父控件给子控件的布局参数。
				// 布局参数，需要向父控件申请。
	            FrameLayout.LayoutParams param = new FrameLayout.LayoutParams(this.getMeasuredWidth(),
	                    this.getMeasuredHeight());//Width、Height是操作之后的图片宽度和高度
	            param.leftMargin = left;//操作之后控件左上角的横坐标
	            param.topMargin = top;//操作之后控件左上角的纵坐标
	            this.setLayoutParams(param);
	            lastX = (int) event.getRawX();
	            lastY = (int) event.getRawY();
	
	抬起
				// 判断单击，不需要判断长按短按，只要没有移动，再抬起，就是单击
	            if (!isMoved){
	                if (onClick != null){
	                    onClick.onClick(this);
	                }
	            }

# RecyclerView #
setHasFixedSize：true 的作用就是确保尺寸是通过用户输入从而确保RecyclerView的尺寸是一个常数。RecyclerView 的Item宽或者高不会变。每一个Item添加或者删除都不会变。如果你没有设置setHasFixedSized没有设置的代价将会是非常昂贵的。因为RecyclerView会需要而外计算每个item的size。 
如果item的内容不改变view布局大小，那使用这个设置可以提高RecyclerView的效率。
## LinearLayoutManager一些api ##
	setReverseLayout():数据逆向填充到布局中
	setStackFromEnd：滑动到最后一个条目
	scrollToPosition(int position)滑动到指定item
	scrollToPositionWithOffset(int position, int offset)，第二个参数是偏移量px单位
## 添加头布局 ##

1.getcount+1。 模拟的头布局。

2.getItemViewType：各个条目，view的类型是什么

3.根据不同的ItemViewType去创建指定的类型的viewholder

	// 增加头布局，条目数目+1
	@Override
	public int getItemCount() {
	    return totalCount + 1;// 增加头布局
	}
	// 根据position确定条目的viewtype
	@Override
	public int getItemViewType(int position) {
	    if (position == 0) {
	        // 这个是头布局
	        return HEAD_TYPE;
	    } else {
	        return CONTENT_TYPE;
	    }
	}
	// 在创建对应的viewholder时，根据viewtype来创建对应的view
	@Override
	public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
	    if (viewType == HEAD_TYPE) {
	        return new HomeHeadHolder(header);
	    } else if (viewType == CONTENT_TYPE) {
	        View view = View.inflate(parent.getContext(), R.layout.item_home, null);
	        return new HomeListHolder(view);
	    } else if (viewType == BUTTOM_TYPE) {
	        View view = View.inflate(parent.getContext(), R.layout.home_bottom, null);
	        return new HomeButtomHolder(view);
	    }
	    return null;
	}

## 是否滑动到头和尾 ##

	public boolean isFirstItem() {
	    return !canScrollVertically(-1);
	}
	
	public boolean isVisBottom() {
	    return !canScrollVertically(1);//
	}
## ListView是否滑动到头部 ##

	setOnScrollListener(new OnScrollListener() {
	        @Override
	        public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
	            
	            if (firstVisibleItem == 0) {
	                View firstVisibleItemView = XListView.this.getChildAt(0);
	                if (firstVisibleItemView != null && firstVisibleItemView.getTop() == 0) {
	                    // 滑动到头部
	                }
	            }
	        }
	
	        @Override
	        public void onScrollStateChanged(AbsListView view, int scrollState) {
	            //do nothing
	        }
	);
## ScrollView是否滑动到头部 ##

	@Override
	public boolean onTouch(View v, MotionEvent event) {
	    switch (event.getAction()) {
	        case MotionEvent.ACTION_MOVE:
	            View childView = getChildAt(0);
	            // 注意，这里如果先判断是否到达底部，会出现如果不满整个屏幕的话， 一直都是消费的到达底部，就不能下拉了。
	            if (getScrollY() == 0) {
	                System.out.println("到达顶部");
	            }  else if (childView != null && childView.getMeasuredHeight() <= getScrollY() + getHeight()) {
	                System.out.println("到达底部了 ");
	            }
	            break;
	    }
	    return false;
	}