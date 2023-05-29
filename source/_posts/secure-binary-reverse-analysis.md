---
title: 软件安全分析--二进制逆向分析
date: 2023-05-26 18:11:48
tags: 安全
---


虚拟机要点
- 主要的东西都在文档下
  - PETools 修改二进制文件
  - BinarySecureit 课程内容
  - 在mscode中编译使用
  
反汇编和反编译
- 用到IDAPro 在文档Tools下，可直接搜索ida
  - 多窗口
  - 右键+text可以在左边显示地址
  - 在编译视图下按F5反编译
  - ctrl+g跳转
  - ctrl+f查找
- 内存读写监控需要用到pin
  - pin -t MemoryTrace.dll -- target.exe
  - 插桩分析，输出的内容在trace.out下
  - 根据trance拿到的地址，再给MemoryHook
- 动态方式注入代码
  - 用IDA和hook
- 静态方式注入
  - 用PETools
  - 打开exe，打开section
  - 先修改.rodata节的大小
  - 再添加.new节