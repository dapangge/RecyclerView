# 滚动字体 #
1. 总金额数据是从后台获取的。123456 ---  

2. 总金额需要进行转换，将金额转成亿万单位。 12 万 3456

3. 如果是数字，需要滚动。单位不需要滚动。

4. 进入界面，获取到数据，就需要自动滚动。

5. 滚动有一定的频率。

   1.继承ScrollView，每一个数字都是一个可以滑动的scrollview

   2.将所有的scrollview放到一个viewgroup，数字生成一个控件，控件可以滚动。

6. 流程：

   1.初始化滚动器

   2.计算要滚动的距离，目标位置 - 当前位置

   3.startScroll（当前x， 当前y， 距离x， 距离y， 时间），滚动计划

   4.当计划完成，computeScroll，调用scrollTo

   5.滚动速率

   ​	public void fling(int velocityY) { 

   ​		   super.fling(velocityY / 5);

   ​	}

初始化滑动器

	mScroller = new Scroller(context);
设置滚动速率

	 /**
	 *  更改速率
	 * @param velocityY
	 */
	@Override
	public void fling(int velocityY) {
	    super.fling(velocityY / 5);
	}
外界设置滚动的位置，默认正确的数字是最后一个，所以提供这个方法，外界调用的时候是传递的最后一个位置  
默认规则：生成10个随机数，如果数字是5，在生成5 个随机数，最后，将正确的数组添加到最后。

	/**
	 *  设置默认选项
	 * @param position
	 */
	public void setSeletion(final int position) {
	    postDelayed(new Runnable() {
	        @Override
	        public void run() {
	            WheelTextView.this.smoothScrollToSlow(0, position * itemHeight, 300 * position);
	        }
	    }, 500);
	}
	
	/**
	 * 调用此方法滚动到目标位置  duration滚动时间
	 */
	public void smoothScrollToSlow(int fx, int fy, int duration) {
	    int dx = fx - getScrollX();//mScroller.getFinalX();  普通view使用这种方法
	    int dy = fy - getScrollY();  //mScroller.getFinalY();
	    smoothScrollBySlow(dx, dy, duration);
	}
	
	/**
	 * 调用此方法设置滚动的相对偏移
	 * @param dx
	 * @param dy
	 * @param duration
	 */
	public void smoothScrollBySlow(int dx, int dy, int duration) {
	
	    //设置mScroller的滚动偏移量
	    mScroller.startScroll(getScrollX(), getScrollY(), dx, dy, duration);//scrollView使用的方法（因为可以触摸拖动）
	    //        mScroller.startScroll(mScroller.getFinalX(), mScroller.getFinalY(), dx, dy, duration);  //普通view使用的方法
	    invalidate();//这里必须调用invalidate()才能保证computeScroll()会被调用，否则不一定会刷新界面，看不到滚动效果
	}
	
	@Override
	public void computeScroll() {
	    //先判断mScroller滚动是否完成
	    if (mScroller.computeScrollOffset()) {
	        //这里调用View的scrollTo()完成实际的滚动
	        scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
	        //必须调用该方法，否则不一定能看到滚动效果
	        postInvalidate();
	    }
	    super.computeScroll();
	}

# 上拉 #

其实上啦可以很多种方式：

1.模拟recyclerview 的footview

2.直接在recyclerview下面增加要拉出来的layout

事件分发  
setOnTouchListener做了什么事情？  

![](imgs/46.bmp)

padding 和 margin 做了什么事情？  
ultra-ptr框架没有提供上拉加载，因为作者是这样一种思想：下拉跟上拉不是一个层次的功能。如果需要上拉加载，需要content内容自己去实现。所以，我们第一个界面：理财，有上拉的动作，那么此时就需要我们自己去实现上拉。我们可以拓展RecyclerView 去实现上拉。

该下拉刷新框架，没有上拉加载的功能，所以更多的是，只能自己去完成功能。  
理财界面，需要设计上拉记载，显示一些信息。此时我们可以通过几种方式来处理：  
1. 将要上拉的布局设置为RecyclerView 的footView，控制其onTouch方法。  
2. 将要上拉的布局，设置到RecyclerView布局的下方，控制其onTouch方法。 

## 设置布局 ##

	<com.m520it.tounaer.views.XRecyclerView
	    android:id="@+id/recycler"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"/>
	
	<LinearLayout
	    android:id="@+id/bottom_iv"
	    android:background="#00ffff"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:layout_alignParentBottom="true"
	    android:layout_below="@+id/recycler"
	    android:layout_centerHorizontal="true"
	    android:orientation="vertical"/>

# 拓展RecyclerView #

		layoutManager = new LinearLayoutManager(context);
	    setLayoutManager(layoutManager);
	    layoutManager.setStackFromEnd(false);// 默认不滑动到最后一行
	    setHasFixedSize(true);// 节约内存开销
	    setOnTouchListener(this);// 设置触摸监听
