# C语言文件操作

**本章重点**

- 文件的打开和关闭
- 文件的顺序读写
- 文件的随机读写
- 文本文件和二进制文件
- 文件读取结束的判定

# 1.文件的打开和关闭

## 1.1 文件指针

缓冲文件系统中，关键的概念是“文件类型指针”，简称“**文件指针**”。

我们可以创建一个FILE*的指针变量:

```c
FILE* pf;//文件指针变量
```

定义pf指针是一个指向FILE类型数据的指针变量。

可以使pf指向某个文件的文件信息区（是一个结构体变 量）。通过该文件信息区中的信息就能够访问该文件。也就是说，**通过文件指针变量能够找到与它关联的文件。**

## 1.2 文件的打开和关闭

文件在读写之前应该先打开文件，在使用结束之后应该关闭文件。

ANSIC 规定使用fopen函数来打开文件，fclose来关闭文件。

```c
//打开文件
FILE * fopen ( const char * filename, const char * mode );
//关闭文件
int fclose ( FILE * stream );
```

**打开方式：**

![image-20221107173132134](C:\Users\14521\AppData\Roaming\Typora\typora-user-images\image-20221107173132134.png)

# 2.文件的顺序读写

## 2.1 文件的顺序读写

| 功能           | 函数名  |   适用于   |
| :------------- | :-----: | :--------: |
| 字符输入函数   |  fgetc  | 所有输入流 |
| 字符输出函数   |  fputc  | 所有输出流 |
| 文本行输入函数 |  fgets  | 所有输入流 |
| 文本行输出函数 |  fputs  | 所有输出流 |
| 格式化输入函数 | fscanf  | 所有输入流 |
| 格式化输出函数 | fprintf | 所有输出流 |
| 二进制输入     | fwrite  |    文件    |
| 二进制输出     |  fread  |    文件    |

## 2.2 对比一组函数

> scanf/fscanf/sscanf 
>
> printf/fprintf/sprintf

参考：[学习笔记--输入输出函数对比]([my_blog/学习笔记--输入输出函数对比.md at main · 1452115078/my_blog (github.com)](https://github.com/1452115078/my_blog/blob/main/c-blog/学习笔记--输入输出函数对比.md))

# 3.文件的随机读写	

## 3.1 fseek

定义：根据文件指针的位置和偏移量来定位文件指针。

```c
int fseek ( FILE * stream, long int offset, int origin );
```

**orign：**

![image-20221107175034019](C:\Users\14521\AppData\Roaming\Typora\typora-user-images\image-20221107175034019.png)

<u>examples：</u>

```c
#include <stdio.h>
int main ()
{
  FILE * pFile;
  pFile = fopen ( "example.txt" , "wb" );
  fputs ( "This is an apple." , pFile );
  fseek ( pFile , 9 , SEEK_SET );
  fputs ( " sam" , pFile );
  fclose ( pFile );
  return 0;
}
```

## 3.2 ftell

定义：返回文件指针相对于起始位置的偏移量

```c
long int ftell ( FILE * stream );
```

<u>examples：</u>

```c
#include <stdio.h>
int main ()
{
  FILE * pFile;
  long size;
  pFile = fopen ("myfile.txt","rb");
  if (pFile==NULL) perror ("Error opening file");
  else
 {
    fseek (pFile, 0, SEEK_END);   // non-portable
    size=ftell (pFile);
    fclose (pFile);
    printf ("Size of myfile.txt: %ld bytes.\n",size);
 }
  return 0;
}
```

## 3.3 rewimd

定义：让文件指针的位置回到文件的起始位置

<u>examples：</u>

```c
#include <stdio.h>
int main ()
{
  int n;
  FILE * pFile;
  char buffer [27];
  pFile = fopen ("myfile.txt","w+");
  for ( n='A' ; n<='Z' ; n++)
    fputc ( n, pFile);
  rewind (pFile);
  fread (buffer,1,26,pFile);
  fclose (pFile);
  buffer[26]='\0';
  puts (buffer);
    return 0;
}
```

# 4.文本文件和二进制文件

根据数据的组织形式，数据文件被称为**文本文件**或者**二进制文件**。 

数据在内存中以二进制的形式存储，如果不加转换的输出到外存，就是**二进制文件**。 

如果要求在外存上以ASCII码的形式存储，则需要在存储前转换。以ASCII字符的形式存储的文件就是**文本文件**。

 一个数据在内存中是怎么存储的呢？

如有整数10000，如果以ASCII码的形式输出到磁盘，则磁盘中占用5个字节（每个字符一个字节），而 二进制形式输出，则在磁盘上只占4个字节

```c
#include <stdio.h>
int main()
{
    int n = 10000;
    FILE* pf = fopen("test.txt", "wb");
    if (!pf)
    {
        perror("fopen()");
        return 1;
    }
    //
    fwrite(&n, 4, 1, pf);

    fclose(pf);
    pf = NULL;
    return 0;
}
```

![image-20221107184405648](C:\Users\14521\AppData\Roaming\Typora\typora-user-images\image-20221107184405648.png)

# 5.文件读取结束的判定 

## 5.1  被错误使用的feof

**重点：在文件读取过程中，不能用feof函数的返回值直接用来判断文件的是否结束。**

而是应用于当文件读取结束的时候，判断是读取失败结束，还是遇到文件尾结束。

1. 文本文件读取是否结束，判断返回值是否为 EOF （ fgetc ），或者 NULL （ fgets ）

   例如：

   -  fgetc 判断是否为 EOF 
   - fgets 判断返回值是否为 NULL 。

2. 二进制文件的读取结束判断，判断返回值是否小于实际要读的个数。

   例如： 

   - fread判断返回值是否小于实际要读的个数。

**<u>二进制文件的例子：</u>**

```
#include <stdio.h>
enum { SIZE = 5 };
int main(void)
{
    double a[SIZE] = {1.,2.,3.,4.,5.};
    FILE *fp = fopen("test.bin", "wb"); // 必须用二进制模式
    fwrite(a, sizeof *a, SIZE, fp); // 写 double 的数组
    fclose(fp);
    double b[SIZE];
    fp = fopen("test.bin","rb");
    size_t ret_code = fread(b, sizeof *b, SIZE, fp); // 读 double 的数组
    if(ret_code == SIZE) {
        puts("Array read successfully, contents: ");
        for(int n = 0; n < SIZE; ++n) printf("%f ", b[n]);
        putchar('\n');
   } else { // error handling
       if (feof(fp))
          printf("Error reading test.bin: unexpected end of file\n");
       else if (ferror(fp)) {
           perror("Error reading test.bin");
       }
   }
    fclose(fp);
}
```

