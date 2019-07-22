# 循环

## 循环

可以用**`条件测试`**和**`跳转组合`** 来实现循环的效果**.**

### do - while 循环

```ruby
通用语句格式:
do
     body-statement
     while (test-expr);
#这个循环效果就是重复执行  body-statement, 对 test-expr 求值,如果求值结果为非零,就继续循环.
#   (这个 body-statement 至少会执行一次)

翻译成 goto 格式:
loop:
     body-statement
     t =  test-expr;
     if(t)
          goto loop;
#每次循环程序会执行循环体内的语句,然后执行测试表达式, 如果测试结果为真,就回去再执行一次循环.

#以下三种形式代码等价:
C代码:   long  fact_do(long n){
            long result = 1;         /*计算阶乘*/
            do{
                result *= n;
                n = n-1;
           }while( n > 1);
           return result;
      }
goto代码:
        long fact_do(long n ){
            long result = 1;
        loop:
            result  *= n;
            n = n-1;
            if( n > 1)
                goto loop;
           return result;
      }
汇编代码:  n是%rdi
       fact_do:
            movl    $1,%rax      将result  = 1
         .L2 
            imulq   %rdi,%rax    乘法运算,  result *= n
            subq    $1, %rdi     减法运算,  n-=1
            cmpq    $1, %rdi     计算减法  n-1  重置条件码寄存器, 整数寄存器%rdi并不会被修改
            jg      .L2          判断,  n > 1 , 根据上面的条件进行判断, 如果为真, 则跳转到L2
            rep;ret;
```

### while 循环

```ruby
通用语句格式:
    while( test-expr )
        body-statement
#在第一次执行 body-statement  之前会先对 test-expr 求值.








```











