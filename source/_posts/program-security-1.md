---
title: 程序设计安全（第一章——引论）
date: 2022-09-05 17:30:21
tags: 安全
---
> 申明：该系列文章为课堂笔记，主要是为了自己整理笔记方便。其中大量内容来自老师课上讲解和教材，所以我不一一标注引用（侵权必删）。

### 1.1 实例——W32.Blaster.Worm

​		2003年8月11日，爆发了Blaster（冲击波）蠕虫病毒，至少800万台计算机被感染，造成经济损失超过30亿美元。它很好地向我们展示了软件中的安全漏洞如何使我们变得易受攻击。

​		![Blaster.png](http://server.killuayz.top:8089/images/2023/05/26/Blaster.png)

**漏洞原理**

​		Blaster是一种具有攻击性的蠕虫病毒，通过TCP/IP传播，利用Windows DCOM和PRC接口中的漏洞。在继续介绍Blaster怎样利用这些漏洞之前，先补充一下PRC 和DCOM。

> **PRC（Remote Procedure Call）**是Windows 操作系统使用的一种远程过程调用协议，提供进程间交互通信机制，允许在无缝地在远程系统上执行代码。 
>
> **DCOM（Distributed Component Object Model）**，即分布式COM，扩展了组件对象模型，使其能够支持在网络上不同计算机的对象之间的通讯。 

​		当Blaster开始运行，它会检查计算机是否已经被感染了，并且正在运行蠕虫，如果确实如此，那么它就停下脚步，不再次感染计算机，否则，它将利用PRC DCOM接口处理远程信息交换时存在的缓冲区溢出漏洞，发送畸形的PRC DCOM请求，以管理员权限执行任意指令。

```c++
error_status_t _RemoteActivation(
 ..., WCHAR *pwszObjectName, ... ) {
 *phr = GetServerPath(pwszObjectName, &pwszObjectName);
 ...
 }

 HRESULT GetServerPath(
 WCHAR *pwszPath, WCHAR **pwszServerPath ){
 WCHAR *pwszFinalPath = pwszPath;
 WCHAR wszMachineName[MAX_COMPUTERNAME_LENGTH_FQDN+1];
 hr = GetMachineName(pwszPath, wszMachineName);
 *pwszServerPath = pwszFinalPath;
 }

 HRESULT GetMachineName(
 WCHAR *pwszPath,
 WCHAR wszMachineName[MAX_COMPUTERNAME_LENGTH_FQDN+1]) 
 {
 pwszServerName = wszMachineName;
 LPWSTR pwszTemp = pwszPath + 2;
 while ( *pwszTemp != L’\\’ )
 *pwszServerName++ = *pwszTemp++;
 ...
 }
```

其中21、22行用于提取出host name，它显然没有进行边界检查。成功感染计算机后，Blaster将在注册表项**HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run**中添加**"windows auto update"="msblast.exe"**，这样一来，蠕虫便能在计算机开机时就启动。接下来，Blaster会随机生成一个IP地址，并尝试去感染该IP地址对应的计算机。不仅如此，蠕虫会通过cmd创建后门，允许黑客通过4444端口远程访问，同时它还会对Windows Update进行DoS攻击，阻止用户下载补丁。



### 1.2 安全概念

#### 1.2.1 安全角色与关系

![roles.png](http://server.killuayz.top:8089/images/2023/05/26/roles.png)

- Programmer 负责源码正确性，效率和安全性。
- System integrator 负责将新创造的或现有的软件整合成一个新的系统，以满足特定需求。
- System administrators 负责管理一个或多个系统，确保它们正常且安全地运作。他们的职责包括安装或卸载软件、补丁，管理系统优先级等。
- Network administrators 负责管理网络的安全操作。
- Security analyst 负责识别安全漏洞，找出它们的特性。
- Vulnerability analyst负责分析现有及已部署程序的安全漏洞。
- Security researchers 负责开发缓和策略和解决方法。
- Attacker是一个有恶意的角色，它利用漏洞来渗透进入计算机系统，造成威胁。



#### 1.2.2 安全策略（Security Policy）

安全策略是一个规则或操作的集合，一般被系统和网络管理员在系统中加以应用，以保护系统安全，抵御威胁。



#### 1.2.3 安全缺陷（Security Flaw）

安全缺陷是一种**会招致安全风险**的软件瑕疵。并非所有的软件瑕疵都有安全风险，但软件瑕疵的安全风险往往被低估，且目前的软件测试技术一般未能全面地考虑安全风险。



#### 1.2.4 漏洞（Vulnerability）

漏洞是允许攻击者违反显式或隐式安全策略的一组条件，并非所有的安全缺陷都会导致漏洞，只有当其可能被恶意用户所利用而导致安全风险时，方可认为是漏洞。在实践中，由于漏报的风险极高，往往把安全缺陷等同于漏洞。



#### 1.2.5 利用（Exploit）

利用指借助软件漏洞来违反显式或隐式安全策略的软件或技术。利用代码一般被简称为**EXP**。除了用于恶意的目的，EXP也具有一定的积极作用，可以用来直观地证实漏洞确实存在，渗透测试技术最核心的技术就是漏洞利用。**漏洞利用技术也是一个严肃的安全学术研究课题**。



#### 1.2.6 PoC（Proof-of-Concept）

**PoC**是一种仅仅用于验证漏洞是否存在及可能的危害程度的软件或技术，PoC所进行的漏洞利用往往是良性的，关注于提供可见的漏洞攻击演示效果。需要特别注意的是：**PoC很容易转换为恶性的EXP**



#### 1.2.7 缓解（Mitigation）

缓解措施是指能够保护或者限制对漏洞进行利用的方法、技术、过程、工具或运行库。最彻底的缓解措施是找到问题根源并进行修正，而不是简单的隔离（如设置防火墙策略）在软件发布后以打补丁的方式修补漏洞代价高昂。有的情况下，打补丁后停机重启是不可接受的，如骨干网路由器。



### 1.3 主要研究语言——C/C++

#### 1.3.1 用C/C++的原因

**流行**，C/C++是运用最广泛的语言，几乎所有系统软件都由C/C++来编写。

![ranking.png](http://server.killuayz.top:8089/images/2023/05/26/ranking.png)


#### 1.3.2 C语言精神

- 信任程序员
- 不阻止程序员做他要做的事
- 保持语言小而简单
- 对一种操作只提供一种方法
- 即使不能保证可移植性，也要使它快速运行



#### 1.3.3 C语言安全问题

- 有指针机制，允许直接访问内存

```c++
int* ptr = 0x11111; 
int integer = *ptr;
```

- 对内存越界访问不进行检查

```c++
int arr[10];
int integer = arr[20];
```

- 由于历史性能原因，又很多标准函数存在较高人安全风险。如strcpy()



### 1.4 总结

- 糟糕的开发编码是大多数信息安全问题的根源，由其引发的损失巨大（若能在早期发现漏洞） 

- 最彻底的缓解措施是找到漏洞并进行修正，最好的解决办法是在开发过程中避免漏洞

- 在系统软件开发中最常用C/C++语言是“不安全”的语言，但应用极为广泛



### 1.5 思考题

> 如何改进C/C++语言中对越界内存访问不进行检查的问题？分析下这种改进的可行性，有什么代价？对已经存在的软件有什么影响？

​		我认为一个可行的方案是类似JAVA，将C/C++数组封装成为一个类Array，这个类维护一个变量size，每次要读写Array的值都会检查传入的index和size比大小，看是否越界。这种做法使得C语言失去了原本的简洁，由于每次读写数据都要进行判断，降低了运行的效率。且对已经写好的程序，凡是有用到数组的地方都要换成Array，十分麻烦。



> 什么是类型安全？什么是弱类型语言？举例说明之。

​		**类型安全**简单来说就是访问被授权的内存位置，不去访问未被授权的内存区域。它可以被用来形容编程语言和程序等。一个类型安全的编程语言会提供相应的保障机制，一段类型安全的程序不会有隐含的类型错误。

​		例如C就不是类型安全的，他支持显示和隐式的类型转换

```c
int i = 'a';//隐式类型转换
char* ptr_char = (int*)ptr_int;//显示类型转换		
```

​		有些情况下会导致错误

```c
int i = 10;
printf("%lf",i);
```


![test1.png](http://server.killuayz.top:8089/images/2023/05/26/test1.png)

​		可见输出为0，而非10，编译器并未提醒。		

​		可以任意访问未经授权的地址

```c
int a[20];
printf("%d",a[999]);
```

​		
       ![test2.png](http://server.killuayz.top:8089/images/2023/05/26/test2.png)

​		编译器能正常通过，a[999]也能被访问到。



**弱类型语言**

​		**弱类型语言**是一种类型可以被忽略的语言，不同于如Java等强类型语言，PHP、JS这类语言中，一个变量被定义后，该变量可以根据环境变化，自动进行隐式类型转换，不需要显示转换。

例如JS：

```javascript
var A = 10;		//number
var B = "5";	//string
res1 = A + B;	//string
res2 = A - B;	//number
```

执行结果为

![test3.png](http://server.killuayz.top:8089/images/2023/05/26/test3.png)

​		在执行A+B时，系统默认+是字符连接符，所以A被隐式地转换成了string类型，与B连接在一起成了'105'。在执行A-B时，系统默认-是算数运算符，所以B被隐式地转换为了number类型，结果就是5 。

​		弱类型语言的好处是，编程方便、灵活，而当工程量较大时，弱类型语言由于缺乏类型定义的严谨性，会产生一些不必要的错误。



### 参考文章

[1]Robert C. Seacord. Secure Coding in C and C++(Second Edition)

[2] [什么是强类型语言，什么是弱类型语言，为什么python也是强类型语言。 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/62570358)

[3] [什么是类型安全？ （C和C++类型安全比较）_啊大1号的博客-CSDN博客_类型安全](https://blog.csdn.net/a3192048/article/details/82499164)