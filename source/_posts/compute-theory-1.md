---
title: 计算理论导论复习知识点总结1-正则语言
date: 2023-06-21 11:25:59
tags: 计算理论导论
math: true
---

# String and Language
> DEFINITION
>
> A string is a finite sequence of symbols in $\Sigma$
>
> A language is a set of strings (finite or infinite)
>
> The empty String $\epsilon$ is the string of length 0
>
> The empty language $\emptyset$ is the set with no strings
>
> 

# Finate Automaton
> DEFINITION 1.5
> 
> A finite automaton is a 5-tuple$(Q, \Sigma, \delta, q_0, F)$ where
> 
> 1.Q is a finite set called the states
>
> 2.$\Sigma$ is a finite set called the alphabet,
>
> 3.$\delta : Q \times \Sigma \rightarrow Q$ is the transition function
>
> 4.$q_0 \in Q$ is the start state, and
>
> 5.$F \subseteq Q$ is the set of accept states
>

An example

![_20230621004959.jpeg](http://server.killuayz.top:8089/images/2023/06/21/_20230621004959.jpeg)

The finite automaton $M_1$

We can descrube $M_1$ formally by writing $M_1 = (Q, \Sigma, \delta, q_1, F)$, where 

1.$Q={q_1, q_2, q_3}$

2.$\Sigma = {0,1}$

3.$\delta$ is described as 

||0|1|
|---|---|---|
|$q_1$ | $q_1$ | $q_2$|
|$q_2$ | $q_3$ | $q_2$|
|$q_3$ | $q_2$ | $q_2$|


4.q_1$ is the start state, and

5.F = {$q_2$}




# More Examples About Finite Automaton

# Formal Definition of Computation
> DEFINITION
>
> Let M = $(Q,\Sigma,\delta,q_0,F)$ be a finite automaton and let $w = w_1w_2w_3...w_n$ be a string where each $w_i$ is a member of the alphabet $\Sigma$. Then M `accepts` w if a sequence of states $r_0, r_1, ..., r_n$ in Q exists with three conditions:
>
> 1.$r_0 = q_0$
>
> 2.$\delta(r_i, w_{i+1}) = r_{i+1}, for i = 0,...,n-1, and$
>
> 3.$r_n \in F$

1->机器从起始状态开始运行

2->机器的运行遵守了状态转移函数

3->机器停止在了接收状态

> DEFINITION
>
> If A is the set of all strings that machine M accepts, we say that `A is the language of machine M and write L(M) = A`. We say that `M recognizes A` or that `M accepts A`.


> DEFINITION 1.16
>
> A language is called a `regular language` if some finite automaton recognizes it.

正则语言和有限自动机能力是相等的

# Designing Finite Automata

# The Regular Operations

> DEFINITION 1.23
>
> Let A and B be languages. We define the regular operations `union`, `concatenation`, and `star` as follow
>
> Union: $A \cup B$ = {$x | x \in A$ or $x \in B$}
>
> Concatenation: $A \circ B$ = {$xy|x\in A$ and $y \in B$}
>
> Star: A* = {$x_1x_2...x_k | k \ge 0$ and each $x_i \in A$}


> THEOREM 1.25
>
> The class of regular languages is `closed` under the `union` operation

证明：

假设我们有正则语言 $A_1$ 和 $A_2$，并且 $M_1$ 接受 $A_1$, $M_2$ 接受 $A_2$。证明的主要思路就是构造一个有限状态自动机 $M$ 去同时模拟 $M_1$ 和 $M_2$ 的行为，具体证明如下：

step1：假设条件

假设$M_1$ 识别 $A_1$, 其中$M_1 = (Q, \Sigma, \delta_1, q_1 F_1)$，$M_2$ 识别 $A_2$, 其中$M_2 = (Q, \Sigma, \delta_2, q_2, F_2)$ 

我们接下来构造M识别$A_1 \cup A_2$, 其中$M = (Q, \Sigma, \delta, q_0, F)$

构造的其实就是根据$M_1$和$M_2$写$Q, \Sigma, \delta, q_0, F$表达式的过程

step2：构造M

$Q=${ $(r_1, r_2) | r_1 \in Q_1 \space and \space  r_2 \in Q_2 $} 

$\Sigma = \Sigma_1 \cup \Sigma_2$ 

$\delta((r_1, r_2), a) = (\delta_1(r_1,a),\delta_2(r_2,a))$ 

$q_0 = (q_1,q_2)$

$F = ${ $(r_1, r_2) | r_1 \in F_1 \space or \space r_2 \in F_2 $ }


综上，M识别$A_1 \cup A_2$，故正则语言在union运算下是封闭的

也可以用NFA证明如下：

![1687428151158-screenshot.png](http://server.killuayz.top:8089/images/2023/06/22/1687428151158-screenshot.png)


> THEOREM 1.26
>
> The class of regular languages is closed under the concatenation operation
>
> In other words, if $A_1$ and $A_2$ are regular languages then so is $A_1 \circ A_2$

可以用NFA证明如下：

![1687428161257-screenshot.png](http://server.killuayz.top:8089/images/2023/06/22/1687428161257-screenshot.png)


> THEOREM 1.49
> 
> The class of regular languages is closed under the star operation

可以用NFA证明如下：

![1687428177663-screenshot.png](http://server.killuayz.top:8089/images/2023/06/22/1687428177663-screenshot.png)


> DEFINITION 1.37
> 
> A nondeterministic finite automaton is a 5-tuple $(Q,\Sigma,\delta,q_0,F)$
> 
> where
> 
> 1.Q is a finite set of states,
> 
> 2.$\Sigma$ is a finite alphabet
> 
> 3.$\delta : Q \times \Sigma_{\epsilon} \rightarrow P(Q)$ is the transition function,
> 
> 4.$q_0 \in Q$ is the start state, and
> 
> 5.$F \subseteq Q$ is the set of accept states 
> 

> THEOREM 1.39
> 
> Every nondeterministic finite automaton has an equivalent deterministic finite automaton

> COROLLARY 1.40
> 
> A language is regular if and only if some nondeterministic finite automaton recogizes it.


