# 加载框 #
![](imgs\21.png)  
## 需求分析 ##
1. 加载框是由一个背景和动态移动的水波组成  
2. 可以绘制一个背景，增加遮罩效果，在绘制贝塞尔path曲线，并且动态的更改贝塞尔曲线的高度。
3. 将一个图片直接画在view中 ，仅仅是说，这个图片是一个背景+前景 合成的一个图片。 
## 功能实现 ##
### 绘制背景图片 ###
1. 需要遮罩，遮罩只能在图片中使用，所以，先生成一张图片  

   	// 从资源文件中获取生成图片
   	bmp = BitmapFactory.decodeResource(getResources(), R.mipmap.aa);
   	// 创建一个空白的图片
   	 xfModeBitmap = Bitmap.createBitmap(bmp.getWidth(), bmp.getHeight(), Bitmap.Config.ARGB_8888);
2. 将原图绘制到空白图片中  

   	canvas = new Canvas(xfModeBitmap);// 根据空白画布，绘制原图
   	 canvas.drawBitmap(bmp, 0, 0, paint);
   	// 设置遮罩
   	 paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
   ![](imgs\23.png)  
   这里就是遮罩模式，注意，先绘制的是dest（背景），后绘制的是src（前景）  

### 绘制线条 ###
贝塞尔曲线：  

	    http://blog.csdn.net/sd19871122/article/details/51282762
Path对象中，有绘制贝塞尔曲线的方法。  
quadTo：二阶贝塞尔曲线。一个点控制曲线。参数为一个控制点，和终点。  
cubicTo：三阶贝塞尔曲线。两个点控制曲线。参数分别为控制点1，控制点2，终点。  
分析动画：根据波纹效果，需要有值改变，才能实现动画。  
x轴增加，y轴向上减小。  

		private float waveY;// 水波高度
		private float cX;// 贝塞尔x轴控制变量
		private float cY;// 贝塞尔y轴控制变量

绘制：  

		// 使用path画一条线
	    if (cX >= bmp.getWidth()){// 如果x方向移动到最末，则表示可以反向递减
	        replay = true;
	    } else if (cX <= 0){// 否则，移动到最左边，就正向递增
	        replay = false;
	    }
	    cX = replay ? cX - 10 : cX + 10;// 每次递增10像素
	    if (cY >= 0){// 如果y轴控制点没有移动到最上面，控制坐标递减，水波高度递减，就是水波上升
	        cY--;
	        waveY--;
	    } else {// 否则，重置数据。
	        waveY = 7 / 8f * bmp.getHeight();
	        cY = 17 / 16f * bmp.getHeight();
	    }
	    path.moveTo(0, waveY);
	    path.cubicTo(cX / 2, waveY - (cY - waveY), (cX + bmp.getWidth()) / 2, cY, bmp.getWidth(), waveY);// 通过控制变量，更改两个控制点的位置
	    path.lineTo(bmp.getWidth(), bmp.getHeight());
	    path.lineTo(0, bmp.getHeight());
	    path.close();// 路径闭合，最后一个点moveTo到第一个点，进行路径闭合。
		canvas.drawPath(path, paint);// 绘制路径
	    paint.setXfermode(null);// 每次绘制完毕之后，重置遮罩效果

背景使用一个drawable位图  
发现一个问题，![](imgs\25.png),发现有脏的像素点。  
​	
	// 在生成空白位图的时候，调用方法，擦除像素点，再绘制drawable图片。
	xfModeBitmap.eraseColor(Color.parseColor("#00ffffff"));

### 自定义属性 ###
优化方案：通过布局文件，调用者可以写入背景图片，以及水波颜色。  
# 自定义键盘 #
![](imgs\27.png)  
## 功能分析 ##
1. 每一个自定义的按钮，都可以存放数据，并且，点击到的时候，需要知道点击的是谁。  
2. 可以为每一个按钮设置背景颜色或者资源图片。  
3. 每一个控件可以设置权重。  
4. 还有一些其他的功能键，比如字符转换，比如删除，比如数字切换。  
5. 需要隐藏系统的键盘。  
### 单个按钮，绘制文字 ###
思考一个问题，如果确定文字的位置？  
想要确定一个文字的位置，需要有什么前提？  
只要，我们能够知道文字的大小，从x坐标的什么位置开始写，基准线y坐标是多少，就能确定文字的位置。  
所以，canvas中绘制文本的方法，其中传递的两个参数，就是x开始坐标和基准线。  

