# step4 实验指导

本实验指导使用的例子为：

```C
1<2
```

## 词法语法分析

### Python 框架

本 step 中引入的运算均为二元运算，在 step3 中引入的二元运算节点中进行修改即可。

# C++ 框架

新引入的每个运算都有一个对应的语法树节点.

| 节点 | 成员 | 含义 |
| --- | --- | --- |
| `LesExpr` | 左操作数 `e1`，右操作数 `e2` | 小于 |
| `GrtExpr` | 左操作数 `e1`，右操作数 `e2` | 大于 |
| `LeqExpr` | 左操作数 `e1`，右操作数 `e2` | 小于等于 |
| `GeqExpr` | 左操作数 `e1`，右操作数 `e2` | 大于等于 |
| `EquExpr` | 左操作数 `e1`，右操作数 `e2` | 等于 |
| `NeqExpr` | 左操作数 `e1`，右操作数 `e2` | 不等于 |
| `AndExpr` | 左操作数 `e1`，右操作数 `e2` | 逻辑与 |
| `OrExpr` | 左操作数 `e1`，右操作数 `e2` | 逻辑或 |

## 语义分析

同 Step2。

## 中间代码生成
针对小于符号，我们显然需要设计一条中间代码指令来表示它，给出的参考定义如下：

> 请注意，TAC 指令的名称只要在你的实现中是一致的即可，并不一定要和文档一致。

| 指令 | 参数 | 含义 |
| --- | --- | --- |
| `LT` | `T0,T1` | 给出 `T0<T1`结果，成立为1，失败为0 |

> 需要特别注意的是，在 C 语言中，逻辑运算符 || 和 && 有短路现象，我们的实现中不要求大家考虑它们的短路性质。

因此，测例可以翻译成如下的中间代码：

```assembly
_T0 = 1
_T1 = 2
_T2 = LT _T0, _T1
```

## 目标代码生成

step4 目标代码生成步骤的关键点与 step3 相同，针对中间代码指令，选择合适的 RISC-V 指令来完成翻译工作。

```assembly
li t0, 1
li t1, 2
slt t2, t0, t1
```

逻辑表达式会麻烦一点，因为 gcc 可能会用跳转来实现`&&`和`||`，比较难以理解，所以下面直接给出 `land` 和 `lor` 对应的不使用跳转的汇编。

| IR       | 汇编                                                |
| ---      | ---                                                 |
| `lor` | `or t3,t1,t2  ;  snez t3,t3` |
| `land` | `snez d, s1; sub d, zero, d; and d, d, s2; snez d, d;` |

> 注意 RISC-V 汇编中的 `and` 和 `or` 指令都是位运算指令，不是逻辑运算指令。

# 思考题

1. 在 MiniDecaf 中，我们对于短路求值未做要求，但在包括 C 语言的大多数流行的语言中，短路求值都是被支持的。为何这一特性广受欢迎？你认为短路求值这一特性会给程序员带来怎样的好处？

# 总结
本步骤中其他运算符的实现逻辑和方法与小于符号类似，可以参考小于符号的实现方法设计实现其他逻辑运算符。

恭喜你！到目前为止，你已经成功实现了一个基于 MiniDecaf 语言的计算器，可以完成基本的数学运算和逻辑比较运算了，成就感满满！然而，目前你的计算器还只能支持常量计算，这大大降低了计算器的使用体验，因此，在下一个 Stage，我们将一起实现对变量以及分支语句的支持。无论如何，当前的任务已经完成，好好休息一下吧☕️