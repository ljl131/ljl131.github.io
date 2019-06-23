---
title: C/C++文件操作
date: 2019-06-23 17:23:07
tags: C语言
---
# C语言对于文件的操作
C语言对于文件操作有两种方式，流式文件操作和I/O文件操作
## 流式文件操作
### 重要的结构
```
typedef struct
{
    short           level;          /* fill/empty level of buffer */
    unsigned        flags;          /* File status flags    */
    char            fd;             /* File descriptor      */
    unsigned char   hold;           /* Ungetc char if no buffer */
    short           bsize;          /* Buffer size          */
    unsigned char   *buffer;        /* Data transfer buffer */
    unsigned char   *curp;          /* Current active pointer */
    unsigned        istemp;         /* Temporary file indicator */
    short           token;          /* Used for validity checking */
}FILE;    /* This is the FILE object */
```
FILE这个结构包含了文件操作的基本属性，对文件的操作都要通过这个结构的指针来进行。
### ANSI C提供的一组标准库函数
#### 打开文件
```
#include <stdio.h>
FILE *fopen(char *pname,char *mode);
打开方式：
"r" 只读，文件必须存在
"w" 只写，若文件存在则长度清0，若文件不存在则建立
"a" 追加，若文件存在则追加到末尾，若文件不存在则建立
"r+" 以读/写方式打开文件，该文件必须存在。如无文件则出错
"w+" 以读/写方式打开文件，若文件存在则文件长度清为零，即文件内容清空。若文件不存在则建立该文件。
上面的每种方式加b，以二进制方式打开文件。
新建文件的权限为0666-umask值

-----------------------------------------------------------------

#include <stdio.h>
FILE *fopen(char *pname,char *mode,FILE *stream);
stream:以打开的文件指针
功能：将原stream锁打开的文件流关闭，然后打开path的文件。
```
#### 三个标准文件
```
stdin 标准输入
stdout 标准输出
stderr 标准错误
```
这三个文件默认是已经打开的。
#### 文件的关闭
```
#include <stdio.h>
int fclose(FILE *fp);
返回值：0，EOF
```
#### 读写一个字符
```
//读一个字符
int fgetc(FILE *fp);
返回读取的字符，EOF
//写一个字符
int fputc(int ch,FILE *fp);
返回写入的字符，EOF
```
#### 读写一个字符串
```
//读取n-1个字符，放到str字符数组中，最后加上一个'\0'
char *fgets(char *str,int n,FILE *fp)
返回字符串的首地址，NULL
注意：
    1、一次读取的最大字符是n-1
    2、遇到换行符或文件结尾时此函数正常返回
    3、fgets在遇到EOF时，返回NULL
//写一个字符串到文件
int fputs(char *str,FILE *fp);
返回写入字符的个数即字符串长度，NULL
```
#### 文件结尾
从文件中读数据在读到文件结尾时要停止。    
1. fgetc()读取文件时发现是文件结尾，返回一个特殊的EOF。  
```
int ch;
while((ch=fgetc(fp)!=EOF))
    fputc(ch,stdout);
```
2. fgets在遇到EOF时，返回NULL，此时要用feof()来判断是否读到了文件结尾
```
char buf[1024]={0};
while(1)
{
    fgets(buf,sizeof(buf),fp);
    if(strlen(buf)>0)
    {
        printf("buf=%s",buf);
        memset(buf,0,sizeof(buf));//每次处理完buf一定要清零
        if(feof(fp))
        {
            break;
        }
    }
}
```
#### 格式化读写数据
```
int fprintf(FILE *fp,char *format,arg_list);
int fscanf(FILE *stream, char *format[,argument...]);
```
#### 二进制读写文件
```
size_t fwrite(*buf,size，nmemb,*fp);
size_t fread(*buf,size.nmemb,*fp);
要写入的内容，块大小，快数目，文件
返回值为块数目，失败返回0；
Tips:
    一般写时块数量为1，读时块大小为1.
```
#### 其他函数
```
//更新缓冲区，不常用，除非需要实时刷新数据

int fflush(FILE* stream);
返回0，EOF
// 返回文件描述符
int fileno(FILE * stream);
```
#### 文件的随机读写
```
//当前读写位置
long ftell(FILE *fp);
//重定位到文章开头
void rewind(FILE *fp);
//移动读写位置
int fseek(FILE *fp,long offset,int base)
offset相对base的偏移量
base:
    SEEK_SET:开头
    SEEK_CUR:当前位置
    SEEK_END:结尾
```
## 直接I/O操作
直接调用系统函数无缓冲区
```
open()         打开一个文件并返回它的句柄
close()        关闭一个句柄
lseek()        定位到文件的指定位置
read()         块读文件
write()        块写文件
eof()          测试文件是否结束
filelength()   取得文件长度
rename()       重命名文件
chsize()       改变文件长度
```
### 1.open()
　　打开一个文件并返回它的句柄，如果失败，将返回一个小于0的值，原型是int open(const char *path, int access [, unsigned mode]); 参数path是要打开的文件名，access是打开的模式，mode是可选项。表示文件的属性，主要用于UNIX系统中，在DOS/WINDOWS这个参数没有意义。其中文件的打开模式如下表。  
**必选项：**
符号 | 含义 
---|---
O_RDONLY | 只读方式 
O_WRONLY | 只写方式
O_RDWR | 读/写方式
**可选项**
符号 | 含义 
---|---
O_NDELAY | 用于UNIX系统 
O_APPEND | 追加方式
O_CREAT | 如果文件不存在就创建
O_TRUNC | 把文件长度截为0 
O_APPEND | 追加方式
O_EXCL 和O_CREAT连用 | 如果文件不存在就创建，如果文件存在返回错误 
O_BINARY | 二进制方式
O_TEXT | 文本方式

