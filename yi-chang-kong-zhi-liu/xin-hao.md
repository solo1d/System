# 信号

## 信号

研究一种更高层次的软件形式的**异常**， 也是一种**软件中断**，称为`Unix`信号，它允许进程中断其他进程。

一个**信号**就是一条小消息，它通知进程系统中发生一个某种类型的事件。

`Linux`系统支持30多种信号,  **`core`文件是 代码和数据内存段 写入到磁盘的的镜像文件.**

| 编号 | 信号 | 对应事件 | 默认动作 |
| :---: | :---: | :--- | :---: |
| 1\* | SIGHUP | 用户退出shell 时,由该shell启动的所有进程将收到这信号 | 终止进程 |
| 2 | SIGINT | 按下 Ctrl+c 组合键时,用户终端正在运行中的由该终端启动程序发出信号 | 终止进程 |
| 3\* | SIGQUIT | 按下 Ctrl+ 组合键时,产生该信号,用户终端向正在运行中的由该终端发出信号 | 终止进程 |
| 4 | SIGILL | cpu 检测到某些进程执行了非法指令. | 终止进程 并产生core文件 |
| 5 | SIGTRAP | 该信号由断点指令或其他 trap 指令产生 | 终止进程 并产生core文件 |
| 6\* | SIGABRT | 调用 abort 函数时产生该信号. 异常终止的信号 | 终止进程 并产生core文件 |
| 7 | SIGBUS | 非法访问内存地址, 包括内存对齐出错 | 终止进程 并产生core文件 |
| 8 | SIGFPE | 发生致命运算错误时发出,包括浮点运算错误,溢出错误,被除数为0错误等 | 终止进程 并产生core文件 |
| 9\* | SIGKILL | 无条件终止进程. 本信号不能被忽略,捕捉 和阻塞. | 终止进程.可以杀死任何进程 |
| 10 | SIGUSE1 | 用户定义的信号. 即程序员可以在程序中定义并使用该信号 | 终止进程 |
| 11\* | SIGSEGV | 指示进程进行了无效内存访问\(段错误\). | 终止进程 并产生core文件 |
| 12 | SIGUSR2 | 另外一个用户自定义信号. 程序员可以在程序中定义并使用该信号 | 终止进程 |
| 13\* | SIGPIPE | Broken pipe 向一个没有读端的管道写数据 | 终止进程 |
| 14\* | SIGALRM | 定时器超过, 超过的时间由系统调用 alarm 设置 | 终止进程 |
| 15 | SIGTERM | 程序结束信号. 该信号可以被阻塞和终止.用来表示程序正常退出 | 终止进程 |
| 16 | SIGSTKFLT | linux 早期版本出现的信号.现仍保留向后兼 | 终止进程 |
| 17\* | SIGCHLD | 子进程结束时, 父进程会收到这信号 | 忽略这信号 |
| 18 | SIGCONT | 如果进程已停止, 则使其继续运行 | 继续 / 忽略 |
| 19\* | SIGSTOP | 停止进程的执行. 信号不能被忽略,处理 和阻塞 | 为终止进程 |
| 20\* | SIGTSTP | 停止终端交互进程的运行. 按下&lt; ctrl + z &gt;组合键时,会产生该信号 | 暂停进程 |
| 21 | SIGTTIN | 后台进程读终端控制台. | 暂停进程 |
| 22 | SIGTTOU | 该信号类似于SIGTTIN,在后台进程要向终端输出数据时发生 | 暂停进程 |
| 23 | SIGURG | 套接字上有紧急数据时,向当前正在运行的程序发些出信号,报告有紧急数据到达 | 忽略该信号 |
| 24 | SIGXCPU | 进程执行时间超过了分配给进程的CPU时间,系统产生该信号并发送给进程 | 终止进程 |
| 25 | SIGXFSZ | 超过文件的最大长度设置 | 终止进程 |
| 26 | SIGVTALRM | 虚拟时钟超时时产生该信号.类似于SIGALRM 但是该信号只能计算该进程占用cpu的使用时间 | 终止进程 |
| 27 | SIGPROF | 类似于 SIGVTALRM, 它光包括该进程占用CPU时间,还包括执行系统调用时间 | 终止进程 |
| 28 | SIGWINCH | 窗口变化大小时发出. | 忽略该信号 |
| 29 | SIGIO | 此信号向进程指示发出了一个异步IO 事件. | 忽略该信号 |
| 30 | SIGPWR | 关机,或电源故障 | 终止进程 |
| 31 | SIGSYS | 无效的系统调用 | 终止进程并产生 core 文件 |
| 34~64 | SIGRTMIN ~ SIGRTMAX | linux的实时信号,他们没有固定的含义,\(可以由用户自定义\) | 终止进程 |



