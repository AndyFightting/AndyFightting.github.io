---
layout: post
title: "Android Studio插件"
date: 2015-11-22 14:10:45 +0800
comments: true
categories: android
---
### * Logger 
推荐一款打印工具[Logger](https://github.com/orhanobut/logger),比自带的打印效果更好，主要是json打印是格式化的！Github上写的地址`compile 'com.github.orhanobut:logger:1.12'`我下载失败，我用的是`compile 'com.orhanobut:logger:1.11'`。
![img](/myimg/android/logger.png)
打印颜色可以在preference里设置，要先`Save as`保存一下，然后把右边的`Use inherited attributes`取消，然后就可以选择颜色了<!--more-->
![img](/myimg/android/logColor.png)

### * Sexy Editor
设置编辑区域背景图的, 这样敲代码也更有力气拉~~哈哈

![img](/myimg/android/screen.png)

怎么添加插件？有如下三种方式`install JetBrains plugin`,`Browse repositories`,`install plugin from disk`。`Sexy Editor`是用第二种方式添加。

![img](/myimg/android/add_plugin.png)

我的`sexy editor`设置

![img](/myimg/android/sexy_editor.png)

### * Mac下的快捷键

Mac中的标识：`⌘:command`, `⌃:control`, `⇧:shift`, `⌥:alt/option`

`⌘ + ⌥ + L`: 代码对齐格式化

`^ + ⌥ + O`: 除去无效的import引用

`F1`: 查看文档

`F2`: 定位到未被使用的成员

`F3`: 添加书签

`F4`: 定位到声明处

`⇧ + F6`: 文件重命名

`⌘ + F12`: 显示内部成员

`⌘ + ⇧ + F7`: 代码高亮

`⌥ + ⏎`: 添加缺少的import

`⌘ + delete`: 删除整行

`⌘ + N`: 生成代码，getter setter 等

`⌃ + H`: 查看类的层级关系

`⌘ + U`: 查看父类的同名方法

`⌘ + J`: 快捷插入常用代码片段

`⌥ + ⇧ + ↑或↓`: 上下移动光标所在的行

`⌘ + ⇧ + ↑或↓`: 上下移动整个方法块

`⌘ + ⌥ + ⏎`: 当前行上面插入一行

`⌘ + ⇧ + U`: 大小写转化

`⌘ + E`: 查看打开过的文件

`⌃ + T`: 重构面板

`⌃ + O`: 选择要重写或者要实现的方法

`⌘ + O`: 快速搜索class

`⌘ + ⇧ + O`: 快速搜索file

`⌘ + ⇧ + F`: 全局搜索

`⌘ + ⇧ + R`: 全局替换

`⌘ + +或-`: 展开或者收起代码块

`⌘ + /`: 注释 //

`⌘ + ⌥ + /`: 注释 /**/

`⌥ + ⏎`: 提示错误解决方法


可能会有人觉得为啥重命名`⇧ + F6`没反应呢？跟 F1 ~ F12 有关的都没反应...

因为Mac系统默认没有启用它们...在系统设置，键盘选项里选中就可以了。

![img](/myimg/android/keyf.png)

当然可以设置自己喜欢的快捷键，我自己把运行和调试改成了`⌘ + R`和`⌘ + D`

![img](/myimg/android/keymap.png)

