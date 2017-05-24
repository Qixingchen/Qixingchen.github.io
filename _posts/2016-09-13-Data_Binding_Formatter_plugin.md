---
layout: post
title: Data Binding Formatter Plugin
---


现已在 [JetBrans Plugin 中心提供下载 ](https://plugins.jetbrains.com/plugin/8616-data-binding-formatter)

直接搜索 Data Binding 就可以找到了.


写一个 Android Data Binding 的实体类是一个非常痛苦的事情,要为所有 getter 函数添加 @Bindable 注解, 对所有 setter 函数加入 notify .如果一个实体类有 10 几个成员对象,自动生成 getter 和 setter 之后的操作把手也累断了.

所以我写了一个 IDEA (Android Studio 基于 IDEA 社区版) 的扩展,来自动完成这些操作.

对于一个实体类
``` java
public class TestBean {
    private String testField1;
    private List<Boolean> testField2;
}
```
调出生成菜单,选择 Data Binding Formatter 即可自动生成以下代码:

``` java
public class TestBean {
    private String testField1;
    private List<Boolean> testField2;
    private transient PropertyChangeRegistry propertyChangeRegistry = new PropertyChangeRegistry();


    @Bindable
    public String getTestField1() {
        return testField1;
    }

    public void setTestField1(String testField1) {
        this.testField1 = testField1;
        notifyChange(BR.testField1);
    }

    @Bindable
    public List<Boolean> getTestField2() {
        return testField2;
    }

    public void setTestField2(List<Boolean> testField2) {
        this.testField2 = testField2;
        notifyChange(BR.testField2);
    }

    private void notifyChange(int propertyId) {
        if (propertyChangeRegistry == null) {
            propertyChangeRegistry = new PropertyChangeRegistry();
        }
        propertyChangeRegistry.notifyChange(this, propertyId);
    }

    @Override
    public void addOnPropertyChangedCallback(OnPropertyChangedCallback callback) {
        if (propertyChangeRegistry == null) {
            propertyChangeRegistry = new PropertyChangeRegistry();
        }
        propertyChangeRegistry.add(callback);

    }

    @Override
    public void removeOnPropertyChangedCallback(OnPropertyChangedCallback callback) {
        if (propertyChangeRegistry != null) {
            propertyChangeRegistry.remove(callback);
        }
    }
}
```

不过现在还需要手动 import BR 类,并向类添加 android.databinding.Observable 接口.

后续应该也能自动完成,不过刚刚开始开发 IDEA 扩展,还不是很会用 Psi 类.

项目地址 : [GitHub - Qixingchen/DataBindingModelFormatter: quickly add data binding getter and setter for a model](https://github.com/Qixingchen/DataBindingModelFormatter)

欢迎指出错误指出,或者提交pr wwww

后续计划:

找出已存在的 getter 和 setter   
使用列表,使得可以选择需要增加 binding 的成员对象.