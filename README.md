# MiniDecaf 编译实验

## 写在前面的

本仓库是 2022 秋编译原理实验的 fork。

原始仓库参见 <https://github.com/decaf-lang/minidecaf-tutorial>；原始页面参见 <https://decaf-lang.github.io/minidecaf-tutorial/>。

---

下面是原始介绍。

## 实验概述
MiniDecaf [^1] 是一个 C 的子集，去掉了`include/define`等预处理指令，多文件编译支持，以及结构体/指针等语言特性。 本学期的编译实验要求同学们通过多次“思考-实现-重新设计”的过程，一步步实现从简单到复杂的 MiniDecaf 语言的完整编译器，能够把 MiniDecaf 代码编译到 RISC-V 汇编代码。进而深入理解编译原理和相关概念，同时具备基本的编译技术开发能力，能够解决编译技术问题。MiniDecaf 编译实验分为多个 stage，每个 stage 包含多个 step，共包含 12 个 step。**每个 step 大家都会完成一个可以运行的编译器**，把不同的 MiniDecaf 程序代码编译成 RISC-V 汇编代码，可以在 QEMU/SPIKE 硬件模拟器上执行。随着实验内容一步步推进，MiniDecaf 语言将从简单变得复杂。每个步骤都会增加部分语言特性，以及支持相关语言特性的编译器结构或程序（如符号表、数据流分析方法、寄存器分配方法等）。下面是采用 MiniDecaf 语言实现的快速排序程序，与 C 语言相同。

```c
int qsort(int a[], int l, int r) {
    int i = l; int j = r; int p = a[(l+r)/2];
    while (i <= j) {
        while (a[i] < p) i = i + 1;        
        while (a[j] > p) j = j - 1;        
        if (i > j) break;        
        int u = a[i]; a[i] = a[j]; a[j] = u;        
        i = i + 1;        
        j = j - 1;    
    }    
    if (i < r) qsort(a, i, r);    
    if (j > l) qsort(a, l, j);    
    return 0;
}
```

2022 年秋季学期基本沿用了 2021 年秋季学期《编译原理》课程的语法规范。为了贴合课程教学内容，提升训练效果，课程组设计了比较完善的编译器框架，包括词法分析、语法分析、语义分析、中间代码生成、数据流分析、寄存器分配、目标平台汇编代码生成等步骤，并采用 C++ 与 Python 两种语言实现。每个 step 同学们都会面对一个完整的编译器流程，但不必担心，实验开始的几个 step 涉及的编译器框架知识都比较初级，随着课程实验的深入，将会循序渐进地引入各个编译器功能模块，并通过文档对相关技术进行分析介绍，便于同学们实现相关编译功能模块。

## 实验起点和基本要求

本次实验一共设置 12 个步骤（其中 step0 为环境配置，主要是 RISC-V 工具链和硬件模拟器的的安装与使用，以及学会使用助教提供的自动测试脚本）。后续的 step1-11 我们将由易到难完成 MiniDecaf 语言的所有特性，由于编译器的边界情况很多，因此你**只需通过我们提供的正例与负例即可**。

我们以 stage 组织实验，各个 stage 组织如下：

<ol start="0">
  <li>
    第一个编译器（step0-step1）。我们给的实验框架可以通过所有测试用例，你需要做的事情为跟着文档阅读学习实验框架代码。请各位同学注意，stage0 尤为重要，掌握好实验框架是高质量和高效率完成后续实验的保证。
  </li>
  <li>
    常量表达式（step2-step4）。在这个 stage 中你将实现常量操作（加减乘除模等）。
  </li>
  <li>
    变量和语句（step5-step6）。在这个 stage 中你将第一次支持变量声明与赋值，以及条件跳转语句。
  </li>
  <li>
    块语句和循环（step7-step8）。在这个 stage 中你将支持块语句，所谓块语句，就是多个语句组成一个块，每个块都是一个作用域。作为一种特殊的块语句，你也将实现循环操作。
  </li>
  <li>
    全局变量和函数（step9-step10）。在这个 stage 中你将支持声明全局变量，并且支持函数的声明和调用。
  </li>
  <li>
    数组（step11-step12）。在这个 stage 中，你将支持数组，包括全局数组和局部数组。
  </li>
</ol>


同时，为了帮助大家通过实验学习语法分析，我们单独设置了一个**手工自顶向下语法分析的小实验**，需要大家手动实现一个支持 step1 - step6 语法规范的手工 parser。

> 设置这个实验的目的是为了帮助大家通过实验学习了解语法分析，parser generator（如 Bison）掩盖了很多语法分析的实现细节。

其中，stage0 为环境配置和框架学习，无需进行编程，不计入成绩。
stage1 - stage3 和手工语法分析器为 4 个基础关卡，你需要通过它们以拿到一定的分数（40%，每个关卡 10%）。
stage4 - stage5 为升级关卡，如果你学有余力，完成它们可以减少期末考试在总评中所占的比重（完成一个关卡，替代占总评 10% 的期末考试成绩）。

我们以 step 组织文档，每个 step 的文档都将以如下形式组织：首先我们会介绍当前 step 需要用到的知识点，其次我们会以一个当前 step 具有代表性的例子介绍它的整个编译流程。在之前 step 中已经介绍的知识点，我们会略过，新的知识点和技术会被详细介绍。