#### 四格线与基线 ####  
![](imgs\28.png)  
![](imgs\29.png)  

#### 获取文字宽高的方法 ####
一般，我们可以采用多种方式来获取到文字的宽高信息。  
1. 通过画笔去测量文本之后获取到文本信息。  

![](imgs\30.png)  
2. 通过画笔的文本字体大小，来获取到文本的路径标准。
   ![](imgs\32.png)  

3. 获取到文本的最小内容边界。  

   // 		    方式三：获取文本的边界，注意，需要知道文本内容
                Rect rect = new Rect();
                textPaint.getTextBounds(mItem.getData(), 0, mItem.getData().length(), rect);
                canvas.drawText(mItem.getData(), width / 2 - (rect.right - rect.left) / 2,
                        height / 2 - rect.top - (-rect.top + rect.bottom) / 2, textPaint
                        );
                // 方式二：获取文字的metric
   //                Paint.FontMetrics pf = textPaint.getFontMetrics();
   //                float textWidth = textPaint.measureText(mItem.getData());
   //                canvas.drawText(mItem.getData(), width / 2 - textWidth / 2 ,
   //                        height / 2 + (-pf.ascent - (pf.descent - pf.ascent) / 2), textPaint);
               // 方式一：使用画笔的方法
   //                canvas.drawText(mItem.getData(), width / 2 - textWidth / 2 ,
   //                        height / 2 + (-textPaint.ascent() - (textPaint.descent() - textPaint.ascent()) / 2), textPaint);

#### measureText 和 getTextBounds 区别 ####
![](imgs\31.png)  
### 每个按钮中的数据对象 ###

	public class KeyItem {

	    private int type;// 表示的类型，数字、拼音、符号或自定义的等等
	    private String data;// 键盘的内容，具体是哪一个字符
	    private int resoure = -1;// 如果是功能键，支持一个本地资源图片，暂时不写
	    private int weight;// 这个按钮所占的权重
	}
### 绘制键盘 ###
1.添加数据。
​	
	Map<Integer, List<KeyItem>> 键为行号，值为每一行的数据。  
2.测量布局宽高，测量子控件宽高  
​	
		// 通过父布局传入的mode来计算控件的宽高
	    Set<Map.Entry<Integer, List<KeyItem>>> entrys = datas.entrySet();
	    Iterator<Map.Entry<Integer, List<KeyItem>>> it = entrys.iterator();
	    int childCount = 0;
	    int height = 0;
	    while(it.hasNext()){
	        Map.Entry<Integer, List<KeyItem>> entry = it.next();
	        List<KeyItem> list = entry.getValue();
	        int allWeight = 0;
	        for (KeyItem item : list){
	            allWeight += item.getWeight();
	        }
	        int childHeightMode = MeasureSpec.makeMeasureSpec((int) evHeight, MeasureSpec.EXACTLY);
	        for (int i = 0; i < list.size(); i++){
	            int totalSize = (int) (evWidth - 2 * margin - padding * (list.size() - 1));
	            int childWidthMode = MeasureSpec.makeMeasureSpec(
	            // 总宽度 - 布局的margin值 - 两边间距 - 中间的间距   /  总权重   *  当前按钮所占的权重。
	                    totalSize / allWeight * list.get(i).getWeight(),
	                    MeasureSpec.EXACTLY);
	            getChildAt(childCount + i).measure(childWidthMode, childHeightMode);
	        }
	        height += evHeight;
	        childCount += list.size();
	    }
	    setMeasuredDimension((int)evWidth, height);
