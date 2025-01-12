# step5 实验指导

## 词法语法分析
如果你使用工具完成词法语法分析，修改你的语法规范以满足要求，自行修改词法规范，剩下的交给工具即可。

语义检查部分，我们需要检查是否（一）使用了未声明的变量、（二）重复声明变量。
为此，我们在生成 IR 的 Visitor 遍历 AST 时，维护当前已经声明了哪些变量。
遇到变量声明（`declaration`）和使用（`primary` 和 `assignment`）时检查即可。
可以把这个要求和后面提到的符号表结合，放到 IR 生成去做。

如果你是手写分析，参见[这里](./manual-parser.md)。

## IR 生成
为了完成 step5 的 IR 生成，我们需要确定 IR 的栈帧布局，请看 [这里](./stackframe.md)。

[step1](../lab1/ir.md)提到，局部变量保存在栈上，这个栈和IR中的运算栈并不是一个概念。
前者指的是汇编中一片可以增长的物理空间（可以称之为物理栈），后者是一个逻辑上的概念。
这二者的关系是，我们之前一直在使用物理栈的空间来实现运算栈。
局部变量存放在物理栈上，运算栈也在物理栈上，它们的内存空间很接近，但是二者是互不干扰的，对运算栈的压栈弹栈操作，不能影响到局部变量的值。
因此我们需要加入访问物理栈内部的 `load`/`store`，以及生成栈上地址的 `frameaddr` 指令。

此外我们还加入一个`pop`指令，这与上面的讨论没有什么关系，是用于别的用途：

| 指令 | 参数 | 含义 | IR 栈大小变化 |
| --- | --- | --- | --- |
| `frameaddr` | 一个非负整数常数 `k` | 把当前栈帧底下开始第 `k` 个元素的地址压入栈中 | 增加 1 |
| `load` | 无参数 | 将栈顶弹出，作为地址 [^1] 然后加载该地址的元素（`int`），把加载到的值压入栈中 | 不变 |
| `store` | 无参数 | 弹出栈顶作为地址，读取新栈顶作为值，将值写入地址开始的 `int` | 减少 1 |
| `pop` | 无参数 | 弹出栈顶，忽略得到的值 | 减少 1 |

IR 生成还是 Visitor 遍历，并且
* 遇到读取变量 `primary: Identifier` 的时候，查符号表确定变量是第几个，然后生成 `frameaddr` 和 `load`。
  - 如果查不到同名变量，应当报错：变量未定义
* 遇到变量赋值的时候，先生成等号右手边的 IR，同上对等号左手边查符号表，生成 `frameaddr` 和 `store`。
> 注意赋值表达式是有值的，执行完它的 IR 后栈顶还保留着赋值表达式的值。这就是为什么 `store` 只弹栈一次。
* 遇到表达式语句时，生成完表达式的 IR 以后记得再生成一个 `pop`，保证栈帧要满足的第 1. 条性质（[这里](./stackframe.md)有说）
* 遇到声明时，除了记录新变量，还要初始化变量。
> 为了计算 prologue 中分配栈帧的大小，IR 除了一个指令列表，还要包含一个信息：局部变量的个数。
* `main` 有多条语句了，它的 IR 是其中语句的 IR 顺序拼接。

> 例如 `int main(){int a=2; a=a+3; return a;}`，显然 `a` 是第 0 个变量。
> 那它的 IR 指令序列是（每行对应一条语句）：
```
push 2 ; frameaddr 0 ; store ; pop ;
frameaddr 0 ; load ; push 3 ; add ; frameaddr 0 ; store ; pop ;
frameaddr 0 ; load ; ret ;
```

## 汇编生成
IR 指令到汇编的对应仍然很简单，如下表。

| IR       | 汇编                                                |
| ---      | ---                                                 |
| `frameaddr k` | `addi sp, sp, -4  ;  addi t1, fp, -12-4*k  ;  sw t1, 0(sp)` |
| `load`    | `lw t1, 0(sp)  ;  lw t1, 0(t1)  ;  sw t1, 0(sp)` |
| `store` | `lw t1, 4(sp)  ;  lw t2, 0(sp)  ;  addi sp, sp, 4  ;  sw t1, 0(t2)` |
| `pop` | `addi sp, sp, 4` |

但除了把 IR 一条一条替换成汇编，step5 还需要生成 prologue 和 epilogue，并且 `ret` 也要修改了，
参见[栈帧文档](./stackframe.md)。

| IR       | 汇编                                                |
| ---      | ---                                                 |
| `ret` | `lw a0, 0(sp)  ;  addi sp, sp, 4  ;  j FUNCNAME_epilogue` |

另外我们还要求 main 默认返回 0：
> **5.4** 执行完 main 函数但没有通过 return 结束时，返回值默认为 0。

显然，如果 main 是通过 return 结束的，按照上面的修改一定是跳到 `main_epilogue`，否则是顺序执行到 `main_epilogue` 的。
因此我们在 `main_epilogue` 之前，所有语句之后，加上 `push 0` 的汇编即可，表示默认返回 0。

# 思考题

1. 描述程序运行过程中函数栈帧的构成，分成哪几个部分？每个部分所用空间最少是多少？
2. 有些语言允许在同一个作用域中多次定义同名的变量，例如这是一段合法的 Rust 代码（你不需要精确了解它的含义，大致理解即可）：

```Rust
fn main() {
  let a = 0;
  let a = f(a);
  let a = g(a);
}
```

其中`f(a)`中的`a`是上一行的`let a = 0;`定义的，`g(a)`中的`a`是上一行的`let a = f(a);`。

如果 MiniDecaf 也允许多次定义同名变量，并规定新的定义会覆盖之前的同名定义，请问在你的实现中，需要对定义变量和查找变量的逻辑做怎样的修改？

# 备注
[^1]: 我们规定 `load` 的地址必须对齐到 4 字节，生成 IR 时需要保证。`store` 也是。
