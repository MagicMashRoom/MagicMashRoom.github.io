---
layout: post
title: Android ConstraintLayout详解
date: 2017-06-26
categories: blog
tags: [android,ConstraintLayout]
description: Android ConstraintLayout详解
---

## 1. 概述

![](http://upload-images.jianshu.io/upload_images/590399-ef127187bb756f30.png?imageMogr2/auto-orient/strip%7CimageView2/2)

在本篇文章中，你会学习到有关ConstraintLayout -- 一种构建于弹性Constraints（约束）系统的新型Android Layout。最终你将会在Android Studio中编辑与构建一个相对复杂的Layout。

###### *收获*

- 新Layout Manager所使用的Constraints系统
- 创建Constraints来构建弹性高效的Layouts 
- 新Layout编辑器的各种功能

###### *需求*

- [Android Studio 2.2 preview](http://tools.android.com/download/studio/builds/android-studio-2-2-preview-1)
- [Android 示例代码](https://github.com/googlecodelabs/constraint-layout)

## 2. 获取示例代码

- 
[直接下载zip文件](https://github.com/googlecodelabs/constraint-layout/archive/master.zip)

- 
使用Git

    $ git clonehttps://github.com/googlecodelabs/constraint-layout.git

## 3. 运行示例代码

1. 打开[Android Studio](https://developer.android.com/studio/index.html)，选择 `File>New>Import Project`，选择步骤2下载的示例代码的文件夹`constraint-layout-start`。
2. 点击`Gradle sync`按钮。
3. 在`Project`面板内打开 `res/layout/activity_main_done.xml`
4. 选择`Design`选项显示最终的layout编辑界面
5. 在编辑器左上角选择`Virtual Device to render the layout with`为`Nexus 5x`
6. 完工

## 4. Constraints 系统概览

Layout引擎使用Contraints指定每个widget来决定他们在layout中的位置。你可以使用Android Studio Layout编辑器界面来手动或者自动指定约束。要更好的理解他，需要我们了解一下他对一个选中的widget的基本控键。

##### *Constraints*

Constraints帮助你保持widgets对齐。你可以使用箭头来决定各widgets的对齐规则。例如（图示 A），从`button 2`左侧控键设置一个constraint到`button 1`的右侧控键意味着：`button 2`会放置于`button 1`右侧`56dp`处

![](http://upload-images.jianshu.io/upload_images/590399-46711a7fe0b9d6aa.png?imageMogr2/auto-orient/strip%7CimageView2/2)

图示 A

##### *控键类型*

![](http://upload-images.jianshu.io/upload_images/590399-c801fcb9d031840f.png?imageMogr2/auto-orient/strip%7CimageView2/2)

图示 B：不同类型的控键

- 
***调整尺寸控键*** - 类似于其他设计/绘图应用，该控键允许你调整widget尺寸

![](http://upload-images.jianshu.io/upload_images/590399-1a932240d99df35d.png?imageMogr2/auto-orient/strip%7CimageView2/2)

- 
***侧约束控键*** -  该控键让你指定widget的位置。例如，你可以使用widget的左侧控键到其他widget的右侧控键相隔`24dp`。

![](http://upload-images.jianshu.io/upload_images/590399-3f34d66b09d0fa21.png?imageMogr2/auto-orient/strip%7CimageView2/2)

- 
***基线约束控键*** - 该控键帮助你对齐任意两个widget的文字部分，与widget的大小无关。例如你有两个不同尺寸的widget但是你想要他们的文字部分对齐。

![](http://upload-images.jianshu.io/upload_images/590399-fabe66bb03bcd08c.png?imageMogr2/auto-orient/strip%7CimageView2/2)

---

#### **《ConstraintLayout从入门到放弃》**

###### *太长；别读*

## 5. ConstraintLayout应用

#### 一）开启

现在，让我们开始来构建你自己的Constraint Layout。

从左侧导航栏打开 `res/layout/activity_main_start.xml`。

- 
**载入constraint-layout依赖**
`constraint-layout`依赖构建在一个分离的支持库里，该依赖支持从Android2.3（Gingerbread）到最新的版本。这个项目在`app/build.gradle`里已经包含了该依赖

    dependencies {
    ...
    compile'com.android.support.constraint:constraint-layout:1.0.0-alpha2'
    }

- 
**回到 `res/layout/activity_main_start.xml`**

该layout已经有了一个空的`ConstraintLayout`。

    <?xml version="1.0" encoding="utf-8"?><android.support.constraint.ConstraintLayoutxmlns:android="http://schemas.android.com/apk/res/android"xmlns:app="http://schemas.android.com/apk/res-auto"android:layout_width="match_parent"android:layout_height="match_parent"></android.support.constraint.ConstraintLayout>

在编辑器底部转换到`Design`选项

![](http://upload-images.jianshu.io/upload_images/590399-9b240ff8af1e54fa.png?imageMogr2/auto-orient/strip%7CimageView2/2)

- **添加`ImageView`**
添加一个`ImageView`到layout。在编辑器内，找到`ImageView`拖到layout内。

![](http://upload-images.jianshu.io/upload_images/590399-8eec1d86d41e5205.png?imageMogr2/auto-orient/strip%7CimageView2/2)

`ImageView`一旦拖到layout中，UI会提示需要resource。`constraint-layout-start`已经包含了resources，请选择`@drawable/singapore`resource。
一旦选中`ImageView`，你可以点击并按住调整尺寸控键来调整图片大小。

![](http://upload-images.jianshu.io/upload_images/590399-983154bb1a32bd7a.png?imageMogr2/auto-orient/strip)

- **添加`TextView`**
找到`TextView`并拖到layout内。

![](http://upload-images.jianshu.io/upload_images/590399-dce361f62fbfcf72.png?imageMogr2/auto-orient/strip%7CimageView2/2)

我们会看到一些警告，因为在`ImageView`以及`TextView`内没有`contentDescription`属性。内容描述（Content Description）属性对于构建可访问应用非常重要。让我们为该属性添加`@string/dummy`。
在右侧，Inspector面板可以改变已选择widget的各种属性。

![](//upload-images.jianshu.io/upload_images/590399-dce361f62fbfcf72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. 选择`ImageView`并添加`@string/dummy`到`contentDescription`属性
2. 在Inspector面板，你可以看到`ImageView`的其他属性。修改`scaleType`为`centerCrop`。
3. 接着，我们选择`TextView`，使用该面板修改`text`值为`@string/singapore`。

#### 二）手动创建Constraints

创建一个约束，你需要在widget的某个控键上点击并按住，然后拖到两一个widget的约束控键内。一旦显示绿色，你就可以松手了最终约束就会被创建。

![](http://upload-images.jianshu.io/upload_images/590399-2c09e1a34f2e5dd0.png?imageMogr2/auto-orient/strip)

> 注意：该部分讲有关手动创建约束的，需要将左上角的自动创建约束按钮关闭
> 
> 
> ![](http://upload-images.jianshu.io/upload_images/590399-f7a75d733cc29c4a.png?imageMogr2/auto-orient/strip%7CimageView2/2)

在开始之前，确保`ImageView`和`TextView`在layout内。我们的目标是在容器、ImageView以及TextView之间创建约束。

假设我们想要`TextView`置于`ImageView`下方。我们可以在`TextView`的顶部控键与`ImageView`的底部控键创建一个约束，如图：

![](http://upload-images.jianshu.io/upload_images/590399-25fe966a4af4a40e.png?imageMogr2/auto-orient/strip)

> 移除约束：移除某个约束只需点击指定约束的控键；移除全部约束需要点击如下按钮：
> 
> 
> ![](http://upload-images.jianshu.io/upload_images/590399-6b41ea59ea2037cc.png?imageMogr2/auto-orient/strip%7CimageView2/2)

下一步，创建`ImageView`跟容器顶部的约束

![](http://upload-images.jianshu.io/upload_images/590399-a65bb166fadfbd09.png?imageMogr2/auto-orient/strip)

最后，创建`ImageView`左右两侧的约束

![](http://upload-images.jianshu.io/upload_images/590399-30c4b59cc0127d99.png?imageMogr2/auto-orient/strip)

89f057b3a8ea3e0b.png

> 创建基线约束 - 连接widget的基线控键到另一个的基线
> 
> 
> ![](http://upload-images.jianshu.io/upload_images/590399-30c4b59cc0127d99.png?imageMogr2/auto-orient/strip)

#### 三）熟悉Inspector

在此部分，我们会了解一下Inspector。它在UI编辑器的右侧。附带有已选择widget的各种相关属性，而且还显示了该视图是如何对齐与约束的。

- 移除`TextView`
- 添加`ImageView`底部约束

此时,UI构建起如下图：

![](http://upload-images.jianshu.io/upload_images/590399-b5d77d4b97974df2.png?imageMogr2/auto-orient/strip%7CimageView2/2)

以下部分描述了不同的元素和他们的使用方法：

**Margins** - widget的外围上下左右为margins。你可以点击按钮设置不同的值来改变margins。在上边截图中，margins设置为`16dp`

**移除constraint** - 在Inspector内点击连接widget与container的线，可以移除约束。当然也可以点击已设置约束的控键来移除。

**相对于约束来放置widget** - 当在一个widget有至少两个相对的连接，比如说顶部和底部，或者左侧和右侧，然后就可以使用滑动条来调节widget在链接中的位置。你还可以改变屏幕方向来进一步调整方位。

![](http://upload-images.jianshu.io/upload_images/590399-f6f76f2342755ddb.png?imageMogr2/auto-orient/strip)

**控制widget内部尺寸** - Inspector内部的线让你可以控制widget内部尺寸。

![](http://upload-images.jianshu.io/upload_images/590399-f6f76f2342755ddb.png?imageMogr2/auto-orient/strip)

![](http://upload-images.jianshu.io/upload_images/590399-ea5715aa385f7642.png?imageMogr2/auto-orient/strip%7CimageView2/2)

**Fixed** - 可以调整widget的宽度和高度

![](http://upload-images.jianshu.io/upload_images/590399-9cbf338ab638b5d8.png?imageMogr2/auto-orient/strip%7CimageView2/2)

**AnySize** - 使得widget占据所有可用的控键来满足约束

![](http://upload-images.jianshu.io/upload_images/590399-9cbf338ab638b5d8.png?imageMogr2/auto-orient/strip%7CimageView2/2)

**AnySize**应用之前

![](http://upload-images.jianshu.io/upload_images/590399-57653136f77d7c2d.png?imageMogr2/auto-orient/strip%7CimageView2/2)

**AnySize**应用之后

![](http://upload-images.jianshu.io/upload_images/590399-152a9d3fb22c8a24.png?imageMogr2/auto-orient/strip%7CimageView2/2)

**Wrap Content** - 含有text或者drawable的widget扩大到填满整个容器

#### 四）自动创建Constraints

`Autoconnect`自动创建widgets之间的连接。开始之前

- 打开`res/layout/activity_main_autoconnect.xml`
- 开启`Autoconnect`（译注：小磁铁图标）

接下来，选中`ImageView`并且拖到layout的中心，如下图所示：

![](http://upload-images.jianshu.io/upload_images/590399-f90293c913217bd5.png?imageMogr2/auto-orient/strip)

下一步，下方的动图展示了以下几个步骤

![](http://upload-images.jianshu.io/upload_images/590399-513086f030493525.png?imageMogr2/auto-orient/strip)

10210fd273ea1a86.png

1. `ImageView`对齐顶部并使用Inspector（AnySize）来确保他扩展到两侧
2. 放置两个button在右下角。使用Inspector面板来修改最右边button的`text`为`@string/upload`以及左侧改为`@string/discard`
3. 将一个`TextView`和一个`Plain Text`放到layout中。
4. 调整`TextView`和`Plain Text`为`48dp`。并自动创建约束。
5. 同样的选中上传button放置到右侧。
6. 最后放置取消button离上传button`32dp`的位置

#### 五）使用Inference创建Constraints

（译注：待更新）

---

**原文链接：**[Using ConstraintLayout to design your views](https://codelabs.developers.google.com/codelabs/constraint-layout/#0)

[ [获取授权](https://101704180005850.bqy.mobi) ]

---

- *如有翻译有误或者不理解的地方，请评论指正*
- *待更新的译注之后会做进一步修改翻译*
- *翻译：[田浩浩](https://github.com/llitfkitfk)*
- *邮箱：[llitfkitfk@gmail.com](mailto:llitfkitfk@gmail.com)*