# 浮点数初始化和运算指令

## 初始化浮点数

和整数运算操作不同, AVX浮点操作**`不能以立即数作为操作数`**, 相反, 编译器**必须为所有的常量值分配和初始化内存空间**.  然后把代码在把这些值**`从内存中读入`**.   下面的摄氏度到华氏度转换的函数就可以说明问题.

```ruby
C代码:
double  cel2fahr(double tmep){
    return 1.8 * temp +32.0;        /* 华氏度= 1.8 *摄氏度 +32.0;  */
}
汇编代码如下:   temp是%xmm0
    cel2fahr:
        vmulsd    .LC2(%rip),%xmm0,%xmm0    #双精度乘法运算,从.LC2位置读取浮点数1.8
        vaddsd    .LC3(%rip),%xmm0,%xmm0    #双精度加法运算,从.LC3位置读取浮点数32.0
        ret
     .LC2:
         .long   3435973837    #这个在%xmm0寄存器的低四字节(这是小端存储,如果是大端存储应该在高位)
         .long   1073532108    #这个在%xmm0寄存器的高四字节(这里还是小端存储, 大端应该在低位)
     .LC3:
         .long   0             # 和上面相同, 值都是十进制的
         .long   1077936128
         
可以看出汇编代码从.LC2和.LC3 内存位置读取浮点数,每一对.long 声明和十进制表示的值,一个低4位,一个高4位.

把这些.long 数解释为浮点值:
    标号 .LC2 的声明有两个值: 3435973837(0xcccccccd) 和1073532108(0x3ffccccc).
        因为机器采用的是小端法字节顺序, 第一个值给出的是低4字节,第二个是高4字节.
         从高位字节可以抽取指数字段为 0x3ff(1024), 减去偏移量 1023 得到指数0.
                 ( E=(e-(2^10 -1)) = 1023-(1024-1) = 0 )
             将两个字段连接起来, 得到小数字段 0xccccccccccccd, 二进制小数表示为0.8
                 完整的浮点数字段是    0x3ffccccccccccccd = 1.8
```

## 加减乘除 和 平方根

```ruby
浮点数运算分为单精度和双精度, 有一个操作数和一个目的地址, 或者两个操作数和一个目标地址.
浮点运算的目标地址必须是 XMM 寄存器


# S1 和 S2 表示源操作数(内存或寄存器),  D 是目的 XMM 寄存器
vaddss    S1, S2, D      #单精度加法运算, S2+S1 结果写入 D
vaddsd    S1, S2, D      #双精度加法运算, S2+S1 结果写入 D

vsubss    S1, S2, D      #单精度减法运算, S2-S1 结果写入 D
vsubsd    S1, S2, D      #双精度减法运算, S2-S1 结果写入 D

vmulss    S1, S2, D      #单精度乘法运算, S2*S1 结果写入 D
vmulsd    S1, S2, D      #双精度乘法运算, S2*S1 结果写入 D

vdivss    S1, S2, D      #单精度除法运算, S2 / S1 结果写入 D
vdivsd    S1, S2, D      #双精度除法运算, S2 / S1 结果写入 D

vmaxss    S1, S2, D      #单精度浮点最大值, S2和S1 谁的值大, 谁就写入 D
vmaxsd    S1, S2, D      #双精度浮点最大值, S2和S1 谁的值大, 谁就写入 D

vminss    S1, S2, D      #单精度浮点最小值, S2和S1 谁的值小, 谁就写入 D
vminsd    S1, S2, D      #双精度浮点最小值, S2和S1 谁的值小, 谁就写入 D

sqrtss    S1, D        #求 S1 浮点数平方根, 结果写入 D



例子:
  C代码:
    double funct(double a, float x, double b, int i){
          return a*x - b\i;
    }

  汇编代码:  a是%xmm0, x是%xmm1, b是%xmm2, i是%rdi
    funct:   
       vunpcklps   %xmm1,%xmm1,%xmm1     #将 x转成双精度浮点数 ,准备和a进行运算
       vcvtps2pd   %xmm1,%xmm1           # x 已经转成 双精度浮点数了
       vmulsd      %xmm0,%xmm1,%xmm0     # a和x 进行双精度乘法运算,结果存储a 
       vcvtsi2sd   %edi, %xmm1,%xmm1     # 将i的值转换成双精度浮点数,存入%xmm1
       vdivsd      %xmm1,%xmm2,%xmm2     # 双精度除法运算, b/i ,(%xmm2)/(xmm1).
       vsubsd      %xmm2,%xmm0,%xmm0     # 双精度减法运算, (a*x)-(b/i) 结果给xmm0
       ret                               # 返回值是double,所以%xmm0存储的是返回值
```

## 位操作

