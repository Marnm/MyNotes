---
title: Linux性能管理
tags: []
---

## 性能测试
### perf
#### 常用的perf命令
- perf stat
    这个命令提供常见的性能事件统计信息，包括说明执行和消耗的时钟周期。
- perf record
    该命令能够将数据记录并生成一个perf.data文件
- perf report
    该命令从perf record生成的perf.data文件中读取并显示事件数据。
- perf list
    列出机器上的可用事件
- perf top
    该命令与`top`命令类似，实时显示性能事件
    ![](http://192.168.85.188:8081/uploads/165292411776693751202205072153614.png)
    
- perf  trace
    该命令与`strace`命令类似，监视指定线程或进程使用的系统调用以及应用程序接收到的所有信号。
- perf help

```bash
perf list   # 查看当前支持的所有事件
# [Software event]软件事件
# [Tool event]
# [Hardware cache event]  硬件缓存事件
# [Kernel PMU event] PMU事件
# [Hardware event]
# Tracepoint event
# 
```



## 性能优化
