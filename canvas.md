# canvas相关
## 1、画笔相关属性、方法

### 1.1、 `mPaint.setAntiAlias(boolean aa);`  设置锯齿效果
### 1.2、`mPaint.setColor(@ColorInt  int color);` 设置画笔颜色,只接收ColorRes
### 1.3、`mPaint.setARGB(int a,int r,int g,int b);`设置分别的颜色
### 1.4、`mPaint.setAlpha(int a);`设置透明度  0-255
### 1.5、`mPaint.setTextSize(float textSize);`设置字体大小，按像素算
### 1.6、`mPaint.setStyle(Style style);` 设置空心还是实心 `Paint.Style.STROKE`空心  `Paint.Style.FILL`实心 `Paint.Style.FILL_AND_STROKE` 实心加边框
### 1.7、`mPaint.setStrokeWidth(float width)` 设置边框宽度
### 1.8、`mPaint.setTextAlign(Align align)` 这只文本相对于矩形框的位置

## 2、画布相关属性及方法
### 2.1、canvas.drawPoint(float x,float y,Paint paint)   画点
### 2.2、canvas.drawLine(float startX,float startY,float stopX,float stopY,Paint paint);  画线
### 2.3、canvas.drawLines(float[] pts,int offset(可省略),Paint paint); 画多条直线，pts中每四个定义一条直线，offset 为跳过的个数
### 2.4、画矩形,RectF 与Rect 不同在于  参数类型不同一个是float,一个是int   RectF 更加精准
```java
    canvas.drawRect(RectF rect,Paint paint)
    canvas.drawRect(Rect rect,Paint paint)
    canvas.drawRect(float l,float t,float r,float b,Paint paint)
```
### 2.5、画圆角矩形
```java
    canvas.drawRoundRect(RectF,float rx,float ry,Paint paint)
    canvas.drawRountRect(float left,float top,float right,float bottom,float rx,float ry,Paint paint);
```
### 2.6、画圆
`canvas.drawCircle(float cx,float cy,float radius,Paint paint);`

### 2.7、画圆弧、扇形
* 矩形四点确定一个圆，开始的角度，圆弧划过的角度，是否需要跟圆心连接

```java
  canvas.drawArc(RectF oval,float startAngle,float sweepAngle,boolean useCenter,Paint paint);
  canvas.drawArc(float left,float top,float right,float bottom,  oval,float startAngle,float sweepAngle,boolean useCenter,Paint paint);
```
### 2.8、画椭圆
```java
  canvas.drawOval(RectF rect,Paint paint)
  canvas.drawOval(float left,float top,float right,float bottom,Paint paint);
```

### 2.9、绘制文本
```java
注意 (x,y)为文本左侧baseline线的位置，基本快到左下角但是没到
canvas.drawText(String text ,float x,float y,Paint paint);
canvas.drawText(String text ,int start,int end,float x,float y,Paint paint);
canvas.drawText(CharSequence text ,int start,int end,float x,float y,Paint paint);
canvas.drawText(Char[] text ,int index,int count,float x,float y,Paint paint);

ascent线: 系统推荐的顶部
descent线:系统推荐的底部
top线：物理顶部
bottom线：物理底部
baseline线: 基准线

FontMetrics包含五个变量 ascent、descent、top、bottom、leading,  他们与上面的线不是一回事
FontMetrics.ascent=ascent线.y-baseline线.y  ，其余类似
FontMetrics.leading= 前一行的descent-下一行的ascent   行间距

如何得到FontMetrics?
他只跟文字的大小有关，所以画笔字体大小设定之后，就可以得到FontMetrics
FontMetrics fontMetrics = paint.getFontMetrics();
float height= fontMetrics.bottom-fontMetrcs.top;
float width = paint.measureText(String text);
由top 点得到baselineY的值即
float baselineY=top -  fontMetrics.top;

由center点得到baseLineY的值即：
baselineY = center + (FontMetrics.bottom - FontMetrics.top)/2 - FontMetrics.bottom;
```
### 2.10、在指定位置上绘制文本（必须执行有所有文字的位置）
```java
canvas.drawPosText(String text,float[] pos,Paint paint);
canvas.drawPosText(Char[] text,int index,int count,float[] pos.Paint paint);
```
### 2.11、绘制路径
```java
Path path = new Path();
path.moveTo(float x,float y)    //移动到一个点上，不会画线
path.lineTo(float x,float y); //从当前点画线到这一点上
path.arcTo(RectF oval,float startAngle,float sweepAngle,boolean forceMoveTo)  //画弧线，弧线所在的椭圆矩形，起始角度，画的角度，是否强制画笔跳过去
path.addArc(RectF oval,float startAngle,float sweepAngle) //相当于上面forceMoveTo=true
path.close(); //封闭字图形可以用lineTo回到起点替换
canvas.drawPath(path,paint);
```

