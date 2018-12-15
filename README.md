# 移动端关于屏幕适配的一些总结

## 一、基本概念

### 1. 物理像素(physical pixel)

一个物理像素是显示器(手机屏幕)上最小的物理显示单元

### 2. 设备独立像素(density-independent pixel)

设备独立像素(也叫密度无关像素)，可以认为是计算机坐标系统中得一个点，这个点代表一个可以由程序使用的虚拟像素(比如: CSS像素)

### 3. 位图像素

一个位图像素是栅格图像(如：png, jpg, gif等)最小的数据单元。每一个位图像素都包含着一些自身的显示信息(如：显示位置，颜色值，透明度等)

### 4. 设备像素比(DevicePixelRadio，简称dpr)

设备像素比 = 物理像素 / 设备独立像素（在某一方向上，x方向或者y方向）

### 5. scale

缩放比：scale = 1/dpr

### 6、逻辑分辨率（point）

pt: point ，点，是一个标准的长度单位，1pt＝1/72英寸.

### 7、设备分辨率(pixel)

px: pixel，像素，屏幕上显示的最小单位。

### 8、屏幕尺寸

​    我们通常所说的iPhone5屏幕尺寸为4英寸、iPhone6屏幕尺寸为4.7英寸，指的是显示屏对角线的长度（diagonal）。

![img](https://pocket-image-cache.com/direct?url=http%3A%2F%2Fcc.cocimg.com%2Fapi%2Fuploads%2F20170728%2F1501209520583815.jpg&resize=w1408)

### 9、像素密度（PPI）

PPI（Pixel Per Inch by diagonal）：表示沿着对角线，每英寸所拥有的像素（Pixel）数目。表示了清晰度。

PPI数值越高，代表显示屏能够以越高的密度显示图像，即通常所说的分辨率越高、颗粒感越弱。

### 10、缩放因子**（****scale factor** between logic point and device pixel）

> #### （1）Scale起源
>
> ​    早期的iPhone3GS的屏幕分辨率是320*480（PPI=163），[iOS](http://lib.csdn.net/base/1)绘制图形（CGPoint/CGSize/CGRect）均以point为单位（measured in points）：
>
> ​    **1 point = 1 pixel**（Point Per Inch=Pixel Per Inch=PPI）
>
> ​    后来在iPhone4中，同样大小（3.5 inch）的屏幕采用了[Retina显示技术](http://www.zhihu.com/question/20220915)，横、纵向方向像素密度都被放大到2倍，像素分辨率提高到(320x2)x(480x2)= 960x640（PPI=326）， 显像分辨率提升至iPhone3GS的4倍（1个Point被渲染成1个2x2的像素矩阵）。
>
> ​    但是对于开发者来说，[iOS](http://lib.csdn.net/base/ios)绘制图形的API依然沿袭point(pt，注意区分印刷行业的“磅”)为单位。在同样的逻辑坐标系下（320x480）：
>
> ​    **1 point = scale\*pixel**（在iPhone4~6中，缩放因子scale=2；在iPhone6+中，缩放因子scale=3）。
>
> ​    可以理解为：
>
> ​    **scale=***绝对长度比\**（*****point/pixel）=***单位长度内的数量比\**（*****pixel/point）**
>
>
>
> ####     （2）UIScreen.scale
>
> ​    **UIScreen.h**中定义了该属性：
>
> ​    // The natural scale factor associated with the screen.(read-only)
>
> ​    @property(nonatomic,readonly) CGFloat **scale**  NS_AVAILABLE_IOS(4_0);
>
> ​    \--------------------------------------------------------------------------------
>
> ​    This value reflects the scale factor needed to convert from the default **logical coordinate space** into the **device coordinate space** of this screen.
>
> ​    The default logical coordinate space is measured using **points**. For standard-resolution displays, the scale factor is 1.0 and one point equals one pixel. For Retina displays, the scale factor is 2.0 and one point is represented by four pixels.
>
> ​    \--------------------------------------------------------------------------------
>
> ​    为了自动适应分辨率，系统会根据设备实际分辨率，自动给UIScreen.scale赋值，该属性对开发者只读。
>
> ####     （3）UIScreen.nativeScale
>
> ​    **iOS8新增了nativeScale属性：**
>
> ​    // Native scale factor of the physical screen
>
> ​    @property(nonatomic,readonly) CGFloat **nativeScale** NS_AVAILABLE_IOS(8_0);
>
> ​    以下是iPhone6+下的输出，初步看来[nativeScale与scale没有太大区别](http://stackoverflow.com/questions/25871858/what-is-the-difference-between-nativescale-and-scale-on-uiscreen-in-ios8)：
>
> ​    \--------------------------------------------------------------------------------
>
> ​        (lldb)p (CGFloat)[[UIScreen mainScreen] scale]
> ​        (CGFloat) $1 = 3
> ​        (lldb) p(CGFloat)[[UIScreen mainScreen] nativeScale]
> ​        (CGFloat) $2 = 3
>
> ​    \--------------------------------------------------------------------------------
>
> #####    **（4）机型判别**
>
> ​    在同样的逻辑分辨率下，可以通过scale参数识别是iPhone3GS还是iPhone4(s)。以下基于nativeScale参数，定义了探测机型是否为iPhone6+的宏：
>
> ​    \--------------------------------------------------------------------------------
>
> ​    // not UIUserInterfaceIdiomPad
> ​    \#define **IS_IPHONE** (UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPhone)
> ​    // detect iPhone6 Plus based on its native scale
> ​    \#define **IS_IPHONE_6PLUS** (IS_IPHONE && [[UIScreenmainScreen] nativeScale] == 3.0f)
>
> ​    \--------------------------------------------------------------------------------
>
> ​    那么，同样的分辨率和scale，如何区分机型iPhone4与4s、iPhone5与5s呢？通过[[[UIDevice currentDevice\] model]](http://blog.csdn.net/like7xiaoben/article/details/8184064)只能判别iPhone、iPad、iPod大类，要判断iPhone具体机型型号，则需要通过[sysctlbyname("hw.machine")](http://www.2cto.com/kf/201412/359301.html)获取详细的设备参数信息予以甄别。

11、高倍图：@1x，@2x，@3x

### 6. 完美视口

```html
<meta name="viewport" content="initial-scale=1,width=device-width,user-scalable=0,maximum-scale=1" />
```



