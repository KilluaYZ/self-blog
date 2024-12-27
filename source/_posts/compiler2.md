---
title: 编译原理实验SysY-语法分析器
date: 2023-05-18 17:55:19
tags: [编译, 编译原理, 编译原理实验, SysY, 语法, 语法分析器, yacc, bison]
---
> 源代码：[KilluaYZ/sys-y-compiler (github.com)](https://github.com/KilluaYZ/sys-y-compiler)
## 实验目的
用YACC(bison)工具生成一个SysY语言的语法分析程序，对SysY 源程序进行语法分析。要求对于一个SysY源代码，能按归约顺序将用到的语法规则序列输出到文件，以及画出语法树。


## 实验环境
系统：wsl: ubuntu 22.04
yacc工具：bison
lex工具：flex
c编译器：g++-9.4

## 目录结构
```
.
├── DotDrawer.hpp    //生成语法树
├── Format.hpp       //c++格式化字符串
├── lex.l            //词法分析器
├── main.h           //yacc.y和lex.l公共头文件
├── makefile        
├── test            //测试
│   ├── SysY        //测试源代码
│   ├── dotout      //测试生成的.dot文件
│   ├── svgout        //测试生成的语法树图
│   └── textout        //测试生成的归约式
└── yacc.y            //语法分析器

```

## 用法
``` bash
./main <inFilePath> <outFilePath> <outDotPath>
#inFilePath,outFilePath,outDotPath必须同时指定，如果不指定的话，默认inFilePath="file.txt", outFilePath = "file.out", outDotPath="file.dot"

make #编译
make run #运行并编译test/SysY目录下的源代码，将生成的文件放在test/textout test/dotout test/svgout中

```

## 设计思路与实现

### 词法
词法部分在原有基础上进行扩展，将每一个keyword都赋予单独的类型
``` c++
//main.h
enum TokenType{
//keywords
T_NONE, T_NULL, T_INT, T_VOID, T_CONST, T_WHILE, T_BREAK, T_CONTINUE, T_DO, T_RETURN, T_IF, T_FOR, T_ELSE, T_VAR,
//symbols
T_LEFT_PARENTHESIS, T_RIGHT_PARENTHESIS, T_LEFT_BRACKET, T_RIGHT_BRACKET, T_LEFT_BRACE, T_RIGHT_BRACE, T_DEFINE, 
//relation symbols
T_EQUAL, T_NOT_EQUAL, T_LARGE, T_LESS, T_LARGE_EQUAL, T_LESS_EQUAL, 
//arithmetic symbols
T_ADD, T_SUB, T_MUL, T_DIV, T_MOD, 
//logical symbols
T_NOT, T_AND, T_OR,
//comment
T_SINGAL_ROW_COMMENT, T_LEFT_MULTI_ROW_COMMENT, T_RIGHT_MULTI_ROW_COMMENT, T_MULTI_ROW_COMMENT,
//other
T_DELIMITER, T_NEWLINE, T_ERRORCHAR, T_COMMA,
//right value
T_IDENT, T_INTEGER_CONST, T_DEC_CONST, T_OCT_CONST, T_HEX_CONST,  
};

```


### 语法
语法部分按照yacc的语法规则编写yacc.y文件，该部分的重点是文法的编写，根据SysY语言定义将定义转换为对应文法。在按照定义编写语法后，出现了移进/归约冲突。
![compiler2/image1.png](/images/compiler2/image1.png)
	
根据bison提示可知是Stmt产生式出现问题。其中有问题的部分如下：
![compiler2/image2.png](/images/compiler2/image2.png)
分析到Stmt时，程序不知道应该归约到Stmt还是继续移进T_ELSE。按照逻辑，在SysY代码中如果出现else语句，应该先移进else，而非将之前的if(...){...}归约为Stmt，所以else的优先级更高。根据yacc的规则，我们只需要规定一下两个产生式最后一个终结符的优先级即可，在这里，我们定义T_ELSE的优先级高于T_RIGHT_PARENTHESIS。
``` c++
//yacc.y
%nonassoc<y_id> T_RIGHT_PARENTHESIS
%nonassoc<y_id> T_ELSE

```
现在重新运行一次bison
![compiler2/image3.png](/images/compiler2/image3.png)
可以看到它没有报错，所以移进归约冲突被解决了。

### lex和yacc衔接
lex中识别出的关键字，标识符，数字等信息需要传给yacc，这样yacc才能正常工作，所以lex和yacc的衔接尤为重要。
衔接主要是通过全局变量yylval进行的，yylval默认是int类型，为了使其能够包含更多的信息，方便lex为yacc传递更多的信息，我重新定义了yylval的类型为一个字定义结构体。

``` c++
//main.h
struct YaccType  
{  
    string y_id;  //储存yytext对应的字符串
    int y_int;    //储存yytext对应的数字
    TreeNode* y_node; //储存当前语法树
    int row = 1, col = 1;
    double y_double; //保留，暂时不用
    char y_op;  //保留，暂时不用
	 
};  
#define YYSTYPE YaccType
//把YYSTYPE(即yylval变量)重定义为struct Type类型，这样lex就能向yacc返回更多的数据了  


```
lex中识别到的关键字，标识符，数字等信息会被赋值到yylval的相应属性中。例如，将识别的标识符，keyword存到yylval中：
``` c++
//lex.l
int install_ident(yytokentype tokenType){ 
	yylval.col += yyleng; //更新col
	yylval.y_id = yytext; //更新y_id
	return tokenType;     //返回当前tokenType
}

```

在yacc指定某个token在yylval中储存的属性：
![compiler2/image4.png](/images/compiler2/image4.png)

这样，在yacc分析时，就可以使用$1这样的方式拿到lex分析的结果了。

### 语法树生成
语法树生成使用dot工具，dot的一个特点是，相同的名字会被当做同一个节点，而我们的语法树中存在不少名字相同但是在逻辑上是不同节点的项。
为了能够满足这个功能，我首先在yacc分析语法的过程中生成一个语法树。下面展示的是语法树构建的一个过程
``` c++
//yacc.y 
CompRoot:
    CompUnit
    {
        tree.rootNode = $1;
    }

CompUnit:
    CompUnit FuncDef
    {
        // cout << "CompUnit -> CompUnit FuncDef"<< endl;
        print_out("CompUnit -> CompUnit FuncDef");
        p = new TreeNode("CompUnit");
        p->childNodes.push_back($1);
        p->childNodes.push_back($2);
        $$ = p;
    }
    | CompUnit Decl
    {
        print_out("CompUnit -> CompUnit Decl");
        p = new TreeNode("CompUnit");
        p->childNodes.push_back($1);
        p->childNodes.push_back($2);
        $$ = p;
    }
    ...

```

每归约一次，就会生成一个新的节点，并把产生式右部作为子节点加入到该节点中，这样当程序归约到CompRoot时，我们就可以拿到整个的语法树。拿到语法树后，我们为语法树的每个节点标记一个深度和一个序号，深度顾名思义就是该节点在这棵树中的深度，序号是该节点在同一层中出现次数的排序，即若名称为x的节点在该层是第一次出现，序号就为0，第二次出现就为1，以此类推。

``` c++
//DotDrawer.hpp
vector<Edge> getSortedSequence()
    {
        // step1: 更新节点depth
        updateNodeDepth();
        // step2: 更新节点idx
        updateTreeNodeIdx();
        // step3：先序遍历树,将结果保存
        vector<Edge> res;
        queue<TreeNode*> q;
        q.push(rootNode);
        while (!q.empty())
        {
            auto treeNode = q.front();
            q.pop();
            for (auto childNode : treeNode->childNodes)
            {
                Edge e(treeNode, childNode);
                res.push_back(e);
                q.push(childNode);
            }
        }
        return res;
    }

```
以上函数是将语法书转为dot的核心，根据之前描述，先调用updateNodeDepth()更新语法树每个节点的深度，再调用updateTreeNodeIdx()更新语法树每个节点的序号，最后先序遍历语法树的所有边并返回。

``` c++
//DotDrawer.hpp
void genarateDot(Tree &tree)
    {
        vector<Edge> sortedEdgeArray = tree.getSortedSequence();
        std::stringstream ss;
        ss << "digraph tree {" << std::endl;
        ss << "\tfontname = \"" << this->fontname << "\"" << std::endl;
        ss << "\tfontsize = " << this->fontsize << std::endl;
        ss << "\tnode[shape = \"" << this->shape << "\"]" << std::endl
           << std::endl;
        for (auto it = sortedEdgeArray.begin(); it != sortedEdgeArray.end(); ++it)
        {
            ss << "\t\""
               << it->getTreeNode().first->getName()
               << "\" -> \""
               << it->getTreeNode().second->getName()
               << "\";" << std::endl;
        }
        ss << "}";
        dot_content = ss.str();
    }

```
拿到返回的语法树的所有边后，我们便可以据此生成.dot文件，生成后，再使用dot命令编译，就可以输出正确的语法树。

### 结果展示
一个简单的例子，源代码如下：
``` c++
//file.txt
int main(){
    int a = 1;
    a = a + 1;
}
```
![compiler2/image5.png](/images/compiler2/image5.png)
输出的file.out如下：

```
Number -> T_INTEGER_CONST 
PrimaryExp -> Number
UnaryExp -> PrimaryExp
MulExp -> UnaryExp
AddExp -> MulExp
Exp -> AddExp
InitVal -> Exp
VarDef -> T_IDENT T_DEFINE InitVal
VarDecl -> T_INT VarDef T_DELIMITER
Decl -> VarDecl
BlockItem -> Decl
LVal -> T_IDENT
LVal -> T_IDENT
PrimaryExp -> LVal
UnaryExp -> PrimaryExp
MulExp -> UnaryExp
AddExp -> MulExp
Number -> T_INTEGER_CONST 
PrimaryExp -> Number
UnaryExp -> PrimaryExp
MulExp -> UnaryExp
AddExp -> AddExp T_ADD MulExp
Exp -> AddExp
Stmt -> LVal T_DEFINE Exp T_DELIMITER
BlockItem -> Stmt
BlockRepeat -> 
BlockRepeat -> BlockItem BlockRepeat
Block -> T_LEFT_BRACE BlockItem BlockRepeat T_RIGHT_BRACE
FuncDef -> T_INT T_IDENT T_LEFT_PARENTHESIS T_RIGHT_PARENTHESIS Block
CompUnit -> FuncDef


```
编译出的语法树如下：
![compiler2/image6.png](/images/compiler2/image6.png)
更多的例子请参考test目录下的文件。

## 体会总结
yacc为利用计算机程序输入的结构提供了一个通用的工具。yacc使用者准备了一种输入处理的规范，包括描述输入结构的规则，这些规则被识别调用的代码，一个低级别程序来执行基本的输入。yacc和lex程序的结构十分类似，重点都是在规则段。尽管有现成的SysY语言定义，但是这些定义可能仍然存在移进归约冲突或归约归约冲突，遇到这些冲突，我们可以使用bison提供的工具来检查并排除，排除的常用方式便是规定优先级。在画图时，需要先构建AST，然后才能根据AST为每个节点一个唯一的名称，这样才能画出正确的语法树。

