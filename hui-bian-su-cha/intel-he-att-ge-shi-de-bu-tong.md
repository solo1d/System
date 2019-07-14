# intel 和 ATT 格式的不同

### intel 和 ATT 格式的不同

**编译出 intel 格式汇编的命令  :  `gcc   -S  -masm=intel  mstore.c`**

* **intel 代码省略了指示大小的后辍.**  指令是 `push` 和 `mov`,  而不是 `pushq` 和 `movq`.
* **intel 代码省略了寄存器名字前面的 '`%` ' 符号**, 用的是  `rbx` ,而不是 `%rbx`
* **intel 代码用不同的方式来描述内存中的位置**.  例如是 '`QWORD PIR [rbx]` ' 而不是 '`(%rbx)`' .
* **在带有多个操作数的情况下, 列出操作数的顺序相反.**  `mov  a ,b`  是a被赋值, `movp a,b` 是b被赋值.