每种信号类型对应于某种系统事件

* 底层的信号。
  * 当底层发生硬件异常，**信号**通知 **用户进程** 发生了这些**异常**。
    * 除以0：发送`SIGILL`信号。
    * 非法存储器引用：发送`SIGSEGV`信号
  * 较高层次的软件事件
    * 键入ctrl+c:发送`SIGINT`信号
    * 一个进程可以发送给另一个进程`SIGKILL`信号强制终止它。
    * 子进程终止或者停止，内核会发送一个`SIGCHLD`信号给父进程。

### 信号术语

传送一个信号到目的进程有两个步骤。

* **发送信号**: 内核通过**更新目的进程上下文的某个状态**，就说**发送**一个信号给目的进程。
  * 发送信号有两个原因
    * **内核检测到一个系统事件**。比如被零除错误，或者子进程终止。
    * **一个进程调用了kill函数**。显示要求进程发送信号给目的进程。
      * 一个进程可以发信号给它自己。
* **接收信号**： 当**目的进程** 被内核强迫以某种方式对信号的发送**做出反应**。目的进程就**接收了信号**。
  * 进程可以忽略这个**信号**，终止。
  * 或者通过一个称为`信号处理程序`\(signal handler\)的用户层函数捕获这个信号。

![](../.gitbook/assets/ping-mu-kuai-zhao-20190904-xia-wu-3.13.12.png)

一个只发出而没有被接收的**信号**叫做`待处理信号`\(pending signal\)

* 一种类型至多只有一个`待处理信号`。
  * 如果一个进程有一个类型为`k`的`待处理信号`。
  * 那么接下来发送到这个进程类型为`k`的信号都会被简单的**丢弃**。

一个进程可以有选择性地**阻塞接收**某种信号

* 它任然可以被发送。但是产生的待处理信号不会被接收。

> 一个待处理信号最多被接收一次。内核为每个进程在`pending`位向量维护着待处理信号的集合，而在`blocked`位向量维护着被阻塞的信号集合。只要传送一个类型为k的信号，内核就会设置`pending`中的第`k`位，而只要接收了一个类型为`k`的信号，内核就会清除`pending`中的第`k`位。

### 发送信号

`Unix`系统 提供大量向进程发送**信号**的机制。所有这些机制都是基于`进程组`\(process group\)。

1. 进程组
   * 每个进程都属于一个`进程组`。
     * 由一个正整数`进程组ID`来标示
       * `getpgrp()`函数返回当前进程的`进程组ID`:

         ```c
           #include<unistd.h>
           pid_t getpgrp(void);
         ```
     * 默认，一个子进程和它的父进程同属于一个`进程组`
       * 一个进程可以通过`setpgid()`来改变自己或者其他进程的进程组。

         ```c
           #include<unistd.h>
           int setpgid(pid_t pid,pid_t pgid);
           如果pid是0 ，那么使用当前进程的pid。
           如果pgid是0，那么使用指定的pid作为pgid(即pgid=pid)。

           例如：进程15213调用setpgid(0,0)
           那么进程15213会 创建/加入进程组15213.
         ```
2. 用`/bin/kill` 命令程序发送信号
   * `/bin/kill`可以向另外的进程发送**任意的信号**。
     * 比如

       ```bash
         unix>/bin/kill -9 15213
       ```

       发送信号9\(`SIGKILL`\)给进程15213。

     * 一个为负的PID会导致信号被发送到进程组PID中的每个进程。

       ```bash
        unix>/bin/kill -9 -15213
       发送信号9(`SIGKILL`)给进程组15213中的每个进程。
       ```
   * 用`/bin/kill`的原因是，有些Unix shell 有自己的`kill`命令
3. 从**键盘**发送信号

   `作业(job)` :对一个命令行求值而创建的**进程**。

   * 在任何时候至多只有一个**前台作业**和0个或多个**后台作业**
     * 前台作业就是需要等待的
     * 后台作业就是不需要等待的
   * 键入`unix>ls  | sort`
     * 创建一个两个进程组成的**前台作业**。
     * **两个进程**通过Unix`管道`链接。
   * `shell`为每个作业创建了一个独立的进程组。
     * 进程组ID取自作业中父进程中的一个。

   在**键盘**输入`ctrl + c` 会发送一个`SIGINT`信号到外壳。外壳捕获该信号。然后发送`SIGINT`信号到这个前台进程组的每个进程。在默认情况下，结果是**终止**前台作业

   ```text
   类似，输入`ctrl-z`会发送一个`SIGTSTP`信号到外壳，外壳捕获这个信号，并发送`SIGTSTP`信号给前台进程组的每个进程，
        在默认情况，结果是**停止(挂起)**前台作业(还是僵死的)
        可以使用 fg 命令唤醒然后放到前台,  使用 bg命令唤醒然后放到后台.
   ```

