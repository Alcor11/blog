## linux加载过程

加载bios，读取bios存储的CPU、设备启动顺序、硬盘内存相关信息
通过磁盘第一个扇区读取MBR，Master Boot Record
GRUB引导
加载内核  /etc/modules.conf
启动运行程序 rc.local文件，可以在rc.local文件后自定义需要启动的用户程序。
执行 /bin/login



## 防火墙

- 查看防火墙状态 
  `systemctl status firewalld`
- 开启防火墙
  `systemctl start firewalld`
- 关闭防火墙
  `systemctl stop firewalld`
- 开启防火墙
  `service firewalld start`

## 端口查询

1. 查询已经对外开放的端口  
   netstat -anp
2. 查询指定端口是否已经开放  
   firewall-cmd --query-port=8848/tcp

_**开放端口号命令：**_  
添加指定需要开放的端口：  
firewall-cmd --add-port=8848/tcp --permanent

重载入添加的端口：  
firewall-cmd --reload

查询指定端口是否开启成功：  
firewall-cmd --query-port=8848/tcp


## 日志查看

- 查看刚部署的tail -f xxx.log
- tail -200f xxx.log（查看最后200行
- ctrl + c推出
- 搜索关键词cat -n filename |grep "keyword"
- vi xxx.log 
- /keyword 查找
- ctrl + f下翻
- ctrl + b上
- shift + g 最后一页
- gg 第一页
- :wq 保存
- :q不保存
- grep、awk、sed

### grep参数

```shell
# 查找ERROR日志、以及后10行
$ grep -A 10 ERROR app.log

# 查找ERROR日志、以及前10行
$ grep -B 10 ERROR app.log

# 查找前后10行
$ grep -C 10 ERROR app.log
```

### 查看某个时间段的log

```shell
# 导出02:14-2.16的日志
awk '/2022-06-24T02:14/,/2022-06-24T02:1[6-9]/'app.log > app0215.
log

sed -n '/2022-06-24T02:14/,/2022-06-24T02:1[6-9]/p' app.log > app0215.log
```

### 查看最近10条错误日志

```shell
# 最容易想到的是tail，但有可能最后1000行日志全是正常日志
$ tail -n 1000 app.log | less

# 最后10条异常, tac会反向读取日志行，然后用grep找到10个异常日志，再用tac又反向一次就正向了
$ tac app.log | grep -n -m10 ERROR | tac
```


**vim编辑**

- 显示行号：set nu
- 定位到指定行 :n 
- 编辑模式定位到指定行：ngg
- vim +n 
- vi和vim
- / string 向后查找
- ? string 向前查找


## 命令

-  **kill和kill-9的区别**
   kill是发送SIGTERM信号给对应程序，程序收到信号后可能会：

1. 立刻停止
2. 释放资源后再停止
3. 继续运行
   kill -9 则是发送SIGKILL信号，即exit ，不会被阻塞，能立刻kill进程

kill java.jar可以让jvm触发信号处理函数，jvm接收到以后会清理资源后停止。

## 使用linux定时执行shell脚本

- 定时任务服务：crontab

- 执行权限：
  chmod 755 hello.sh

- 手动启动crontab
  service crond start

- crontab -e 然后添加相应的任务，wq存盘退出。 
- crontab -l -u 列出调度任务



- vim /etc/crontab
  `*/5 * * * * root /hello.sh`

crontab -e root/shell.sh

## java部署

- 查看java进程
  ps -aux | grep java
- 停止进程
  kill -9 id
- 启动java
  nohup java -jar xxx.jar > /log/ xxxx >log &
- tail -f xxx.log

mkdir 创建文件夹
mkdir -p xx/xxx/xx递归创建文件夹


nginx -t 查看nginx配置文件位置

端口查看：netstat | grep 80


后台：unweb.gthitx.com
企信：http://unqualified.gthitx.com/


- 修改文件权限
  touch text
  chmod a+rw xxx.sh 所有人可读写
  chmod u+rwx xxx.sh
  4-r
  2-w
  1-x
- ls
  ls -d -- 仅列出目录本身
  ls -a -- 列出全部文件，包括隐藏文件
  ls -l -- 列出权限信息等

## 文件操作

- 转移文件mv
- 删除命令rm
- 复制文件cp