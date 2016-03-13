---
layout: post
title: idea快捷键
categories: tool
tags: idea
---

*   [代码颜色](#color_git)
*   [快捷键](#short_cut)
    *   [编辑](#edit)
    *   [查找替换](#search_replace)
    *   [usage　search](#usage_search)
    *   [编译运行](#compile_run)
    *   [调试](#debug)
    *   [导航](#nav)
    *   [重构](#refactor)
    *   [vcs/local history](#vcs)
    *   [模板](#template)
    *   [通用](#general)

<h2 id="color_git">代码颜色</h2>

![git](/images/git/git.png)

|颜色 |状态|
|--|--|
|蓝色	|工作区和本地仓库不一致|
|绿色	|已经add到index中，但是不在本地仓库中|
|白色	|工作区和本地仓库代码一致|
|红色	|只在本地工作区|

<h2 id="short_cut">快捷键</h2>

<h3 id="in practice">实用至上</h3>

Command,Shift,Option,Ctrl参看[mac按键](/2016/02/03/mackeyboard)

|mac|pc|作用|
|---|---|
|Cmd + E |Ctrl + E	|最近使用的文件(不一定编辑)|
|Cmd + Shift + O|Ctrl + Shift + N	|跳转到指定文件或者目录|
|Cmd + Option + O|Ctrl + Shift + Alt + N|	跳转到指定的符号|
|Cmd + 1|Alt + 1	|project工具窗口|
|Esc|Esc	|返回编辑器|
|Cmd + B |Ctrl + U	|跳转到父类或者接口|
|Cmd + Option + B |Ctrl + Alt + B|	跳转到实现|
|Ctrl + H |Ctrl  + H|类的继承关系|
|Ctrl + Option + H |Ctrl + Shift  + H　我的给改成Ctrl + Alt + Shift + H了|方法的继承关系|
|Cmd + Shift + H |Ctrl + Alt + H	|方法的调用关系|
|Cmd + Shift + Option + U |Ctrl + Alt  + U	|UML图展示类的层次关系 （F5 美化 空格 添加新类）|
|Cmd + Option + F7 |Alt + F7|查看使用处 show usages|
|Cmd + Shift + E |Ctrl + Shift + E	|最近被编辑过的文件|
|Cmd + Shift + \`|Ctrl + Alt + \[|上一个工程|
|Cmd + \` |Ctrl + Alt + \]|下一个工程|


<h3 id="edit">Editing</h3>

|mac| pc | 作用  |
|---|---| 
|Ctrl + Space |Ctrl + Space| Basic code completion (the name of any class,method or variable)|
| |Ctrl + Shift + Space| Smart code completion (filters the list of methods and variables by expected type)|
| |Ctrl + Shift + Enter| Complete statement
| |Ctrl + P |Parameter info (within method call arguments)
| |Ctrl + Q |快速文档查看
| |Shift + F1 |External Doc
| |Ctrl + mouse over code |Brief Info
| |Ctrl + F1 |Show descriptions of error or warning at caret
| |Alt + Insert |产生代码(get/set方法，构造器，hasCode/equals，toString以及继承来的方法)
| |Ctrl + O |重写方法
| |Ctrl + I |实现方法
| |Ctrl + Alt + T |Surround with… (if..else, try..catch, for,synchronized, etc.)
| |Ctrl + / |行注释
| |Ctrl + Shift + / |块注释
| |Ctrl + W |递增选择代码
| |Ctrl + Shift + W |递减当前选择
| |Alt + Q |Context info
| |Alt + Enter |Show intention actions and quick-fixes
| |Ctrl + Alt + L |格式化代码
| |Ctrl + Alt + O |整理导入
| |Ctrl + Alt + I |Auto-indent line(s)
| |Tab / Shift + Tab| Indent/unindent selected lines
| |Ctrl + X or Shift + Delete |剪切当前行或者所选区域
| |Ctrl + V or Shift + Insert |复制
| |Ctrl + Shift + V |Paste from recent buffers...
| |Ctrl + D |复制当前行或者所选区域
| |Ctrl + Y |删除当前行
| |Ctrl + Shift + J |合并两行
| |Ctrl + Enter |增加空行
| |Shift + Enter |增加新行
| |Ctrl + Shift + U | 切换大小写
| |Ctrl + Delete |删除以当前光标为首的单词
| |Ctrl + Backspace |删除以当前光标为尾的单词
| |Ctrl + NumPad+/- |Expand/collapse code block
| |Ctrl + Shift + NumPad+ |Expand all
| |Ctrl + Shift + NumPad- |Collapse all
| |Ctrl + F4 |Close active editor tab|

<h3 id="search_replace">Search/Replace</h3>

| 快捷键| 作用  |
| -----  | ----- | 
|Double Shift | 全局搜索
|Ctrl + F |文件内查找
|F3 |查找下一个
|Shift + F3 |查找上一个
|Ctrl + R |文件内替换
|Ctrl + Shift + F |全局查找
|Ctrl + Shift + R |全局替换
|Ctrl + Shift + S |Search structurally (Ultimate Edition only)
|Ctrl + Shift + M |Replace structurally (Ultimate Edition only)

<h3 id="usage_search">usage search</h3>

| 快捷键| 作用  |
| -----  | ----- | 
|Alt + F7 / Ctrl + F7 |Find usages / Find usages in file
|Ctrl + Shift + F7 |Highlight usages in file
|Ctrl + Alt + F7 |Show usages

<h3 id="compile_run">编译运行</h3>

| 快捷键| 作用  |
| -----  | ----- | 
|Alt + Shift + F10 |Select configuration and run
|Alt + Shift + F9 |Select configuration and debug
|Shift + F10 |运行
|Shift + F9 |调试


<h3 id="debug">调试</h3>

| 快捷键| 作用  |
| -----  | ----- | 
|F8 |单步跳过
|F7 |单步跳入
|Shift + F7 |Smart step into
|Shift + F8 |跳
|Alt + F9 |运行到光标
|Alt + F8 |Evaluate expression
|F9 |执行到断代呢
|Ctrl + F8 |切换断点
|Ctrl + Shift + F8 |查看所有断点

<h3 id="nav">导航</h3>

| 快捷键| 作用  |
| -----  | ----- | 
|Ctrl + N |跳到给定名字的类
|Ctrl + Shift + N |跳到给定名字的文件
|Ctrl + Alt + Shift + N |跳到给定名字的符号(注意，以上这三个快捷键都可以使用缩写，比如要找一个名字为QunarSystem的类，可以只输入QS)
|Alt + Right/Left |Go to next/previous editor tab
|F12 |Go back to previous tool window
|Esc |Go to editor (from tool window)
|Shift + Esc |Hide active or last active window
|Ctrl + Shift + F4 |Close active run/messages/find/... tab
|Ctrl + G |跳到第几行
|Ctrl + E |最近用到的文件
|Ctrl + Alt + Left/Right |前进/后退
|Ctrl + Shift + Backspace |Navigate to last edit location
|Alt + F1 |Select current file or symbol in any view
|Ctrl + B or Ctrl + Click |跳到声明处
|Ctrl + Alt + B |跳到实现处
|Ctrl + Shift + I |Open quick definition lookup，快速查看定义
|Ctrl + Shift + B |跳到类型声明处
|Ctrl + U |跳到超类或超方法
|Alt + Up/Down |跳到上一个方法/下一个方法
|Ctrl + ] / [ |光标移动到代码块开始/结束
|Ctrl + H |类型层次
|Ctrl + Shift + H |方法层次
|Ctrl + Alt + H  (我给改成Ctrl + Alt + Shift + H)了|调用层次
|F2 / Shift + F2 |Next/previous 错误
|F4 | 查看源码
|F11 |切换书签
|Ctrl + F11 |Toggle bookmark with mnemonic
|Ctrl + #[0-9] |跳到第几个标签
|Shift + F11 |显示标签
|Alt + F7 |查看方法使用

<h3 id="refactor">重构</h3>

| 快捷键| 作用  |
| -----  | ----- | 
|F5 |Copy
|F6 |移动
|Alt + Delete |Safe Delete  
|Shift + F6 |重命名
|Ctrl + F6 |Change Signature
|Ctrl + Alt + N |Inline
|Ctrl + Alt + M |提取方法
|Ctrl + Alt + V |提取变量
|Ctrl + Alt + F |提取字段
|Ctrl + Alt + C |提取常量
|Ctrl + Alt + P |提取参数

<h3 id="vcs">vcs/local history</h3>

| 快捷键| 作用  |
| -----  | ----- | 
|Ctrl + K |Commit project to VCS
|Ctrl + T |Update project from VCS
|Alt + Shift + C |View recent changes

<h3 id="template">模板</h3>

| 快捷键| 作用  |
| -----  | ----- |     
|Ctrl + Alt + J |Surround with Live Template
|Ctrl + J |插入现有的模板(geti,psf,psfi,psfs,psvm,St等)
|iter | jdk1.5之后迭代风格 for-each
|inst | 用instancof检测类的类型，并向下转
|itco | 迭代java.util.Collection的元素
|itit | 迭代java.util.Iterator的元素
|itli | 迭java.util.List的元素
|psf | public static final
|thr | throw new

<h3 id="general">通用</h3>

| 快捷键| 作用  |
| -----  | ----- | 
|Ctrl + Alt + Y |Synchronize
|Ctrl + Shift + F12 |切换最大化编辑模式
|Alt + Shift + F |Add to Favorites
|Alt + Shift + I |以当前配置检查代码
|Ctrl + BackQuote (`) |Quick switch current scheme
|Ctrl + Alt + S |Open Settings dialog
|Ctrl + Alt + Shift + S |Open Project Structure dialog
|Ctrl + Shift + A |Find Action
|Alt + Shift + insert | 块编辑模式
|Ctrl + Alt + U | 生成类图


IDEA已经为你准备了很好的快捷键文档：菜单栏 -> Help -> Default Keymap Reference   
idea生成builder代码    
在Editor里右键----Refactor---Replace Constructor With Builder