4. 用`kill函数`发送信号
   * 进程通过调用`kill函数`发送信号给其他进程，类似于`bin/kill`

     ```c
     #include  <sys/types.h>
     #include  <signal.h>
         int kill(pid_t pid, int sig);
             成功返回0, 失败返回 -1
     ```

   * `pid`&gt;0,发送信号`sig`给进程`pid`
   * `pid`&lt;0,发送信号`sig`给进程组`abs(pid) 也就是命令 kill  -9 -11  #负值代表进程组.`
   * 事例:`kill(pid,SIGKILL)`
5. 用`alarm`函数发送信号

   进程可以通过调用`alarm`函数向它自己`SIGALRM`信号。

   ```c
    #include<unistd.h>

    unsigned int alarm(unsigned int secs);

    返回:前一次闹钟剩余的秒数, 如果前面没有设置过闹钟, 那么会返回0
   ```

   `alarm`函数安排内核在`secs`秒内发送一个`SIGALRM`信号给调用进程

   * 如果`secs=0` 那么不会调度闹钟，当然不会发送`SIGALRM`信号。
   * 在任何情况，对`alarm`的调用会取消待处理\(`pending`\)的闹钟，并且会返回被取消的闹钟还剩余多少秒结束。如果没有`pending`的话，返回0

```c
#include <stdio.h>
#include <unistd.h>
int main(void){
    printf("one %u \t",alarm(2));
    sleep(1);
    printf("two %u \t",alarm(2));
}
/* 输出结果为
    ont 0   two 1
```

### 接收信号

> 信号的处理时机是在从内核态切换到用户态时，会执行do\_signal\(\)函数来处理信号

当内核从一个**异常处理程序**返回，准备将控制传递给进程`p`时，它会检查进程`p`的**未被阻塞的待处理信号的集合**\(`pening&~blocked`\)。

* 如果这个集合为空，内核将控制传递到p的逻辑控制流的下一条指令。
* 如果非空，内核选择集合中某个信号k\(通常是最小的k\)，并且强制p接收k。收到这个信号会触发进程某些行为。一旦进程完成行为，传递到p的逻辑控制流的下一条指令。
  * 每个信号类型都有一个预定义的**默认类型**，以下几种.
    * **进程终止**
    * **进程终止并转储存器\(dump core\)**
    * **进程停止直到被`SIGCONT`信号重启**
    * **进程忽略该信号**
  * 进程可以通过使用`signal函数`修改和信号相关联的默认行为。
    * `SIGSTOP`,`SIGKILL`是不能被修改的例外。

      ```c
        #include <signal.h>
        typedef void (*sighandler_t)(int);

        sighandler_t signal(int signum,sighandler_t handler);
            如果函数出错, 则返回   SIG_ERR   这个函数指针的typedef 定义.
      ```

    * `signal`函数通过下列三种方式之一改变和信号`signum`相关联的行为。
      * 如果`handler`是`SIG_IGN`,那么忽略类型为`signum`的信号
      * 如果`handler`是`SIG_DFL`,那么类型为`signum`的信号恢复为默认行为。
      * 否则，`handler`就是用户定义的函数地址，这个函数称为**信号处理程序**
        * 只要进程接收到一个类型为**signum**的信号，就会调用handler。
        * **设置信号处理程序**：把函数传递给signal改变信号的默认行为。
        * 调用信号处理程序，叫**捕获信号**
        * 执行信号处理程序，叫**处理信号**
  * 当处理程序执行它的return语句后，**控制**通常传递回控制流中进程被信号接收中断位置处的指令。
* 信号处理程序可以被 另外的一个信号处理程序中断. 

> 信号处理程序是计算机并发的又一个示例。信号处理程序的执行中断，类似于底层异常处理程序中断当前应用程序的控制流的方式。因为信号处理程序的逻辑控制流与主函数的逻辑控制流重叠，信号处理程序和主函数`并发`地运行。

**自我思考**：信号是一种`异常/中断`，当接收到信号的时候，会停下当前进程所做的事，立马去执行信号处理程序。并不是`多线程/并行`,但确是`并发`的.

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>

void sig(int t){
    return;
}

unsigned int snooze(unsigned int secs){
    unsigned int rc = secs;
    secs  = sleep(rc);
    printf("\n Slept for %u, of %u \n",rc,rc-secs);
    return rc-secs;
}

