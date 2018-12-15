# 移动端关于屏幕适配的一些总结

## 一、基本概念

### 1、位图像素

一个位图像素是栅格图像(如：png, jpg, gif等)最小的数据单元。每一个位图像素都包含着一些自身的显示信息(如：显示位置，颜色值，透明度等)

### 2、逻辑分辨率（point）

pt: point ，点，是一个标准的长度单位，1pt＝1/72英寸.

### 3、设备分辨率（pixel）

px: pixel，像素，屏幕上显示的最小单位。

![img](https://github.com/Yggdrasill-7C9/screen-adaptation/raw/master/src/2018%E6%AC%BEiPhone%E5%B1%8F%E5%B9%95%E9%80%82%E9%85%8D.png)





### 4、 物理像素(physical pixel)

一个物理像素是显示器(手机屏幕)上最小的物理显示单元

### 5、设备独立像素(density-independent pixel)

设备独立像素(也叫密度无关像素)，可以认为是计算机坐标系统中得一个点，这个点代表一个可以由程序使用的虚拟像素(比如: CSS像素)

css像素是由视口缩放比和设计稿的像素决定的。

### 6、设备像素比(DevicePixelRadio，简称dpr)

设备像素比 = 物理像素 / 设备独立像素（在某一方向上，x方向或者y方向）

![img](https://github.com/Yggdrasill-7C9/screen-adaptation/raw/master/src/iPhone%E5%B1%8F%E5%B9%95%E5%B0%BA%E5%AF%B8%E8%A7%84%E6%A0%BC.png)

### 7、initial-scale， minimum-scale，maximum-scale

缩放比：scale = 1/dpr

iOS下，对于2和3的屏，正常按dpr实际值处理，其余的用1倍方案。

1. 在 dpr = 1 的设备上设置：

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no" />
```

如 Galaxy S5 中，设置前获取屏宽 `window.innerWidth` 为 `360`，设置后还是 `360`。

1. 在 dpr = 2 的设备上：

```html
<meta name="viewport" content="width=device-width, initial-scale=0.5, minimum-scale=0.5, maximum-scale=0.5, user-scalable=no" />
```

如 iPhone6 中，设置前获取屏宽 `window.innerWidth` 为 `375`，设置后是 `750`（与物理像素匹配）。

1. 在 dpr = 3 的设备上：

```html
<meta name="viewport" content="width=device-width, initial-scale=0.3333333333333333, maximum-scale=0.3333333333333333, minimum-scale=0.3333333333333333, user-scalable=no" />
```

### 8、缩放因子**（****scale factor** between logic point and device pixel）

$ scale = point / pixel $

高倍图：@1x，@2x，@3x

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

### 9、屏幕尺寸（inch）

​    我们通常所说的iPhone5屏幕尺寸为4英寸、iPhone6屏幕尺寸为4.7英寸，指的是显示屏对角线的长度（diagonal）。

### 10、像素密度（PPI）

PPI（Pixel Per Inch by diagonal）：表示沿着对角线，每英寸所拥有的像素（Pixel）数目。表示了清晰度。

PPI数值越高，代表显示屏能够以越高的密度显示图像，即通常所说的分辨率越高、颗粒感越弱。 

### 12、 完美视口

```html
<meta name="viewport" content="initial-scale=1,width=device-width,user-scalable=0,maximum-scale=1" />
```



## 二、移动web适配

### 1、网易彩票的设计方案

[网易彩票首页](http://caipiao.163.com/t/)

- 采用scale = 1.0写死viewport
- 采用媒体查询来确定html根元素的font-size值，即rem值
- rem + 百分比布局

媒体查询的css代码如下：

```css
//网易彩票的响应式布局是采用媒体查询来改变rem值实现的
//媒体查询css
#media-query{
    @media screen and (min-width: 240px) {
        html,body,button,input,select,textarea {
            font-size:9px!important;
        }
    }

    @media screen and (min-width: 320px) {
        html,body,button,input,select,textarea {
            font-size:12px!important;
        }
    }

    @media screen and (min-width: 374px) {
        html,body,button,input,select,textarea {
            font-size:14px!important;
        }
    }

    @media screen and (min-width: 400px) {
        html,body,button,input,select,textarea {
            font-size:15px!important;
        }
    }
    // 省略
}
```

### 2、天猫的设计方案

[天猫首页](https://www.tmall.com/#/main)

- 采用scale = 1.0写死viewport
- 不采用rem，body的font-size=14px写死
- px + flexbox布局

### 3、淘宝的设计方案

[淘宝首页](https://m.taobao.com/)

- 通过js处理获取手机dpr参数，然后动态生成viewpoint
- 获取手机物理像素宽度，分成10份，每一份的宽度即是rem的尺寸。
- 根据设计稿的尺寸（px）分三种情况进行处理，采用px + rem布局

相关的脚本如下：

```javascript
$(document).ready(function(){
    var dpr, rem, scale;
    var docEl = document.documentElement;
    var fontEl = document.createElement('style');
    var metaEl = document.querySelector('meta[name="viewport"]');
    var view1 = document.querySelector('#view-1');
    if(window.screen.width < 540){
        dpr = window.devicePixelRatio || 1;
        scale = 1 / dpr;
        rem = docEl.clientWidth * dpr / 10;
    }else{
        dpr = 1;
        scale =1;
        rem = 54;
    }
//貌似最新的淘宝网站又去掉了，只是限制了主体内容的宽度

    // 设置viewport，进行缩放，达到高清效果
    metaEl.setAttribute('content', 'width=' + dpr * docEl.clientWidth + ',initial-scale=' + scale + ',maximum-scale=' + scale + ', minimum-scale=' + scale + ',user-scalable=no');
    
    // 设置整体div的宽高
    view1.setAttribute('style', 'width:'+ docEl.clientWidth+'px; height:'+ docEl.clientHeight+'px');

    // 设置data-dpr属性，留作的css hack之用
    docEl.setAttribute('data-dpr', dpr);

    // 动态写入样式
    docEl.firstElementChild.appendChild(fontEl);
    fontEl.innerHTML = 'html{font-size:' + rem + 'px!important;}';
    $('body').attr('style', 'font-size:' + dpr * 12 +'px');

    // 给js调用的，某一dpr下rem和px之间的转换函数
    window.rem2px = function(v) {
        v = parseFloat(v);
        return v * rem;
    };
    window.px2rem = function(v) {
        v = parseFloat(v);
        return v / rem;
    };

    window.dpr = dpr;
    window.rem = rem;
})
```

### 4、总结：

从以上的分析我们不难看出：

- 网易彩票的方案上手快，开发效率高，兼容性好，但是不够灵活和精细；
- 天猫的设计思路比较简单，flexbox非常灵活，但是flexbox的兼容性方面需要好好处理，不够精细；
- 淘宝的方案几乎解决了移动端遇到的所有问题，堪称完美的解决方案，但是开发效率低、成本比较高。

## 三、Android && iOS原生适配

iOS : masnory 第三方框架自动布局，[iOS常用布局方案](https://github.com/vsouza/awesome-ios#layout)

Android：[*AndroidSwipeLayout*](https://github.com/daimajia/AndroidSwipeLayout)，[flexbox-*layout*](https://github.com/google/flexbox-layout) ，[awesome-Android-layout](https://github.com/wasabeef/awesome-android-ui/blob/master/pages/Layout.md)



## 四、参考链接

[IOS开发：尺寸和适配](https://blog.csdn.net/u012160319/article/details/80591958)

[2018 iPhone XS、Max、XR 适配攻略](https://www.ui.cn/detail/401348.html)

[移动端高清、多屏适配方案](http://www.html-js.com/article/Mobile-terminal-H5-mobile-terminal-HD-multi-screen-adaptation-scheme%203041)

[移动端布局方案探究](https://github.com/tywei90/mobile_study)

[iOS像素和点的转换](http://www.cocoachina.com/ios/20170728/20045.html)

[iOS 屏幕尺寸、分辨率、适配、UI规范](https://blog.csdn.net/doubleface999/article/details/77193823)

[设备像素比devicePixelRatio简单介绍](https://www.zhangxinxu.com/wordpress/2012/08/window-devicepixelratio/)

[iPhone屏幕分辨率和适配规则](https://www.jianshu.com/p/1b24ca5e8c12)

[Android屏幕适配全攻略](https://www.jianshu.com/p/759375113de9)

[Android 屏幕适配：最全面的解决方案](https://www.jianshu.com/p/ec5a1a30694b)

[移动端高清、多屏适配方案](http://www.html-js.com/article/Mobile-terminal-H5-mobile-terminal-HD-multi-screen-adaptation-scheme%203041)

[wap手机端页面根据dpr和宽度计算出font-size对应数值 ](https://my.oschina.net/kenblog/blog/373341?fromerr=OQeOo5fR)

[移动前端开发之viewport的深入理解](https://www.cnblogs.com/2050/p/3877280.html)

[Retina屏的移动设备如何实现真正1px的线？](http://jinlong.github.io/2015/05/24/css-retina-hairlines/)

[前端：『REM』手机屏幕高清适配方案](https://github.com/hbxeagle/rem/blob/master/HD_ADAPTER.md)

[了解真实的『REM』手机屏幕适配](https://github.com/hbxeagle/rem)

[iOS 屏幕适配浅谈](https://juejin.im/entry/5a01bec2f265da43215375d5)