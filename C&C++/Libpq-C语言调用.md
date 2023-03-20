libpq 是一个库函数的集合，它们允许客户端程序传递查询给 `postgesql` 服务器并且接收这些查询的结果。
libpq库是 `postgesql` 的C接口。它是一组库函数，允许客户端程序与 `postgesql` 交互。它也是其他PostgreSQL应用程序接口的基础引擎，包括为C++、Perl、PHP、Ruby、Python和Tcl编写的应用程序接口。

#### 获取
在安装 `postgesql` 后，在`/usr/lib`下会得到`libpq`，可以使用`find /usr -name 'libpq*'`来寻找文件是否存在，若不存在：
- 方法一：`/usr/lib/postgresql`下寻找到库文件后，通过更改`/etc/ld.so.conf.d/`添加到`linux`的库文件查询目录中，
- 方法二： [Index of /pub/latest/ (postgresql.org)](https://ftp.postgresql.org/pub/latest/)中获取`postgresql`源码，编译其中的`libpq`并安装
- 方法三： `sudo apt-get install libpq-dev`

#### 使用
使用时，必须包含`libpq-fe.h`头文件，头文件位于`usr/include/postgresql`
增删改查示例：

```c
#include <stdio.h>
#include <libpq-fe.h>

int main(void)
{
    // 连接数据库
    const char* conninfo = "host=localhost user=postgresql  dbname=gva  password=1";
    PGconn* conn = PQconnectdb(conninfo);
    if (PQstatus(conn) == CONNECTION_BAD)
    {
        printf("failed to connect database！");
        PQfinish(conn);
        return 1;
    }
    // 创建表
    PGresult *res = PQexec(conn, "create table student (id int,name text)");
    if (PQresultStatus(res) != PGRES_COMMAND_OK)
    {
        printf("create failed!%s \n", PQresultErrorMessage(res));
    }
    // 更改
    PQclear(res);
    res = PQexec(conn, "insert into student values(1,'hanjingyi')");
    if (PQresultStatus(res) != PGRES_COMMAND_OK)
    {
        printf("insert failed!\n");
    }
    PQclear(res);
    res = PQexec(conn, "insert into student values(2,'liushun')");
    if (PQresultStatus(res) != PGRES_COMMAND_OK)
    {
        printf("insert failed\n!");
    }
    PQclear(res);
    // 查询操作
    res = PQexec(conn, "SELECT * FROM student");
    if (PQresultStatus(res) != PGRES_TUPLES_OK)
    {
        printf("SELECT failed！");
        PQresultErrorMessage(res);
        PQclear(res);
        return 1;
    }
    int i = PQntuples(res);
    int t = PQnfields(res);
    int s = 0;
    // 循环取值
    for (; s < i; s++)
    {
        int k = 0;
        for (; k < t; k++)
        {
            printf("%s \n", PQgetvalue(res, s, k));
        }
    }
    PQclear(res);
    // 删除操作
    res = PQexec(conn, "DELETE FROM student WHERE id=3");
    if (PQresultStatus(res) != PGRES_COMMAND_OK)
    {
        printf("DELETE executed failed！%s,\n", PQresultErrorMessage(res));
        PQresultErrorMessage(res);
        PQclear(res);
        return 1;
    }
    else
    {
        printf("DELETE operation executed succeed！\n");
        PQclear(res);
    }

    return 0;
}

```

> 使用`gcc main.c -lpq -I/usr/include/postgresql -o main`编译

可以发现，`libpq`本质上就是一个连接器，具体的操作执行仍然是使用`sql`语句，相当于为C打开了`postgresql`最顶层的控制界面。