3.子控件排版  

	@Override
	protected void onLayout(boolean changed, int l, int t, int r, int b) {
	    // 将各个控件放在viewgroup中
	    Set<Map.Entry<Integer, List<KeyItem>>> entrys = datas.entrySet();
	    Iterator<Map.Entry<Integer, List<KeyItem>>> it = entrys.iterator();
	    int line = 0;
	    int childCount = 0;
	    while(it.hasNext()){
	        l = r = 0;// 每一行
	        Map.Entry<Integer, List<KeyItem>> entry = it.next();
	        List<KeyItem> list = entry.getValue();
	        // 每一个
	        for (int i = 0; i < list.size(); i++){
	            View view = getChildAt(childCount + i);
	
	            if (i == 0){
	                // 第一个
	                l = 0 + padding;
	            }
	            r = l + view.getMeasuredWidth();
	            view.layout(l,(int)(line * evHeight + padding), r,(int)((line + 1) * evHeight));
	            l = r + padding;
	
	        }
	        childCount += list.size();
	        line++;
	    }
	}
4.设置每个按钮的点击事件。  
​	
	public interface IOnclickKey {
		// 点击普通按钮，有按钮对应的数据
	    void onClickKey(KeyboardButton keyboardButton, String data);
		// 点击功能按钮，有每个功能键对象
	    void onClickFunctionKey(KeyboardButton keyboardButton, KeyItem item);
	}
5.可以自己定义一个复杂的键盘，为功能键提供切换键盘的方法。  
6.结合自定义的键盘，需要解决在模拟器中使用电脑键盘输入的问题。  
​	
	((SecretEditText)this.editText).setOnSelectionChangedListener(new ICursorChanged() {
	        @Override
	        public void onCursorChanged(int selStart, int selEnd) {
	            setCursor(selStart);// 如果是系统键盘，那么，这个地方还是可以输入东西的。
	            // 当手动改变输入框的焦点的时候，会执行这个回调方法
				// 模拟器点击，也会执行这个方法
	        }
	    });
	
	// 当点击edittext的时候，设置cursor
	public void setCursor(int cursor) {
	    this.cursor = cursor;
	}


	 @Override
	public void onKeyDown(KeyboardButton keyboardButton, String data) {
	    content.insert(cursor, data);// 应该是使用插入的方法，而不是直接添加到最后。插入的位置就是cursor
	    this.editText.setTextKeepState(content.toString());
	    cursor++;// 自定义键盘上面增加数据
	    this.editText.setSelection(cursor);
	}

7.禁止软键盘弹出。  

	/**
	 * 强制隐藏输入法键盘
	 */
	public static void hideInput(Context context, EditText view) {
	    InputMethodManager inputMethodManager =
	            (InputMethodManager) context.getSystemService(Context.INPUT_METHOD_SERVICE);
	    inputMethodManager.hideSoftInputFromWindow(view.getWindowToken(), 0);
	}
	// 禁止软键盘弹出1
	public static void disableShowSoftInput(EditText et) {
	    ((Activity)et.getContext()).getWindow().setSoftInputMode(
	            WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_HIDDEN);
	
	    int currentVersion = android.os.Build.VERSION.SDK_INT;
	    String methodName = null;
	    if (currentVersion >= 16) {
	        // 4.2
	        methodName = "setShowSoftInputOnFocus";
	    } else if (currentVersion >= 14) {
	        // 4.0
	        methodName = "setSoftInputShownOnFocus";
	    }
	
	    if (methodName == null) {
	        et.setInputType(InputType.TYPE_NULL);
	    } else {
	        Class<EditText> cls = EditText.class;
	        Method setShowSoftInputOnFocus;
	        try {
	            setShowSoftInputOnFocus = cls.getMethod(methodName,
	                    boolean.class);
	            setShowSoftInputOnFocus.setAccessible(true);
	            setShowSoftInputOnFocus.invoke(et, false);
	        } catch (NoSuchMethodException e) {
	            et.setInputType(InputType.TYPE_NULL);
	            e.printStackTrace();
	        } catch (IllegalAccessException e) {
	            // TODO Auto-generated catch block
	            e.printStackTrace();
	        } catch (IllegalArgumentException e) {
	            // TODO Auto-generated catch block
	            e.printStackTrace();
	        } catch (InvocationTargetException e) {
	            // TODO Auto-generated catch block
	            e.printStackTrace();
	        }
	    }
	}
	
	// 禁止软键盘弹出2
	private void disableShowSoftInput() {
	    if (Build.VERSION.SDK_INT <= 10){
	        this.editText.setInputType(InputType.TYPE_NULL);
	    } else  {
	        Class<EditText> cls = EditText.class;
	        Method method;
	        try{
	            method = cls.getMethod("setShowSoftInputOnFocus", boolean.class);
	            method.setAccessible(true);
	            method.invoke(this.editText, false);
	        }catch ( Exception e){
	            ;
	        }
	        try{
	            method = cls.getMethod("setSoftInputShownOnFocus", boolean.class);
	            method.setAccessible(true);
	            method.invoke(this.editText, false);
	        }catch ( Exception e){
	            ;
	        }
	    }
	}
