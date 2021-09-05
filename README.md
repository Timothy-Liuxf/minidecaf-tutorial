# C-- 编译实验

## 实验概述
C-- [^1] 是一个 C 的子集，去掉了`include/define`等预处理指令，多文件编译支持，以及`struct/指针`等语言特性。 本学期的编译实验要求同学们通过多次“思考-实现-重新设计”的过程，一步步实现从简单到复杂的 C-- 语言的完整编译器，能够把 C-- 代码编译到 RISC-V 汇编代码。进而深入理解编译原理和相关概念，同时具备基本的编译技术开发能力，能够解决编译技术问题。C-- 编译实验分为多个 stage，每个 stage 包含多个 step，共包含 11 个 step。**每个 step 大家都会完成一个可以运行的编译器**，把不同的 C-- 程序代码编译成 RISC-V 汇编代码，可以在 QEMU/SPIKE 硬件模拟器上执行。随着实验内容一步步推进，C-- 语言将从简单变得复杂。每个步骤都会增加部分语言特性，以及支持相关语言特性的编译器结构或程序（如符号表、数据流分析方法、寄存器分配方法等）。下面是采用 C-- 语言实现的快速排序程序，与 C 语言相同。为了简化实现，C-- 不支持以数组作为函数参数，因此快速排序的数组以全局数组的形式给出：

```c
int a[1000];
int qsort(int l, int r) {
    int i = l; int j = r; int p = a[(l+r)/2];
    while (i <= j) {
        while (a[i] < p) i = i + 1;        
        while (a[j] > p) j = j - 1;        
        if (i > j) break;        
        int u = a[i]; a[i] = a[j]; a[j] = u;        
        i = i + 1;        
        j = j - 1;    
    }    
    if (i < r) qsort(i, r);    
    if (j > l) qsort(l, j);    
    return 0;
}
```

2021 年秋季学期基本沿用了 2020 年秋季学期《编译原理》课程的语法规范，为降低实验难度，进一步去掉了指针等语言特性。和 2020 年秋季学期课程实验所不同的是，为了贴合课程教学内容，提升训练效果，课程组设计了比较完善的编译器框架，包括词法分析、语法分析、语义分析、中间代码生成、数据流分析、寄存器分配、目标平台汇编代码生成等步骤，并采用 C++ 与 Python 两种语言实现。每个 step 同学们都会面对一个完整的编译器流程，但不必担心，实验开始的几个 step 涉及的编译器框架知识都比较初级，随着课程实验的深入，将会循序渐进地引入各个编译器功能模块，并通过文档对相关技术进行分析介绍，便于同学们实现相关编译功能模块。

## 实验起点和基本要求

本次实验一共设置 12 个步骤（其中 step0 为环境配置，主要是 RISC-V 工具链和硬件模拟器的的安装与使用，以及学会使用助教提供的自动测试脚本）。后续的 step1-11 我们将由易到难完成 C-- 语言的所有特性，由于编译器的边界情况很多，因此你**只需通过我们提供的正例与负例即可**。

我们以 stage 组织和发布实验，各个 stage 组织如下：

* 第一个编译器（step1）。我们给的实验框架可以通过所有测试用例，你需要做的事情为跟着文档阅读学习实验框架代码。请各位同学注意，stage1 尤为重要，掌握好实验框架是高质量和高效率完成后续实验的保证。
* 常量表达式（step2-step4）。在这个 stage 中你将实现常量操作（加减乘除模等）。
* 变量和语句（step5-step6）。在这个 stage 中你将第一次支持变量声明与赋值，以及条件跳转语句。
* 块语句和循环（step7-step8）。在这个 stage 中你将支持块语句，所谓块语句，就是多个语句组成一个块，每个块都是一个作用域。作为一种特殊的块语句，你也将实现循环操作。
* 全局变量和函数（step9-step10）。在这个 stage 中你将支持声明全局变量，并且支持函数的声明和调用。
* 数组（step11）。在这个 stage 中，你将支持数组，包括全局数组和局部数组。

其中，stage1-stage4 为基础关卡，你需要通过它们以拿到一定的分数。stage5-stage6 为升级关卡，如果你学有余力，完成它们可以减少期末考试在总评中所占的比重。

> TODO：需要根据教学安排更新比重。

关于文档，我们以 step 组织文档，每个 step 的文档都将以如下形式组织：首先我们会介绍当前 step 需要用到的知识点，其次我们会以一个当前 step 具有代表性的例子介绍它的整个编译流程。在之前 step 中已经介绍的知识点，我们会略过，新的知识点和技术会被详细介绍。


## 实验提交
> TODO: 下面的是 2020 年的实验安排，需更新到 2021 年的版本。

你需要使用 **git** 对你的实验做版本维护，然后提交到 **git.tsinghua.edu.cn**。
大家在网络学堂提交帐号名后，助教给每个人会建立一个私有的仓库，作业提交到那个仓库即可。
关于 git 使用，大家也可以在网上查找资料。

每次除了实验代码，你还需要提交 **实验报告**，其中包括
* 你的学号姓名
* 简要叙述，为了完成这个 step 你做了哪些工作（即你的实验内容）
* 指导书上的思考题
* 如果你复用借鉴了参考代码或其他资源，请明确写出你借鉴了哪些内容。*并且，即使你声明了代码借鉴，你也需要自己独立认真完成实验。*

**晚交扣分规则** 是：
* 晚交 n 天，则扣除 n/15 的分数，扣完为止。例如，晚交三天，那你得分就要折算 80%。


## 备注
[^1]: 关于名字由来，我们的语法规范基本是 C 语言的子集，既然是 C 的子集，那干脆就叫 C-- 好了。