## 3、canvas相关重点属性
### 1、canvas.translate(float dx,float dy);
* 画布平移，其实就是坐标系原点的平移
### 2、canvas.rotate(float degree)
* 画布旋转角度，其实就是坐标系旋转一个角度
### 3、canvas.save();
* 保存当前画布坐标系的状态，包括他的原点平移位置，旋转角度
### 4、canvas.restore()
* 必须与save()一起用，先合并画布内容，然后拿出save()之前的坐标系状态，然后继续绘制

## 4、ColorMatrix操作bitmap图片像素
### 1、ColorMatrix 其实就是一个4 * 5的矩阵，程序中长度为20一维数组进行表示

* 每行分别依次控制着 红、绿、蓝 、透明度
*  每行的最后列表控制着  该行所代表的颜色 占有的偏移量

### 2、如何修改一个图片的色调，饱和度，以及亮度，有对应的Api
```java
                //色调
                ColorMatrix hueMatrix=new ColorMatrix();
                huMatrix.setRotate(int axis, float degrees);
                axis 分别可以为0,1,2  红，绿，蓝
                degrees, 为角度0-360

                //饱和度
                ColorMatrix saturation=new ColorMatrix();
                saturation.setSaturation(float saturation);
                saturation 0-1之间取值

                //亮度
                ColorMatrix lumMatrix=new ColorMatrix();
                lumMatrix.setScale(float rScale,float gScale,float bScale,float aScale);
                参数都是0-1，分别代表 红，绿，蓝，透明度

                //连接上面三个数组组成一个完整的ColorMatrix
               ColorMatrix colorMatrix=new ColorMatrix();
               colorMatrix.postConcat(hueMatrix);
               colorMatrix.postConcat(saturation);
               colorMatrix.postConcat(lumMatrix);
```
### 3、直接根据矩阵数据得到一个ColorMatrix
```java
float[] colorMatrix=new float[20];
原始矩阵，及不改变图片颜色的矩阵
                  1     0      0     0     0
                  0     1      0     0     0
                  0     0      1     0     0
                  0     0      0     1     0
colorMatrix=[1    , 0    ,  0  ,   0  ,   0,
             0  ,   1  ,    0  ,   0   ,  0,
             0    , 0    ,  1  ,   0    , 0,
             0   ,  0   ,   0   ,  1   ,  0];

1. 灰度效果的矩阵
                 0.33   0.59   0.11   0   0
                 0.33   0.59   0.11   0   0
                 0.33   0.59   0.11   0   0
                 0         0      0   1   0

2. 图像反转（颜色互换，黑变白 白变黑）
                -1     0    0    1   1
                 0    -1    0    1   1
                 0     0   -1    1   1
                 0     0    0    1   0

3. 怀旧效果
                 0.393   0.769    0.189   0   0
                 0.349   0.686    0.168   0   0
                 0.272   0.534    0.131   0   0
                 0           0        0   1   0

4. 去色效果
                 1.5    1.5    1.5    0   -1
                 1.5    1.5    1.5    0   -1
                 1.5    1.5    1.5    0   -1
                 0        0      0    1    0

5. 高饱和度
                 1.438     -0.122    -0.016   0   -0.03
                -0.062      1.378     0.016   0    0.05
                 0.062      0.122     1.483   0    0.02
                 0             0          0   1       0
```
## 4、修改图片的颜色
* 因为bitmap是不支持直接修改的，所以我们需要把bitmap复制一份出来填充颜色
```java
//拿到原图bitmap
BitMap bitmap=BitmapFactory.decodeResource(getResources(),R.drawable.XXX)
//创建一个空白的copyBitmap
Bitmap copy=Bitmap.createBitmap(bitmap.getWidth(),bitmap.getHeight(),Config.ARGB_8888);
//用空白的bitmap 创建一个画布
Canvas canvas=new Canvas(copy);
//拿到一个配置好的ColorMatrix,用2,3 都可以
ColorMatrix colorMatrix=new ColorMatrix(float[] colorMatrix);
//用ColorMatrix 对画笔进行配置
Paint paint=new Paint();
paint.setColorFilter(new ColorMatrixColorFilter(colorMatrix));
//绘制copy
canvas.drawBitmap(bitmap,0,0,paint);
imageView.setBitmap(copy);
```
## 5、更加精确的处理bitmap像素
```java
//获取原图bitmap
BitMap bitmap=BitmapFactory.decodeResource(getResources(),R.drawable.XXX)
//获取原图像素点数组
int[] oldPx=new int[bitmap.getWidth(),bitmap.getHeight()];
bitmap.getPixels(oldPx,0,width,0,0,width,height);
//根据老像素点 构造新像素点
int[] newPx=new int[bitmap.getWidth(),bitmap.getHeight()];
int color,r,g,b,a;
for(i=0;i<oldPx.length;i++){
    color=oldPx[i];
    r=Color.red(color);
    g=Color.green(color);
    b=Color.blue(color);
    a=Color.alpha(color);

    r1=根据原像素构造
    g1=根据原像素构造
    b1=根据原像素构造
    a1=根据原像素构造

    newPx[i]=Color.argb(a,r,g,b);
}
//根据新像素点填充新图
Bitmap newBitmap=Bitmap.createBitmap(bitmap.getWidth(),bitmap.getHeight(),Config.ARGB_8888);
newBitmap.setPixels(newPx,0,bitmap.getWidth(),0,0,bitmap.getWidth().bitmap.getHeight());
```
## 6、常用像素构造公式
```java
1. 底片效果：
                         r=255-r;
                         g=255-g;
                         b=255-b;
2. 老照片效果：
                          r= (int) (0.393*r+0.769*g+0.189*b);
                          g= (int) (0.349*r+0.686*g+0.168*b);
                          b= (int) (0.272*r+0.534*g+0.131*b);
3. 浮雕效果：
                         preColor = oldPx[i-1];
                         a = Color.alpha(preColor);
                         r = Color.red(preColor);
                         g = Color.green(preColor);
                         b = Color.blue(preColor);

                         color = oldPx[i];
                         r1 = Color.red(color);
                         g1 = Color.green(color);
                         b1 = Color.blue(color);

                          r = r1 - r + 127;
                          g = g1 - g + 127;
                          b = b1 - b + 127;
```

