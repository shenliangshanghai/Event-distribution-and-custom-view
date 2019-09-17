## 自定义view
### 1、页面绘制过程
```
Activity创建完成之后--->DecorView添加到Window上----->ViewRootImpl实现与DecoreView进行绑定
----->ViewRootImple调用performTraversals---->performMeasure、performLayout、performDraw
------>（以measure为例）调用DecoreView的measure方法------>measure调用onMeasure方法
----->onMeasure方法会调用子view的measure
```
### 2、注意点
```
   1、onMeasure方法调用完成之后，可以使用getMeasureWidth,getMeasureHeight获取View的测量宽高，大部分情况下就是最终宽高，特殊情况下不是
   2、Layout过程之后可以确定View的left,top,right,bottom,和实际宽高，getWidth,getHeight就是最终值
   3、contentView 的id是android.R.id.content,设置进去的view可以通过 View contentView= findViewById(android.R.id.content);  View view=contentView.getChildAt(0);
```
### 3、MeasureSpec
```
   1、ParentView.MeasureSpec  +  View.LayoutParams  = View.MeasureSpec
   2、View.MeasureSpec  --->  View.MeasureHeight、View.MeasureWidth,不一定是Width、Height
   3、MeasureSpec(32位)  =  specMode(2位) + specSize（30位）
```
### 4、SpecMode
```
   1、UNSPECIFIED （未指定）  没有限制，需要多大给多大，一般用于系统内部
   2、EXACTLY (精确的) 对应于Match_parent 和 具体数值
   3、AT_MOST (最多) 对应于wrap_content,但是不能大于parentView指定的大小 SpecSize
```
### 5、getChildMeasureSpec伪代码
```java
   publica stataic getChildMeasureSpec(int parentSpec,int paddingAndMarggin,int childDimension){
       int parentSpecMode=MeasureSpec.getSpecMode(parentSpec);
       int parentSpecSize=MeasureSpec.getSpecSize(parentSpec);
	   //父布局规则的尺寸去掉子view的剩余尺寸，即子view可以用的最大尺寸
	   int remainSize=Math.max(0,parentSpecSize-paddingAndMarggin);
	   int childSize=0;
	   int childMode=0;
	   switch(parentSpecMode){
	     case MeasureSpec.EXACTLY:
		   if(childDimension>=0){
		      childMode=MeasureSpec.EXACTLY;
			  childSize=childDimension;
		   }else if(childDimension==LayoutParams.MATCH_PARENT){
		      childMode=MeasureSpec.EXACTLY;
			  childSize=remainSize;
		   }else if(childDimension==LayoutParams.WRAP_CONTENT){
		      childMode=MeasureSpec.AT_MOST;
			  childSize=remainSize;
		   }
           breake;
		 case MeasureSpec.AT_MOST:
		   if(childDimension>=0){
		      childMode=MeasureSpec.EXACTLY;
			  childSize=childDimension;
		   }else if(childDimension==LayoutParams.MATCH_PARENT){
		      childMode=MeasureSpec.AT_MOST;
			  childSize=remainSize;
		   }else if(childDimension==LayoutParams.WRAP_CONTENT){
		      childMode=MeasureSpec.AT_MOST;
			  childSize=remainSize;
		   }
		   break;
		case MeasureSpce.UNSPECIFIED:
		   if(childDimension>=0){
		      childMode=MeasureSpce.EXACTLY;
			  childSize=childDimension;
		   }else if(childDimension==LayoutParams.MATCH_PARENT){
		      childMode=MeasureSpce.UNSPECIFIED
			  childSize=0;
		   }else if(childDimension==LayoutParams.WRAP_CONTENT){
		      childMode=MeasureSpce.UNSPECIFIED;
			  childSize=0;
		   }
		   break;
	   }

   }
```
   由上可知，child的SpecMode 为at_most的时候，SpecSize均为parentSize
