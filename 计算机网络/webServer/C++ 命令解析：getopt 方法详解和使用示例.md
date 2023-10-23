
# main函数的参数传递
`int main(int argc, char *argv[])`
是main函数的标准签名。它是程序的入口点，用于接收命令行参数。

argc是一个整数，表示命令行参数的数量（包括程序名称本身）。
argv是一个字符指针数组，它存储了每个命令行参数的字符串。
当你在命令行中执行一个程序时，操作系统会解析你输入的命令，并将解析后的命令行参数传递给程序的main函数。操作系统会将命令行参数分割成多个字符串，并将它们存储在argv数组中。argc参数表示argv数组中的元素数量。

argv[0]存储的是程序的名称，例如./myprogram。argv[1]、argv[2]等依次存储了其他命令行参数的字符串。

以下是一个示例，演示如何使用`argc`和`argv`来处理命令行参数：

```
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("命令行参数的数量为：%d\n", argc);
    for (int i = 0; i < argc; i++) {
        printf("参数 %d: %s\n", i, argv[i]);
    }
    return 0;
}
```

当你在命令行中执行该程序时，例如`./myprogram arg1 arg2`，它会输出：

```
命令行参数的数量为：3
参数 0: ./myprogram
参数 1: arg1
参数 2: arg2
```


# 简介
参考[C/C++ 命令解析：getopt 方法详解和使用示例_c getopt_阿飞__的博客-CSDN博客](https://blog.csdn.net/afei__/article/details/81261879)
getopt() 方法是用来分析命令行参数的，该方法由 Unix 标准库提供，包含在 <unistd.h> 头文件中。

 

# 定义
```
int getopt(int argc, char * const argv[], const char *optstring);
 
extern char *optarg;
extern int optind, opterr, optopt;
```

getopt 参数说明：

- argc：通常由 main 函数直接传入，表示参数的数量
- argv：通常也由 main 函数直接传入，表示参数的字符串变量数组
- optstring：一个包含正确的参数选项字符串，用于参数的解析。例如 “abc:”，其中 -a，-b 就表示两个普通选项，-c 表示一个必须有参数的选项，因为它后面有一个冒号
外部变量说明：

- optarg：如果某个选项有参数，这包含当前选项的参数字符串
- optind：argv 的当前索引值
- opterr：正常运行状态下为 0。非零时表示存在无效选项或者缺少选项参数，并输出其错误信息
- optopt：当发现无效选项字符时，即 getopt() 方法返回 ? 字符，optopt 中包含的就是发现的无效选项字符
# 示例
```

void Config::parse_arg(int argc, char*argv[]){
    int opt;
    const char *str = "p:l:m:o:s:t:c:a:";
    while ((opt = getopt(argc, argv, str)) != -1)
    {
        switch (opt)
        {
        case 'p':
        {
            PORT = atoi(optarg);
            break;
        }
        case 'l':
        {
            LOGWrite = atoi(optarg);
            break;
        }
        default:
            break;
        }
    }
}
```
 每个字符后面都是**必须带上一个参数**，因为str中字母后面有一个冒号
 有两个冒号表示**可选带参数**
 opt表示**带上的参数类型**
 optarg表示**带上的参数的值**