## 复写onTouch方法 ##

	@Override
	public boolean onTouch(View v, MotionEvent event) {
	    // 如果是触摸到了滑动控件，如果下拉的时候，没有到第一行，就不让下拉控件作用
	    switch (event.getAction()) {
	        case MotionEvent.ACTION_DOWN:
	            currentY = event.getY();// 记录点击的位置
	            break;
	        case MotionEvent.ACTION_MOVE:
	            if (isButtom()) {
	                float cY = currentY - event.getY();// 这个是为正，表示向上拉，为负，表示向下拉
	                if (cY <= 0 && !isShow) {// 如果是小于0，并且没有显示，表示是最后一条，但是是向下滑动
	                    return false;
	                }
	                if (isShow && cY >= 0) {// 如果是大于0，并且显示了，说明一直往上拉，没效果
	                    return true;// 返回true，表示不让recyclerview滑动，滑动监听到此为止，避免已经拉出来了布局，但是还是可以滑动上面的布局。
	                }
	                if (Math.abs(cY) >= maxHeight) {
	                    // 校验是否超过了最高值
	                    if (cY < 0) {
	                        cY = 0;
	                    } else {
	                        cY = maxHeight;
	                    }
	                }
	                updateMargin(cY);
	                return true;
	            }
	            break;
	        case MotionEvent.ACTION_CANCEL:
	        case MotionEvent.ACTION_UP:
	            if (isButtom()) {
	                float cY = currentY - event.getY();
	                if (cY > 0) {// 向上拉
	                    if (cY >= maxHeight / 2) {
	                        // 超过一半，就直接显示
	                        cY = maxHeight;
	                        isShow = true;
	                    } else {
	                        // 缩回去
	                        if (isShow) {// 如果向上拉，小于了一半，而且已经显示了，还是最大高度
	                            cY = maxHeight;
	                        } else {
	                            cY = 0;
	                            isShow = false;
	                        }
	                    }
	                } else {
	                    if (Math.abs(cY) <= maxHeight / 2) {
	                        // 超过一半，就直接显示
	                        cY = maxHeight;
	                        isShow = true;
	                    } else {
	                        // 缩回去
	                        cY = 0;
	                        isShow = false;
	                    }
	                }
	                updateMargin(cY);
	                return false;
	            }
	            break;
	    }
	    return false;
	}
## 更新界面 ##
	public void updateMargin(int cY){
	    RelativeLayout.LayoutParams lp = (RelativeLayout.LayoutParams) buttomView.getLayoutParams();
	    RelativeLayout.LayoutParams lp1 = new RelativeLayout.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
	    if (isShow && cY < 0){// 如果已经显示了，但是往下拉，那么就要重新设置这个值
	        cY = maxHeight + cY;
	    }
	    lp.setMargins(0, -cY, 0, 0);
	    lp1.setMargins(0, 0, 0, cY);
	    buttomView.setLayoutParams(lp);
	    setLayoutParams(lp1);
	    layoutManager.setStackFromEnd(true);// 设置view滚动到最后一条，这样表现出是被下面的布局顶上去的。
	}
# VectorDrawable #
大家觉得我们去定义的画对勾的方法如何？  
麻烦！而且对勾不好看。 

SVG的全称是Scalable Vector Graphics，叫可缩放矢量图形。它和位图（Bitmap）相对，SVG不会像位图一样因为缩放而让图片质量下降。它的优点在于节约空间，使用方便
Android 5.0中引入了 VectorDrawable 来支持矢量图(SVG)，同时还引入了 AnimatedVectorDrawable 来支持矢量图动画，在最近几次Support包更新之后，SVG的兼容性问题得以大大改善。  

VectorDrawable 并没有支持所有的 SVG 规范，目前只支持 PathData 和有限的 Group 功能。另外还有一个 clip-path 属性来支持后面绘图的区域。 所以对于使用 VectorDrawable 而言，我们只需要了解 SVG 的 PathData 规范即可。  

	// 阿里矢量图标库	
	http://www.iconfont.cn/home/index?spm=a313x.7781069.1998910419.1.V1n5x6

![](imgs\41.png)   
在浏览器打开这个矢量图，点击右键，查看源码。 

 
两种方式去创建一个Vectorable  
![](imgs\42.png)  
或者，直接创建一个图片资源文件  
![](imgs\43.png)  

## 属性基本介绍 ##

