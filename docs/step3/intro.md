# 实验指导 step3：加减乘除模
step3 我们要增加的是：加 `+`、减 `-`、乘 `*`、整除 `/`、模 `%` 以及括号 `(` `)`。

语法上我们继续修改 `expression`，变成

<pre id='vimCodeElement'><code></code><div class="changed">
<span class="SpecRuleStart">expression</span>
<span class="SpecRuleIndicator">    :</span> <span class="SpecRule">additive</span>

<span class="SpecRuleStart">additive</span>
<span class="SpecRuleIndicator">    :</span> <span class="SpecRule">multiplicative</span>
<span class="SpecRuleIndicator">    |</span> <span class="SpecRule">additive</span> <span class="SpecOperator">(</span><span class="SpecToken">'+'</span><span class="SpecOperator">|</span><span class="SpecToken">'-'</span><span class="SpecOperator">)</span> <span class="SpecRule">multiplicative</span>

<span class="SpecRuleStart">multiplicative</span>
<span class="SpecRuleIndicator">    :</span> <span class="SpecRule">unary</span>
<span class="SpecRuleIndicator">    |</span> <span class="SpecRule">multiplicative</span> <span class="SpecOperator">(</span><span class="SpecToken">'*'</span><span class="SpecOperator">|</span><span class="SpecToken">'/'</span><span class="SpecOperator">|</span><span class="SpecToken">'%'</span><span class="SpecOperator">)</span> <span class="SpecRule">unary</span>

<span class="SpecRuleStart">unary</span>
<span class="SpecRuleIndicator">    :</span> <span class="SpecRule">primary</span>
<span class="SpecRuleIndicator">    |</span> <span class="SpecOperator">(</span><span class="SpecToken">'-'</span><span class="SpecOperator">|</span><span class="SpecToken">'~'</span><span class="SpecOperator">|</span><span class="SpecToken">'!'</span><span class="SpecOperator">)</span> <span class="SpecRule">unary</span>

<span class="SpecRuleStart">primary</span>
<span class="SpecRuleIndicator">    :</span> <span class="SpecToken">Integer</span>
<span class="SpecRuleIndicator">    |</span> <span class="SpecToken">'('</span> <span class="SpecRule">expression</span> <span class="SpecToken">')'</span>
</div></pre>

新特性的语义、优先级、结合性和 C 以及常识相同，例如 `1+2*(4/2+1) == 7`。

我们这种表达式语法写法可能比较繁琐，但它有几个好处：
1. 和 [C17 标准草案](../../REFERENCE.md)保持一致
2. 把优先级和结合性信息直接编码入语法里，见[优先级和结合性](./precedence.md)一节。

你需要：
1. 改进你的编译器，支持本节引入的新特性，通过相关测试。
2. 完成实验报告（具体要求请看实验指导书的首页）。实验报告中需要包括：
  * 你的学号姓名
  * 简要叙述，为了完成这个 stage 你做了哪些工作（即你的实验内容）
  * 指导书上的思考题
  * 如果你复用借鉴了参考代码或其他资源，请明确写出你借鉴了哪些内容。*并且，即使你声明了代码借鉴，你也需要自己独立认真完成实验。*
  * 如有代码交给其他同学参考，也必须在报告中声明，告知给哪些同学拷贝过代码（包括可能通过间接渠道传播给其他同学）。