```ruby
位操作指令作用于封装好的数据,即它们会更新整个目的XMM寄存器,对两个源寄存器的所有位都实施指定的位级操作.
 也就是说以下的位操作指令对一个 XMM 寄存器中的所有128 位进行布尔操作.

vxorps  32位内存或寄存器s1,32位内存或寄存器s2,XMM寄存器     #单精度 位级异或   s2 ^ s1
vxorpd  64位内存或寄存器s1,64位内存或寄存器s2,XMM寄存器     #双精度 位级异或   s2 ^ s1

vandps  32位内存或寄存器s1,32位内存或寄存器s2,XMM寄存器     #单精度 位级与或   s2 & s1
vandpd  64位内存或寄存器s1,64位内存或寄存器s2,XMM寄存器     #单精度 位级与或   s2 & s1


例子:
 double simplefun(double x){
     x = x^0x80000000000000;          /* 将x的符号位反过来 */
     return x & 0x7FFFFFFFFFFFFFFF;    /* 将x的符号位置空, 也就是变成0 */
}
汇编代码:
 simplefun:
     vmovsd   .LC1(%rip),%xmm1
     vxorpd   %xmm1,%xmm0,%xmm0
     vmovsd   .LC2(%rip),%xmm1
     vandpd   %xmm1,%xmm0,%xmm0
  .CL1:
      .long   0                   #存储在低位 
      .long   -2147483648         #这个是十进制,变成十六进制是 0x80000000000000
      .long   0                   #按顺序以此存储在高位,  这是两个double值.
      .long   0            
  .CL2:     
      .long   4294967295          #十六进制是 0xffffffff
      .long   2147483647          #十六进制是 0x7fffffff
      .long   0
      .long   0

```

## 浮点比较操作

```ruby
浮点数的比较指令与 cmpq 类似, 不会修改寄存器或者内存, 只会修改条件码.
AVX2 提供了两条用于比较浮点数值的指令: 

vucomiss     32位内存或寄存器s1,必须是XMM寄存器s2    #单精度比较, s2 - s1
vucomisd     64位内存会寄存器s1,必须是XMM寄存器s2    #双精度比较, s2 - s1

上面指令的第二个参数 必须是 XMM 寄存器. 第一个参数可以是寄存器也可以是内存.

浮点比较指令会设置三个条件码: ZF零标志位, CF进位标志位, PF奇偶标志位(偶1,奇0).
    对于整数的操作最近的一次算数或逻辑运算产生的值 的最低位字节是偶校验位.(即这个字节中有偶数个1)
    但是对于浮点比较,当两个操作数中的任意一个是NaN(不是一个数)时, 会把 PF奇偶标志位 设置为1.
        在C语言中如果有个参数是 NaN 那么就认为比较失败,例如,当x为NaN时,比较 x==x 就会得到0.

条件码的设置条件如下:
    顺序 S2:S1        CF     ZF    PF
     无序的            1      1     1            #任意一个操作数为NaN时,就是无序的,结果为0
     S2 < S1          1      0     0
     S2 = S1          0      1     0
     S2 > S1          0      0     0


例子:
    typedef enum {NEG, ZERO, POS, OTHER } range_t;
    reange_t find_range(float x){
         int result;
         if( x < 0 )
             result NEG;       /* 0 */
         else if ( x == 0 )
             result = ZERO;    /* 1 */
         else if (x > 0)
             result = POS;     /* 2 */
         else 
             result = OTHER;   /* 3 */
         return result;
     } 
 汇编:        gcc -S -Og -mavx2  find_range.c
     find_range:
         vxorps     %xmm1,%xmm1,%xmm1    #使用异或运算将xmm1 (临时值)变成0
         vucomiss   %xmm0,%xmm1          #进行对比 x-%xmm1(0) 来 重置符号位, 单精度
         ja         .L5                  #当 x > %xmm1 时跳转
         vucomiss   %xmm1,%xmm0          #镜像对比 %xmm1(0) - x 重置符号位,单精度
         jp         .L8                  #通过判断PF条件码来跳转(偶1,奇0), x == NaN 时跳转
         movl       $1,%eax              #将 result的值赋值为1
         je         .L3                  #还是判断 x == %xmm1(0)  ,相等时跳转
     .L8:
         vucomiss   .LC0(%rip),%xmm0     #x是NaN(不是一个数)时跳转到这里,内存中取值给x,单精度,如果前面没有跳转,那么也会来到这里.
         setbe       %al                 #x==NaN时将al设置为1, 前面三个跳转指令都没有执行时, al设置为0
         movzbl      %al,%eax            #将 %rax 的高位都置为0
         addl        $2,%eax             #%eax+2 结果=2时是x<0, 结果=3时是x== NaN
         ret                             #执行返回操作, POS 和 OTHER
     .L5:
         movl        $0,%eax             # x>0时跳转到这里, 返回值设置为0
     .L3:
         rep;ret                         #x==0 时跳转到这里, 返回0
```