### vector元素 ###
有以下几个属性：  
  name：定义该矢量图形的名字。通过名字找到这个矢量图。  

  width，height：定义该矢量图形的固有宽高(必须的，矢量图内部的宽高intrinsic) 

  viewportHeight，viewportWidth：定义画布(viewport)的大小，不需要指定单位。但大小可以理解为一个虚拟单位，将drawable的宽高分成多少等份，在定义path的时候所有数值都是说取drawable宽高的多少份。如viewportWidth和viewportHeight分别为32，32，在path中(16,16)便表示在drawable宽高的中间。所有控制点都必须在viewportWidth和viewportHeight内，超出的部分交不予显示。该属性为必需值。
​    
  alpha：图片的不透明度。  

  tint 定义该 drawable 的 tint 颜色。默认是没有 tint 颜色的。  

  tintMode 定义 tint 颜色的 Porter-Duff blending 模式，默认值为 src_in。  

  autoMirrored 设置当系统为 RTL (right-to-left) 布局的时候，是否自动镜像该图片。比如 阿拉伯语。  

### path元素 ###

 name：路径的名称。可以在其他地方来引用。  

​	要画动画了，路径1， 到路径2.

 fillColor：图形的填充颜色。设置该属性值后，得到的svg图形就会填充满。 

 strokeColor：边界的颜色。  

 strokeWidth：边界的宽度。 

 strokeAlpha：边界透明度。 

 **pathData**：定义控制点的位置。 

 trimPathEnd：从开始到结束，显示百分比，0，不显示，1 显示。

 trimPathStart：从开始到结束，隐藏百分比，0 不隐藏，1 隐藏。 

 trimPathOffset：设置截取范围。  

 strokeLineCap 设置路径线帽的形状，取值为 butt, round, square.  
 ![](imgs\45.png)  
 strokeLineJoin 设置路径交界处的连接方式，取值为 miter,round,bevel.

 strokeMiterLimit 设置路径交叉时候，斜角的上限。当 strokeLineJoin 为 “round” 或 “bevel” 的时候，这个属性无效。为miter时，锐角相交，可能斜面会很长，不协调。所以就为交界的斜面设置上限。默认是10.意味着一个斜面的长度不应该超过线条宽度的 10 倍。  
### group ###
有时候我们需要对几个路径一起处理，这样就可以使用 group 元素来把多个 path 放到一起。group 主要是用来设置路径做动画的关键属性的。  

 name 定义 group 的名字  

 rotation 定义该 group 的路径旋转多少度  

 pivotX 定义缩放和旋转该 group 时候的 X 参考点。该值相对于 vector 的 viewport 值来指定的。  

 pivotY 定义缩放和旋转该 group 时候的 Y 参考点。该值相对于 vector 的 viewport 值来指定的。  

 scaleX 定义 X 轴的缩放倍数  

 scaleY 定义 Y 轴的缩放倍数  

 translateX 定义移动 X 轴的位移。相对于 vector 的 viewport 值来指定的。  

 translateY 定义移动 Y 轴的位移。相对于 vector 的 viewport 值来指定的。  

### pathData ###

所有的参数，都可以大写小写，大写表示是绝对位置。小写是相对位置

所有的值，都可以用空格来间隔开，或者用逗号间隔开

    M：move to 移动绘制点, 一个坐标
    L：line to 直线，一个坐标
    Z：close 闭合，不要参数
    C：三次贝塞尔曲线，三个坐标，前两个为贝塞尔曲线的控制点的坐标，最后一个终点的坐标。
    S：同C，但比C要更平滑。
    Q：二次贝塞尔曲线，两个坐标，第一个表示贝塞尔曲线的控制点坐标，第二个终点的坐标。
    T：同Q，但比q平滑。
    A：ellipse 圆弧，七个参数，
    	(rx ry rotation big_flag sweep_flag x y)
    	rx ry，弧线所属椭圆的半轴的xy的长度，如果xy相等，那么就是一个圆。
    	rotation 旋转角度
    	big_flag 是否大圆 1 为 是， 0 不是
    	sweep_flag 是否顺时针 1 为 是， 0 不是
    	x y 终点坐标
    H:水平画一条直线到指定位置，一个坐标
    v:垂直画一条直线到指定位置 ，一个坐标
    //指令的大小写分别代表着绝对定位与相对定位。绝对定位指的是这个点在drawable中的坐标，而相对定位指的是这个点相较于前一个点移动的坐标。

### clip_path ###
vector 还支持 clip-path 元素。定义当前绘制的剪切路径。注意，clip-path 只对当前的 group 和子 group 有效。 

name 定义 clip path 的名字  

pathData 和 android:pathData 的取值一样。   


VectorDrawable，最低到api 7，但AnimatedVectorDrawableCompat到api 11。如果想要兼容之下的版本。需要对gradle进行配置。

	android {
	    defaultConfig {
	        vectorDrawables.useSupportLibrary = true
	    }
	}

