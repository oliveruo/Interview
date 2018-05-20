### 其他

---
3.1为什么使用补码

我们都知道计算机中的负数是用补码来表示的，而负数的补码是原码符号位不变，其他位按位取反再加一。但是为什么必须这样？为什么非要取反再+1？这个定义是怎么来的？

首先我们用我们熟悉的十进制来思考问题，假设现在我们只考虑两位数字的运算，比如56 + (-28) = 56 - 28，这里如果作正常的减法运算，就需要借位，也就是先让"6"减去"8"的时候，发现不够减，所以要向高位“借一位”，所以"5"只能借一位给“6”，然后“5”变成“4”，这样的“借位规则”如果用电路来实现，会很复杂（至于如何用电路实现，感兴趣的同学可以去看《编码》的一到十二章)，但幸运的是我们有办法来避免这样的借位：

56 + (-28) = 56 - 28 + 99 - 99       //加99再减99表达式值不变

           = 56 - 28 + 99 + 1 - 100  //把-99写成1 - 100

           = (99 - 28 + 1) + 56 - 100  //交换一下位置         

如上面最后一步所示，如果按照其从左到右的顺序依次运算，是完全不需要借位的，这里的99其实可以看成“两位数字能表示的最大的数值“，如果是三位数字，那就是999。最后在减去100之前，其实已经”溢出“了，这时候只要把最高位舍去就行了。

换到二进制也是一个道理，假设我们只能用八位数字来表示一个数值，那么上面的28写成二进制就是00011100，然后我们要用“八位数字能表示的最大的数值”减去它，就是11111111 - 00011100，这时候你会发现，这样做的结果刚好就是00011100全部都取反一下（0变1,1变0），变成11100011，然后再加一变成11100100，这时候再加上56（也就是二进制的00111000）:

    11100100

    00111000

---

   100011100

然后把结果中多出来的一位最高位舍去，就得到了00011100，也就是十进制的28（56 - 28 = 28）！

写到这里我想大家应该已经明白了，只要把-28（二进制的-00011100）用72（二进制的11100100）来表示，就能直接用加法得到正确结果了。这也是为什么负数的符号位是1的原因，因为，比如用八位二进制来表示一个数值，如果要包含负数的话，那我们只能舍弃一半原来的正数，让这部分正数来表示负数，很自然地我们用“更大的那一半”来表示负数，比如原来可以表示0到256，现在我们就把128到256“分给负数”，剩下的0到127继续表示正数（如果用128到256来表示正数，0到127表示负数，那么0到127和-127到0这部分数值就没人表示了）。所以，“一串01”在计算机中到底表示几，还得看它能不能表示负数。

3.2C语言中的内存泄漏

 (1)内存申请未成功，然后进行使用；//在编程时经常用if (p == NULL) 进行判断；

 (2)内存申请成功，但是没有初始化，会造成内存出错；

 (3)内存初始化成功，但是操作越界，比如在数组的操作当中加一；char a [5] = "hello";会造成段错误,没有考虑到'\0'的存储空间，若越界访问的内存空间是空闲的，则程序可能不受影响。若空间已经被占用，若执行了非法操作，程序可能奔溃。

(4)忘记释放内存或者释放一部分则会造成内存泄露。

3.3进制转换

任意进制转换为10进制；

long  int toTen(char a[], int bit)

		 {

		     int length = strlen(a);

		     int i, b = 1, sum = 0; //i要做数组a的下标，注意其起止范围 

		     for (i = length - 1; i >= 0; i--)

			     {

			         if (a[i] >= 'A')

				         {

				             sum += (a[i] - 'A' + 10) *b;

				             b *= bit;

				         }

			         else

				         {

				             sum += (a[i] - '0') *b;

				             b *= bit;

				         }

			     }

		     return sum;

		 }

十进制转换为任意进制（利用栈实现）

    #include<iostream>
    #include<stack>
    using namespace std;
    
    int main()
    {
        int r,n;
        stack<int> s;
        cin>>n>>r;
        while(n)
        {
            s.push(n%r);
            n/=r;
        }
        while(!s.empty())
        {
            switch(s.top())
            {
                case 10:cout<<'A';break;
                case 11:cout<<'B';break;
                case 12:cout<<'C';break;
                case 13:cout<<'D';break;
                case 14:cout<<'E';break;
                case 15:cout<<'F';break;//这些都是为了能转化为十六进制。
                default:cout<<s.top();break;
            }
            s.pop();
        }
        return 0;
     } 
    
    

任意进制转换成任意进制

int main()

{

	char s1[] = "E";

	int a = toTen(s1, 16);//利用前面10进制转换成任意进制

	cout << a << endl;

	char temp[200];

	int b= 15;

	_itoa(a, temp, 16);//能够将十进制的int型转换成任意进制的字符串输出 

	cout << temp;

        system("pause");

	return 0;

}

3.4自己编写strlen/strcpy/strcmp/strcat

一、字符串拷贝strcpy

函数strcpy的原型是char* strcpy(char* des , const char* src)，des 和 src 所指内存区域不可以重叠且 des 必须有足够的空间来容纳 src 的字符串。

