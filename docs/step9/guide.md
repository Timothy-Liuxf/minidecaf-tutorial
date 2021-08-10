# step9 实验指导

## 词法语法分析
如果你使用工具完成词法语法分析，修改你的语法规范以满足要求，自行修改词法规范，剩下的交给工具即可。

如果你是手写分析，参见[这里](./manual-parser.md)。

## 名称解析
类似符号表，我们需要一张表维护函数的信息。
当然，函数不会重名，所以不用解析名称。
这张表主要目的是记录函数本身的信息，方便语义检查。

就 step9 而言，这个信息包括
1. 参数个数（step12 开始还需要记录参数和返回值类型）。对应的语义检查：9.4
2. 是否已经有定义，还是只有声明。对应的语义检查：9.2

## IR 生成
step9 之前因为只有一个函数，所以一个 MiniDecaf 程序的 IR 就只是一个指令序列。
现在有了函数了，一个 MiniDecaf 程序的 IR 应当包含一系列 **IR 函数**，源代码中每个函数都对应一个 IR 函数。
而一个 **IR 函数** 需要包含
1. 函数自身的信息：函数名、需要 prologue 中分配的“栈帧”大小 [^3] 等；
2. 函数体对应的 IR 指令序列。

对于函数声明和定义，IR 生成的 Visitor 遍历 AST 时，
* 函数声明结点：没有函数体，无须生成任何 IR
* 函数定义结点：继续遍历子结点，拿到上面的两种信息，然后创建一个 IR 函数

函数调用是一个比较复杂的操作，见 [这里](./calling.md)。
为了支持它，我们需要引入 `call` 指令，并且修改 `ret` 指令让它不要把栈顶返回值弹出。

| 指令 | 参数 | 含义 | IR 栈大小变化 |
| --- | --- | --- | --- |
| `call` | 一个字符串表示函数名 | 调用作为参数的函数[^1]，调用完后栈顶是 callee 的返回值 | 增加 1[^2] |
| `ret` | 无参数 | （返回值已经在栈顶了）终止当前函数执行，返回 caller | 不变 |

## 汇编生成
`ret` 的汇编不变，`call` 的如下表。

| IR       | 汇编                                                |
| ---      | ---                                                 |
| `call FUNC` | `call FUNC`，然后有几个参数就执行几次 `pop`，然后 `addi sp, sp, -4  ;  sw a0, 0(sp)` |

我们已经在 IR 处理了传参，所以汇编生成时不用再考虑传参。

如果你采用非标准的调用约定，`prologue` 和 `epilogue` 也不用改，也不用处理 caller-save 寄存器。
否则你可能还需要增加 caller/callee-save 寄存器保存与恢复的代码。

# 总结
引入了概念 **调用约定**，并且描述了栈帧的变化。

# 思考题
1. MiniDecaf 的函数调用时参数求值的顺序是未定义行为。试写出一段 MiniDecaf 代码，使得不同的参数求值顺序会导致不同的返回结果。

# 备注
[^1]: `call` 指令不包含准备参数。
[^2]: `call` 的变化是指，整个 callee 执行完成返回 `call` 指令后，IR 栈大小相对执行 `call` 前的大小变化。
[^3]: 这个栈帧加了引号，因为它没有包含运算栈