---
title: 怎么写烂代码？（bushi）
date: 2022-08-28 11:40:53
tags:
---
#### 前言

在刚开始学习编程时,我面对老师布置的大作业时常感觉无从下手，花了一个又一个下午的时间，大作业没写出来，大BUG倒是写了一堆。。TAT，相信很多同学跟我有一样的感受。本文我根据这两年在RUC的学习经验，结合贪吃蛇游戏开发的例子，给大家分享一下写大作业时程序构建思路和一些防范BUG出现的编程小技巧。

#### 设计思路

##### 自顶向下

在生活中，复杂的问题比比皆是，当我们在面对这类复杂问题时，自顶向下便是一种相对有效且可靠的分析方法。自顶向下的主要本质是**分解**，**将复杂的问题分解为一个个子问题，从顶层的抽象逻辑开始考虑，逐步分解到底层的具体实现**。

![ziwenti](http://43.138.62.72/data/bad-code-proj/image/ziwenti.png)

相信不少人看了以上概念也难以把握它，我也一样，看完这个介绍后半懂不懂，心中有许多疑问，问题要怎么拆解，拆到什么时候停......我不擅长弄概念，现在学什么东西都是通过例子来学，所以这里以贪吃蛇游戏的开发为例，分享我开发贪吃蛇的“心路历程”。刚接触到这个项目，我也一时犯难，但是用自顶向下这套思维方法分析下来，贪吃蛇也比较简单。首先关注最顶层的逻辑，贪吃蛇要实现的功能是**实时打印当前的地图**，**通过按键控制蛇的行动**，**蛇对不同物件的响应**（即蛇撞到自己或者墙时会导致游戏结束，当吃到食物时蛇身会变长，分数会相应地增加），但是这些功能还是比较抽象，找到了要实现的功能，接下来的工作便是想想这几个功能逻辑先后的排布。

```c++
while(1){
	key_num = get_key();//得到当前的按了什么键
	snake_turn(key_num);//根据按键转向
	walk();//蛇向着当前的方向走一步
	react();//蛇对当前蛇头碰到的物件的响应
	update_map();//更新地图
	Sleep(time);//给玩家一小段反应的时间
}
```

现在我们大概知道要做什么了，下面的任务便是将上述的几个功能进行分解。以snake_turn(eky_num)为例，该部分需要实现对不同的按键，进行不同的处理，逻辑简单，我就直接粘代码了，可见，该部分被分成了转向、停止游戏、退出游戏几个小的分支。

```c++
switch (key_num) {
    case UP_KEY:
        turn(snake, UP);//蛇向上转
        break;
    case RIGHT_KEY:
        turn(snake, RIGHT);
        break;
    case DOWN_KEY:
        turn(snake, DOWN);
        break;
    case LEFT_KEY:
        turn(snake, LEFT);
        break;
    case SPACE_KEY:
        game_pause();
        break;
    case ESC_KEY:
        game_exit();
        break;
    }
```

紧接着又是继续向下分解，这里以turn函数为例，直接上源码：

```c++
void turn(Snake& snake, direction new_dir) {
    //方向正好相对，不改变
    if (abs(snake.head.dir - new_dir) % 2 == 0) {
        return;
    }
    snake.head.dir = new_dir;
}
```

可以看到当问题分解到turn函数这里，函数已经有了具体的实现了，已经不可再往下分了，所以我们就此止步。其他的game_pause(); game_exit();也是同样的道理，**将问题不停往下分解，直到可以用代码具体地实现**。

一个例子显然不够，让我们再看一个例子！个人认为整个贪吃蛇游戏中核心的部分是让蛇动起来，可是怎么动？上层逻辑是个不折不扣的甩手掌柜，它可不管蛇具体怎么动，它只管在调完snake_turn()函数后调用walk()函数，在它眼里，调用了walk()这个函数，蛇就算是走了一步了，至于下层要怎么弄，是要把蛇整个的删掉再重新创建一条蛇出来放在新的位置，还是把原来的蛇砍成几段，再新的位置重新拼接出一条蛇来，让下层自己去折腾。只苦了我们程序员，既要站在上层逻辑的高度“宏观调控”，又要同底层的函数一起深入细节。好了，废话不多说，我们继续向下分析walk()函数要怎么实现，让蛇动起来！

首先我们来思考一下，一条象素蛇的运动方式，请看：

![image-20220825145437585](http://43.138.62.72/data/bad-code-proj/image/snake1.png)

假设这条蛇要向右走，最先移动的部分必定是蛇的头部



![image-20220825145635230](http://43.138.62.72/data/bad-code-proj/image/snake2.png)

接下来便是第0段身体移动到头部之前所在的位置

![image-20220825151033271](http://43.138.62.72/data/bad-code-proj/image/snake3.png)

紧接着是第1段身体移动到第0段身体所在的位置

![image-20220825151121715](http://43.138.62.72/data/bad-code-proj/image/snake4.png)

至此蛇成功迈出一步！如果当前蛇不是向右走，而是向上或向下走，与上同理，不再赘述。

![image-20220825151623041](http://43.138.62.72/data/bad-code-proj/image/snake5.png)

不难看出蛇移动的逻辑：

**1、头向当前位置走一步（snake.head.dir方向）**

**2、第0个身体的向头的上一个位置移动**

**3、第n个身体向第n-1个身体位置移动**

因为头需要根据前进方向不同，向不同格子移动，而身体只需要顺次移动到前一段身体上一次所在的位置就好，所以很自然地，我将移动部分(walk)分为头部移动(snake_head_walk)和身体移动(snake_body_walk)

```c++
//蛇朝着当前方向走一步
void walk(Snake& snake) {
    auto last_head_pos = snake_head_walk(snake);
    snake_body_walk(snake, last_head_pos);
}
```

根据自顶向下的设计方法，再往下看，这里以蛇的头部分怎么移动为例，该部分主要的逻辑是根据当前蛇前进的方向(snake.head.dir)算出下一步蛇头所在的位置，将当前蛇头删去，再在新的位置生成蛇头，这就完了吗？不是的，为了方便，我在蛇头部移动的函数中加入了一些判断，用来判断如果蛇向这个方向走，会发生什么。如果撞到了墙或者撞到了自己，那就通过throw EndGameException的方式停止游戏，如果吃到了食物，那就增加分数，增加蛇的长度。Talk is cheap, show me your code!

```c++
//蛇头朝当前方向走一步，同时返回上一次的头的位置
pair<int, int> snake_head_walk(Snake& snake) {
    int new_head_x = snake.head.x + dx[snake.head.dir];
    int new_head_y = snake.head.y + dy[snake.head.dir];

    //撞到墙
    if (get_map(new_head_x, new_head_y) == WALL) {
        //抛出结束游戏异常
        throw EndGameException(EatWall);
    }

    //撞到自己
    if (get_map(new_head_x, new_head_y) == BODY) {
        //抛出结束游戏异常
        throw EndGameException(EatSelf);
    }

    //吃到食物
    if (get_map(new_head_x, new_head_y) == FOOD) {
        //加分
        add_grade(snake);
        cursor_print_score(snake.score);
        lenthen_snake(snake);
        random_gen_food();
    }

    auto last_pos = make_pair(snake.head.x, snake.head.y);
    //删掉上次的头
    rm_snake_head(snake);
    //设置头到新的位置
    set_snake_head_pos(snake, new_head_x, new_head_y);
    return last_pos;
}
```

顺着自顶向下的思路继续往下，再分析一下怎么把蛇头删除，怎么再新建蛇头。这里以负责设置头到新的位置的函数set_snake_head_pos为例，再往下拆解。要怎么实现设置蛇头的位置呢？一个很简单的思路是直接对snake.head.x，snake.head.y进行操作，修改他们的值，同时将新的值更新到map上，并且在屏幕的row行col列打印出蛇头就好了。当然在snake.head.x，snake.head.y对操作之前少不了对传入参数row和col的合法性检查，废话不多说，直接贴代码给大伙看看吧。

```c++
//蛇移动
void set_snake_head_pos(Snake& snake, int row, int col) {
    if (row >= ROW || row < 0 || col >= COL || col < 0) {
        throw MyException("[ERROR] set_snake_head_pos::row or col is out of bound!");
    }

    //检查该位置是否可以走
    if (get_map(row, col) == WALL || get_map(row, col) == BODY) {
        throw MyException(string("[ERROR] set_snake_head_pos::position") 
                          + to_string(row) 
                          + string(",") 
                          + to_string(col) 
                          + string(") is not valid!"));
    }

    snake.head.x = row;
    snake.head.y = col;
    set_screen_map(row,col,HEAD);
}
```

在代码的最后，我们还调用了一个set_screen_map函数，这个函数的作用就是将当前蛇的状态信息在map上更新，同时也在屏幕上更新。聪明的同学可能会问，为什么要用这种蹩脚的方式更新地图和屏幕，而不是在对map操作完后，根据当前map存储地信息，整体地将整个屏幕刷新一遍，就像开头整体思路那写的那样呢？原因很扎心，为了用户体验。整体刷新一遍的效果太差了，我测试时，整个界面一直在闪，玩得我头晕，所以只能麻烦些，用另一种方式，能不更新的就不更新，不得不更新的，才将光标移动到待更新的位置，然后打印输出，这样的话，界面只会在需要被更新的地方更新，不用更新的地方——比如墙，就不用再耗费资源重绘了。上面这段话也已经简单介绍了set_screen_map函数的作用了，为了少说废话，我就此止步，不再向下分析了，又兴趣的同学可以看源码。通过上面的两个例子，相信大家应该对自顶向下的设计方法有了一些直观的感受和体会了吧。

下面我将放出整个项目主要函数的调用图，看了这个图相信你对这部分会有更深的体会。下面这张图可能会有不严谨或遗漏的地方，但是这些在我看来都不重要，重要的是看到整个项目的框架！项目的函数主要调用就像一棵树，整个问题从main函数开始逐步分解，顶层的逻辑被一步步分解成了底部可以被具体实现的函数。

![main](http://43.138.62.72/data/bad-code-proj/image/main.png)

##### 分层

在计算机领域，分层的思想可谓无处不在。例如大家在CSAPP中都学过的储存器金字塔（图源：CSAPP）：

![储存器](http://43.138.62.72/data/bad-code-proj/image/储存器.png)

再看linux内核（[图源](https://www.cnblogs.com/holyxp/p/9452307.html)），整个系统的设计也被分成了用户层、内核层、硬件层。

![linux-kernel](http://43.138.62.72/data/bad-code-proj/image/linux-kernel.png)

当我们看b站，刷知乎时默默做出奉献的TCP/IP协议同样也是分层的（图源：TCP/IP详解 卷1）。

![tcpip](http://43.138.62.72/data/bad-code-proj/image/tcpip.png)

一个大型的软件项目更是需要有合理的层级结构，例如下面这个线上商城项目的业务架构（[图源](https://github.com/qishanzhiruan/basemall)）

![Smart Mall](http://43.138.62.72/data/bad-code-proj/image/Smart-Mall.webp)

除了上述的分层例子外还有很多例如MVC，MVVM架构等等，都是分层结构的。由此可见层级结构在计算机中无处不在，应用广泛，当我们在编写大型程序的同样也要将整个项目进行合理的分层。

现在我们将注意力回到贪吃蛇上，毫无疑问，项目的最顶层应该就是展示给用户的界面和给用户操作的按键，在这层之下便是游戏的逻辑，即对按键的响应，对当前不同事件的处理等，最底层的应该就是数据层和打印层，即对整张地图数据的操作，以及元素的打印。整个项目的层级表示如下：

![分层](http://43.138.62.72/data/bad-code-proj/image/分层.png)

图可能不太严谨，甚至部分和代码还对不上号。。但是没关系，我们关注的重点不在细节，而在更大的方面——层的划分。整个项目我粗略地划分成了三大层，用户层、逻辑层和数据、打印层。最顶层是**用户层**，它只需要负责好给用户展现贪吃蛇游戏的界面，读入用户当前按键，中间的**逻辑层**就需要负责处理读取到的按键，根据当前的map信息和蛇的状态，定期地更新map数据以及更新屏幕显示，底层的**数据层**负责修改和查询map的信息，**打印层**负责把逻辑层传来的打印任务打印到屏幕。各个层各司其职，只通过层与层之间的API进行交流。

这样做的好处有三。

一是**便于测试和调试**。由于各层各司其职，只对自己的层负责，所以一旦程序跑出问题了，通过分析问题的类型，报错信息等，便可很快地锁定BUG在哪个层，进而找到是哪个层，那个模块出了问题，方便我们调试。当我们需要对程序进行测试，以验证其正确性时，不需要将一次性测试整个程序，而是可以测试每个层功能是否正常，进而一定程度上反映程序运行是否正常。

二是**便于扩展**。如果我们想给程序增加点新内容，例如我们想让这个简易的贪吃蛇程序走向精彩的互联网世界中，将游戏进度保存在云端，这时候，我们只需要增加一个网络层，他将调用数据层提供的API，读取Map信息，蛇的状态等，实现与云端服务器通信的逻辑，将本地信息上传到云端，或者反过来将云端数据下载的本地，更新本地的map和蛇的状态等。按照这种方式设计，我们便不需要改动数据层、打印层、用户层的代码，只需要增加网络层的代码和修改逻辑层的部分代码便可实现功能，做到了尽可能少地修改已有的代码，便于我们扩展程序。

![new_snake_struct](http://43.138.62.72/data/bad-code-proj/image/new_snake_struct.png)

三是**降低复杂度**。下图是一个APP设计的架构，以下讲解以一款天气预报的APP为例。在天气预报APP中，用户层承担与用户交互的任务，中间逻辑层承担从数据层取天气预报数据，解析数据，排布布局元素之类的活儿，数据层就负责得到天气的数据，当逻辑层有需要时呈递给它。三层各司其事，用户层**不关系**你逻辑层怎么解析数据，怎么排布元素，在它眼里，它只负责把用户点击、滑动的数据给逻辑层，怎么折腾逻辑层你自己看着办，折腾完了把东西给它，它展现给用户就行。同样地，逻辑层**不关心**数据层的数据究竟是从程序员网上ctrl+c, ctrl+v一顿操作放到txt里来的，还是从本地数据库里查询到的，还是从网络上由其他电脑发来的，还是由服务器发来的。逻辑层对数据层说的还是那句话：数据层老弟，这些天气预报的数据怎么搞，从哪搞，你自个儿想办法，折腾完了把数据按规定格式放在数据仓库等我来拿就好。以这种分层的方式设计整个程序，逻辑层不用加上冗余的逻辑来判断究竟时从本地还是云端读取数据，要怎么读取数据，只需要干好自己的事情，把这些问题甩给数据层就好，降低了单层代码的复杂程度。

![层级结构](http://43.138.62.72/data/bad-code-proj/image/层级结构.png)

**分层虽好可不要贪用**。凡事均有利弊，过度的分层会把整个程序切分得太细，一方面影响运行效率，另一方面在程序运行时东调一个函数，西调一个函数，会对debug造成一定的障碍，总之一句话，**过犹不及**。至于这个度的把握，我觉得只有在不断的编程实践中慢慢积累经验。



##### 模块化

简单的模块化很容易理解，就是把具有某一特定功能的代码封装成一个函数或是一个类。函数的封装容易理解，就是把功能差不多的代码封装到一个函数里，用到了就调用这个函数，不要反复的copy同一段代码

```c++
void fun() {
    // do something
    for (int i = 0; i < 100; ++i) {
        for (int j = 0; j < 100; ++j) {
            cout<<map[i][j];
        }
        cout<<endl;
    }
    
    //do something
    for (int i = 0; i < 100; ++i) {
        for (int j = 0; j < 100; ++j) {
            cout<<map[i][j];
        }
        cout<<endl;
    }
    //....
}
```

```c++
void show(){
    for (int i = 0; i < 100; ++i) {
        for (int j = 0; j < 100; ++j) {
            cout<<map[i][j];
        }
        cout<<endl;
    }
}

void fun() {
    // do something
    show();

    //do something
    show();
    //....
}
```

至于类的封装大家就参考网络上的OOP吧，随便拉出来一个都可能写得比我好。在编写本文的例子——贪吃蛇游戏时我也有意识地将各个逻辑上可分的代码划分成了几个模块，例如map是一个模块，蛇snake是一个模块，print是一个模块等等。在我看来，将代码模块化和分层由诸多相似的地方，只不过**分层更多关注的是整个项目，模块化更多关注的是层内的逻辑**。也因此，模块化和分层的优点是类似的，大家对比着思考就好。

#### 形式

相信大家一定知道形式与内容的辩证关系。内容和形式是辩证的统一。没有无形式的内容，也没有无内容的形式。当形式适合于内容时，它对内容的发展起着有力的促进作用，反之，就起严重的阻碍作用....balabala。我不能再念了，再念大家可能要头疼了。虽然但是，上面这段话也不是随便说说的，是为了强调一下形式对代码的重要作用。个人认为，一个优雅的代码，应同时具有逻辑上的优雅和形式上的优雅的。逻辑上的优雅的一个方面在前文已有了简略的介绍，接下来的主要内容便是形式上的优雅。（下文很大一部分参考了这篇[文章](https://zhuanlan.zhihu.com/p/516564022)）

##### 变量命名要有意义

```c++
int a = 43;  //bad
int age = 43;//good
```

##### 变量命名风格一致

```c++
//bad
int wWidth = 10;  
int w_heigh = 20;
```

```c++
//good
int windowWidth = 10;
int windowHeight = 10;
```

##### 拒绝深层嵌套（不要堆shi山）

```c++
//bad
if(condition1){
	if(condition2){
		if(condition3 && condition4){
			if(condition5 || condition6){
				printf("OK");
			}
		}
	}
}
```

```c++
//good 
if(!condition1){
	break;//or return;
}
if(!condition2){
	break;
}
if(!condition3 || !condition4){
	break;
}
if(!condition5 && !condition6){
	break;
}
printf("OK");
```

##### 多写注释

```c++
//bad
const int stn = 700;
```

```c++
//good

//The total number of student of the school is 700 according to the data.
//the data can be seen on http://goodschool.edu.cn.example/student_data.html
const int StudentTotalNumber = 700; 
```

##### 尽可能少用全局变量

```c++
//bad
int x = 5;
void fun(){
	x = x * x;
}
fun();
```

```c++
//good 
int x = 5;
int fun(int x){
	return x * x;
}
x = fun(x);
```

##### 多写纯函数（pure function）

纯函数是函数式编程里一个重要的概念，它具体是什么呢？我从[这里](https://jax.readthedocs.io/en/latest/notebooks/Common_Gotchas_in_JAX.html)看到了一个容易理解的解释，一个纯函数需要做到**all the input data is passed through the function parameters, all the results are output through the function results. A pure function will always return the same result if invoked with the same inputs**就是你喂给它数据，他全部吃掉，整个对外部的影响只有return，运行期间不会干坏事，不会产生其他影响（side effect）的函数。多写纯函数的好处就是方便测试、调试。

```c++
//bad
int add(int a, int b, char c){
	printf("%c",c);//side effect
	int tmp = a + b;
	print("I am stucked!!\n")//side effect
	Sleep(24*60*60*1000);//side effect
	return tmp;
}
```

```c++
//good
int add(int a,int b){
	return a + b;
}
```

##### 对齐、缩进

```c++
//bad
void print_map() {
 for (int i = 0; i < ROW; ++i) {for (int j = 0; j < COL; ++j) {
       print_map_elem(i, j);
}
  printf("\n");}
                          }
```

```c++
//good
void print_map() {
    for (int i = 0; i < ROW; ++i) {
        for (int j = 0; j < COL; ++j) {
            print_map_elem(i, j);
        }
        printf("\n");
    }
}
```

显而易见，对齐缩进后的函数更加容易理解是吧



#### 后记

第一次写博客，不知不觉写了这么多hh。一方面我现在也只是个本科学生，专业知识有所欠缺，另一方面，我最近还有别的事要忙，都是在半夜写博客，精神状态不佳，所以文中难免有错误、纰漏，欢迎大家指正！邮箱：ziyang12315@ruc.edu.cn