### 6、View的onMeasure流程伪代码
```java
   protected void onMeasure(int widthMeasureSpec,int heightMeasureSpec){
      setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(),widthMeasureSpec),getDefaultSize(getSuggestedMinimumHeight(),heightMeasureSpec));
   }
   //获取的背景的宽度与 mMinWidth属性的最大值
   protected int getSuggestedMinimumWidth(){
      return (mBackground==null) ? mMinWidth : max(mBackground.getMinimumWidth(),mMinWidth);
   }

   //根据父布局的计算逻辑得到MeasureSpec,重新计算下specSize,主要修改UNSPECIFIED状态下的specSize=0的数据
   protected stataic int getDefaultSize(int suggestedMinimunSize(取最小宽度),measureSpec){
       int specMode=MeasureSpce.getMode(measureSpec);
	   int specSize=MeasureSpce.getSize(measureSpec);
	   int defaultSize=suggestedMinimunSize;
	   switch(specMode){
	      case MeasureSpec.UNSPECIFIED:
		    defaultSize=suggestedMinimunSize;
		    break;
		  case MeasureSpec.AT_MOST:
		  case MeasureSpec.EXACTLY:
		    defaultSize=specSize;
			break
	   }
   }
```
代码执行流程
```
getChildMeasureSpec()----------->得到childView的MeasureSpec------->
childView根据上面得到的MeasureSpec执行onMeasure-------->执行setMeasuredDimension()重新设置specSize
-------->发现结果atMost的时候，specSize都为parentSize----->不符合实际意义（wrapconent 跟 match_parent 一个意义）------自定义view需要对AT_MOST时setMeasureDimension 参数重新赋值
```

### 7、ViewGroup的measure过程

* 1、viewGroup 有一个measureChildren方法，里面遍历执行了measureChild(),这里面没有把
   margin参数计算在内，注意与measureChildWithMargin的区分

* 2、measureChildren方法，自身并没有在其他方法中被调用，只在少数几个实现类中使用到，
   如：AbsoluteLayout，NumPadKey，ProfilePictureView中使用到，这个方法大概是保留给我们使用的，
   比如LinearLayout 完全没有用到此方法

* 3、每个ViewGrooup的实现类，onMeasure的逻辑都是不一样的，所以ViewGroup 并没有重写onMeasure方法，
   因为没有意义


### 8、关于如何在页面一打开的时候就得到  width，height
#### 1、onWindowFocusChanged(boolean  hasfocus)； 此方法会被调用多次，当View初始化完毕之后，Activity 获取焦点/失去焦点 ，onResume(),onPause()都会执行;

#### 2、
```java
View.post（new  Runnable(){
         int width=view.getMeasureWidth();
         int height=view.getMeasureHeight();
	  });  //投递到消息队列中，View 初始化完成时候，Looper会处理此消息
```
#### 3、
```java
ViewTreeObserver的onGlobalLayoutListener
      ViewTreeObserve viewTreeObserve=view.getViewTreeObserve();
	  viewTreeObserve.setGlobalLayoutListener(new GlobalLayoutListener(){
	       publica  void  onGlobalLayout(){
		      view.getViewTreeObserve.removeGlobalLayoutListener(this);
			  int width=view.getMeasureWidth;
		   }
	  })//此方法也会执行多次，所以获取到之后就移除监听
```
#### 4、view.measure(widthMeasureSpec,heightMeasureSpec);通过手动调用measure方法对出结果，前提是widthMeasureSpec,heightMeasureSpec可以构造成功

##### 4.1、match_parent模式
* 不行，因为无法得到当前View 在ViewGroupp的可用空间，可用空间=ViewGroup.size - otherChildren.totalSize

##### 4.2、warp_content模式
* `int withMeasureSpec=MeasureSpec.makeMeasureSpec((1<<30)-1,MeasureSpce.AT_MOST);`1左移30位，1后面30个0，减1变成30个1，最大值