在 2022 年秋季学期，我们通过[问答墙](https://docs.qq.com/doc/DY1hZWFV0T0N0VWph)来集中解决大家在环境配置及完成实验中遇到问题。如果你遇到了任何问题，都可以在[问答墙](https://docs.qq.com/doc/DY1hZWFV0T0N0VWph)中检索；如果尚未有其他人提问，也欢迎提问，助教会尽快回复的。

## 实验提交

大家在网络学堂提交 **git.tsinghua.edu.cn** 的帐号名后，助教会给每个人建立一个私有的仓库，URL 为 https://git.tsinghua.edu.cn/compiler-22/minidecaf-你的学号 ，将作业提交到那个仓库即可。
每个 stage 会对应于一个 branch，当切换到一个新的 branch 上实现时，你可以用 `git merge` 来合并前一个 branch 所作的修改。

本学期我们使用清华大学代码托管服务（git.tsinghua）的 CI（持续集成）来**测试**大家的代码实现及**提交实验报告**。
`.gitlab-ci.yml` 中描述了如何运行 CI，你**不允许**修改此文件；
`prepare.sh` 是在测试前会运行的准备脚本，包括安装所需的依赖（python）及编译（c++），如果你想添加新的依赖或者修改编译流程，请修改此文件。
在 CI 中会检查是否通过所有测例及是否有提交报告，只有通过所有测例且提交报告，才会被视为通过 CI。

我们只接受 pdf 格式的实验报告，其提交方式是放在仓库的 `./reports/<branch-name>.pdf`，比如 stage 1 的实验报告需要放在 `stage-1` 这个 branch 下的 `./reports/stage-1.pdf`。
实验报告中需要包括：
* 你的学号姓名
* 简要叙述，为了完成这个 stage 你做了哪些工作（即你的实验内容）
* 指导书上的思考题
* 如果你复用借鉴了参考代码或其他资源，请明确写出你借鉴了哪些内容。*并且，即使你声明了代码借鉴，你也需要自己独立认真完成实验。*
* 如有代码交给其他同学参考，也必须在报告中声明，告知给哪些同学拷贝过代码（包括可能通过间接渠道传播给其他同学）。

## 评分标准

对于每个阶段（stage）：
* 80% 的成绩是自动化测试的结果，你可以直接在 **git.tsinghua** 的 CI 测试中看到。
* 20% 的成绩是实验报告，其中对实验内容的描述占 10%，对思考题的回答占 10%。

评分会以 2023 年 1 月 1 日 24:00 前每个 stage 的 branch 最后一次触发的 CI 及触发此次 CI 的 commit 里的实验报告为准，详见[补交政策](#补交政策)。

如果你认为成绩有问题，请及时与助教联系。

## 补交政策

* 假设 a 日 24:00 是某个 stage 的截止时间；
* a + k (k <= 9) 日 24:00 前补交，此 stage 得分乘以 1 - k / 20；
* a + 9 日 24:00 及之后，至 1 月 1 日（第十六周周日）24:00 前补交，此 stage 得分乘以 1 / 2；
* 提交/补交时间是 1 月 1 日 24:00 前这个 stage 的 branch 最后一次触发 CI 的时间；
    * 更具体来讲是创建 pipeline 的时间，而不是 pipeline 更新的时间，这几乎等于你 push 到远端仓库的时间。并且，你也可以随意地 retry 反复运行 CI，这不会对你的提交/补交产生影响。
    * 如果在多次 retry 中你的代码会得到不一致的结果，请联系助教。
* 1 月 1 日 24:00 及之后不接受补交，此 stage 的得分为 0。

## 学术规范

由于实验有一定难度，同学之间相互学习和指导是提倡的。
对于其他同学的代码（包括实验报告中思考题的回答），可以参考，但禁止直接拷贝。
如有代码交给其他同学参考，必须在报告中声明，告知给哪些同学拷贝过代码（包括可能通过间接渠道传播给其他同学）。
请所有同学不要将自己的代码托管至任何公开的仓库上（如 GitHub），托管至私有仓库的请不要给其他同学任何访问权限。
我们将会对所有同学的代码作相似度检查，如发现有代码雷同的情形，拷贝者和被拷贝者将会得到同样的处罚，除非被拷贝的同学提交时已做过声明。

代码雷同情节严重的，课程组有权上报至院系和学校，并按照相关规定严肃处理。

## 相关资源

- [实验指导书（首页有实验报告提交要求）](https://decaf-lang.github.io/minidecaf-tutorial/)
- [实验指导书勘误表](https://decaf-lang.github.io/minidecaf-tutorial/docs/step0/errate.html)
- [课程问答墙](https://docs.qq.com/doc/DY1hZWFV0T0N0VWph)
- [实验思路指导与问答墙](https://docs.qq.com/doc/DY05QVmJFcGNWcllo)
- [实验框架介绍视频](https://cloud.tsinghua.edu.cn/d/d1d80ebd16a44c179d44/)
- [实验环境虚拟机下载链接（使用方法参见网络学堂公告）](https://cloud.tsinghua.edu.cn/d/5281023e65fd4032a69b/)

## 参考资料
- [Writing a C Compiler: by Nora Sandler](https://norasandler.com/2017/11/29/Write-a-Compiler.html)
  - [nqcc](https://github.com/nlsandler/nqcc)
- [http://scheme2006.cs.uchicago.edu/11-ghuloum.pdf](http://scheme2006.cs.uchicago.edu/11-ghuloum.pdf)

## 备注
[^1]: 关于名字由来，由于往年的实验叫 Decaf，我们在新的且更简单的语言规范下复用了 Decaf 的编译器框架，所以今年的实验就叫 MiniDecaf 了。
