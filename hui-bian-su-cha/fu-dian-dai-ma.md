# 浮点代码, 浮点传送和转换操作

## 浮点代码

处理器的**`浮点体系结构`**包括多个方面, 会影响对浮点数据操作的程序如何被映射到机器上,包括以下内容:

* 如何存储和访问浮点树枝.  通常是通过某种寄存器方式来完成.
* 对浮点数据操作的指令.
* 向函数传递浮点数参数和从函数返回浮点数结果的规则.
* 函数调用过程中保存寄存器的规则 .   `例如:一些寄存器被指定为调用者保存寄存器,而其他的被指定为被调用者保存寄存器.`

#### 媒体\(media\)指令 , \(MMX指令,SSE指令, AVX指令\)

**`媒体指令`**支持图形和图像处理,.  **`这些指令的本意是允许多个操作以并行模式执行,`**称为 **`单指令多数据`** 或 **`SIMD`**.在这种模式中,对多个不同的数据并行执行同一个操作.

近些年来**`媒体指令`**一直在被拓展,从 **`MMX`** 到 **`SSE`**\(流式SIMD拓展\), 以及最新的 **`AVX`**\(高级向量拓展\).

每一代中,都有不同的版本. 每个拓展都是管理存储器组中的数据, 这些寄存器组名称也不相同

* **`MMX`** 中称为 **"MM"** 寄存器     \(64位\)
* **`SSE`** 中称为 **"XMM"** 寄存器  \(128位\)
* **`AVX`** 中称为 **"YMM"** 寄存器  \(256位\)  &lt;可以存放 8个32值,4个8位值, 这些值可以是整数,也可以是浮点数&gt;

2000年 Pentium4 引用 SSE2 , 媒体指令开始包括对那些**`标量浮点数据`**进行操作的指令. 使用**`XMM`** 或 **`YMM`** 寄存器的低32位或64位中的单个值.  这个标量模式提供了一组寄存器和指令, 它们更类似于其他处理器支持浮点的方式.  **所有能够执行 x86-64代码的处理器都支持 SSE2 或更高的版本, `因此 x86-64 浮点数是基于 SSE 或 AVX的`, 包括传递过程参数和返回值的规则.**

#### 以下代码基于 AVX2 , \(2013年Core i7引用\) 可以在GCC 命令中给出 -mavx2  来生成 AVX2代码

\( 基于不同版本的 SSE 以及第一个版本的 AVX 的代码从概念上来说是类似的, 只不过指令名和指令格式有所不用\)



AVX 浮点体系结构允许数据存储在 **16个 YMM** 寄存器中, 他们的名字为 **`%ymm0 ~ %ymm15`** .每个 **YMM** .存储器都是 `256位(32字节)` .**当对标量数据操作时,这些就寄存器`只保存浮点数`. `而且只是用低32位(对于float)`或 `64位(对于 double).`**

### 汇编代码用寄存器的 SSE XMM  寄存器名字 %xmm0 ~%xmm15 来引用它们. 每个 XMM寄存器都是对应的 YMM 寄存器的低 128位\(16字节\).

