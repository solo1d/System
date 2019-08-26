# 链接

**链接是将各种代码和数据片段手机并组合成为一个单一文件的过程, 这个文件可被加载\(复制\)到内存并执行.**

* 链接可以执行于**编译**时, 也就是在源代码被翻译成机器代码时;
* 也可以执行与加**载**时, 也就是在程序被**加载器**加载到内存并执行时.
* **运行**时, 也就是程序应用来执行.

**链接是由连接器的程序自动执行的.**

**链接**将**分离编译**成为可能.\(`多文件编译`\).

* **理解链接器的好处**
  * **有利于大型程序构造**
  * **避免危险的编程错误**
  * **理解语言的作用域的实现**
  * **理解其他重要的系统概念**
  * **利用共享库**

## 编译器驱动程序

* **gcc驱动程序**
  * **`gcc  参数和选项  文件`**
* **cpp  预处理     将 `.c` 文件翻译成  ASCII码的  `.i`   中间文件**
  * **`cpp  main.c -o main.i`**
* **ccl  编译器     将   `.i` 文件翻译成ASCII码的  `.s`  汇编文件**
  * **`ccl   main.i   -o  main.s`**
* **as  汇编器   将 `.s` 文件翻译成一个可重定位目标文件 `.o`**
  * **`as  main.s   -o  main.o`**
* **ld    链接器   将 `.o` 文件以及一些必要的`系统目标文件`组合起来,创建一个可执行目标文件.**
  * **`ld  main.s   -o  prog`**
* **shell 调用操作系统中一个叫做 `加载器的函数` , 它将可执行目标文件的代码和数据复制到内存,然后将控制转移到这个程序的开头.**
  * **`linux>   ./prog`**   

### 静态链接  **ld**

**数据节 : 可重定位目标文件\( .o \)由各种不同的代码和数据节组成, 每个节都是一个连续的字节序列.指令在一节中,初始化了的全局变量在另一节中, 未初始化的变量又在另一个节中**

* **符号解析**
  * **目标文件重定义和引用符号,\(符号对于一个函数,全局变量,或一个静态变量\).**
  * **目的是 将每个符号引用正好和一个符号定义关联起来.**
* **重定位**
  * **编译器和汇编器生成从地址0开始的代码和数据节.**
  * **链接器通过把每个符号定义于一个内存位置关联起来,从而重定位这些节.**
  * **然后修改所有对这些符号的引用, 使得它们指向那个内存位置.**

### **目标文件  \( .o \)**

* **三种目标文件形式**
  * **可重定位目标文件**
    * 就是**`.o 文件`**, 包含二进制代码和数据, 可以在编译时与其他可重定位目标文件合并起来,创建一个可执行文件.
    * 编译器和链接器 可以生成重定位目标文件. **`(ccl 和 as)`**
  * **可执行目标文件**
    * 包含二进制代码和数据, 可以直接复制到内存并执行. 也就是  **`.out 文件`**
    * 链接器可以生存执行目标文件.  \(**`ld )`**
  * **共享目标文件**
    * 一种特殊类型的可重定位目标文件, 可以在加载或运行时被动态地加载进内存并链接. 也就是 **`.dll 或 .so 动态库文件`**
    * gcc 可以生成这种文件

**一个`目标模块` 是一个`字节序列`, `目标文件`是以文件形式存放在磁盘中的`目标模块.`**

**Linux和Unix 系统使用可执行链接格式 `ELF`.**

### **可重定位目标文件**

![ELF&#x53EF;&#x91CD;&#x5B9A;&#x4F4D;&#x76EE;&#x6807;&#x6587;&#x4EF6;](.gitbook/assets/ping-mu-kuai-zhao-20190826-shang-wu-9.15.57.png)

**夹在 ELF 头和节头部表之间的都是节. 一个典型的ELF可重定位文件包含下面几个节:**

* **.text**
  * **已编译程序的机器代码**
* **.rodata**
  * **只读数据, 比如 printf 语句中的格式串和开关语句的跳转表.**
* **.data**
  * **一初始化的全局和静态C变量.     局部的C变量在运行时被保存在栈中,既不出现在.data节中,也不会出现在.bss 节中.**
* **.bss**
  * **为初始化的全局和静态C变量, 以及所有被初始化为0的全局或静态变量. 在目标文件中这个节不占据实际的空间,它仅仅是一个占位符.**
* **.symtab**
  * **一个符号表, 它存放 在程序中定义和引用的函数和全局变量信息.**
* **.rel.text**
  * **一个 .text 节中位置的列表, 当链接器把这个目标文件和其他文件组合时,需要修改这些位置. 一般而言,任何调用外部函数或者引用全局变量的指令都修改修改, 而调用本地函数的指令则不需要修改.**
* **.rel.data**
  * **被模块引用或定义的所有全局变量的重定位信息. 一般而言,任何已初始化的全局变量, 如果它的初始值是一个全局变量地址 或者 外部定义函数的地址, 都需要被修改.**
* **.debug**
  * **一个调试符号表, 七条牧师程序中定义的局部变量和类型定义,程序中定义和引用的全局变量,以及原始的C文件.  \(`只有 -g 选项调用编译器驱动程序时,才会得到这张表`\)**
* **.line**
  * **原始C源程序中的行号和 .text 节中机器指令之间的映射. `(只有以 -g 选项调用编译器驱动程序时,才会得到这张表)`**
* **.strtab**
  * **一个字符串表, 其内容包括  .symtab 和 .debug 节中的符号表,以及头部中的节名字. \(字符串表就是以 null 结尾的字符串序列\).**

\*\*\*\*

\*\*\*\*

\*\*\*\*

\*\*\*\*

\*\*\*\*

\*\*\*\*

\*\*\*\*

\*\*\*\*

\*\*\*\*

\*\*\*\*

\*\*\*\*

\*\*\*\*