###### 为什么这种方式可以？

* 因为ViewGroup都会重写onMeasure,At_most模式都是测量实际大小，然后跟父布局的SpecSize比较，这里用最大值肯定会得到view的实际大小

### 9、layout伪代码过程

    ViewRootImpl执行performTraversals------>performTraversals内部执行performLayout
    -------->performLayout 执行内部的DecoreView的layout方法（也就是view的layout方法）
	  ------>if(positionChanged || isRequestLayout){onLayout()}------->
    View 及ViewGroup的都未实现onLayout

```java
	LinearLayout的onLayout
	publica void  onLayout(int left,int top,int right,int bottom){
	     int childLeft=left-paddingLeft;
		 int width=right-left;
		 int childSpace=width-paddingleft-paddingright;
		 int childTop=0;
		 for(int i=0,i<childCount(),i++){
		    if(isHasDivider){
			   childTop+=diviDerHeight;
			}
			childTop+=marginTop;
			child.setFrame();
			childTop=childHeight+childTop
		 }
	}
```
### 10、getMeasureWidth 跟 getWidth,什么情况下会不相等
  View的onMeasure重会调用setMeasuredDimensionRaw对measureWidth进行赋值，他用的就是measureSpecSize 中的值
	View的getWidth计算方式是根据Right-left，与onLayout中的参数值相关

### 11、常用的回调方法
#### onFinishInflate():从xml加载组件后回调
#### onSizeChanged():组件大小改变时回调


### 12、onDraw
#### 1、属性定义有几种类型？
```xml
<declare-styleable name="TestAttr">
        <!--颜色-->
        <attr name="attrColor" format="color"></attr>
        <!--字符串-->
        <attr name="attrStr" format="string"></attr>
        <!--布尔-->
        <attr name="attrBoolean" format="boolean"></attr>
        <!--浮点值-->
        <attr name="attrFloat" format="float"></attr>
        <!--枚举-->
        <attr name="attrEnum">
            <enum name="enumName1" value="1"/>
            <enum name="enumName2" value="2"/>
        </attr>
        <!--资源引用-->
        <attr name="attrRef" format="reference"></attr>
        <!--尺寸-->
        <attr name="attrDim" format="dimension"></attr>
        <!--整型-->
        <attr name="attrInt" format="integer"></attr>
        <!--百分比-->
        <attr name="attrFraction" format="fraction"></attr>
        <!--位运算类型，自带可以混合使用-->
        <attr name="attrFlag">
            <flag name="top" value="0x30"/>
            <flag name="bottom" value="0x40"/>
            <flag name="left" value="0x50"/>
            <flag name="right" value="0x60"/>
        </attr>
        <!--混合类型-->
        <attr name="attrMix" format="color|reference"></attr>
    </declare-styleable>
```
#### 2、属性值的获取
##### 2.1、命名空间
* 我们在布局文件中使用属性的时候（`android:layout_width="match_parent"`）发现前面都带有一个android：，这个android就是上面引入的命名空间`xmlns:android="http://schemas.android.com/apk/res/android”`，表示到android系统中查找该属性来源。只有引入了命名空间，XML文件才知道下面使用的属性应该去哪里找（哪里定义的，不能凭空出现，要有根据）。 如果我们自定义属性，这个属性应该去我们的应用程序包中找，所以要引入我们应用包的命名空间`xmlns:app="http://schemas.android.com/apk/res-auto”`，res-auto表示自动查找（适用于android studio）

##### 2.2、获取方法
```java
    TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.MyTextView);
    String text = ta.getString(R.styleable.MyTextView_android_text);
    int mTextColor = ta.getColor(R.styleable.MyTextView_mTextColor, Color.BLACK);
    int mTextSize = ta.getDimensionPixelSize(R.styleable.MyTextView_mTextSize, 100);
    ta.recycle();  //注意回收
```
