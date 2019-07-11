---
description: centos 7.6 平台
---

# Linux 系统笔记

## 常会用到的命令

想要查看机器代码文件的内容, 可以使用反汇编器\(objdump\).

### gcc

```bash
$gcc -Og -S mastore.c
    #无优化编译文件,生成汇编文件 mastore.s
 
$gcc -Og -c maastore.c
    #无优化编译文件,生成.o二进制文件  mastore.o
    
```

### objdump    反汇编器

```bash
$objdump -d mastore.o
    #对mastore.o文件执行反汇编, 显示可执行部分的汇编内容.(输出内容: 左侧编号是地址,右侧是汇编)
```

### vim

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