int main(int argc, char* argv[]){
    if(argc != 2)
        exit(1);    /* 接收一个命令行参数,用作snooze 休眠时间参数 */
    
    if( signal(SIGINT, sig) == SIG_ERR)
        fprintf(stderr,"signal error");
    printf("\n argc = %d\n",argc);
    (void) snooze(atoi(argv[1]));
    return 0;
}
```

### 阻塞和解除阻塞信号

Linux提供**阻塞信号**的**隐式**和**显式**的机制, 内核默认使用隐式阻塞机制.

* **隐式阻塞机制**
  * 阻塞任何当前处理程序正在处理信号类型的待处理信号.\( 正在处理信号K, 那么新的信号K就阻塞\)
* **显示阻塞机制**
  * 应用程序可以使用 sigprocmask\(\) 函数和它的辅助函数, 明确的阻塞和解除阻塞选定的信号.

当一个程序要捕获多个信号时，一些细微的问题就产生了。

* 待处理信号被阻塞
  * `Unix` 信号处理程序通常会**阻塞** 当前处理程序**正在处理** 的类型的**待处理信号**。
* 待处理信号\(被抛弃了\)不会排队等待
  * 当有两个同类型信号都是待处理信号时，有一个会被抛弃。
  * 关键思想：存在一个待处理的信号`k`仅仅表明至少一个一个信号`k`到达过。
* 系统调用可以被中断（在某些**unix系统**会出现）
  * 像`read`,`wait`和`accept`这样的系统调用潜在的阻塞一段较长的时间，称为**慢速系统调用**。
    * 当处理程序捕获一个信号，被中断的慢速系统调用在信号处理程序返回后将不在继续，而是立即返回给用户一个错误条件，并将`errno`设置为`EINTR`。

```c
#include <signal.h>

int sigprocmask( int how, const sigset_t* set, sigset_t* oldset);
    改变当前阻塞的信号集合 ( blocked中的). 具体行为依赖于 how 的值;
        参数:  how : SIG_BLOCK 把set最终的信号添加到 blocked中 ( blocked | set)
                    SIG_UNBLOCK  从 blocked 中删除 set中的信号 (blocked &~set)
                    SIG_SETMASK:  用set的值覆盖掉 blocked原有值 (blocked = set)
              set  :  需要阻塞的信号集合,这个参数可以使用下面这几个函数来设定.
              oldset: 备份原有的 blocked 的值.
       返回值: 成功返回0, 出错返回 -1
       
int sigemptyset(sigset_t* set);
    对信号集合进行操作, 将 set 初始化为空集合.
       返回值: 成功返回0, 出错返回 -1
       
int sigfillset(sigset_t* set);
    对信号集合进行操作, 把每个信号都添加到 set , 就是阻塞全部信号.
       返回值: 成功返回0, 出错返回 -1
       
int sigaddset(sigset_t* set,  int  signum);
    对信号集合进行操作, 将 signum 代表的信号添加到 set 中.
        参数  signum : 可以通过查询得到系统的信号值. (SIGXXXX 之类的)
       返回值: 成功返回0, 出错返回 -1
       
int sigismember( const sigset_t* set, int signum);
    判断 signum 这个信号是否已经在 set 中被设置.
       返回值: 如果已经被设置则返回 1 . 如果不是则返回0,  若出错返回 -1
   
--------------------------
临时阻塞接受一个信号 和还原 范例:
#include <signal.h>
    sigset_t mask, prev_mask;
    sigemptyset( &mask );                /* 清空 阻塞信号集合 */
    sigaddset( &mask , SIGINT );        /* 将 SIGINT 信号添加到阻塞信号集合里 */
    sigprocmask( SIG_BLOCK, &mask, &prev_mask);    /* 设置阻塞信号, blocked | mask */
    .....                                           /*省略若干无用步骤 */
    sigprocmask( SIG_SETMASK, &prev_mask, NULL);   /* 还原被设置阻塞的信号 */