![&#x5A92;&#x4F53;&#x5BC4;&#x5B58;&#x5668;](../.gitbook/assets/ping-mu-kuai-zhao-20190807-12.19.45.png)

## 浮点传送和转换操作

下面是一组在**`内存和 XMM 寄存器之间`** 以及 **`从一个 XMM 寄存器到另一个不作任何转换的转送浮点数`**的指令.

\(引用内存的指令是标量指令, 意味着它们**`只对单个`**`而不是一组封装好的数据进行操作`.\)

\(数据可以在内存中**`(M32和M64),`**要么在XMM寄存器中\(**`图中 X 表示)`**.  无论数据对齐与否,这些指令都能正确执行, 不**过优化建议是 32位内存数据满足4字节对齐, 64位内存满足8字节对齐**\).

内存引用的制定方式与MOV 指令相同.

#### 所有带有字母 a 的指令,在写入内存时, 内存地址必须保证 16字节 对齐,否则会导致异常.

### 浮点数出传送指令

```ruby
浮点传送指令  :  接受浮点数的寄存器是 %xmm0 ~ %xmm15 这十六个128位寄存器(64*2 或 32*4)
vmovss    32位内存或浮点寄存器,32位内存或浮点寄存器    #传输单精度浮点数(32位),内到寄,或寄到内
vmovsd    64位内存或浮点寄存器,64位内存或浮点寄存器    #传输双精度浮点数(64位),内到寄,或寄到内
vmovaps   浮点寄存器,浮点寄存器       #传送对齐的封装好的单精度数(32), 从寄存器到寄存器
vmovapd   浮点寄存器,浮点寄存器       #传送对齐的封装好的双精度数(64), 从寄存器到寄存器

示例:
  C代码:
    float float_mov(float v1, float *src, float *dst){
        float  v2 = *src;
        *dst   = v1;
        return = v2;
    }
 汇编代码:  v1是%xmm0, src是%rdi, dst是%rsi  ##注意,第一个参数是浮点数寄存器不同
  float_mov:
      vmovaps   %xmm0,%xmm1    将v1的值保存起来,因为返回值是v2,所以%xmm0 应该存储v2
      vmovss    (%rdi),%xmm0   读取src指向在内存中的值,然后存入%xmm0寄存器( v2)
      vmovss    %xmm1,(%rsi)   将v1的值传递到内存, v1 -> %dst
      ret                      将%xmm0 寄存器作为返回值,传递出去.
```

### 浮点数到整数的转换指令

```ruby
浮点数转换成整数时,指令会执行 截断,把值向0进行舍入. (就是完全抛弃小数部分,没有进位)

vcvttss2si   32位内存或浮点寄存器,32位通用寄存器     #把单精度浮点数转换成4字节整数 float -> int
vcvttsd2si   64位内存或浮点寄存器,64位通用寄存器     #把双精度浮点数转换成4字节整数 double-> int
vcvttss2siq  32位内存或浮点寄存器,64位通用寄存器     #把单精度浮点数转换成8字节整数 float -> long
vcvttsd2siq  64位内存或浮点寄存器,64位通用寄存器     #把双精度浮点数转换成8字节整数 double-> long
```

### 整数到浮点数的转换指令

```ruby
第一个参数是整数(源), 第二个与第三个相同都是目的寄存器(目标)
vcvtsi2ss   32位内存或32位通用寄存器,浮点寄存器,浮点寄存器   #4字节整数转换成单精度浮点数 int ->float
vcvtsi2sd   32位内存或32位通用寄存器,浮点寄存器,浮点寄存器   #4字节整数转换成双精度浮点数 int ->double
vcvtsi2ssq  64位内存或64位通用寄存器,浮点寄存器,浮点寄存器   #8字节整数转换成单精度浮点数 long->float
vcvtsi2sdq  64位内存或64位通用寄存器,浮点寄存器,浮点寄存器   #8字节整数转换成双精度浮点数 long->double

例子:  
    vcvtsi2sdq   %rax,%xmm1,%xmm1    
            #从寄存器%rax读取长整型 long,把它转换成双精度double 存入%xmm1寄存器低8字节.
```

### 单/双 精度数互转指令

```ruby
以下两条指令可以执行
         单精度转双精度
      
vunpcklps    %xmm0,%xmm0,%xmm0
    #交叉前两个寄存器的值,放入第三个寄存器, 假设%xmm0[s3,s2,s1,s0], 经过交叉之后结果如下
    #    %xmm0 的值会被更新位 [s1,s1,s0,s0], 其余的被丢弃扔了(每个值都是4字节,共128位)
vcvtps2pd    %xmm0,%xmm0
    #经过上面的交叉, 这里把%xmm0的值进行拓展,变成双精度 
    # %xmmm[s1,s1,s0,s0] 变成了  [ds0,ds0]  (ds0占8字节,共2*8=16字节,共128位)
    #    [s1] 还是被丢弃了

目前最新的指令   单精度转双精度 ( 一条指令就可以完成转换 )
    vcvtss2sd	%xmm0, %xmm0, %xmm0         #指令的 ss表示单精度,sd表示双精度
    



以下两条指令可以执行
        双精度转单精度
vmovddup    %xmm0,%xmm0 
    #假设%xmm0存储值为[x1,x0]两个双精度数(8字节),这条指令会把它变成 [x0,x0] 两个双精度数
vcvtpd2psx  %xmm0,%xmm0
    #这条指令会把%xmm0[x0,x0]两个双精度(8字节) 转换成单精度(4字节),并放到寄存器的低位一半
    # 中,并将高位一半设置为0, 结果得到 %xmm0[0.0, 0.0, x0,x0],这时候x0占4字节


目前最新的指令   双精度转单精度( 一条指令就可以完成转换 )
    vcvtsd2ss  %xmm0,%xmm0,%xmm0         #指令的 ss表示单精度,sd表示双精度



示例:
 C代码:
    double fcvt(int i,float *fp, double *dp,long *lp){
          float f = *fp;     double d = *dp;      long l = *lp;
          *lp = (long)   d;   /* double -> long   双精度转8字节整数*/
          *fp = (float)  i;   /* int  -> float   4字节整数转单精度*/
          *dp = (double) l;   /* long -> double  8字节整数转双精度*/
          retunr (double) f;  /* float -> double 单精度转双精度*/

  汇编代码: i是%edi,  fp是%rsi,  dp是%rdx, lp是%rcx
    fcvt:
      vmovss    (%rsi),%xmm0          #传送单精度的 *fp 到 f  (f是返回值)
      movq      (%rcx),%rax           #取出*lp 的值到%rax , long值 lp被取出暂存.
      vcttsd2siq   (%rdx),%r8         #双精度double d 转8字节整数, 临时存放到%r8
      movq      %r8,(%rcx)            #将%r8 的8字节整数传送给  lp(long)内存中
      vcvtsiss  %edi,%xmm1,%xmm1      #32整数i 转换成单精度,临时存入 %xmm1
      vmovss    %xmm1,(%rsi)          #单精度传送指令, %xmm1 放入到内存 i-> *fp
      vcvtsi2sdq   %rax,%xmm1,%xmm1   #8字节整数转浮点, lp备份值被转成 double
      vmovsd    %xmm1,(%rdx)          #双精度浮点数存入内存  l -> *dp
               #从这里开始就是返回值计算,将从 *fp 取出的值由单精度转双精度
      vunpcklps    %xmm0,%xmm0,%xmm0  #交叉值,高位剔除,低位堆叠 [x1,x1,x0,x0]
      vcvtps2pd    %xmm0,%xmm0        #将单进度进行拓展,变成双精度 [x0,x0]
      ret                             # %xmm0 被当作返回值 进行返回.
```
























