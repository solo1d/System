# 循环 和 switch

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

翻译成goto代码:
    t = test-expr;
    if( !t )
        goto done;
    loop:
        body-statement
        t = test-expr;
        if(t)
            goto loop;
    done:
    
#while有两种实现策略, 一种是 跳转到中间(使用gcc -Og), 一种是guarded-do(使用gcc -O1)

C代码:
    long fact_while(long n){
        long result = 1;
        while( n > 1){
            result  *= n;
            n = n -1;
        }
        return result;
    }
两种goto模式和对应的汇编,  跳转到中间:
            long fact_while_jm_goto(long n ){
                long result = 1;
                goto test;
              loop:
                result *= n;
                n = n-1;
              test:
                if( n > 1 )
                    goto loop;
                return result;
            }
     汇编:    n是 %rdi
         fact_while:
             movl    $1,%rax     设置result = 1
             jmp     .L5          goto test
          .L6
             
          

第二种gouto模式和对应汇编, guarded-do :
            long fact_while_gd_goto(long n ){
               long result = 1;
               if( n <= 1)        /* 这个判断很重要 */
                  goto done;
            loop:
               result *= n;
               n = n-1;
               if( n!= 1 )
                   goto loop;
            done:
                return result;
            }
     汇编:    n是 %rdi
          fact_while:
              cmpq    $1,%rdi     对比 n-1
              jle     .L7         if  n<=1  goto 到 done
              movl    $1,%rax     设置 result =1
          .L6:                    这里是 loop:
              imulq   %rdi,%rax   乘法 result*=n;
              subq    $1,%rdi     n递减
              cmpq    $1,%rdi     对比 n-1
              jne     .L6         if n!= 1
              rep;ret
          .L7:
              movl    $1,%rax
              ret
```

### for 循环

```ruby
for基本格式:
for(init-expr; test-expr; update-expr)
      body-statement;


有基于 while 两种模式的转换, 取决于优化等级.
跳转中间策略:  gcc -S -O1
    init-expr;
    goto  test;
 loop:
    body-statement
    update-expr;
 test:
    t = test-expr;
    if(t)
       goto loop;

而guarded-do 策略:  gcc -S -Og
   init-expr;
   t = test-expr;
   if(!t)
      goto done;
loop:
   body-statement;
   update-expr;
   t = test-expr;
   if(t)
      goto loop;
done:


最根本的操作还是将for 转换成 while , 然后再转换汇编代码.

C 代码:
   long fact_for(long n ){
      long i ;
      long restult = 1;
      for(i = 2; i <= n; i++)
         restult *= i;
      return result;
   }

翻译成 while 代码:
    long fact_while(long n ){
       long i =2;
       long result = 1;
       while( i <= n){
          result *= i;
          i++;
       }
       return result;
    }

GOTO 代码 的一种翻译形式:  跳转到中间形式.
   long fact_goto(long n ){
      long i = 2;
      long result = 1;
      goto test;
   loop:
      result *= i;
      i++;
   test:
      if( i <= n )
         goto loop;
      return result;
   }
   
翻译成汇编代码:
   n是 %rdi
      fact_for:
         movl   $1,%eax
         movl   $2,%edx
         jmp    .L8
      .L7:
         imulq   %rdx,%rax
         addq    $1,%rdx
      .L8:
         cmpq    %rdi,%rdx
         jle     .L9
         rep;ret
```

## switch  整数索引值进行多重分支

```ruby
switch是通过跳转表数组来实现的, 它的效率比if-else 嵌套要高.
 运算符  &&  是指向代码位置的指针 ( 不是 并且运算 ).







```