对于多个要求，可以用"|"运算符来连接，如O_APPEND|O_TEXT表示以文本模式和追加方式打开文件。  
例：inthandle=open("c:\\msdos.sys",O_BINARY|O_CREAT|O_WRITE)

### 2.close()
关闭一个句柄，原型是int close(int handle);如果成功返回0
例：close(handle)

### 3.lseek()
定位到指定的位置，原型是：long lseek(int handle,long offset, int fromwhere);  
参数offset是移动的量，fromwhere是移动的基准位置，取值和前面讲的fseek()一样，SEEK_SET：文件首部；SEEK_CUR：文件当前位置；SEEK_END：文件尾。此函数返回执行后文件新的存取位置。

例：lseek(handle,-1234L,SEEK_CUR);//把存取位置从当前位置向前移动1234个字节。  
x=lseek(hnd1,0L,SEEK_END);//把存取位置移动到文件尾，x=文件尾的位置即文件长度

### 4.read()
　　从文件读取一块，原型是int read(int handle, void*buf, unsigned len);参数buf保存读出的数据，len是读取的字节。函数返回实际读出的字节。
例：char x[200];read(hnd1,x,200);

### 5.write()
　　写一块数据到文件中，原型是int write(int handle,void *buf, unsigned len);参数的含义同read()，返回实际写入的字节。
例：char x[]="I LoveYou";write(handle,x,strlen(x));

### 7.eof()
　　类似feof()，测试文件是否结束，是返回1，否则返回0;原型是：inteof(int handle);
例：while(!eof(handle1)){……};

### 8.filelength()
　　返回文件长度，原型是long filelength(int handle);相当于lseek(handle,0L,SEEK_END)
例：long x=filelength(handle);

### 9.rename()
　　重命名文件，原型是int rename(const char*oldname, const char *newname); 参数oldname是旧文件名，newname是新文件名。成功返回0
例：rename("c:\\config.sys","c:\\config.w40");

### 10.chsize();
　　改变文件长度，原型是int chsize(int handle, longsize);参数size表示文件新的长度，成功返回0，否则返回-1，如果指定的长度小于文件长度，则文件被截短；如果指定的长度大于文件长度，则在文件后面补''\0''。
例：chsize(handle,0x12345);
setbuffer（设置文件流的缓冲区）
相关函数 setlinebuf，setbuf，setvbuf
表头文件 #include<stdio.h>
定义函数 void setbuffer(FILE * stream,char * buf,size_t size);
函数说明 在打开文件流后，读取内容之前，调用setbuffer()可用来设置文件流的缓冲区。参数stream为指定的文件流，参数buf指向自定的缓冲区起始地址，参数size为缓冲区大小。

### 11.remove()
remove(const char *filename);//删除文件

# C++对文件的操作
## C++的四个流类对象
**cin**:标准输入流
**cout**:标准输出流
**cerr**:标准错误流
**clog**:标准日志流
## 两个操作符号
**<<**:插入运算符
**>>**:提取运算符
## 格式化I/O
### ios类中的枚举常量
#### 设置输入输出格式
1. skipws:跳过当前位置及后面的所有连续的空白符（空格、指标、回车、换行）
2. left,right,internal:左对齐，右对齐，数值符号左对齐而数值本身右对齐，默认为right.
3. dec,otc,hex:十进制、八进制和十六进制。默认十进制
4. showbase:是否显示进制符号(0,0x),默认不设置。
5. showpoint:强制输出小数点和小数点尾部无效的0.
6. uppercase:十六进制和浮点数中的字母为大写，默认不设置，
7. showpos:使正数前带有正号，默认不设置。
8. sicentific,fixed:科学计数法，定点计数法，默认系统根据数值自己确定。
#### 文件打开方式
enum open_mode {in,out,ate,app,trunc,nocreate,noreplace,binany};
ate:打开时文件指针定位到文件尾
nocreate:如果文件不存在也不创建文件，直接打开失败。
noreplace:如果文件已存在则打开失败
#### 文件指针的定位操作
enum seek_dir {beg,cur,end};
### ios成员函数
## 文件流
头文件：<fstream.h>提供三个类:ifstream,ofstream,iofstream来定义用户所需要的文件对象。
### 文件的打开与关闭
打开：
```
void open(const char * filename,int mode,int port);
port决定文件的访问方式：
0：普通文件（默认值）
1：只读文件
2：隐含文件
4：系统文件
一般情况下保持默认
```
关闭：
```
对象.close()
```
### 文件的读写
文件的读写可直接使用流运算符(>>,<<)或流成员函数
输出到文件：put,write  
从文件输入：get,getline,read
### 文件的随机读写
1. 输出流随机访问函数
```
seekp(<流中的位置>);
seekp(<偏移量>,<流中的位置>);
tellp();//直接定位到开头
```
2. 输入流随机访问函数
```
seekg(<流中的位置>);
seekg(<偏移量>,<流中的位置>);
tellg();//直接定位到开头
```