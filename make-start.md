`make` 不管对于大多数静态语言，作为构建脚本都有着非常重要的作用；
现在最流行的是 `gnu-make`，基本概念使用是使用 `make` 脚本，调用 `make tool` 间接调用了目标语言的编译器进行操作。

### 基本语法

使用 `:` 表示前者依赖于后者，并在其后跟随命令 
```shell
    target ... : prerequisites ...  
            command  
            ...  
            ...
```

##### 概览
```shell
# 设置目标源文件
SRC = $(wildcard *.c)
# 设置构建依赖
OBJ = $(patsubst %.c, %.o, $(SRC))

# 命令1：make all
ALL: hello.out
 
hello.out: $(OBJ)
        gcc $< -o $@
 
$(OBJ): $(SRC)
        gcc -c $< -o $@
```

> **变量定义**
> `$^` 表示所有的依赖文件  
> `$@` 表示生成的目标文件  
> `$<` 代表第一个依赖文件
> **变量赋值**
> `=` 是动态赋值，变量取最后一次出现 `=` 地方的赋值
> `:=` 是静态赋值，变量取当前 `:=` 对应的赋值
> `?=` 是询问式的赋值，若变量未赋值则取本次赋值，否则维持不变

```shell
## 预定义编译器
CC = arm-linux-gnueabihf-gcc  
CXX = arm-linux-gnueabihf-g++ 
AR = arm-linux-gnueabihf-ar
==========================
VIR_A = A 
VIR_B = $(VIR_A) B 
VIR_A = AA 
# VIR_B 最后取值为 AAB
==========================
VIR_A = A 
VIR_B = $(VIR_A) B 
VIR_A = AA 
# VIR_B 最后取值为 AB
==========================
VIR := old_value
VIR ?= new_value
# VIR 最后会取old_value
```

> **函数**
> `wildcard` 通配词，将符合修饰词的所有文件纳入变量
> `patsubst` 替换词，按规则将所有符合条件的文件置换

```shell
# 匹配目录下所有.c 文件，并将其赋值给SRC变量
SRC = $(wildcard *.c)
# 取出SRC中的所有值，然后将.c 替换为.o 最后赋值给OBJ变量
OBJ = $(patsubst %.c, %.o, $(SRC))
```

> **清理**
> `clean` 通俗的清理命令，可以命名为其他，后面编写清理编译文件需要的命令即可

```shell
SRC = $(wildcard *.c)
OBJ = $(patsubst %.c, %.o, $(SRC))
 
ALL: hello.out
 
hello.out: $(OBJ)
        gcc $< -o $@
 
$(OBJ): $(SRC)
        gcc -c $< -o $@
# 会使用rm命令删除变量OBJ对应的文件（此处是.o文件），以及最后生成的hello.out
clean:
        rm -rf $(OBJ) hello.out
```

> **伪目标 `.PHONY`**
>  `make` 命令 同名的文件时 执行make 命令就会出现错误，
>  解决办法就是使用伪目标，其含义就是将 `make` 后跟随的字符定义为虚伪的目标，而非真实的文件

```shell
# 此时，目录下如果有clean文件，ALL文件，就不会对其执行make而报错
.PHONY: clean ALL
```

> **嵌套的 `make`**
> 在大型项目中，会为其中的小项目写 `makefile` ，此时则会需要跳转到对应文件夹中，执行 `make`
> 	总控 `Makefile` 的变量可以传递到下级的 `Makefile` 中，但是不会覆盖下层 `Makefile` 中所定义的变量，除非指定了 `-e` 参数
> 		使用 `export` 指定需要传递的变量
> 		使用 `unexport` 指定不需要传递的变量
> 		若不指定变量是否传递，默认会传递

```shell
subsystem:
            cd dir && $(MAKE)
# 其等价于：
subsystem:
            $(MAKE) -C dir
===============================
export variable = value
# 等价于
variable = value
===============================
export variable
# 等价于
export variable := value
# 等价于
variable := value
===============================
export variable
# 如果需要传递所有变量，那么只要一个export就行了。后面什么也不用跟，表示传递所有变量
```

> **指定头文件与指定库文件**
> `-I` 指定头文件目录
> `-L` 指定库文件目录 ，特别的仅仅在链接阶段指定的库文件则 `-l`

如在 `C` 语言编译时，指定头文件目录 `include` ，库文件为 `src` ，`lib`

```shell
# 随便定义一个变量，对于C习惯命名为CFLAGS
CFLAGS=-I./include
# 随便定义并在等号后连续键入
LDFLAGS=-L./src -L./lib
LIBS = -lpthread -liconv
# 在编译时引用
main:*.c 
	gcc $(CFLAGS) -o main $(LDFLAGS) $(LIBS)
```

##### 实战
构建如下目录

.
├── build
│   └── [makefile](./example/make-test/build/makefile)
├── include
│   └── [include.h](./example/make-test/include/include.h)
├── lib
│   ├── [makefile](./example/make-test/lib/makefile)
│   └── [lib.c](./example/make-test/lib/lib.c)
├── main
│   ├── [main.c](./example/make-test/main/main.c)
│   └── [makefile](./example/make-test/main/makefile)
├── [makefile](./example/make-test/makefile)
└── src
    ├── [makefile](./example/make-test/src/makefile)
    └── [src.c](./example/make-test/src/src.c)

> 想要尽量减少 `makefile` 的情况下，就会增加一个 `makefile` 中出现的 `&&` 连接命令





