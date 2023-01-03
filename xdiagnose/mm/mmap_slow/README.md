# 内核mmap_sem锁调测工具

## 【功能描述】

showbt 驱动用来定位内核维护的用户态进程的 struct rw_semaphore 读写锁的持有时间、持有者、以及调用栈信息。

1. 当被监控进程内的任何线程，持有读写锁时间过久（比如，mmap慢），则打印锁的 owner（高版本内核4.18及之后内核支持） 和调用栈。
2. 当被监控进程内的任何线程，读写内存时触发page fault且执行时间过久，则打印线程调用栈以及文件名（对于文件映射）。

## 【使用方法】

需要设置进程名和对应的 uid。

## 【加载命令】


```
insmod showbt.ko comm="java" uid=2201 dims=500
```

## 【参数说明】

comm: 进程名
uid:  进程的uid

dims: dump interval ms。根据该值周期性打印可疑线程信息和内核态调用栈。此值设置太小可能会冲日志，设置太大可能会抓不到，请结合实际表现设置经验值。

fims: find interval ms。根据该值周期性查找被监控的进程，直到找到为止。建议使用默认值。
ttms: tolerate time ms。容忍时间，持有锁或者调 mmap 耗时超过此阈值会打印告警，或者主动 panic。默认取 dims 值的一半。建议不设置。
sop: 如果设置为1，当检测到长时间卡且达到 ttms 阈值，则主动 panic。

## 【编译方法】

执行 make 即可。

## 【实现依赖】

基于 kprobe、kretprobe 和 workqueue 机制。