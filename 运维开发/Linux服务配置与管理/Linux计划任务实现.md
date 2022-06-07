---
title: Linux计划任务实现
tags:
  - cron
  - crontab
---

### crontab
> 提交和管理用户的需要周期性执行的任务

```
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

```bash
# crontab任务格式
minute hour day month week command 顺序：分 时 日 月 周
```

说明：
- minute: 表示分钟，可以是从0到59之间的任何整数
- hour: 表示小时，可以是从0到23之间的任何整数
- day: 表示日期，可以是1-31之间的任何整数
- month: 表示月份，可以是1-23之间的任何整数，也可以是英文
- week: 表示星期几，可以是0-7之间的任何整数，（0和7代表星期日），也可以是英文
- command: 要执行的命令，可以是系统命令，也可以是自己编写的脚本

----

- 星号(\*) : 代表所有可能的值，例如month字段如果是星号，则表示在满足其他字段的制约条件后每月都执行该命令的操作
- 逗号(,)：可以用逗号隔开的值指定一个列表范围，例如1,2,4,7,9
- 中杠(-)：可以用整数之间的中杠表示一个整数范围，例如2-6表示2,3,4,5,6
- 正斜线(/)：可以用正斜线指定时间的间隔频率，例如0-23/2表示每两个小时执行一次。通知正斜线可以和星号一起使用，例如*/10，在minute字段表示每十分钟执行一次

```bash
# 每分钟执行一次
* * * * * command

# 每小时的第三和第15分钟执行一次
3,15 * * * * command

# 在上午8点到11点的第3和第15分钟执行
3,15 8-11 * * * command

# 每隔两天的上午8点到11点的第3和第15分钟执行一次
3,15 8-11 */2 * * command

# 每个星期一的上午8点到11点的第3和第15分钟执行
3,15 8-11 * * 1 command

# 每晚21:30重启smb
30 21 * * * /etc/init.d/smb restart

# 每月1、10、22日的4:45重启smb
45 4 1,10,22 * * /etc/init.d/smb restart

# 每周六，周日的1:10重启smb
10 1 * * 6,7(或者6,0) /etc/init.d/smb restart

# 每天18:00至23:00之间每隔30分钟重启smb
*/30(0,30) 18-23 * * * /etc/init.d/smb restart

# 每周六的晚上11重启smb
0 23 * * 6 /etc/init.d/smb restart

# 每小时重启smb
* */1 * * * /etc/init.d/smb restart

# 每月4号与每周一到周三的11点重启smb
0 11 4 * 1-3(mon-wed) /etc/init.d/smb restart

# 一月一号的4点重启smb
0 4 1 jan * /etc/init.d/smb restart

# 每小时执行/etc/cron.hourly目录内的脚本
01 * * * * root run-parts /etc/con.hourly
```


### at
> 在指定时间执行一个任务

```bash
# 三天后的下午5点钟执行/bin/ls
at 5pm+3 days
at> /bin/ls
at> <EOT>
Ctrl+D

# 明天17点钟，输出时间到指定文件内：
at 17:20 tomorrow
at> date >/root/2022.log
at> <EOT>
Ctrl+D
```

**atq 查看系统执行工作任务**

**atrm x 删除任务x**

**at -c x 查看已经设置的任务内容**
