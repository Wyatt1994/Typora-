ps -aux|grep 

netstat -nlp|grep 

**找到最耗CPU的进程**

- 执行top -c ，显示进程运行信息列表
- 键入P (大写p)，进程按照CPU使用率排序

**找到最耗CPU的线程**

- top -Hp 10765 ，显示一个进程的线程运行信息列表
- 键入P (大写p)，线程按照CPU使用率排序

**将线程PID转化为16进制**

工具：printf

方法：printf “%x” 10804

**查看堆栈，找到线程在干嘛**

工具：pstack/jstack/grep

方法：jstack 10765 | grep ‘0x2a34’ -C5 --color

- 打印进程堆栈
- 通过线程id，过滤得到线程堆栈

查看堆信息

**jmap**命令+java进程号即可

```
javap是 Java class文件分解器，可以反编译（即对javac编译的文件进行反编译），也可以查看java编译器生成的字节码。用于分解class文件
```

**Linux查看cpu占用率高的进程**
用top命令找到最占CPU的进程
[https://www.cnblogs.com/pangguoping/p/5715848.html](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/pangguoping/p/5715848.html)
[https://blog.csdn.net/hrn1216/article/details/51426741](https://link.zhihu.com/?target=https%3A//blog.csdn.net/hrn1216/article/details/51426741)
**查看占用某端口的进程**
1、lsof -i:端口号
2、netstat –tunlp | grep 端口号
**查看某进程监听的端口**
1、先查看进程pid
ps -ef | grep 进程名
2、通过pid查看占用端口
netstat -nap | grep 进程pid
10 如何查询日志文件中的所有ip，正则表达式

查看java进程的配置参数

jinfo -pid

jconsole 命令，弹出GUI连接窗口

**排查内存泄漏**

https://blog.csdn.net/fishinhouse/article/details/80781673

jps -l或者ps -aux|grep tomcat 获得pid(若为本地虚拟机，即vmid=pid)，然后jstat -gcutil vmid 1000ms，即1000ms查询一次空间占用情况

然后jmap获得堆转储快照，jmap vmid，jmap -histo:live vmid，即可查看存活的对象情况。

如果还不清晰，就是用**MAT**工具软件可视化查看