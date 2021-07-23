---
description: centos 7.6 平台
---

- **文件是对I/O 设备的抽象表示**
- **虚拟内存是对主存和磁盘I/O设备的抽象表示**
- **进程则是对处理器, 主存 和 I/O 设备的抽象表示**

# Linux 系统笔记

## 常会用到的命令

想要查看机器代码文件的内容, 可以使用反汇编器\(objdump\).

### gcc

```bash
$gcc -Og -S mastore.c
    #无优化编译文件,生成汇编文件 mastore.s
 
$gcc -Og -c maastore.c
    #无优化编译文件,生成.o二进制文件  mastore.o

大于 2GB的程序可以用 -mcmodel=medium  (中型代码模型)
    或者 -mcmodel=large  (大型代码模型来编译)
```

### objdump    反汇编器

```bash
$objdump -d mastore.o
    #对mastore.o文件执行反汇编, 显示可执行部分的汇编内容.(输出内容: 左侧编号是地址,右侧是汇编)
在linux下，用readelf来看ELF头部或者其它各section的内容，用objdump来对指定的内容
    （.text, .data等）进行反汇编。

但是mac os X下没有这两个命令，可以用brew来安装，
        macos> brew update && brew install binutils
    然后用greadelf和gobjdump 
```

## 处理目标文件工具

```bash
Linux 系统中有大量可用的工具来处理目标文件, GUN binutils 包是最有用的.可以运行在每个linux 平台上

     ar : 创建动态库,  插入, 删除, 列出  和提取成员. ($ ar rcs libxx.a  add.o sub.o )
strings : 列出一个目标文件中所有可打印的字符串.
  strip : 从目标文件中删除符号表信息
     nm :  列出一个目标文件的符号表中定义的符号.
   size : 列出目标文件中节的名字和大小.
readelf :  显示一个目标文件的完整结构, 包括ELF头中编码的所有信息.包含 size 和nm 的功能.
objdump : 所有二进制工具之母. 能够显示一个目标文件中所有的信息. 他最大的作用是反汇编 .text节中的二进制指令.
    ldd : 列出一个可执行文件在运行时所需要的共享库.
```

### **readelf  可重定向目标文件查看工具   \(就是 .o 文件\)**

**GUN  的 `readelf` 程序是一个查看目标文件内容很方便的工具. \(就是命令\)     `linux> readelf -a x.o  >   file.txt     #把结果放到文本内,方便查看`**

### 编译命令

```bash
linux> cpp   a.c    #预处理命令, 生成一个 ASCII码的中间文件 a.i                                    
linux> ccl   a.i    #编译器命令, 生成一个 ASCII码的汇编文件 a.s
linux> as    a.s    #汇编命令,   生成一个 可重定位目标文件  a.o
linux> ld    a.o    #链接命令, 生成一个可执行的目标文件. a.out

linux> ar  rcs libxx.a  a.o     #静态库生成命令,将 a.o 可重定位目标文件,打包成.a 静态库文件
linux> gcc  main.o -L. -lxx     #使用静态库, -L. 指定本级目录, -lxx 静态库名简写(libxx.a)
                                    #可以生成一个可执行文件

linux> gcc -rdynamic -o a.out  main.c -ldl        #生成一个会使用动态库或共享库的执行文件
linux> gcc -fpic -shared -o  libvector.so  a.c    #生成一个动态库或共享库( -fpic 表示位置无关代码)

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

```bash
strace  进程     #跟踪系统调用
```





## vim

```bash
    十六进制显示  :%!xxd      将当前的文本转换成十六进制来显示.
    文本格式显示  :%!xxd -r   将当前文本转换回文本格式显示.
    代码排版      gg=G  .
    替换操作      r / R ( r 一个字符,是光标后面的字符, R 替换多个,从光标后的位置向后替换).
    整行替换      :s/需要替换的内容/替换后的内容/g  (把这一行的内容替换掉).
    指定行替换    :12,29s/work/newlink/g  (把12到29行中的 work 替换成 newlink).
    全文替换      :%s/work/newlink/g      (全文的 work 替换成 newlink).
    提示替换      :s/work/gc    (c 关键字会提示用户是否替换,上面都能使用这个选项).
```

## 操作数格式

![操作数格式](.gitbook/assets/ping-mu-kuai-zhao-20190805-14.29.03.png)





## 电脑启动过程

1. 开机键按下, CPU上电后 会初始化寄存器 CS 和 IP 寄存器
   1. 将CS 初始化为 0xFFFF,  代码段寄存器
   2. 将IP  初始化为 0x0000
   3. **整合之后就是 CS:IP  =   0xFFFF0 从这里开始执行代码, 只有16字节, 一般放置 jmp 指令 跳转到另一个区域**
      1. 从 0xFFFF0 向  0xFFFFF 处执行,  从低到高
   4. **在开机的一瞬间，CPU 的 PC 寄存器被强制初始化为 0xFFFF0**。
      1. 再说具体些，CPU 将 段基址寄存器 cs 初始化为 0xF0000，将偏移地址寄存器 IP 初始化为 0xFFFF0.
      2. 根据实模式下的最终地址计算规则，将段基址CS 左移 4 位，加上偏移地址，得到最终的物理地址也就是抽象出来的 PC 寄存器地址为 0xFFFF0  ;  ( 0xFFFF << 4  = 0XFFFF0   顶出的位会被记录 )
2. **会来到 `ROM  BIOS` 中执行, 将 ROM BIOS的内容 映射到内存 (这是系统 BIOS)**
   1. **BIOS会存放到内存的最高处,   占据了 最顶端的 64KB  ( 0xFFFFF 到 0xEFFFF)**
   2. **CPU来到 0xFFFF0 内存地址处开始执行**
3. **根据存在于 0xFFFF0 的代码, 跳转到 0xFE05B 位置，开始执行**
   1. **这部分的 BIOS 代码 主要做的内容就是初始化硬件**
4. **根据 BIOS 最后的内容, 会将 磁盘的第一个扇区  512b 字节的数据读取到一个内存位置 `0x07C00`, 然后通过 jmp指令 跳转到这个位置开始执行, 也就是转移控制权**
   1. 这个磁盘的第一个扇区 就是 **主引导扇区 (MBR)**
      1. **这个扇区的特点是 :  521字节中, 最后两位字节永远是 0x55 和 0xAA**
      2. 如果查找不到, 就返回没有启动盘的信息
   2. CS:IP = 0x07C00  内存地址就是 0x07C00,  CS = 0x0000 , IP = 0x7C00
   3. **把磁盘换成 USB 也是同理,  BIOS 可以让你选择使用哪个设备启动, 那么哪个设备的第一个扇区就会拷贝到固定位置,再移交控制权**
5. **主引导扇区会 加载内核和其他的内容到内存, 然后 再移交控制权给内核即可**



## 跳转步骤

1. 按下开机键，CPU 将 PC 寄存器的值强制初始化为 0xffff0，这个位置是 BIOS 程序的入口地址（一跳）
2. 该入口地址处是一个跳转指令，跳转到 0xfe05b 位置，开始执行（二跳）
3. 执行了一些硬件检测工作后，最后一步将启动区内容加载到内存 0x7c00，并跳转到这里（三跳）
4. 启动区代码主要是加载操作系统内核，并跳转到加载处（四跳）



