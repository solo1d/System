# 异常的类别

## 异常的类别

**`异常可以分为四类 : 中断(interrupt),  陷阱(trap),  故障(fault),   终止(abort)`**

![](../.gitbook/assets/ping-mu-kuai-zhao-20190831-xia-wu-5.05.53.png)

**`异步异常`是由处理器外部的 I/O 设备中的事件产生的.  `同步异常`是执行一条指令的直接产物.**

### 1.中断

**中断是异步发生的,是来自处理器外部I/O 设备的信号结果.**

**硬件终端不是由任何一条专门的指令造成的, 从这个意义上来说它是移步的. 硬件中断的异常处理程序常常称为 `中断处理程序`.**

* I/O设备通过向处理器芯片的一个引脚发信号，并将异常号放到系统总线上，以触发中断。
* 在当前指令执行完后，处理器注意到中断引脚的电压变化，从系统总线读取异常号，调用适当的中断处理程序。
* 当处理程序完成后，它将控制返回给下一条本来要执行的指令。
*   ![](../.gitbook/assets/ping-mu-kuai-zhao-20190831-xia-wu-6.28.59.png) 
* 剩下的异常类型（陷阱，故障，终止）是**同步发生**，执行当前指令的结果。我们把这类指令叫做**故障指令**（faulting instruction）.

### 2.陷阱和系统调用

* `陷阱`是**有意**的异常，是执行一个指令的结果。也会返回到下一跳本来要执行的指令。
* `陷阱`最重要的用途是在用户程序和内核之间提供一个像过程一样的接口，叫做**系统调用**
  * 用户程序经常需要向内核请求服务。
    * 读文件\(read\)
    * 创建进程\(fork\)
    * 新的程序\(execve\)
    * 终止当前进程\(exit\)
  * 为了运行对这些内核服务的受控访问，处理器提供了一条特殊的`syscall n`的指令
  * 系统调用是运行在内核模式下，而普通调用是用户模式下。
    * 用户模式限制了函数可以执行的指令的类型,而且只能访问与调用函数相同的栈.
    * 内核模式允许系统调用执行特权指令,并访问定义在内核中的栈.
  * 

![](../.gitbook/assets/ping-mu-kuai-zhao-20190831-xia-wu-7.53.31.png)

### 3.故障

* **故障**由错误引起，可能被故障处理程序修正。
  * 如果能被修正，返回引起故障的指令。
  * 否则返回`abort`例程，进行终结。
* 

![](../.gitbook/assets/ping-mu-kuai-zhao-20190831-xia-wu-7.52.07.png)

### 4.终止

* **终止**是不可恢复的致命错误造成的结果，通常是一些硬件错误，比如DRAM和SRAM被损坏。
* 终止处理程序从不将控制返回给应用程序。返回一个`abort`例程。

![](../.gitbook/assets/ping-mu-kuai-zhao-20190831-xia-wu-7.54.43.png)

### Linux/x86-64 系统中的异常

* 有高达256种不同的异常
  * 0~31 由Intel架构师定义的异常，对任何IA32系统都一样。
  * 23~255 对应操作系统定义的中断和陷阱。

![](../.gitbook/assets/ping-mu-kuai-zhao-20190831-xia-wu-7.57.13.png)

#### 1. Linux/IA32 故障和终止

* 除法错误
* 一般保护故障
* 缺页
* 机器检查

#### 2. Linux/IA32 系统调用

![](../.gitbook/assets/ping-mu-kuai-zhao-20190831-xia-wu-7.59.06.png)

* 在IA32系统中，系统调用是通过一条称为`int n`的陷阱指令完成，其中n可能是IA32异常表256个条目中任何一个索引，历史中，系统调用是通过异常128\(0x80\)提供的。
* C程序可用`syscall`函数来直接调用任何系统调用
  * 实际上没必要这么做
  * C库提供了一套方便的**包装函数**。这些包装函数将参数打包到一起，以适当的系统调用号陷入内核，然后将系统调用的返回状态传递回调用函数。
  * 我们将系统调用与他们相关联的包装函数称为**系统级函数**。

> **研究程序如何使用int指令直接调用Linux 系统调用是很有趣的。所有到Linux系统调用的参数都是通过通用寄存器而不是栈传递。**

> 惯例
>
> * **%eax 包含系统调用号**
> * **%ebx,%ecx,%edx,%esi,%edi,%ebp包含六个任意的参数。**
> * **%esp不能使用，进入内核模式后，内核会覆盖它。**

* **系统级函数写的hello world**

  ```text
  #include <unistd.h>
  int main()
  {
      write(1,"hello,world\n",13);   /* 1代表 stdout, 中间是字符串参数, 13是字符串长度*/
      _exit(0);
  }
  ```

* **汇编写的hello world**

  ```text
  .section .data
  string:
          .ascii "hello, world\n"        #在MACos中是 .asciz "hello, world\n"
  string_end:
          .equ len, string_end - string
  .section .text
  .globl main
  main:
          # First, call write(1, "hello, world\n", 13)
          movq $1, %rax        # %rax 包含系统调用编号1      
          movq $1, %rdi        # 第一个 write()系统调用的参数 1 标号
          movq $string, %rsi   # 第二个字符串参数
          movq $len, %rdx      # 第三个 字符串长度 size_t 参数
          syscall              # 进行系统调用 write()

          # Next, call _exit(0)
          movq $60, %rax       # _exit is system call 60     
          movq $0, %rdi        # Arg1: exit status is 0      
          syscall              # Make the system call  
  ```
