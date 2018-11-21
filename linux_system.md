查看系统版本：
```bash
lsb_release -a
```
查看文件系统：

```bash
df -h
```

查看目录磁盘占用：

```bash
du -lh -d 1
```

查看内存状况：

```bash
free -m
```


NUMA架构

```bash
numactl --hardware
```

查看进程信息：

```bash
top
```

获取某进程的线程数：

```bash
cat /proc/10826/status | grep threads -i
```

查看硬盘旋转
```bash
cat /sys/block/*/queue/rotational
```
查看物理CPU个数
```bash
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
```
查看每个物理CPU中core的个数(即核数)
```bash
cat /proc/cpuinfo| grep "cpu cores"| uniq
```
查看逻辑CPU的个数
```bash
cat /proc/cpuinfo| grep "processor"| wc -l
```
查看CPU频率
```bash
cat /proc/cpuinfo | grep 'cpu MHz'
```