```

### 安全,正确,可移植的信号处理程序

* 信号处理程序有几个熟悉使得它们很难推理分析.
  * 处理程序与主程序并发运行, 共享同样的全局变量,因此可能与主程序和其他的处理程序互相干扰.
  * 如何以及何时接收信号的规则常常有违人的直觉.
  * 不同系统有不同的信号处理语义.

#### 基本规则

* **安全的信号处理**
  * **G0 , 处理程序要尽可能简单.**
  * **G1, 在处理程序中只调用异步信号安全函数.**
    * 异步信号安全的函数就是能够被信号处理程序安全的调用, **原因有二 :**
      * `要么是可重入的` \(如访问局部变量\)
      * `要么它不能被其他信号处理程序中断`. \(也就是在处理信号时, 阻塞其他信号\).
  * **G2, 保存和恢复 errno   这个全局变量.**
  * **G3, 阻塞所有的信号,保护对共享全局数据结构的访问.**
  * **G4, 用  volatile  声明  全局变量 `(强迫编译器每次都要去内存中读取这个值,不能缓存在寄存器中.)`**
  * **G5,  用 sig\_atomic\_t  声明标志 \(就是个全局int值, 异常处理程序将收到的信号记录在这个值内\)**
* **正确的信号处理**
  * **未处理的信号是不排队的,** 
    * **当正在处理信号k时, 又到达个2个同类型 k 信号时, 第二个会被丢弃,第一个会在阻塞列表中**
  * **不可以用信号来对其他进程中发生的时间计数.**
* **可移植的信号处理**
  * 不同系统之间，信号处理语义的差异\(比如一个被中断的慢速系统调用是重启，还是永久放弃\)是Unix信号系统的一个缺陷
    * **signal 函数的语义各不相同.**
    * **系统调用可以被中断 . \(有的系统可以被中断, 有的系统不可以被中断\)**
    * 为了处理这个问题，`Posix`标准定义了`sigaction`函数，它允许与`Linux`和`Solaris`这样与`Posix`兼容的系统上的用户，明确指明他们想要的信号处理语义。
      * **`可以使用 sigaction() 函数来设置需要的信号处理语意.`**
        * 一般这个函数会被封装,然后实现如下功能
          * 只有这个处理程序当前正在处理的那种类型被阻塞。
          * 和所有信号实现一样，信号不会排队等待。
          * 只要可能，被中断的系统调用会自动重启
          * 一旦设置了信号处理程序，它就会一直保持，直到`Signal`带着handler参数为`SIG_IGN`或者`SIG_DFL`被调用。
            * 在某些比较老的Unix系统，信号处理程序被使用一次后，又回到默认行为。

```c
#include<signal.h>

int sigaction(int signum,stuct sigaction *act,struct sigaction *oldcat);
//若成功则为1，出错则为-1
```

`sigaction`函数应用不广泛，它要求用户设置**多个结构条目**。

一个更简洁的方式，是定义一个包装函数，称为`Signal`,它调用`sigaction`。

```c
handler_t *Signal(int signum, handler_t *handler) 
{
    struct sigaction action, old_action;

    action.sa_handler = handler;  
    sigemptyset(&action.sa_mask); /* 阻止处理类型的标志 */
    action.sa_flags = SA_RESTART; /* 如果可能，重新启动系统调用 */

    if (sigaction(signum, &action, &old_action) < 0)
	unix_error("Signal error");
    return (old_action.sa_handler);
}
```

* 它的调用方式与signal函数的调用方式一样。
* `Signal`包装函数设置了一个信号处理程序，其信号处理语义如下（**设置标准**）：
  * 只有这个处理程序当前正在处理的那种类型被阻塞。
  * 和所有信号实现一样，信号不会排队等待。
  * 只要可能，被中断的系统调用会自动重启
  * 一旦设置了信号处理程序，它就会一直保持，直到`Signal`带着handler参数为`SIG_IGN`或者`SIG_DFL`被调用。
    * 在某些比较老的Unix系统，信号处理程序被使用一次后，又回到默认行为。

#### G1 中的安全函数

| 安全函数 |  |  |  |
| :--- | :--- | :--- | :--- |
| \_Exit | fexecve | poll | sigqueue |
| \_exit | fork | posix\_trace\_event | sigset |
| abort | ffstat | pselect | sigsuspend |
| accept | fstatat | raise | sleep |
| access | fsync | read | sockatmark |
| aio\_error | ftruncate | readlink | socket |
| aio\_return | futimens | readlinkat | socketpair |
| aio\_suspend | getegid | recv | stat |
| alarm | geteuid | recvfrom | symlink |
| bind | getgid | fecvmsg | symlinkat |
| cfgetispeed | getgroups | rename | tcdrain |
| cfgetospeed | getpeername | renameat | tcflow |
| cfsetispeed | getpgrp | rmdir | tcflush |
| cfsetospeed | getpid | select | tcgetattr |
| chdir | getppid | sem\_post | tcgetpgrp |
| chmod | getsockname | send | tcsendbreak |
| chown | getsockopt | sendmsg | tcsetattr |
| clock\_gettime | getuid | sendto | tcsetpgrp |
| close | kill | setgid | time |
| connect | link | setpgid | timer\_getoverrun |
| creat | linkat | setsid | timer\_gettime |
| dup | listen | setsockopt | time\_settime |
| dup2 | lseek | setuid | times |
| execl | lstat | shutdown | umask |
| execle | mkdir | sigaction | uname |
| execv | mkdirat | sigaddset | unlink |
| execve | mkfifo | sigdelest | unlinkat |
| faccessat | mkfifoat | sigemptyset | utime |
| fchmod | mknod | sigfillset | utimensat |
| fchmodat | mknodat | sigismember | utimes |
| fchown | open | signal | wait |
| fchownat | openat | sigpause | waitpid |
| fcntl | pause | sigpending | write |
| fdatastnc | pipe | sigprocmask |  |



### 同步流以避免讨厌的并发错误

如何编写读写**相同存储位置**的并发流程序的问题，困扰着数代计算机科学家。

* 流可能**交错** 的数量是与**指令数** 量呈**指数关系**
  * 有些交错会产生正确结果，有些可能不会。

所谓`同步流`就是。以某种方式`同步`**并发流**，从而得到 **最大的可行交错的集合** ，每个交错集合都能得到正确的结果。

**并发编程**是一个很深奥，很重要的问题。在第12章详细讨论。

现在我们只考虑一个并发相关的智力挑战。

![](http://i.imgur.com/mh0wN05.png)

```c
#include "csapp.h"

