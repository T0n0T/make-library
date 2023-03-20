#### c方式编译和c++方式编译
一般.cpp文件是采用g++去编译，.c文件是采用gcc编译，然而这不是绝对的。
1. gcc和g++都可以编译`.c`文件，也都可以编译`.cpp`文件。g++和gcc是通过后缀名来辨别是c程序还是c++程序的(这一点与Linux辨别文件的方式不同，Linux是通过文件信息头辨别文件的)。
2. 在gcc看来，`.c`文件会以c方式去编译，`.cpp`文件则是以c++的方式去编译，注意，gcc不会主动去链接c++用到库stdc++，所以用gcc编译cpp文件时需要手动指定链接选项-lstdc++。而对于g++，不管是`.c`还是`.cpp`文件，都是以c++方式去编译。
3. 还需要注意，并不是说 `__cpluscplus` 是g++编译器才会定义的宏，确切的说，是只有以c++编译的方式去编译文件的宏才会定义的宏，这样说来，gcc编译.cpp文件、g++编译.c、.cpp文件,这个  `__cplusplus`都会被编译器定义。

#### c++调用c程序
1. 假设c程序是之前写好的具有价值的静态库，该库是由add.o和sub.o编译而成，而add.c和sub.c是由c语言写的
- add.h和add.c
```c
//add.h
#ifndef __ADD_H__
#define __ADD_H__
int add(int, int);
#endif /* __ADD_H__ */
==========================
//add.c
#include "add.h"

int add(int a, int b)
{
	return a + b;
}
```

- sub.h和sub.c
```c
//sub.h
#ifndef __SUB_H__
#define __SUB_H__
int sub(int, int);
#endif /* __SUB_H__ */

//sub.c
#include <stdio.h>
#include "sub.h"
int sub(int a, int b)
{
    printf("%d - %d = %d\n", a, b, a - b);
    return 0;
}
```

2. `main.cpp`如下
```c++
//使得add.h、sub.h里面的代码用c语言的方式去编译 
extern "C"{ 
#include "add.h" 
#include "sub.h" 
}
#include <stdio.h>

int main(void)
{
    int c = add(1, 6);
    sub(8, 2);
    printf("c = %d\n", c);
    return 0;
}
```


>  cpp编译器是兼容c语言的编译方式的，所以在编译cpp文件的时候，调用到.c文件的函数的地方时，需要用extern "C"指定用c语言的方式去编译它
>  注意，extern “C”是g++才具有的关键字，gcc并没有，所以如果用gcc编译cpp而不加以宏判断直接使用extern “C”那么就会出现语法错误

3. 编译文件，完成C++调用C函数
```bash
# 打包静态库.a
gcc -c add.c
gcc -c sub.c
ar cqs libadd.a *.o

# 若要编译成动态库，则
gcc -shared -fPIC *.o -o libadd.so

# c++联合c库编译
g++ main.cpp -L./ -ladd -o main
```

