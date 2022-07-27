## Linux面试命令参考

参考1：cnblogs.com/myseries/p/11214977.html  【**查看日志**常用命令】

---

### whereis

> whereis  xx  ==>  查找文件--将与xx相关的文件查找出来。
>
> 该指令会在特定目录中查找符合条件的文件。这些文件应属于原始代码、二进制文件，或是帮助文件。
>
> 该指令只能用于查找二进制文件、源代码文件和man手册页，一般文件的定位需使用**locate**命令。

### which xx  [-a] command  显示一个二进制文件或可执行文件的完整路径【指令搜索？？】

### locate fileName

### find / -name "xxx"

---

**grep与fgrep的区别**：grep支持正则表达式，fgrep不支持正则表达式；由于后者不支持正则表达式，因此查询的速度也要比前者快；

---

查看某个端口的连接情况：netstat 、lsof



### lsof  ==  查看系统打开的文件

```shell
lsof filename // 显示打开指定文件的所有进程
lsof -u username // 显示所属user进程打开的文件
lsof -i // 用以显示符合条件的进程情况
lsof -p PID? // 列出某个进程号打开的文件
lsof -c mysql // 列出某个程序打开的文件信息

lsof -i:port // 根据端口查进程
netstat -nap | grep port // 根据端口查进程
ps -ef | grep PID // 查看指定PID的进程
```

### netstat  ==  检查本机网络状况

```shell
netstat -anp | grep 9090  //  端口号占用情况查看
```

### ps  ==  进程查看

```shell
ps -ef | grep PID // 查看指定PID的进程
```

### find 命令



### wc命令 

```shell
wc 命令 -c[统计字节数] -l[统计行数] -w[统计字数]
```





### sort命令



### 压缩命令



### paste命令





### 软/硬链接





### 查看后台任务

```shell
job -l
```



### 让一个命令在后台运行

```shell
在命令最后面加上 &
```



### 查看用过的命令列表

```shell
history
```



### 查看磁盘使用空间？

```shell
df -ah
```



### du和df的使用和区别



### awk的使用



### 给命令绑定一个宏或者按键

```shell
bind
```



### scp





### 孤儿进程

### 僵尸进程
