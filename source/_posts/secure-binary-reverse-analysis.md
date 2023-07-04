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

- IDA python （必做）
  - 为IDA编写python脚本
    - 输出exe文件中所有的函数名以及起始地址和结束地址
  - 两种模式
    - 命令行方式
      - 输入命令时log文件和脚本文件要打引号

- pin (选做)
  - 编译
    - 文档/Tools/Pin/source/mypintool/用vs打开
    - 编译之前要修改MyPinTool.vcproj，找到PlatformToolTest，把所有的110改成143
    - 然后点击绿三角就可以开始编译了
    - 也可以直接build，这个东西没法运行，因为它生成的东西是一个动态库
    - 生成的东西在release/debug下的*.dll文件
  - pintool用法
    - pin -t <已生成的dll文件 *.dll>  -- <target *.exe>
    - 然后会在执行目录下生成一个trace.out


模糊测试技术
基本概念
- 流程
  - 初始化
  - 测试用例筛选
  - 对筛选出的测试用例进行变异
  - 执行这些测试用例，获取对应反馈信息



