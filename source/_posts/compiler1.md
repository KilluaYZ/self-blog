---
title: 编译原理实验SysY-词法分析器
date: 2023-05-18 15:03:58
tags: 编译 编译器 编译原理 lex flex SysY
---
> 源代码：[KilluaYZ/sys-y-compiler (github.com)](https://github.com/KilluaYZ/sys-y-compiler)
# 1.实验目标
设计并实现SysY语言的词法分析器

# 2.词法分析器设计
SysY是C语言的一个子集，与C语言类似，他将包含变量、函数、循环语句、逻辑表达式、算术表达式等。该阶段目标是设计并实现基本的SysY词法分析器，所以我的目标主要是识别.sy文件，提取出其中的token，该阶段的产出将被下一阶段语法、语义分析器使用。
我设计的词法分析器将识别以下语言：

# 3.词法分析器实现
词法分析器借助flex工具实现。lex源代码分四个部分——声明、辅助定义、识别规则、用户子程序。
声明部分定义了后面程序需要用到的宏，使用一个enum类型定义了TokenType来表示Token的类型。TokenType的内容前文已经交代，不再赘述。
- 宏ASSERT(condition, errMsg)用于DEBUG，如果condition为false,那么程序就会把errMsg输出到stderr,停止程序运行。
- 宏NOT_REACHED()用于检测特殊情况，有些程序分支是不可能被执行到的，如果被执行到，说明程序出错，为了检测这种情况，定义该宏，将其安插到不可能到达的分支中。
- 宏DEFINE_INSTALL_FUNCTION(ttype)用于将当前的TokenType放入到yylval中，使用这个宏是为了提高程序扩展性。
- 宏PRINT_TYPE_1和PRINT_TYPE_2是两种Token打印方式，一个用于与实验要求对齐，另一个是自己调试使用。
辅助定义部分定义了能够匹配关键词的正则式，每个关键词都对应一个TokenType。
识别规则部分为每个关键词定义了识别后的动作，具体来说，就是为每个TokenType调用了宏DEFINE_INSTALL_FUNCTION，将其放到yylval中，并返回TokenType,让程序能够识别出类型并输出。
用户子程序部分定义了程序的main函数，main函数解析了命令行参数，并根据命令行参数进行解析SysY语言。目前main函数支持命令行模式，以及指定输入输出文件。


# 4.实现结果
命令行模式：将lex源代码编译后运行即可进入命令行模式，输入简单的测试样例可以得到如下结果：

![compiler1/image1.png](/images/compiler1/image1.png)

其中划线部分为源代码，下面的是词法分析器输出的结果。方框中的部分是识别了'\n'，表示新的一行。
编译解析样例1.sy输出到文件1.txt。

![compiler1/image2.png](/images/compiler1/image2.png)
打开1.txt就能看到结果
![compiler1/image3.png](/images/compiler1/image3.png)
实验结果与源码比对，符合预期要求。