#include <assert.h>

#include <stdio.h>  

char* strcpy(char* des, const char* src)  

{  

    assert((des!=NULL) && (src!=NULL));   

    char *address = des;    

    while((*des++ = *src++) != '\0')    

        ;    

    return address;  

}  

要知道 strcpy 会拷贝'\0'，还有要注意：

- 源指针所指的字符串内容是不能修改的，因此应该声明为 const 类型。
- 要判断源指针和目的指针为空的情况，思维要严谨，这里使用assert（见文末）。
- 要用一个临时变量保存目的串的首地址，最后返回这个首地址。
- 函数返回 char* 的目的是为了支持链式表达式，即strcpy可以作为其他函数的实参。

二、字符串长度strlen

函数strlen的原型是size_t strlen(const char *s)，其中 size_t 就是 unsigned int。

#include <assert.h>

#include <stdio.h>  

int strlen(const char* str)  

{  

    assert(str != NULL);  

    int len = 0;  

    while((*str++) != '\0')  

        ++len;  

    return len;  

}  

strlen 与 sizeof 的区别：

- sizeof是运算符，strlen是库函数。
- sizeof可以用类型、变量做参数，而strlen只能用 char* 变量做参数，且必须以\0结尾。
- sizeof是在编译的时候计算类型或变量所占内存的大小，而strlen的结果要在运行的时候才能计算出来，用来计算字符串的长度。
- 数组做sizeof的参数不退化，传递给strlen就退化为指针了。

三、字符串连接strcat

函数strcat的原型是char* strcat(char* des, char* src)，des 和 src 所指内存区域不可以重叠且 des 必须有足够的空间来容纳 src 的字符串。

#include <assert.h>  

#include <stdio.h>    

char* strcat(char* des, const char* src)   // const表明为输入参数   

{    

    assert((des!=NULL) && (src!=NULL));  

    char* address = des;  

    while(*des != '\0')  // 移动到字符串末尾  

        ++des;  

    while(*des++ = *src++)  

        ;  

    return address;  

}   

四、字符串比较strcmp

函数strcmp的原型是int strcmp(const char *s1,const char *s2)。

- 若s1==s2，返回零；
- 若s1>s2，返回正数；
- 若s1<s2，返回负数。

即：两个字符串自左向右逐个字符相比（按ASCII值大小相比较），直到出现不同的字符或遇\0为止。

#include <assert.h>  

#include <stdio.h>  

int strcmp(const char *s1,const char *s2)  

{  

    assert((s1!=NULL) && (s2!=NULL));  

    while(*s1 == *s2)  

    {  

        if(*s1 == '\0')  

            return 0;             

        ++s1;  

        ++s2;  

    }  

    return *s1 - *s2;  

}   

附：assert()断言(调试常用)

assert是宏，而不是函数。它的原型定义在头文件 assert.h 中：

1. void assert( int expression );  

宏 assert 经常用于在函数开始处检验传入参数的合法性，可以将其看作是异常处理的一种高级形式。assert 的作用是先计算表达式expression，然后判断：

- 如果表达式值为假，那么它先向stderr打印错误信息，然后通过调用 abort 来终止程序运行。
- 如果表达式值为真，继续运行后面的程序。

注意：assert只在 DEBUG 下生效，在调试结束后，可以通过在#include <assert.h>语句之前插入#define NDEBUG来禁用assert调用。

5.C、C++以及Java之间的区别和各自优缺点

C/C++：

           

C/C++代码——编译（不同的系统编译出不同的机器码，所以同一个C/C++文件不一定可以在某些系统执行，因为编译出的机器码不同）——机器码————在操作系统中由硬盘读取到内存中运行——内存——CPU——输出结果

Java：             

Java代码————编译得到字节码文件(.class)————JVM执行字节码文件（字节码在虚拟机上运行，虚拟机相当于翻译官，不同的系统JVM不同，转换规则不同，把同一个字节码文件转换为相应的系统的机器码）————机器码在相应系统运行——内存——CPU——结果

由于JVM的存在，只需在不同的系统上安装相应的JVM，同一个.class文件在相应的系统的JVM运行就会输出相应系统能解析的机器码，从而成功运行。

这就是，一次编译，到处运行。

打个比方，就是：

一本汉字写的书（源码），去到不同的国家（系统），每个国家有相应的翻译官（JVM虚拟机）把汉字翻译成其所在国家的文字（比如这本书传到英国被翻译成英文书），之后就可以在这个国家流传开了（相当于程序成功运行）。

另外，补充一点： 

JAVA有两特性：

移植性：一次编译，到处运行（上面已解释）

安全性：自动回收内存中不常用的命令垃圾，防止内存溢出。

#### 参考资料

           https://blog.csdn.net/melody_1016/article/details/52966355

           https://www.oschina.net/translate/cpp-virtual-inheritance

           https://blog.csdn.net/lisonglisonglisong/article/details/44278013
           《编码》

>Contributes: oliveruo
>
>Reviewers : Hollis, Kevin Lee
