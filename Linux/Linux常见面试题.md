## 1. 说说常用的linux命令

常见的文件和目录操作 cp 、 mv、vim 、 rm -rf 、pwd、tree、ls

查询操作 cat、tail -300 test.txt 、 tail -f log.txt

服务操作 service、systemctl

网络和进程操作 ifconfig、ps -ef 

端口占用 netstat -lnp|grep 80



### 关机、重启

```shell
# 关机、重启
shutdown -h
init
```

### 服务相关

```shell
# 服务相关
service <服务名> status/start/stop/restart
systemctl status/start/stop/restart <服务名>
## 开机自启动
systemctl enable/disable <服务名>
```

### 网络和进程相关

```shell
# 网络和进程相关
## 查看⽹络接⼝属性
ifconfig

## 查看所有监听端口
netstat -lntp
## 查看TCP/UDP的状态信息
netstat -lutp
## 查看指定端口信息

## 查看所有进程
ps -ef
## 过滤出你需要的进程
ps -ef  | grep codesheep

## 杀掉进程
kill 

## 每1秒采⼀次系统状态，采20次
vmstat 1 20
## 查询cpu使⽤情况（1秒⼀次，共10次）
sar -u 1 10
## 查询磁盘性能
sar -d 1 10
```

### 查看系统相关信息

```shell
# 查看系统相关信息
## 查看内核/OS/CPU信息
uname -a 
## 查看linux版本信息
cat /proc/version
## 查看cpu信息
cat /proc/cpuinfo
## 查看系统环境变量
env
## 查看内存和交换区使用量
free

```

### 查看磁盘和分区

```shell
# 查看磁盘和分区
## 查看磁盘使用情况和挂载点
df -h 
## 查看所有交换分区
swapon -s

```

