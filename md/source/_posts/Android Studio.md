---
title: Android Studio
categories: 工具
toc: true 
theme: next
date: 2017-07-02 15:41:34
---

官网：https://developer.android.google.cn/studio/index.html
要科学上网
https://pan.baidu.com/s/1nuABMDb 
安装过程也很简单，一直点击 Next，其中选择安全组件的时候建议全部勾上

1. Tip of the Day 值得关注
2. IntelliJ IDEA 的官网 https:www.jetbrains.com/idea/whatsnew ,可以找到很多实用 IDEA 新功能

## 小技巧
1. 进入演示模式，View菜单下面 `enter Presetation Mode`等模式
2. 按照驼峰移动光标，Settings -> Editor -> Smart Keys -> `Use CamelHumps words`，用`Ctrl + Left/Right`来进行光标移动
3. 鼠标放上去可以看到文档提示，Settings -> Editor -> Show quick documention on mouse move, 根据需要自行定义
4. 多光标编辑，'Alt + Shift + 鼠标单击'，增加新光标

## 断点
1. 断点加条件，Enable 停用/启动断点
    + 右键单击断点
    + 在 Condition 中写下断点条件
2. 临时断点，`Ctrl + Shift + Alt + F8`
3. 异常断点，
    + 打开 `Run-View breakpoints` 界面
    + 左上角 `+` ,选择`Java Exception Breakpoints`
    + 输入要监听的异常
4. 日志断点，不用重新编译，会在断点处输出你需要的日志信息
    + 在需要的地方打断点，右键单击断点
    + suspend 为 False
    + Log evaluated expression 中写入日志信息


## 快捷键
1. 双击`shift`，Search Everywhere，快速查找
2. `Ctrl + Shift + A` ，Search Action，操作入口
3. `Ctrl + E`，Recent Files，最近的浏览过的文件
4. `Ctrl + Shift + E`，Recent Edited Files，最近修改的文件
5. `Ctrl + Tab`，Switcher，文件界面切换
6. `Ctrl + Alt + Left/Right`，代码返回，操作记录
7. `Shift + Alt + Left/Right`，整行移动
8. `Alt + F7`，Find Uages，查找方法调用
9. `Ctrl + B`，进入方法
10. `Ctrl + P`，查看方法参数定义与文档
11. `Ctrl + Shift + Up/Down`，快速行移动
12. `Ctrl + X`，删除
13. `Ctrl + D`，复制行
14. `Ctrl + Shift + Enter`，快速完成
15. `Ctrl + space`，代码提示，`Ctrl + Shift + space`，更加智能
    + Tab 键补全提示，会把后面已输入内容覆盖
    + Enter 补全提示，不会覆盖后面内容
16. ``，
17. ``，
18. ``，
19. ``，
20. ``，
21. ``，
22. ``，
23. ``，
24. ``，
25. ``，
26. ``，
27. ``，
28. ``，

2. 代码提示  ctrl+alt+space
代码格式化   ctrl+alt+l
选中代码  ctrl+w
代码向上/下移动   ctrl+shift+up/down
复制到下一行  ctrl+d
剪切 ctrl+x
删除 ctrl+y
类中的方法间移动   alt+up/down
查找一个文件  ctrl+shift+n
查找类  ctrl+n
查找方法 ctrl+shift+alt+n
查找类中方法  ctrl+F12
查看方法的调用  ctrl+alt+h
查看方法实现 ctrl+shift+i  

查看父类 ctrl+u
复写父类方法 ctrl+o
显示类结构图 ctrl+h
查看变量声明 ctrl+b



代码返回  ctrl+alt+left
窗口返回 alt+left
代码折叠展开  ctrl+plus
隐藏工程面板 alt+1
跳到大括号开头结尾  ctrl+[/]
选择代码，快速添加if，for  ctrl+alt+t
代码生成 ctrl+j
替换   ctrl+r
查找  ctrl+f
打开最近文件  ctrl+e
单步调试 f8
进入方法 f7
调到下一个断点 shift+f8


====》 107 代码提示