# 异质的数据结构: 结构struct,联合union

## 异质的数据结构

C语言提供了**两种**不同类型的**对象**组合到一起**创建数据类型的机制**: **`结构`**(`将多个对象集合到一个单位中`), **`联合`**\(`允许用几种不同的类型来引用一个对象`\).

## 结构 struct

将可能不同类型的对象聚合到一个对象中. 用名字来引用结构的各个组成部分,类似于数组的实现.

结构的所有组成部分都存放在内存中一段连续的区域内, 而指向结构的指针**就是结构第一个字节的地址.**

编译器维护关于每个结构类型的信息, **`指示每个字段( field) 的字节偏移`**. 它以这些偏移作为内存引用指令中的**位移**, 从而产生对结构元素的引用.

结构的各个字段的选取完全是在编译时处理的, 机器代码**`不包含`**关于字段声明或字段名字的信息.

```ruby
将指向结构的指针从一个地方传递到另一个地方, 而不是复制它们, 这是很常见的.

下面这个结构包括4个字段, 两个4字节int
struct rec { 
    int i;
    int j;
    int a[2];
    int *p;
};

struct rec A;
struct rec *pA = &A;

*(pA).i  等价于  pA->i  等价于 A.i
// 以成员中 占字节数最大的成员为对齐标准
```

![&#x7ED3;&#x6784; rec &#x7684;&#x6570;&#x636E;&#x504F;&#x79FB;&#x5185;&#x5BB9;](../.gitbook/assets/ping-mu-kuai-zhao-20190804-12.33.45.png)

```ruby
struct rec *r = malloc(sizeof(struct rec));

r->p = & r->a[r->i + r->j];   /* 这条语句翻译成汇编代码如下 */
  开始时 r 在寄存器 %rdi 中.
    movl    4(%rdi),%eax
    addl    (%rdi),%eax    
    cltq				         			将%eax 寄存器4字节的值以补码方式拓展成为 %rax 8字节的值
    leaq    8(%rdi,%rax,4),%rax
    movq    %rax,16(%rdi)



 另一种代码: ( 是一种单链表 )
   C 代码:
     struct ELE{  
          long v;
          struct ELE * p;
      };
      
      long fun(struct ELE* ptr){
           long temp = 0;
           while( ptr != NULL ){
               temp += ptr->v;
               ptr = ptr->p;
           }
           return temp;
       }
   转换成汇编代码:
      fun:
        movl   $0,%eax
        jmp    .L2
      .L3:
        addq   (%rdi), %rax
        movq   8(%rdi),%rdi
      .L2:
        testq  %rdi,%rdi
        jne    .L3            ptr !=  NULL 时跳转
        rep;ret  
```

## 联合 union

联合提供了一种方式能够规避C 语言的类型系统, 允许以多种类型来引用一个对象.

联合和结构它们是用不同的字段来引用相同的内存块.

一个联合总的大小等于它最大字段的大小.

联合还可以用来访问不同数据类型的位模式.

当联合来将不同大小的数据类型结合到一起时, 字节顺序问题就变得很重要.

```ruby
union t { 
    int   cat[2];
    long  tep;
};
    这个t.cat很重要, 在小端法机器上, cat[0] 是 tep 的低4个字节, cat[1] 是 tep 的高4个字节.
    在大端法的机器上正好相反
低               高
_________________
|cat[0] | cat[1]|
-----------------
      tep       |
-----------------
```

### 使用联合和枚举类型来优化二叉树结构

```c
一个普通的二叉树结构体
struct node_s{
    struct node_s *left;
    struct node_s *right;
    double data[2];
}

优化后的联合 和 枚举类型
typedef enum  { N_LEAF, N_INTERNAL } nodetype_t;    /* 用来指示这个是数据还是节点*/
struct node_t{
    nodetype_t type;      /* 节点或者数据判断 */
    union{
        struct {
            struct node_s *left;
              struct node_s *right;
        }internal;
        double data[2];
    }info;
};
```





























