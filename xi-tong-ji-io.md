# 系统级 I/O

`输入/输出(I/O)` : 是指`主存`和`外部设备`\(如磁盘，终端，网络\)之间拷贝数据过程。

输入: 从 **`I/O 设备`**复制数据到**`主存`**.

输出:  从**`主存`** 复制数据到**`I/O设备`**.

* 高级别`I/O`函数
  * `scanf`和`printf`
  * `<<`和`>>`
  * 使用系统级`I/O`函数实现
* 系统级`I/O`函数。
  * `Q`:大多数时候高级别`I/O`函数都运行良好，为什么我们还要学`Unix I/O`
  * `A`:
    * 了解`Unix I/O`将帮助你理解其他的系统概念。
      * 要深入理解其他概念，必须理解`I/O`。
    * 有时你除了使用`Unix I/O`别无选择
      * 标准`I/O库`没有提供读取文件`元数据`的方式。
        * 如`文件大小`或`文件创建时间`。
      * 用于`网络编程`十分冒险。

## Unix I/O

  
一个`Unix 文件`就是一个`m`个字节的序列:

![](http://i.imgur.com/2gltscx.png)

* 所有`I/O`设备都被模型化为`文件`。
* 而所有的输入和输出都被当做相应**文件**的读和写。

`设备`优雅地映射成文件，允许`Unix`内核引出一个**简单**，**低级**的应用接口。叫做`Unix I/O`

* 使得所有的输入输出都能以一种统一且一致的方式来执行。
* * **打开文件**: 应用程序要求内核打开文件
  * * `内核`返回一个小的`非负整数`，叫做`描述符`
    * * 等于`内核`分配一个文件名，来标示当前的文件。
      * `内核`记录有关这个打开文件的所有信息。应用程序只需要记住标示符。
    * `Unix`外壳创建进程时都有三个**打开的文件**
    * * **标准输入**\(标示符`0`\)
      * **标准输出**\(标示符`1`\)
      * **标准错误**\(标示符`2`\)
      * 头文件`<unistd.h>`定义了常量代替**显式**的描述符值
      * * `STDIN_FILENO`
        * `STDOUT_FILENO`
        * `STDERR_FILENO`
  * **改变当前文件的位置\(非文件目录\)**
  * * 对于每个打开的文件，内核保持一个`文件位置k`
    * * 初始为`0`。
      * `文件位置`即是从文件开头起始的字节偏移量。
    * 执行`lseek`操作，可以显式地设置文件位置。
  * **读写文件**。
  * * 一个读操作就是从文件拷贝`n`个字节到存储器,然后将`k`增加到`k+n`。
    * * 给定一个大小为`m`字节的文件，当`k>=m`时**执行读操作**会触发一个称 为`end-of-file(EOF)`的条件。
      * * 应用程序能检测到这个`条件`\(或者说信号?\)
        * 文件结尾并没有这样的符号。
    * `写操作`就是从存储器拷贝`n`个字节到一个文件，从当前`文件位置`k开始，然后更新`k`。
  * **关闭文件** :当应用程序完成了文件的访问，通知`内核`关闭文件。
  * * 响应
    * * `内核`释放文件打开时创建的数据结构。
      * 将`描述符`恢复到可用的描述符池中。
    * 无论一个进程因为何种原因被关闭，内核会关闭所有它打开的文件。

## 文件

每个Linux 文件都有一个类型 来表明它在系统中的信息.

* **普通文件 :** 包含任意数据.
  * 应用程序常常要区分`文本文件`和`二进制文件`
    * **文本文件**  是指含有 ASCII 或 unicod 字符的普通文件,包含一个 **`文本行序列`**
    * **二进制文件**  是所有其他的文件, \(对内核而言,二进制文件和文本文件没有区别\)
* **目录  :**   是包含一组链接的文件, 其中每个链接都将一个文件名条目 映射到一个文件, 这个文件可能是另一个目录
* **套接字 :**  用来与另一个进程进行网络通信的文件.
* 命名通道
* 符号链接
* 字符和块设备

Linux内核集那个所有文件都组织成一个目录层次结构,

![](.gitbook/assets/ping-mu-kuai-zhao-20190918-shang-wu-9.28.34.png)

**作为其上下文的一部分, 每个进程都有一个当前工作目录, 来确定其 在目录层次结构中的当前位置.**

**目录层次结构中的位置用路径名来指定. \(路径名是一个字符串\)**

* **绝对路径名 :** 以斜杠开始,表示从根节点开始的路径.
* **相对路径名:**  以文件名开始, 表示从当前工作目录开始的路径.

## 打开和关闭文件

`进程`是通过调用 `open`函数来**打开**一个已存在的文件或者**创建**一个新文件的

```c
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>

int open(char *filename,int flags,mode_t mode);

                        //返回:若成功则为新文件描述符，若出错为-1
```

`open`函数将`filename`转换为一个`文件描述符`，并且返回描述符数字。

* 返回的`描述符`总是在进程当前没有打开的`最小描述符`。
* `flags`参数指明了进程打算如何访问这个文件:
  * 可是以一个多个`掩码`的或。\(拿二进制思想思考\)
  * `O_RDONLY`: 只读
  * `O_WRONLY`: 只写
    * `O_CREAT` : 如果文件不存在，就创建一个`截断的(truncated)`\(空\)文件。
    * `O_TRUNC` : 如果文件已存在，就`截断它`\(长度被截为`0`，属性不变\)
    * `O_APPEND`: 在每次写操作前，设置文件位置到文件的结尾
  * `O_RDWR`: 可读可写

    ```c
      例子代码

      //已只读模式打开一个文件
      fd = Open("foo.txt",O_RDONLY,0);
      //打开一个已存在的文件，并在后面面添加一个数据
      fd = Open("foo.txt",O_WRONLY|O_APPEND,0);
    ```
* `mode`参数指定了`新文件`的访问权限位。
  * 每个进程都有`umask`
    * `权限掩码`，或 `权限屏蔽字`
    * 所有被设置的权限都要减去这个`权限掩码`才是实际权限。
      * `777-022=755` 或者是 `777&~022`。
    * 通过`umask()`函数设置
  * `mode`并不是实际权限
    * 文件的权限位被设置为`mode & ~umask`，也可以表示两者相减。
  * 例子

    ```c
      #define DEF_MODE S_IRUSR|S_IWUSER|S_IRGRP|S_IWGRP|S_IROTH|S_IWOTH
      //所有人都能读和写
      #define DEF_UMASK S_IWGRP|S_IWOTH //屏蔽了用户组的写和其他人的写

      umask(DEF_UMASK);
      fd=oepn("foo.txt",O_CREAT|O_TRUNC|O_WRONLY,DEF_MODE);
      //创建了一个新文件，文件的拥有者有读写权利，其他人只有读权限。(屏蔽了用户组的写和其他人的写)
    ```

| 掩码 | 描述 | 八进制表示 |
| :---: | :--- | :--- |
| S\_IRWXU | 使用者\(拥有者\) 能够读 写 执行这个文件 | 00700 |
| S\_IRUSR | 使用者\(拥有者\) 能够读这个文件 | 00400 |
| S\_IWUSR | 使用者\(拥有者\) 能够写这个文件 | 00200 |
| S\_IXUSR | 使用者\(拥有者\) 能够执行这个文件 | 00100 |
| S\_IRWXG | 使用者\(拥有者\) 所在的组成员 能够读 写 执行这个文件 | 00070 |
| S\_IRGRP | 使用者\(拥有者\) 所在的组成员 能够读这个文件 | 00040 |
| S\_IWGRP | 使用者\(拥有者\)  所在的组成员 能够写这个文件 | 00020 |
| S\_IXGRP | 使用者\(拥有者\)  所在的组成员 能够执行这个文件 | 00010 |
| S\_IRWXO | 其他人\(任何人\) 能够读 写 执行这个文件 | 00007 |
| S\_IROTH | 其他人\(任何人\) 能够读这个文件 | 00004 |
| S\_IWOTH | 其他人\(任何人\) 能够写这个文件 | 00002 |
| S\_IXOTH | 其他人\(任何人\) 能够执行这个文件 | 00001 |
| S\_ISUID | 在执行时设置用户ID | 0004000 |
| S\_ISGID | 在执行时设置组 ID | 0002000 |
| S\_ISVTX | 粘着位, 必须拥有该目录的所有权限 700,而且只允许这用户访问 | 0001000 |

`close`函数关闭一个打开的文件

```c
#include <unistd.h>

int close(int fd);

                //返回: 若成功则为0，若出错则为-1
```

关闭一个已关闭的描述符会出错。

##  读和写文件

调用`read`和`write`完成输入输出

```c
#include <unistd.h>

ssize_t read(int fd,void *buf,size_t n);
//read函数从描述符fd的当前文件位置拷贝最多n个字节到存储器buf

                    返回:若成功则为读的字节数，若EOF则为0，若出错为 -1.
ssize_t write(int fd,const void *buf,size_t n)
//write函数从存储器位置buf拷贝至多n个字节到描述符fd的当前文件位置

                    返回:若成功则为写的字节数，若出错则为-1
```

展示了一个程序使用`read`和`write`调用一次一个字节的从`标准输入`拷贝到`标准输出`。

![](http://i.imgur.com/28b8QuN.png)

![](http://i.imgur.com/2W78uw5.png)

通过调用`lseek`函数，应用程序能够显示地修改当前文件的位置

> ssize\_t 和 size\_t 有什么区别  
>
>
> * size\_t:被定义为`unsigned int`
> * ssize\_t:被定义为`int`
>   * 为了出错的时候，返回-1.
>   * 有趣的是，因为这个-1，使得read的最大值减小了一半。

在某些情况，`read`和`write`传送的字节比应用程序要求的要少，有以下原因。  
这样的情况返回的值叫做`不足值`。

* 读时遇到EOF。
* 从终端读文本行\(`stdin`和`STDIN_FILENO`\)
  * `不足值`等于文本行的大小。
* 读和写网络套接字\(`socket`\)
  * 内部缓冲约束和较长的网络延迟会引起`read`和`write`返回不足值。
  * 你向创建健壮的诸如`Web服务器`这样的网络应用，就必须反复调用`read`和`write`处理不足值，知道所有需要的字节传送完毕。

一般的磁盘文件除了`EOF`外，一般不会遇到不足值的问题。

## 用RIO包健壮地读写

`RIO`包: 全称 `Robust I/O`包，健壮的`I/O`包。会自动的处理上文中所述的`不足值`。

提供两类不同的函数:

* **无缓冲的输入输出函数**
  * 直接在存储器和文件之间传送数据，**没有应用级缓冲**。
  * 他们对二进制读写到**网络**和从**网络**读写二进制数据尤其有用。
* **带缓冲的输入函数**
  * 允许你**高效**地从文件中读取文本行和二进制数据。
  * 这些文件内容`缓存`到应用级缓冲区内。
* **带缓冲的`RIO`输入函数是`线程安全`的，在同一个描述符可以交错调用**

###  RIO的无缓冲的输入输出函数

通过调用 rio\_readn 和 rio\_writen 函数, 应用程序可以在内存和文件之间直接传送数据.

```c
这是个自定义的  RIO 函数, 不是系统自带的

ssize rio_readn( int fd, void* usrbuf, size_t n);
ssize rio_writen( int fd, void* usrbuf, size_t n);
    返回: 成功返回传送的字节数, 若 EOF 则为 0 (只对rio_readn 而言), 若出错为 -1
```

* 与普通`read`,`write`区别
  * 在读写`网络套接字`的时候不会产生`不足值`
    * 即`rio_writen`不可能返回`不足值`
  * 线程安全的。
  * 当`wirte`,`read`被应用信号处理程序的返回中断时，允许手动重启。

#### 源代码

rio\_readn\(\)

```c
ssize_t rio_readn(int fd, void *usrbuf, size_t n) 
{
    size_t nleft = n;
    ssize_t nread;
    char *bufp = usrbuf;

    while (nleft > 0) {
	if ((nread = read(fd, bufp, nleft)) < 0) {
	    if (errno == EINTR) /* Interrupted by sig handler return */
		nread = 0;      /* and call read() again */
	    else
		return -1;      /* errno set by read() */ 
	} 
	else if (nread == 0)
	    break;              /* EOF */
	nleft -= nread;
	bufp += nread;
    }
    return (n - nleft);         /* Return >= 0 */
}
```

rio\_writen\(\)

```c
ssize_t rio_writen(int fd, void *usrbuf, size_t n) 
{
    size_t nleft = n;
    ssize_t nwritten;
    char *bufp = usrbuf;

    while (nleft > 0) {
	if ((nwritten = write(fd, bufp, nleft)) <= 0) {
	    if (errno == EINTR)  /* Interrupted by sig handler return */
		nwritten = 0;    /* and call write() again */
	    else
		return -1;       /* errno set by write() */
	}
	nleft -= nwritten;
	bufp += nwritten;
    }
    return n;
}
```

### RIO的带缓冲的输入函数

一个`文本行`就是由一个换行符结尾的`ASCII`码字符序列。

* 在Unix系统里，换行符\(`\n`\)与ASCII码换行符`LF`相等，数字值为`0x0a`

**rio\_readnb和rio\_readlineb 引入**

假设我们要编写一个程序**计算文本文件中文本行的数量**如何实现?

* 方案1: `read`函数一次一个字节地从文件传送到用户存储器，检查每个字节来查找换行符。
  * 效率低，每读取文件中的一个字符都要求陷入内核。
* 更好的方法是调用一个包装函数`rio_readlineb`。
  * 它从一个内部`读缓冲区`拷贝一个文本行。
    * 当缓冲区变空时，会调用`read`重新填满缓冲区。
  * 为什么这样子更快?
    * 利用了空间局部性原理
* 使用`rio_readn`的带缓冲区版本`rio_readnb`.
  * 对于即包含文本行也包含二进制数据的文件\(例如 11.5.3节会提到的`HTTP`响应\).
  * 和`rio_readlineb`一样的读缓冲区中传送原始字节。

**rio\_readinitb 和 rio\_readnb，rio\_readlineb 实例**

每打开一个`描述符`都要调用一次`rio_readinitb`函数。

* 它将描述符`fd`和 **地址`rp`处的一个类型为`rio_t`的读缓冲区**联系起来。

![](http://i.imgur.com/T3iH2Er.png)

* `rio_readlineb(&rio,buf,MAXLINE)` 函数
  * `rio_readlineb` 函数从`rio`\(缓冲区\)读出一个文本行\(包括结尾的换行符\)，将它拷贝到存储器位置`buf`，并用`\0`字符结束这个文本行。
  * `rio_readlineb` 最多读`MAXLINE-1`个字符，其余被截断，末尾永远是`\0`.
* `rio_readnb(&rio,buf,n)`
  * `rio_readnb`函数从`rio`最多读`n`个字节到`buf`
* 对同一描述符，对`rio_readlineb`和`rio_readnb`的调用可以任意交叉。
  * 但是带缓冲的 和 不带缓冲的 不应该交叉引用。

剩余部分给出大量`RIO`函数的实例。

![](http://i.imgur.com/moWl8ko.png)

* 图10-5展示了一个`读缓冲区`的格式，以及初始化它的`rio_readinitb`的代码。
  * `rio_readinitb`函数创建了一个空的读缓冲区，并且将一个打开的文件描述符与之关联。

![](http://i.imgur.com/7FOjo1V.png)

* 图10-6所示的`rio_read`函数是RIO读程序的核心。
  * `rio_read`是`Unix read`函数的带缓冲版本。
    * 当调用`rio_read`要求读`n`个字节.
    * 此时如果缓冲区为空，调用`read`填满，不过也未必会满。
    * 读`缓冲区`内 `min(n,rp->rio_cnt)`个未读字节。
* 对于一个应用程序，`rio_read`和`Unix read`函数拥有相同的语义。
  * 只是可能有时返回的`不足值`可能会不同。
    * 所以如果抛开不足值的话，两者是一样的。
    * 即包装它，让他读满。
    * 即后文的`rio_readn`和`rio_readnb`。
  * 两者的相似性，使得在某些情况也能互相替换。
    * 如后文的`rio_readn`和`rio_readnb`。

![](http://i.imgur.com/gxx93Rv.png)

![](http://i.imgur.com/MEGxdfn.png)

## 读取文件元数据

应用程序能够通过调用`stat`和`fstate`函数，检索到关于文件的信息\(有时也称为文件的`元数据(metadata)`\)

```c
#include<unistd.h>
#include<sys/stat.h>

int stat(const char *filename,struct stat *buf);
int fstat(int fd,struct stat *buf);
//填写stat数据结构中的各个成员
                    返回 : 成功0 ,出错-1
```

```c
	stat 结构体原型：
    struct stat{ 
        dev_t     st_dev;    // 文件的设备编号
        ino_t     st_ino;    // 节点
        mode_t    st_mode;   // 文件的类型和存取的权限
        nlink_t   st_nlink;  // 连接到该文件的硬链接数目, 刚建立的文件值为1
        uid_t     st_uid;    // 用户ID
        gid_t     st_gid;    // 组ID
        dev_t     st_rdev;   // (设备类型) 若此文件为设备文件,则为其设备编号
        off_t     st_size;   //  文件字节大小
        blksize_t st_blksize; // 块大小 (文件系统的 I/O 缓冲区大小)
        blkcnt_t  st_blocks; // 块数
        time_t    st_atime;  // 最后一次访问时间
        time_t    st_mtime;  // 最后一个修改时间
        time_t    st_ctime;  // 最后一次改变时间
     };
```

* `st_size`成员包含了文件的`字节数大小`。
* `st_mode`成员则编码了文件`访问许可位`和`文件类型`
  * 文件类型
    * **普通类型**:就是我们一般所说的`文件`
    * **目录文件**:包含关于其他文件的信息
    * **套接字**: 是一种用来通过网络与其他进程通信的文件。
  * Unix提供的`宏指令`根据`st_mode`来确定文件类型，以下是其中一部分。
    * `S_ISREG()` \#这是一个普通文件吗
    * `S_ISDIR()` \#这是一个目录文件吗
    * `S_ISSOCK()` \#这是一个网络套接字吗
    * 在`sys/stat.h`中定义

图10-10展示了如何使用`宏`和`stat`函数来读取和解释

![](http://i.imgur.com/DjwE5hT.png)

## 读取目录内容

应用程序可以用opendir 函数来打开目录,  用 readdir 系列函数来读取目录的内容,

```c
#include <sys/types.h>
#include <dirent.h>

DIR* opendir(const char* name);
        参数 是路径名
        返回: 若成功 则为指向目录流的指针, 若出错 则为NULL
```

```c
#include <dirent.h>

struct  dirent*  readdir(DIR* dirp);
    返回 :若成功 则为指向下一个目录项的指针, 若没有更多的目录项 或出错,则为NULL
检测错误方法:  首先备份 errno 的值,  然后在 dirent 执行完毕后. 对比目前 errno的值和原值是否相等.
                如果不相等 则表明出错,  如果相等 则表明没有更多目录项了(也就是流结束).
```

```c
目录项指针结构体.
    struct dirent {
        ino_t d_ino ;    // 此目录进入点的Inode
        ff_t  off   ;    // 目录文件开头至此目录进入点的位移
        signed short int d_reclen;  // d_name 的长度, 不含字节NULL
        unsigned char    d_type  ;  // d_name 所指的文件的类型
        har   d_name[256] ;   // 文件名
    };
```

```c
用下面宏定义来判别 目录下的文件都是哪些类型.
    d_type  :
        DT_BLK  - 块设备
        DT_CHR  - 字符设备
        DT_DIR  - 目录
        DT_LNK  - 软链接
        DT_FIFO - 管道
        DT_REG  - 普通文件
        DT_SOCK - 套接字
        DT_UNKNOWN - 未知
```

* `st_size`成员包含了文件的`字节数大小`。
* `st_mode`成员则编码了文件`访问许可位`和`文件类型`
  * 文件类型
    * **普通类型**:就是我们一般所说的`文件`
    * **目录文件**:包含关于其他文件的信息
    * **套接字**: 是一种用来通过网络与其他进程通信的文件。
  * Unix提供的`宏指令`根据`st_mode`来确定文件类型，以下是其中一部分。
    * `S_ISREG()` \#这是一个普通文件吗
    * `S_ISDIR()` \#这是一个目录文件吗
    * `S_ISSOCK()` \#这是一个网络套接字吗
    * 在`sys/stat.h`中定义

图10-10展示了如何使用`宏`和`stat`函数来读取和解释

![](http://i.imgur.com/DjwE5hT.png)

##  共享文件

除非你很清楚`内核`是如何表示打开的文件，否则`文件共享`的概念相当难懂。

`内核`有三个相关的数据结构来表示打开的文件:

* `描述符表(descriptor table)`:
  * 每个进程都有它**独立**的`描述符表`。
  * 它的`表项`是由进程打开的`文件描述符`来索引的。
  * 每个打开的`描述符表项`指向`文件表`的一个表项。
* `文件表`:打开文件的集合是由一张`文件表`表示的。
  * 所有的进程**共享**这张表。
  * 每个`文件表项`的部分组成是
    * 当前的文件位置
    * `引用计数(reference count)`:即当前指向该表项的`描述符项`数。
      * 关闭一个`描述符`会减少相应`文件表表项`中的`引用计数`。
      * 当`引用计数`变为`0`。内核会删除这个文件表表项。
    * 以及一个指向`v-node`表中对应表项的指针。
* `v-node`
  * 所有的进程**共享**这张表。
  * 每个表项包含`stat`结构的大多数信息。
    * `st_mode`
    * `st_size`

打开文件有三种可能的情形:

**最常见的类型**  
![](http://i.imgur.com/NTWwK07.png)

* 就是打开两个不同的文件，且文件磁盘位置也不一样。
* 没有进行**共享**.

**共享情况1**  
![](http://i.imgur.com/YbMDzzQ.png)

* 多个`描述符`也可以通过引用不同的`文件表表项`来引用同一个`文件`。
* 内容相同，`文件位置`不同\(指向的磁盘位置是同一块\)
* 例子
  * 如果以同一个`filename`调用`open`两次，就会出现这种情况。
  * 每个`描述符`都有它自己的文件位置，所以对不同`描述符`的读操作可以从文件的不同位置获取数据。

**子父进程共享情况**  
![](http://i.imgur.com/I9dqQdZ.png)

我们也能理解父子进程如何共享文件。

* 调用`fork`后，子进程有一个父进程`描述符表`副本。
* 父子进程共享相同的打开`文件表`。
  * 共享相同的`文件位置`。
* 一个很重要的**结果**
  * 在内核删除对应文件表表项之前，父子进程**必须都关闭它们的描述符**。
  * 不要以为父进程`close(fd 1)`就好了。
    * 子进程也要`close(fd 1)`

## I/O 重定向

`Linux shell` 提供了 `I/O` 重定位操作符, 允许用户将磁盘文件和标准输入输出联系起来,

* 例如

  ```text
    unix> ls > foo.txt
  ```

  * 使得`shell`加载和执行`ls`程序，将标准输出重定向到磁盘文件`foo.txt`。

* 一个`Web`程序代表客户端允许`CGI`程序时，也执行一种相似类型的重定向。

`I/O`重定向如何工作?

* 使用`dup2`函数

  ```text
    #include<unistd.h>

    int dup2(int oldfd,int newfd);

                返回:若成功则为非负的描述符，若出错则为-1
  ```

  * `dup2`函数拷贝描述符表表项 `oldfd` 到描述符表表项 `newfd` ，覆盖`newfd`。
    * 如果`newfd`已经打开，`dup2`会在拷贝`oldfd`之前关闭`newfd`。\(废话，不是肯定打开吗?\)
    * 也就是说 `newfd` 和 `oldfd` 指向同一个文件表.

![](http://i.imgur.com/dpxhwRt.png)

> 左边和右边的`hoinkies`
>
> * `右hoinkies` : `>`
> * `左hoinkies` : `<`

## 标准I/O

`ANSI C`定义了一组高级输入输出函数，称为`标准I/O`库。

* 这个`库(libc)`提供了
  * 打开和关闭文件的函数\(`fopen`和`fclose`\)
  * 读和写字节\(`fread`和`fwrite`\)
  * 读和写字符串的函数\(`fgets`和`fputs`\)
  * 以及复杂的格式化`I/O`函数 \(`scanf`和`printf`\)
* 标准`I/O`库将一个打开的文件模型化为一个`流`
  * 对于程序员来说，一个`流`就是一个指向`FILE`类型的结构的指针。
  * 每个`ANSI C`程序开始时都有三个打开的`流`
    * `stdin` 标准输入
    * `stdout` 标准输出
    * `stdout` 标准错误

      ```text
        #include<stdio.h>
        extern FILE *stdin;
        extern FILE *stdout;
        extern FILE *stderr;
      ```
* 类型为`FILE`的流是对`文件描述符`和`流缓冲区`的抽象。
  * `流缓冲区`的目的和`RIO读缓冲区`的目的一样
    * 就是使开销较高的`Unix I/O`系统调用的数量尽可能的少。

## 综合 : 我该使用哪些 I/O 函数?

![](http://i.imgur.com/yWLI3IM.png)

图总结了我们讨论过的各种`I/O`包。

* `Unix I/O`。
* `RIO I/O`
* `标准I/O`
  * `磁盘`和终端设备之选。
  * 在`网络输入输出`使用，有一些问题。
    * `Unix`对网络的抽象是一种叫做`套接字`的文件类型。
      * 和任何`Unix`文件一样，`套接字`也是用`文件描述符`来引用的，称为`套接字描述符`。
      * 应用进程通过读写`套接字描述符`来与运行在其他计算机上的进程**通信**。
  * 大多数C程序员，生涯中只用`标准I/O`

**使用的指导原则:**

* **只要有可能就是用标准** `I/O` **. 但是对于 网络套接字就不可以使用这个标准I/O  ,而是使用** `RIO`**或** `unix I/O`
* **不要使用 sacnf 或 rio\_readlineb 来读二进制文件.**
* **对于网络套接字** `I/O` **要使用** `RIO` **或者** `unix I/O`



`标准I/O流` 某种意义上而言是`全双工`的，因为程序能够在同一个`流`上执行输入输出。\(就是清空缓冲区\)然而，对`流`的限制和对`套接字`的限制，有时会互相冲突，而又极少有文档描述这些现象:

* 限制一: 跟在输出函数之后的输入函数。
  * 如果中间没有插入对`fflush`,`fseek`,`fsetpos`或者`rewind`的调用，一个输入函数不能跟在输出函数之后。
    * `fflush`函数清空与流相关的缓冲区。
    * 后三个函数使用`Unix I/O`的`lseek`函数来重置当前的文件位置。
* 限制二: 跟在输入函数之后的输出函数。
  * 如果中间没有插入对`fseek`,`fsetpos`或者`rewind`的调用，一个输出函数不能跟随在一个输入函数之后，除非该输入函数遇到了一个\`EOF。

![](http://i.imgur.com/4pMtV0z.png)

因此，我们建议你在`网络套接字`不要使用`标准I/O`来进行输入和输出。而要使用`RIO 或 unix I/O`

* 如果需要格式化的输出
  * 使用`sprintf`函数在存储器格式化一个字符串。
  * 然后用`rio_writen`把它发送到套接口。
* 如果需要格式化输入
  * 使用`rio_readlinb`来读一个完整的文本行
  * 然后使用`sscanf`从文本行提取不同的字段。

**unix I/O 必标准I/O 更加适用于网络应用程序.**

