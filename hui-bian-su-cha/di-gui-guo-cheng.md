# 递归过程

## 递归过程

```ruby
和调用过程相同, 注意%rbx 和%rbp 等寄存器的 入栈和出栈.

C代码:
    long rfun(unsigned long x){
        if( x== 0)
            return 0;
        unsigned long nx = x >>2 ;
        long rv = rfun(nx);
        return  x+rv;
汇编代码:   x是%rdi,   被放入%rbx中的值是x ( 会pushq 入栈 )
    rfun:
        pushq    %rbx
        movq     %rdi, %rbx
        movl     $0, %eax
        testq    %rdi, %rdi
        je       .L2
        shrq     $2, %rax
    .L2
```

