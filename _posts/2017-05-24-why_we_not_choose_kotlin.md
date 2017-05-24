---
layout: post
title: 为什么我们当初没有选择 Kotlin
---



今年的 Google I/O 终于钦定了 Kotlin 作为 Android 第二个官方的开发语言.
一年前,我还是实习生的时候,我司就在尝试 Kotlin 了,并运用在了 [家校通](http://a.app.qq.com/o/simple.jsp?pkgname=com.ci123.noctt),赛诚PAD 上.  
但是为什么后续的应用没有使用 Kotlin 开发呢?  

# 最初遇到的问题

### lint
Kotlin 在当时不支持 lint ,很多本可以发现的问题,在使用 Kotlin 后就无法发现了.  
甚至我实习的第一周,就在 家校通 的多个页面错误的使用了 getDrawable(int, android.content.res.Resources.Theme) 这个 API 21 的方法.导致上线后所有低于 5.0 的设备闪退.当然这大部分责任是我们没有完善测试的锅,但没有 lint 很大程度上导致了后续开发时我们都战战兢兢,生怕用错了方法.  
后来 Kotlin 支持了 lint ,然而很长时间内都只支持到 Android Studio 1.5 为止.于是我在 AS 的四个频道之外,又放了一个 1.5 的包,每次开发 Kotlin 的应用来使用.

### Data Binding
我司开发时使用了 Data Binding,并且使用的很多.当时 Data Binding 本身还不稳定,有时会出一些奇怪的错误.和 Kotlin 混用后,遇到的错误更多了,还无法确定究竟是哪方面出了错误.  
那为什么选择了继续使用 Data Binding 而不是选择停用 Data Binding 使用 Kotlin 呢? 因为 Data Binding 真的有用,虽然最初踩了一些坑,但使用起来之后,不管在代码量上,代码结构上,Debug 难度上 都有很大的益处.  
然而 使用 Kotlin 后,我们没有发现任何益处.

# Kotlin 的问题

使用一个新的技术,一定要有一个使用它的理由,比如可以节约时间,或者可以减少错误.当然官方选择也是一个重要的理由.  
那么 Kotlin 有什么优势呢?

### data class

 每次介绍 Kotlin ,都会拿 data class 进行开场.的确,对于一个基础的数据类,Kotlin 看上去很简单.然而实际使用上就是另一回事了.  
 如果你需要一些自定义的 setter 呢?(比如使用一个类来设定多个属性),如果你需要使用 Data Binding 呢?  
 你就会发现 Kotlin 的 Data Class 没有任何意义.getter 和 setter 都需要再次重写.  
 而且在Java 中 getter ,setter, toString 等都可以使用 IDE 自动生成.包括 Data Binding 属性也可以使用 [Data Binding Formatter](https://plugins.jetbrains.com/plugin/8616-data-binding-formatter) 来生成.如果我们当时使用 Kotlin 来写 data class, 我想我也许会在写一个 Kotlin 版本的 Data Binding Formatter,如果 IDE 允许的话.

### null 安全 
Kotlin 另一个重要优势,"Null 安全"实际上也没什么优势.  
Android 已经有了 [Annotations](https://developer.android.com/studio/write/annotations.html#adding-nullness) 来处理 Null 的问题.   
另外由于 Android 的生命周期的特性,例如在 OnCreate 中赋值的变量,在 Kotlin 中依旧需要声明为可空变量,并没有比 Java 高明多少.  
"?."的语法糖,实际上大部分如果可能为空的情况,空和非空都需要进行处理(例如对用户进行提示),所以这个语法糖基本用不上.  

### 多样的写法
Kotlin 的写法很多,从一个方法返回就有多种写法.也许有人觉得这是优点,然而对于企业级的开发协作来说是难以接受的.最终使用那种方法来写,又可以争论个几天.

### 生态的缺失
当然最重要的还是生态不完善.Java 有大量的 扩展, 活动模板 等帮助开发.然而当时 Kotlin 什么都没有.  
另外,Kotlin 的 与 Java 完全兼容说的是 Kotlin 调用 Java 时,千万不要觉得反过来也是如此.  
也就导致了想用 Kotlin 的项目发现,写底层方法很适合 Kotlin ,然而 Java UI 层调用就坑了.  
不会遇到 Java 层调用的 UI 层,又发现 Kotlin 没什么用武之地,而且没有扩展与活动模板等帮助,还得纯手写.


# 结论

当然,本文并不是说 Kotlin 没有优势,只不过想提醒你, Kotlin 并没有宣传的那么好.  
  

后续我们会再次使用 Kotlin 么?  
当然会.既然 Kotlin 已经是 Android 官方的开发语言了,有理由相信,在某一个时间点后,我们会选择使用 Kotlin 来开发新的需求.
 