# SpannableString #
![](imgs\26.png)  
查看app，我们可以看到，在注册信息界面，以及理财首页，都有不同颜色，不同大小的字体。如何实现这种功能呢？我们第一反应应该是自定义，因为不可能自己使用一大堆的textView去拼接。  

TextView 有一个方法，可以设置SpannableString对象。该SpannableString可以通过构造器初始化一个字符串。SpannableString对象有一个很重要的方法：setSpan。  

setSpan 需要四个参数：

        sb.setSpan(new ForegroundColorSpan(Color.RED), 2, 2 + number.length(), Spannable.SPAN_INCLUSIVE_EXCLUSIVE);

参数一：需要传递一个Span对象。
​	
	1、BackgroundColorSpan 背景色 
	2、ClickableSpan 文本可点击，有点击事件
	3、ForegroundColorSpan 文本颜色（前景色）
	4、MaskFilterSpan 修饰效果，如模糊(BlurMaskFilter)、浮雕(EmbossMaskFilter)
	5、MetricAffectingSpan 父类，一般不用
	6、RasterizerSpan 光栅效果
	7、StrikethroughSpan 删除线（中划线）
	8、SuggestionSpan 相当于占位符
	9、UnderlineSpan 下划线
	10、AbsoluteSizeSpan 绝对大小（文本字体）
	11、DynamicDrawableSpan 设置图片，基于文本基线或底部对齐。
	12、ImageSpan 图片
	13、RelativeSizeSpan 相对大小（文本字体）
	14、ReplacementSpan 父类，一般不用
	15、ScaleXSpan 基于x轴缩放
	16、StyleSpan 字体样式：粗体、斜体等
	17、SubscriptSpan 下标（数学公式会用到）
	18、SuperscriptSpan 上标（数学公式会用到）
	19、TextAppearanceSpan 文本外貌（包括字体、大小、样式和颜色）
	20、TypefaceSpan 文本字体
	21、URLSpan 文本超链接 

参数二：该span作用的起始位置。  
参数三：该span作用的结束为止。
参数四：

	//前面包括，后面不包括
	public static final int SPAN_INCLUSIVE_EXCLUSIVE = SPAN_MARK_MARK;  
	// 前面包括后面包括  
	public static final int SPAN_INCLUSIVE_INCLUSIVE = SPAN_MARK_POINT;  
	// 前不包括后面不包括
	public static final int SPAN_EXCLUSIVE_EXCLUSIVE = SPAN_POINT_MARK;  
	// 前面不包括后面包括
	public static final int SPAN_EXCLUSIVE_INCLUSIVE = SPAN_POINT_POINT; 

包括的含义是：TextView可以使用append方法追加文本，前包括表示：该样式可以应用的追加到前面的文本。后包括表示：该样式可以应用到追加到后面的文本。 

# 自定义不可见的输入框 #
一般，我们见过这种输入框。可以明文密文切换。
我们可以自定义一个Viewgroup，去包裹一个输入框以及一张图片。今天我们使用另外一种方式。  
在EditText中，有一个这样的方法：  

	android:drawableRight="@drawable/password_hide"