定义好的矢量图文件，在布局中使用app:srcCompat标签，需要使用activity继承于AppCompatActivity，需要用这个标签 ，首页的HomeActivity继承于AppCompatActivity，而AppCompatActivity也继承了FragmentActivity 
### 画太极 ###
![](imgs\44.png)

	<vector xmlns:android="http://schemas.android.com/apk/res/android"
	    android:height="50dp"
	        android:width="50dp"
	        android:viewportHeight="100.0"
	        android:viewportWidth="100.0"
	    >
	    <!--左边的是两个四分之一圆-->
	    <path
	        android:fillColor="#000000"
	        android:name="黑鱼"
	        android:pathData="M0 50,
	            A50 50 0 0 1 50 0,
	            A25 25 0 0 0 50 50,
	            A25 25 0 0 1 50 100,
	            A50 50 0 0 1 0 50"/>
	    <path
	        android:fillColor="#ffffff"
	        android:name="白眼"
	        android:pathData="M43 75,A7 7 0 1 1 57 75, A7 7 0 1 1 43 75"/>
	    <!--右边画一个半圆-->
	    <path
	        android:fillColor="#FFFFFF"
	        android:name="白鱼"
	        android:pathData="M50 0,
	            A50 50 0 1 1 50 100,
	            A25 25 0 1 0 50 50,
	            A25 25 0 1 1 50 0"/>
	    <path
	        android:fillColor="#000000"
	        android:name="黑眼"
	        android:pathData="M43 25,A7 7 0 1 1 57 25, A7 7 0 1 1 43 25"/>
	</vector>	

### 画对勾 ###
如何达到镂空效果呢？在代码中，我们可以设置遮罩类型，但是在矢量图中，不能设置镂空，所以我们只能不在镂空处绘制，相当于就镂空了。只是这样稍微线段多一点而已。

	<?xml version="1.0" encoding="utf-8"?>
	<vector xmlns:android="http://schemas.android.com/apk/res/android"
	    android:name="对勾"
	        android:height="20dp"
	        android:width="20dp"
	        android:viewportHeight="100.0"
	        android:viewportWidth="100.0"
	    >
	    <!--
	        上半部分和下班部分分开绘制，这样才能达到镂空的效果。
	    -->
	    <path
	        android:name="圆形1"
	        android:fillColor="#e9d94c"
	        android:pathData="M0 50, A50 50 0 1 1 100 50,L70 50 70 30 30 30 30 50z"/>
	    <path
	        android:name="圆形2"
	        android:fillColor="#e9d94c"
	        android:pathData="M0 50, A50 50 0 1 0 100 50,L70 50 70 70 30 70 30 50z"/>
	    <!--两条线相交-->
	   <path
	        android:name="对勾"
	        android:strokeWidth="8"
	        android:strokeColor="#ff0000"
	        android:strokeLineCap="square"
	        android:strokeLineJoin="round"
	        android:pathData="M30 40, L45 70, L80 25"/>
	</vector>

### VectorDrawable动画 ###
1. 创建animator_vector 资源文件
2. 在资源文件中定义target，指定为某个path或者group去设定动画
3. 创建objectAnimator资源文件。注意，这个文件，需要定义在values/animator文件夹下。

定义一个笑脸，从哭脸变成笑脸。  

	<?xml version="1.0" encoding="utf-8"?>
		<vector xmlns:android="http://schemas.android.com/apk/res/android"
	    android:name="smile"
	    android:width="30dp"
	    android:height="30dp"
	    android:viewportHeight="100"
	    android:viewportWidth="100"
	>
	    <!--脸-->
	    <path
	        android:name="smile_circle"
	        android:fillColor="#f7ec22"
	        android:pathData="M0 50, A50 50 0 0 1 100 50, A50 50 0 0 1 0 50"/>
	    <!--嘴巴-->
	    <path
	        android:name="mouth"
	        android:strokeWidth="5"
	        android:strokeColor="#00ff00"/>
	</vector>

定义笑脸动画：

	<?xml version="1.0" encoding="utf-8"?>
	<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
	                android:duration="1000"
	                android:propertyName="pathData"
	                android:valueFrom="M30 70, Q50 50 70 70"
	                android:valueTo="M30 70, Q50 90 70 70"
	                android:valueType="pathType"
	    >
	</objectAnimator>
关联动画和矢量图  

	<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
	     android:drawable="@drawable/smile"
	    >
	
	    <target
	        android:animation="@animator/smile_anim"
	        android:name="mouth"/>
	</animated-vector>

在布局中使用vector矢量图文件  

	<ImageView
	    android:id="@+id/iv"
	    android:background="#000000"
	    android:layout_width="60dp"
	    android:layout_height="60dp"
	    app:srcCompat="@drawable/smile_anim"
	    />

在代码中开启动画

	ImageView iv = (ImageView) findViewById(R.id.iv);
	Animatable a = (Animatable) iv.getDrawable();
	a.start();
