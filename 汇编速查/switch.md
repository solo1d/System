# switch

## switch  整数索引值进行多重分支

```ruby
switch是通过跳转表数组来实现的, 它的效率比if-else 嵌套要高.
 运算符  &&  是指向代码位置的指针 ( 不是 并且运算 ).
#执行switch 语句的关键步骤是通过跳转表来访问代码位置.

C 代码范例:
 void switch_eg(long x, long n, long* dest){
     long val = x;
     switch(n){
        case 100:   val*=13;
                    break;
        case 102:   val +=10;  /* 注意这里没有 brack, 上面也没有101选项 */
        case 103:   val+= 11;
        case 104:                /* 虽然104 没有任何内容, 但他还会向下继续执行*/
        case 106:   val *= val;
                    break;
        default:    val = 0;
        }
    *dest = val;
    }
#############################################
转换成 goto :
  void switch_eg_impl(long x, long n , long* dest){
       static void *jt[7] = { &&loc_A,&&loc_def,&&loc_B,
                                  &&loc_C,&&loc_D,&&loc_def,&&loc_D}; /*仔细看*/
       /* jt 初始化的是代码表指针, 也就是跳转表 */
       unsigned long index = n-100;    /* 选项都是100以上,那么这里-100 也便于计算 */
       long val ;
       if(index > 6 )     /* 所有的switch 只有1~6 这些选项 */
           goto loc_def;
       goto *jt[index];   /* 这步是核心, 他会跳转到指定的代码位置开始执行 */
       
    loc_A:    val = x * 13;  goto done;    /* 100 选项, 也就是 index = 0 */
    loc_B:    x = x+10;                    /* 102 选项, 没有brack 结尾, index = 2 */
    loc_C:    val = x + 11;  goto done;    /* 103 选项,  index = 3 */
    loc_D:    val = x*x;     goto done;    /* 104 选项,  index = 4 */
    loc_def:  vla = 0;                     /* default其他值的默认选项,最上面的if会来到这里*/
    
    done:     *dest = val;     /* 这部分已经不是switch 范围了 */
  }
#############################################
转换成汇编代码:   gcc -S -Og
 /*跳转表声明部分, 这部分声明叫做 rodata 只读数据 */
     .section       .rodata        /* 声明只读数据 */
     .align 8                     /* 将地址对齐到8的倍数 */
  .L4:                            /* 这个L4 是跳转表的地址,也是基址 */
      .quad    .L3         /* 100 选项位置 , 计算方法是  .L4(,x,8) = .L4 + 0 =.L3 */
      .quad    .L8         /* 101 选项位置 */
      .quad    .L5         /* 102 选项位置 */
      .quad    .L6         /* 103 选项位置 */
      .quad    .L7         /* 104 选项位置 */
      .quad    .L8         /* 105 选项位置 */
      .quad    .L7         /* 106 选项位置 */

/* 执行汇编部分, x是%rdi, n是%rsi, dest是%rdx */
 switch_eg:   
     subq     $100, %rsi      执行 index = n-100;
     cmpq     $6, %rsi        执行 index:6 重置条件码寄存器
     ja       .L8             如果结果index >6 那么就执行跳转
     jmp      *.L4(,%rsi,8)   这步很重要  goto *jt[index] ;查看idnex的值,然后计算.
  .L3:
      leaq    (%rdi,%rdi,2),%rax     这里是 loc_A
      leaq    (%rdi,%rax,4),%rdi
      jmp     .L2                    loc_A有break, 所以直接跳出switch, 去*dest=val处.
  .L5:                               这里是 loc_B
      addq    $10,%rdi
  .L6:                               这里是 loc_C
      addq    $11,%rdi
      jmp     .L2
  .L7:                               这里是 loc_D
      imulq    %rdi,%rdi
      jmp      .L2
  .L8:                               这里是 loc_def
      movl    $0,%edi
  .L2:                               这里是 loc_done
      movq    %rdi,(%rdx)
      ret

```