可以为EditText设置上下左右的图片。  

		// 取出后边的可见符号，增加点击时间
	    mDrawable = getCompoundDrawables()[2];// 该数组有四个元素，分别是左上右下四个drawable，如果返回null，表示那个位置没有设置对应的drawable
	    mDrawable.setBounds(0, 0, mDrawable.getIntrinsicWidth(), mDrawable.getIntrinsicHeight());// 指定隐藏的矩形区域
	    mShowDrawable = getResources().getDrawable(R.drawable.password_see);
	    mShowDrawable.setBounds(0, 0, mShowDrawable.getIntrinsicWidth(), mShowDrawable.getIntrinsicHeight());// 指定显示的矩形区域，表示绘制图片的时候，需要将这个drawable绘制到那个矩形范围中。
	    // 设置input 为 password，那么就可以实现先显示明文，后显示密文这种效果，否则就一直是密文一直是明文
	    setTransformationMethod(PasswordTransformationMethod.getInstance());
## 点击切换 ##
1. 需要能够点击到这个位图  
2. 需要能够切换明文密文
   如何点击呢？drawable是一个位图，不是一个控件，我们不能直接为其添加点击事件监听。  
   所以，我们用onTouch触摸方法，来模拟一下点击到了这个位图。  
### onTouch ###

			//getTotalPaddingRight():图标左边缘至控件右边缘的距离
	        //getPaddingRight():图标右边缘至控件右边缘的距离
	        boolean touchable = event.getX() > (getWidth() - getTotalPaddingRight())
	                && (event.getX() < ((getWidth() - getPaddingRight())));
	        if (touchable){
	            if (event.getAction() == MotionEvent.ACTION_UP){
	                // 如果点击了，就显示内容或者是显示星星星
	                isShowContent = !isShowContent;
	                // 注意光标应该是留在当前光标位置
	                if (isShowContent){// 显示明文
	                    setTransformationMethod(HideReturnsTransformationMethod.getInstance());// 切换成明文
	                    setCompoundDrawables(null, null, mShowDrawable, null);// 更换图标
	                } else {
	                    setCompoundDrawables(null, null, mDrawable, null);
	                    setTransformationMethod(PasswordTransformationMethod.getInstance());
	                }
	                postInvalidate();
	            }
	        }

# 首页框架搭建 #
## 界面布局 ##

页面布局  

	 <!--标签设置到最下面，这个FrameLayout是显示内容的真正区域-->
	<FrameLayout
	    android:layout_width="match_parent"
	    android:layout_height="0dp"
	    android:id="@+id/tabContent"
	    android:layout_weight="1"
	    />
	<com.m520it.tounaer.views.FragmentTabHost
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"
	    android:id="@+id/tabs"
	    >
	    <LinearLayout
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:orientation="vertical"
	        >
	        <!--删除tabwidget，设置下面的内容为0dp-->
	        <FrameLayout
	            android:layout_width="0dp"
	            android:layout_height="0dp"
	            android:id="@android:id/tabcontent"
	            />
	    </LinearLayout>
	</com.m520it.tounaer.views.FragmentTabHost>
自定义的FragmentTabHost  

修改doTabChanged方法中的这句代码  

	//这个方法会触发onDestoiyView,hide不会触发任何生命周期方法
	//ft.detach(mLastTab.fragment);// 不能让tab在切换的过程中重新创建布局，执行生命周期方法。
	ft.hide(mLastTab.fragment);

创建TabSpec

		titles = getResources().getStringArray(R.array.tab_title);
	    Class[] f_classs = {// 
	            InverstFragment.class,
	            DynamicFragment.class,
	            WelfareFragment.class,
	            MyAccountFragment.class};
	    int[] selector ={
	            R.drawable.inverst_select,
	            R.drawable.ico_dynamic_select,
	            R.drawable.ico_welfare_select,
	            R.drawable.myaccount_select};
	
	    host = (FragmentTabHost) findViewById(R.id.tabs);
	    //containId 放置Fragment的容器id
	    host.setup(HomeActivity.this,getSupportFragmentManager(),R.id.tabContent);
	
	    for(int i=0;i<titles.length;i++){
	        View view = getTabView(i,titles,selector);// 创建自定义tab样式
	        TabHost.TabSpec tmp =  host.newTabSpec(i+"");
	        //自定义控件的UI，自定义每个tab的样式
	        tmp.setIndicator(view);
	        //添加标签到tabhost中
	        host.addTab(tmp, f_classs[i],null);// tab和页面进行绑定
	    }