void initjobs()
{
}

void addjob(int pid)
{
}

void deletejob(int pid)
{
}

/* $begin procmask1 */
/* WARNING: This code is buggy! */
void handler(int sig)
{
    int olderrno = errno;
    sigset_t mask_all, prev_all;
    pid_t pid;

    Sigfillset(&mask_all);
    while ((pid = waitpid(-1, NULL, 0)) > 0) { /* Reap a zombie child */
        Sigprocmask(SIG_BLOCK, &mask_all, &prev_all);
        deletejob(pid); /* Delete the child from the job list */
        Sigprocmask(SIG_SETMASK, &prev_all, NULL);
    }
    if (errno != ECHILD)
        Sio_error("waitpid error");
    errno = olderrno;
}
    
int main(int argc, char **argv)
{
    int pid;
    sigset_t mask_all, prev_all;

    Sigfillset(&mask_all);
    Signal(SIGCHLD, handler);
    initjobs(); /* Initialize the job list */

    while (1) {
        if ((pid = Fork()) == 0) { /* Child process */
            Execve("/bin/date", argv, NULL);
        }
        Sigprocmask(SIG_BLOCK, &mask_all, &prev_all); /* Parent process */  
        addjob(pid);  /* Add the child to the job list */
        Sigprocmask(SIG_SETMASK, &prev_all, NULL);    
    }
    exit(0);
}
/* $end procmask1 */
```

如果发生以下情况，会出现**同步错误**。

* 父进程执行`fork`函数，内核调度新创建的子进程运行，而不是父进程。
* 在父进程再次运行前，子进程已经终止，变成僵死进程，需要内核一个`SIGCHLD`信号给父进程
* 父进程处理信号，调用`deletejob`.
* 调用`addjob`。

显然`deletejob`必须在`addjob`之后，不然添加进去的job永久存在。这就是**同步错误**。

这是一个称为`竞争`\(race\)的经典同步错误的示例。

* `main`中的`addjob`和处理程序中调用`deletejob`之间存在竞争。
* 必须`addjob`赢得进展，结果才是正确的，否则就是错误的。但是`addjob`不一定能赢，所以有可能错误。即为同步错误。
* 因为内核的调度问题，这种错误十分难以被发现。难以调试。

{% hint style="info" %}
Q：如何消除竞争？

A：可以在fork之前，阻塞`SIGCHLD`信号，在调用`addjob`后取消阻塞。
{% endhint %}

* 注意，子进程继承了阻塞，我们要小心地接触子进程中的阻塞。
* 消除竞争的原则就是，让该赢得竞争的对象在任何情况下都能赢。

#### 修改过的,保证父进程在相应的 deletejob 之前执行 addjod

```c
#include "csapp.h"

void initjobs()
{
}

void addjob(int pid)
{
}

void deletejob(int pid)
{
}

/* $begin procmask2 */
void handler(int sig)
{
    int olderrno = errno;
    sigset_t mask_all, prev_all;
    pid_t pid;

    Sigfillset(&mask_all);
    while ((pid = waitpid(-1, NULL, 0)) > 0) { /* Reap a zombie child */
        Sigprocmask(SIG_BLOCK, &mask_all, &prev_all);
        deletejob(pid); /* Delete the child from the job list */
        Sigprocmask(SIG_SETMASK, &prev_all, NULL);
    }
    if (errno != ECHILD)
        Sio_error("waitpid error");
    errno = olderrno;
}
    
