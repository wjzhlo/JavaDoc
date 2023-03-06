

# CentOS 记录

## 一、进程管理

### 1.1 查看占用的进程信息

https://blog.csdn.net/baidu_21349635/article/details/87854332

```
# 列出端口使用情况
netstat -ltnp
```

```
# 检查某个具体的端口
netstat -tlnp | grep 80

------------------------------------------------------
例如： 其中 13412 为进程号
[root@izbp1dou23krq0ayz4k4f7z bin]# netstat -ntpl |grep 8080
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      13412/java 
```

```
# 常见参数
-a (all)显示所有选项，默认不显示LISTEN相关
-t (tcp)仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化成数字。
-l 仅列出有在 Listen (监听) 的服務状态

-p 显示建立相关链接的程序名
-r 显示路由信息，路由表
-e 显示扩展信息，例如uid等
-s 按各个协议进行统计
-c 每隔一个固定时间，执行该netstat命令。
```

得到端口使用情况后，可以使用 ps 命令查看具体是哪个进程占用

```
# 查看具体的进程信息
ps 【进程号】

-----------------------------------
例如：可以看到，实际是在 /opt 目录下的 zookeeper 占用了8080端口
[root@izbp1dou23krq0ayz4k4f7z bin]# ps 13412
PID TTY      STAT   TIME COMMAND
13412 pts/0    Sl     0:01 /opt/java/bin/java -Dzookeeper.log.dir=/opt/zookeeper1/bin/../logs -Dzookeeper.log.file=zookeeper-root-server-izbp1dou23krq0ayz4k4f7z.log -Dzookeeper.root.l
```

可以通过正常关闭，但也可以直接杀死进程

```
# 杀死进程
 kill -9 【进程号】
```









