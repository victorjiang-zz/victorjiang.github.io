---
layout: post
title: "在iOS中使用Icon Font"
date: 2014-06-17 10:16:03 +0800
comments: true
categories: 技术
---


<!-- photo 1 -->
![photo 1](http://victorjiang.github.io/images/2014/icon_font_1.jpg)

最近公司在做一个OA项目,项目中有一个天气模块,这边天气的数据接口是由[国家气象局](http://cj.weather.com.cn/)提供,接口同时提供了天气图标,不过需要通过图标的网络地址去加载,而且大小风格固定,不一定适合App的样式.是就将用该图标还是让美工设计一套?

<!--more-->

于是乎打算去网上找找有没有类似的天气类的应用,在GitHub上找到一个[CZWeatherKit](https://github.com/CZWeatherKit/CZWeatherKit)的开源应用,运行了下,这界面,这图标正是我想要的iOS7风格,但在工程里并没有找到任何天气的图标,遂研究一下他的代码.

最终在绘制界面的代码中找到了答案:

```
self.weatherView.conditionIconLabel.text = [NSString stringWithFormat:@"%c", condition.climaconCharacter];
```
但是,可是,怎么会用一个`UILabel`去展示一个图标,是不是错了?怎么是`conditionIconLabel`,于是从label.text的赋值查找,`condition.climaconCharacter`的类型是`Climacon`,看定义是一个枚举类型,每一个元素都是一个字符.

```
typedef enum {
    ClimaconCloud                   = '!',
    ClimaconCloudSun                = '"',
    ClimaconCloudMoon               = '#',
    
    ClimaconRain                    = '$',
    ClimaconRainSun                 = '%',
    ClimaconRainMoon                = '&',
    
    ClimaconRainAlt                 = '\'',
    ClimaconRainSunAlt              = '(',
    ClimaconRainMoonAlt             = ')',

    ClimaconDownpour                = '*',
    ClimaconDownpourSun             = '+',
    ClimaconDownpourMoon            = ',',
    
    ClimaconDrizzle                 = '-',
    ClimaconDrizzleSun              = '.',
    ClimaconDrizzleMoon             = '/',
    
    ClimaconSleet                   = '0',
    ClimaconSleetSun                = '1',
    ClimaconSleetMoon               = '2',
    
    ClimaconHail                    = '3',
    ClimaconHailSun                 = '4',
    ClimaconHailMoon                = '5',
    
    ClimaconFlurries                = '6',
    ClimaconFlurriesSun             = '7',
    ClimaconFlurriesMoon            = '8',
    
    ClimaconSnow                    = '9',
    ClimaconSnowSun                 = ':',
    ClimaconSnowMoon                = ';',
    
    ClimaconFog                     = '<',
    ClimaconFogSun                  = '=',
    ClimaconFogMoon                 = '>',
    
    ClimaconHaze                    = '?',
    ClimaconHazeSun                 = '@',
    ClimaconHazeMoon                = 'A',
    
    ClimaconWind                    = 'B',
    ClimaconWindCloud               = 'C',
    ClimaconWindCloudSun            = 'D',
    ClimaconWindCloudMoon           = 'E',
    
    ClimaconLightning               = 'F',
    ClimaconLightningSun            = 'G',
    ClimaconLightningMoon           = 'H',
    
    ClimaconSun                     = 'I',
    ClimaconSunset                  = 'J',
    ClimaconSunrise                 = 'K',
    ClimaconSunLow                  = 'L',
    ClimaconSunLower                = 'M',
    
    ClimaconMoon                    = 'N',
    ClimaconMoonNew                 = 'O',
    ClimaconMoonWaxingCrescent      = 'P',
    ClimaconMoonWaxingQuarter       = 'Q',
    ClimaconMoonWaxingGibbous       = 'R',
    ClimaconMoonFull                = 'S',
    ClimaconMoonWaningGibbous       = 'T',
    ClimaconMoonWaningQuarter       = 'U',
    ClimaconMoonWaningCrescent      = 'V',
    
    ClimaconSnowflake               = 'W',
    ClimaconTornado                 = 'X',
    
    ClimaconThermometer             = 'Y',
    ClimaconThermometerLow          = 'Z',
    ClimaconThermometerMediumLoew   = '[',
    ClimaconThermometerMediumHigh   = '\\',
    ClimaconThermometerHigh         = ']',
    ClimaconThermometerFull         = '^',
    ClimaconCelsius                 = '_',
    ClimaconFahrenheit              = '\'',
    ClimaconCompass                 = 'a',
    ClimaconCompassNorth            = 'b',
    ClimaconCompassEast             = 'c',
    ClimaconCompassSouth            = 'd',
    ClimaconCompassWest             = 'e',
    
    ClimaconUmbrella                = 'f',
    ClimaconSunglasses              = 'g',
    
    ClimaconCloudRefresh            = 'h',
    ClimaconCloudUp                 = 'i',
    ClimaconCloudDown               = 'j'
} Climacon;
```

又产生一个疑问,既然是字符,`UILabel`不是应该显示该字符吗?`conditionIconLabel`在初始化的时候设置了字体为:

```
// UIFont name of the Climacon font
#define CLIMACON_FONT @"Climacons-Font"

...

[self.conditionIconLabel setFont:[UIFont fontWithName:CLIMACON_FONT size:fontSize]];
```

原来是用的自定义的Icon字体,字体文件`Climacons.ttf`.下面进入正题.

## 一、如何使用自定义字体

在讲Icon Font之前,先来看看普通自定义字体是如何使用的.

### 1.导入字体文件

<!-- photo 2 -->
![photo 2](http://victorjiang.github.io/images/2014/icon_font_2.png)

### 2.配置.plist文件

在.plist文件中注册自定义字体

<!-- photo 3 -->
![photo 3](http://victorjiang.github.io/images/2014/icon_font_3.png)

### 3.找到自定义字体名称
注册完之后,需要检测自定义字体是否注册成功,并获取新字体名称,检测方法就是遍历所有安装字体,查看是否包含新注册字体:

```
for (NSString *familyName in [UIFont familyNames]) {
        NSLog(@"familyName = %@", familyName);
        
        for (NSString *fontName in [UIFont fontNamesForFamilyName:familyName]) {
            NSLog(@"    fontName = %@", fontName);
        }
    }
```

查看控制台打印出来的所有字体是否有新注册的字体,如果有,则注册成功,并记住字体名(`Kaushan Script`),后面会用到.

<!-- photo 4 -->
![photo 4](http://victorjiang.github.io/images/2014/icon_font_4.png)

### 4.使用自定义字体

只要设置`UILabel`的`font`为自定义字体即可

```
self.label.font = [UIFont fontWithName:@"Kaushan Script" size:35];
    self.label.text = @"Hello World";
    self.label.textColor = [UIColor blueColor];
```

<!-- photo 5 -->
![photo 5](http://victorjiang.github.io/images/2014/icon_font_5.png)

## 二、如何使用Icon Font

这里我使用上面天气的图标字体文件`Climacons.ttf`

### 1.导入图标字体文件

### 2.配置.plist文件

前两部和普通字体一样将字体注册到项目中

### 3.找到图标对应的Unicode编码

<!-- photo 6 -->
![photo 6](http://victorjiang.github.io/images/2014/icon_font_6.png)

从图中可以看出类似`E800`就是各Icon的Unicode码

### 4.使用Icon Font

使用图标字体只需要两步:

1. 设置控件的font属性为Icon Font
2. 赋值Unicode码

```
self.label.font = [UIFont fontWithName:@"fontello" size:40];
self.label.text = @"\ue800";
```

这样UILabel上就显示了上面的Google图标.

这里注意如何使用Unicode码,

1. 使用`\u`前缀,后面跟四位Unicode码
2. 使用`\U`前缀,后面跟八位Unicode码

比如上面的赋值还可以这样:

```
self.label.text = @"\U0000e800";
```

Icon Font的使用虽然很简单,只有两步,但是可读性却很低,不对照上面的图,根本不知道`E800`对应的是哪张图.可以用一些常量来命名,使阅读者可以清楚其意义

```
static NSString * const CustomIconFont_Google = @"\ue800";
static NSString * const CustomIconFont_GitHub = @"\ue801";
static NSString * const CustomIconFont_CSS3 = @"\ue802";
static NSString * const CustomIconFont_Apple = @"\ue803";
static NSString * const CustomIconFont_HTML5 = @"\ue804";
```

	注:可以从上面的`CZWeatherKit`看出,他定义了一个enum的`Climacon`,这边不是通过Unicode码来使用Icon Font的,他是通过Name来使用的.
	
这里引出一个问题,笔者到现在还没有解决,网上关于Icon Font的资料都是通过Unicode来使用,但是在`CZWeatherKit`的项目中,是通过Name来使用,打开两者的字体文件,都可以查看到Unicode和Name对应的值,笔者尝试更换Icon Font的使用方式,在`CZWeatherKit`中通过Unicode对应的值给UILabel赋值,在上面的学习项目中修改通过Name对应的值来给UILabel赋值,结果都不成功.

以及如何通过Index Mode来访问Icon Font,就这一问题,笔者在[知乎](http://www.zhihu.com/question/24095915)上也提问了,希望有知道的同学不吝赐教.

参考文章:[在iOS中使用icon font](http://ued.taobao.org/blog/2013/09/icon-font-in-ios/)