int main(int argc, char **argv)
{
    int pid;
    sigset_t mask_all, mask_one, prev_one;

    Sigfillset(&mask_all);
    Sigemptyset(&mask_one);
    Sigaddset(&mask_one, SIGCHLD);
    Signal(SIGCHLD, handler);
    initjobs(); /* Initialize the job list */

    while (1) {
        Sigprocmask(SIG_BLOCK, &mask_one, &prev_one); /* Block SIGCHLD */
        if ((pid = Fork()) == 0) { /* Child process */
            Sigprocmask(SIG_SETMASK, &prev_one, NULL); /* Unblock SIGCHLD */
            Execve("/bin/date", argv, NULL);
        }
        Sigprocmask(SIG_BLOCK, &mask_all, NULL); /* Parent process */  
        addjob(pid);  /* Add the child to the job list */
        Sigprocmask(SIG_SETMASK, &prev_one, NULL);  /* Unblock SIGCHLD */
    }
    exit(0);
}
/* $end procmask2 */

```

### 显示的等待信号

sleep 和 pause  这两个函数比较浪费时间和CPU, 而且还不好掐算时间和信号.

解决方法是使用下面这个函数:

```c
#include <signal.h>
int sigsuspend( const  sigset_t* mask);
  使用 mask替换当前阻塞集合(一般是设置阻塞时的原阻塞备份),然后挂起该进程,直到收到一个信号,其行为要么
      是运行一个信号处理程序, 要么是终止该进程. ( 根据收到的信号来区分).
返回值: 这个函数一直返回 -1

这个函数是三个函数的原子操作的集合体, 代表它不被任何情况中断. 绝对会执行完成.
  sigprocmask(SIG_SETMASK, &mask, &prev);    /*代表 prev = blocked 和 blocked = mask */
  pause();                                   /*等待一个信号,任何信号都可以, 随后继续执行 */
  sigprocmask(SIG_SETMASK, &prev, NULL);     /*执行 blocked = prev   . 这三步不可中断 */
```

使用sigsuspend 来显示的等待信号.

```c
/* $begin sigsuspend */
#include "csapp.h"

volatile sig_atomic_t pid;

void sigchld_handler(int s)
{
    int olderrno = errno;
    pid = Waitpid(-1, NULL, 0);
    errno = olderrno;
}

void sigint_handler(int s)
{
}

int main(int argc, char **argv) 
{
    sigset_t mask, prev;

    Signal(SIGCHLD, sigchld_handler);
    Signal(SIGINT, sigint_handler);
    Sigemptyset(&mask);
    Sigaddset(&mask, SIGCHLD);

    while (1) {
        Sigprocmask(SIG_BLOCK, &mask, &prev); /* Block SIGCHLD */
        if (Fork() == 0) /* Child */
            exit(0);

        /* Wait for SIGCHLD to be received */
        pid = 0;
        while (!pid) 
            Sigsuspend(&prev);

        /* Optionally unblock SIGCHLD */
        Sigprocmask(SIG_SETMASK, &prev, NULL); 

        /* Do some work after receiving SIGCHLD */
        printf(".");
    }
    exit(0);
}
/* $end sigsuspend */
```



## 非本地跳转

C语言提供一种用户级异常控制流形式，称为`非本地跳转(nonlocal jump)`。

* 它将控制直接从一个函数转移到另一个当前正在执行的函数。不需要经过正常的**调用-返回**序列。
* 非本地跳转是通过`setjmp`和`longjmp`函数来提供。

  ```c
    #include<setjmp.h>

    int setjmp(jmp_buf env);
        参数类型定义在 setjmp.h 中, 就是个用来保存当前程序状态的缓冲区结构体.(无信号值)
            第一次调用返回值是0, 第二次通过longjmp()调用返回这个函数指定的 retval 的值.
  
    int sigsetjmp(sigjmp_buf env,int savesigs);    //信号处理程序使用
       参数类型定义在 setjmp.h 中, 就是个用来保存当前程序状态的缓冲区结构体.
                                     (有信号值, 但没有 待处理信号表 和 阻塞信号表)
            第一次调用返回值是0, 第二次通过siglongjmp()调用返回这个函数指定的 retval 的值.
    //参数savesigs若为非0则代表搁置的信号集合也会一块保存 
  ```

  * `setjmp`函数在`env`缓冲区保存当前调用环境，以供后面`longjmp`使用，并返回0

    * 调用环境包括**程序计数器**，**栈指针**，**通用目的寄存器**。

    ```c
    #include
      void longjmp(jmp_buf env,int retval);
      void siglongjmp(sigjmp_buf env,int retval);//信号处理程序使用
    ```

  * `longjmp`函数从env缓冲区中恢复调用环境，然后触发一个从最近一次初始化env的`setjmp`调用的返回。然后`setjmp`返回，并带有非零的返回值`retval`（看清楚从`setjmp`返回）
    * `setjmp`返回多次，第一次是`0`，第二次是`retval`.
    * `longjmp`从不返回。

* `非本地跳转`的重要应用是允许从一个深层嵌套的函数调用立即返回。一般是发现了**错误**。
  * 不用费力解开栈。
  * 直接返回到一个普通的本地化的错误处理程序。
  * 会造成内存泄漏

{% hint style="info" %}
C++和Java 中的软件异常

C++和Java提供的异常机制是较高层次的，是C语言setjmp和longjmp函数的更加结构化的版本。你可以把try语句中的catch字句看作setjmp函数。相似地,throw语句就类似与longjmp函数。
{% endhint %}

 

**一个实例: 使用了 非本地跳转从深层潜逃的函数调用中的错误情况恢复, 而不需要解开整个栈的基本结构.**

```c
/* $begin setjmp */
#include "csapp.h"

