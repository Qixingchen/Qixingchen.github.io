---
layout: post
title: Data Binding Formatter 的更新
---

写完第一版的 [Data Binding Formatter](https://loli.xing.moe/Data_Binding_Formatter_plugin/) 后,闲置了快一年,突然我又拾起来更新了下.

## 使用 IDEA 源码版本查看 Plugin SDK 源码

作为 Java 开发者,看源码是习惯性动作.然而默认情况下,只能看到混淆后的代码.可以按照 [JetBrans 的指导](https://www.jetbrains.com/help/idea/configuring-intellij-platform-plugin-sdk.html)将 IDEA 源码附加在 Plugin SDK 上.  
值得注意的是,Plugin SDK 依旧需要使用 IDEA 可执行版本,而不能选择源码版本,不然你会得到一片 import 失败的壮观景象(不要问我是怎么知道的)

## PSI 相关
PSI 是什么呢?全称是 Program Structure Interface,IDEA 都使用 PSI 表现代码.通过对 PSI 的操作,可以直接修改对应的代码.  
有个很好的工具,Tools 下的 PSI viewer,可以查看当前文件的 PSI 组织.不过需要加载至少一个 plugin module 才会显示.可以按照  [JetBrans 的指导](https://www.jetbrains.com/help/idea/viewing-psi-structure.html) 强制开启.  
Data Binding Formatter 中的 [WriterUtil](https://github.com/Qixingchen/DataBindingModelFormatter/blob/master/src/moe/xing/databindingformatter/WriterUtil.java) 使用了 [增加一整个方法](https://github.com/Qixingchen/DataBindingModelFormatter/blob/e7961a6be920deb29119eb3a28ad463208676765/src/moe/xing/databindingformatter/WriterUtil.java#L126-L140),[增加方法的 Annotation](https://github.com/Qixingchen/DataBindingModelFormatter/blob/e7961a6be920deb29119eb3a28ad463208676765/src/moe/xing/databindingformatter/WriterUtil.java#L142-L151),[向一个方法内部增加一行语句](https://github.com/Qixingchen/DataBindingModelFormatter/blob/e7961a6be920deb29119eb3a28ad463208676765/src/moe/xing/databindingformatter/WriterUtil.java#L168-L182),[增加一个变量](https://github.com/Qixingchen/DataBindingModelFormatter/blob/e7961a6be920deb29119eb3a28ad463208676765/src/moe/xing/databindingformatter/WriterUtil.java#L213-L216)等操作.  
而且,[FieldUtils](https://github.com/Qixingchen/DataBindingModelFormatter/blob/e7961a6be920deb29119eb3a28ad463208676765/src/moe/xing/databindingformatter/utils/FieldUtils.java)提供了寻找变量 getter 与 setter 的方法.

## Data Binding Formatter
目前 Data Binding Formatter 已经具备了列出所有变量,寻找其 getter 与 setter 判断是否含有 Data Binding 部分.能够自动向现有的 getter 与 setter 增加 Data Binding 部分代码的所有功能.后续可能允许自定义 notifyChange 方法的命名,因为这个明明可能会引发歧义,与其他模板类似的明明冲突.  
不过截至目前,Data Binding Formatter 已经比较完善了,可以在各种需要 Data Binding 的地方进行使用www.