## 7、图形的特效变换
* 初始矩阵为3*3阶矩阵(就相当于参数),new float[9]，X0,Y0相当于原始图上的像素点
```java
    1      0       0                    X0                     X
    0      1       0        x           Y0           =         Y
    0      0       1                      1                    1
    Matrix matrix=new Matrix();
```
### 7.1、平移变换的矩阵
```java
                    1      0      △x
                    0      1      △y
                    0      0        1
                    matrix.setTranslate(float dx,float dy);
```
### 7.2、绕原点的旋转矩阵，γ为旋转角度
```java
                    cosγ     -sinγ    0
                    sinγ     cosγ     0
                    0          0      1
                    此为绕原点旋转的矩阵，如果不饶原点旋转，
                    1、先做平移变换，把原点平移到对应的点上（画布平移），
                    2、旋转完成，
                    3、最后还原原点
                    matrix.setRotate(float degree,float px,float py);   px py为旋转原点
                    matrix.setRotate(float degree);
```
### 7.3、缩放变换
```java
                    K1     0      0
                    0       K2    0
                    0       0      1
                    matrix.setScale(float sx,float sy,float px,float py);  px，py 为不变的点，以此为支点
                    matrix.setScale(float sx,float sy);
                    可以看出横向，纵向缩放比例可以不同
```
### 7.4、错切变换
```java
                    1      K1      0
                    K2     1       0
                    0      0        1
                     matrix.setSkew(float kx,float ky);
                     matrix.setSkew(float kx,float ky,float px,float py) px,py为支点
```
## 8、图形形状变化方法二，像素块方式处理，把一张图分成多个像素块机型坐标变换
```java
                     drawBitmapMesh(Bitmap  bitmap,int meshWidth,int meshHeight,float[] verts,int vertsOffset,int[] colors,int colorOffset,Paint paint);
                     bitmap: 原图
                     meshWidth:横向需要的网格数，不是线数
                     meshHeight:纵向需要的网格数
                     verts:网格交叉 点坐标数组（包括边界交点），至少（meshWidth + 1）*（meshHeight + 1）* 2 + vertOffset
                     vertsOffset:verts数组中开始跳过的坐标对的数目
                     colors:可能为空。 指定每个顶点的颜色，该颜色在单元格中进行插值，其值乘以相应的位图颜色。
                                   如果不为null，则数组中必须至少包含（meshWidth + 1）*（meshHeight + 1）+ colorOffset值。
                      colorOffset: 跳过的颜色个数
                      paint:画笔，可以为空

                      主要需要计算出 verts的值

                      原始坐标得到方式：
                      float  bitmapWidth=bitmap.getWidth();
                      float bitmapHeight=bitmap.getHeight();
                      int  index=0;
                      float[] orig=new float[(meshWidth+1)*(meshHeight+1)];
                      float[] verts=new float[(meshWidth+1)*(meshHeight+1)];
                      for(int i=0;i<=meshHeight;i++){
                             float fy=bitmapHeight*i/meshHeight;
                            for(int j=0;j<=meshWidth;j++){
                                 float fx=bitmapWidth*j/meshWidth;
                                 orig[index*2+0]=verts[index*2+0]=fx;
                                 verts[index*2+1]=verts[index*2+1]=fy;
                                 index+=1;
                            }
                      }
```
## 9、画笔特效处理——PorterDuffXfermode
                      PorterDuffXfermode:处理两个图层重叠时候的显示效果
                      dst:先画   src:后画
                      srcIn:只保留重叠部分，颜色用src
                      dstIn:只保留重叠部分，颜色用dst
                      paint.setXfermode(Xfermode xfermode);

                      PorterDuffXfermode继承自Xfermode

                      paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));