jmp_buf buf;

int error1 = 0; 
int error2 = 1;

void foo(void), bar(void);

int main() 
{
    switch(setjmp(buf)) {
    case 0: 
	foo();
        break;
    case 1: 
	printf("Detected an error1 condition in foo\n");
        break;
    case 2: 
	printf("Detected an error2 condition in foo\n");
        break;
    default:
	printf("Unknown error condition in foo\n");
    }
    exit(0);
}

/* Deeply nested function foo */
void foo(void) 
{
    if (error1)
	longjmp(buf, 1); 
    bar();
}

void bar(void) 
{
    if (error2)
	longjmp(buf, 2); 
}
/* $end setjmp */

```

**例子2:  当用户键入 Ctrl + C 时, 使用非本地跳转来重启动它自身的程序**

```c
/* $begin restart */
#include "csapp.h"

sigjmp_buf buf;

void handler(int sig) 
{
    siglongjmp(buf, 1);
}

int main() 
{
    if (!sigsetjmp(buf, 1)) {
        Signal(SIGINT, handler);
	Sio_puts("starting\n");
    }
    else 
	Sio_puts("restarting\n");

    while(1) {
	Sleep(1);
	Sio_puts("processing...\n");
    }
    exit(0); /* Control never reaches here */
}
/* $end restart */

```

## 操作进程的工具

* `STRACE`\(痕迹\):打印一个正在运行的程序和它的子进程调用的每个系统调用的轨迹。
  * 用`-static`编译，能得到一个更干净，不带有大量共享库相关的输出的轨迹。
* `PS`\(**Processes Status**\)： 列出当前系统的进程\(包括僵死进程\)
* `TOP`\(因为我们关注峰值的几个程序，所以叫TOP\):打印当前进程使用的信息。
* `PMAP`\(**rePort Memory map of A Process**\): 查看进程的内存映像信息
* `/proc`:一个虚拟文件系统，以ASCII文本格式输出大量内核数据结构。
  * 用户程序可以读取这些内容。
  * 比如，输入`"cat /proc/loadavg`，观察Linux系统上当前的平均负载。

## 小结

* `异常控制流(ECF)`发生在计算机系统的各个层次，是计算机系统中提供并发的基本机制。
* 在**硬件层**，`异常`是处理器中的事件出发的控制流中的突变。控制流传递给一个异常处理程序，该处理程序进行一些处理，然后返回控制被中断的控制流。
  * 有四种不同类型的异常：中断，故障，终止和陷阱。
    * 定时器芯片或磁盘控制器，设置了处理器芯片上的**中断引脚**时，`中断`会**异步**发生。返回到`Inext`
    * 一条指令的执行可能导致`故障`和`终止`同时出现。
      * `故障`可能返回调用指令。
      * `终止`不将控制返回。
    * `陷阱`用于`系统调用`。结束后，返回`Inext`
* 在**操作系统层**，内核用`ECF`提供进程的基本概念。`进程`给应用两个重要抽象:
  * **逻辑控制流**
  * **私有地址空间**
* 在**操作系统和应用程序接口处**，有**子进程**，和**信号**。
* 最后，C语言的`非本地跳转` 完成应用程序层面的异常处理。

至此，`异常`贯穿了从底层硬件，到抽象的软件层次。