每个页面都需要下拉，所以，我们应该思考一个问题：  
1. 将各个界面抽取成一个BaseFragment，这个超类中，处理下拉刷新，刷新成功失败等等一系列的逻辑。
2. 使用下拉控件：  

    compile 'in.srain.cube:ultra-ptr:1.0.11'  
   /*
   	继承与Viewgroup，支持包裹任何view  
   	和 Google 官方推出 SwipeRefreshLayout 是相同的设计思路，但对比 SwipeRefreshLayout ， UltraPTR 更灵活，更容易拓展。	
   */
### ultra_ptr下拉框架介绍： ###

1. 首先抽象出了两个接口，功能接口和 UI 接口。

PtrHandler 代表下拉刷新的功能接口，包含刷新功能回调方法以及判断是否可以下拉的方法。用户实现此接口来进行数据刷新工作。

PtrUIHandler 代表下拉刷新的 UI 接口，包含准备下拉，下拉中，下拉完成，重置以及下拉过程中的位置变化等回调方法。通常情况下， Header 需要实现此接口，来处理下拉刷新过程中头部 UI 的变化。  

2. 整个项目围绕核心类 PtrFrameLayout 。 PtrFrameLayout 代表了一个下拉刷新的自定义控件。

PtrFrameLayout 继承自 ViewGroup ，有且只能有两个子 View ，头部 Header 和内容 Content 。通常情况下 Header 会实现 PtrUIHandler 接口， Content 可以为任意的 View 。  

3. PtrHandler.java

       // 提供给调用者，判断是否可以下拉刷新。如果返回false，则不能下拉。
       public boolean checkCanDoRefresh(final PtrFrameLayout frame, final View content, final View header)
4. PtrDefaultHandler.java，为上面的方法提供了默认实现。  
5. PtrUIHandler.java 下拉刷新 UI 接口，提供了刷新的各个时期的方法。
6. PtrClassicFrameLayout.java  
    继承 PtrFrameLayout.java，经典下拉刷新实现类。添加了 PtrClassicDefaultHeader 作为头部，用户使用时只需要设置 Content 即可。

BaseFragment布局中使用下拉标签PtrClassicFrameLayout

	<in.srain.cube.views.ptr.PtrClassicFrameLayout
	    android:id="@+id/pull_layout"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent">
	    <RelativeLayout
	        android:id="@+id/rl_content"
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:background="#FFFFFF"
	        android:gravity="center_horizontal"
	        android:orientation="vertical">
	    </RelativeLayout>
	</in.srain.cube.views.ptr.PtrClassicFrameLayout>

下拉设置头布局控件

	pullLayout.setHeaderView(header);

### 是否处理上拉下拉 ###

			public boolean checkCanDoRefresh(PtrFrameLayout frame, View content, View header1) {

                // 检查是否可以下拉，这里如果有listview 或者 gridview的话，需要处理这里的逻辑，否则会事件冲突
                if (dispatchPullDown()){// 如果自己处理，就自己处理，就返回自己的处理，否则就返回默认的处理方式
                    return customPullDown();
                }
                return PtrDefaultHandler.checkContentCanBePulledDown(frame, rl_content, header1);
            }


	    // 是否拦截下拉方法，如果不拦截下拉，就返回默认的下拉处理方式
	    public abstract  boolean dispatchPullDown();
	    // 如果拦截下拉方法，是否是需要下拉
	    protected abstract boolean customPullDown();
	    public abstract void setCanPullDown(boolean canPullDown);

### 进度加载 ###

	/**
	 *  进入界面刷新完毕，则隐藏中间水波加载框
	 */
	public void loadFinish(){
	    mLoadView.setVisibility(View.GONE);
	}
	public void loadStart(){
	    mLoadView.setVisibility(View.VISIBLE);
	}