## 10、画笔特效----shader
* 一般用法都是对画笔设置属性后，绘制各种图形，填充图形中的内容效果
### 1、BitmapShader   位图shader
`BitmapShader(Bitmap bitmap,TileMode tileX,TileMode tileY)`
### 2、LinearGradient  线性渐变
`LinearGradient(float x0,float y0,float x1,float y1,int colors[],float positions[],TileMode  tileMode);`
* 取值：positions  0f--1f

### 3、RadialGradient   环形光束渐变（可用来实现水波纹效果）
`RadialGradient(float centerX,float centerY,float radius,int colors[],float stops[],TileMode tileMode);  `
* stops  0f--1f

### 4、SweepGradient   扫描/梯度渲染（可用来实现雷达扫描效果）
`SweepGradient(float cx,float cy,int colors[],float positions[])`
* positions  0f--1f

### 5、ComposeShader  混合shader
`ComposeShader(Shader shader1,Shader shader2,Xfermode xfermode)`

                       填充模式
                       1、CLAMP   拉伸图片的最后一个像素，不断重复
                       2、REPEAT  横向 纵向  不断重复
                       3、MIRROR   镜像重复

## 11、画笔特效------PathEffect 路径绘制效果
### 1、CornerPathEffect   使路径绘制转角处变得圆滑一点
### 2、DiscretePathEffect  在路径上随机偏离一些杂点
### 3、DashPathEffect  虚线
### 4、PathDashPathEffect  路径上的点绘制形状
### 5、ComposePathEffect  组合效果
