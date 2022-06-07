---
title: Linux Shell编程
tags:
  - shell
  - bash
---

## 条件判断
### if...then语句
> if语句会运行if后面的那个命令，
> 如果该命令的退出状态码是0（该命令成功运行）；位于then部分的命令就会被执行，
> 如果该命令的退出状态码是其他值，then部分的命令就不会被执行；
> shell会继续执行脚本中的下一条命令

### if..then..else语句
> 当if语句中的命令返回退出状态码0时，这跟if-then语句一样。
> 当if语句中的命令返回非零退出状态码时，shell会执行else部分中的命令

## 循环判断
### for

### while

### until

## 嵌套

【find】
atime: access，当文件内容被取用的时候，就会更新这个 读取时间，例如：使用cat命令取读取一个文件的内容，就会更新该文件的atime

ctime: status time 当文件的 状态 改变时，就会更新这个时间，例如：像是 权限 和 属性 被更改了，都会更新这个时间


mtime: modify，当文件的 内容数据 更改时，就会更新这个时间。内容数据指的是文件的内容，而不是文件的属性或权限

amin
文件最后一次访问在 n分钟前

cmin
文件的状态最后的一次更改在 n分钟前

mmin
文件的数据最后一次修改在 n分钟前

【shell】
()
数组赋值a=(value0 value1 value2 value3),=左右不要有空格
子shell赋值var="" 不等于 (var=lookback; echo $var),()内的命令会新开一个子shell
$()

( ( ) )
运算 echo $((4+4))
（三元运算）if-else
判断?前面的条件是否成立，是:否

[ ]


[ [ ] ]


{ }
通配，不允许空格 {1,2,3}  {a..f}
代码块/匿名函数
特殊的替换结构 ${var:-value} ${var:+value} ${var:=value} ${var:?value}
匹配模式替换结构 

{ { } }

' '
" "

字符串截取
[http://c.biancheng.net/view/1120.html](http://c.biancheng.net/view/1120.html)

数值比较
.png

字符串比较
.png

文件比较
.png


```bash
# 查看是否存在子shell
echo $BASH_SUBSHELL
# 0，没有子shell
# 1，或其他数字，表名存在一个或多个子shell
```

## 变量*
### 环境变量
> bash shell中，环境变量分为两类

- **全局变量**
    
- **局部变量**

所有的环境变量名均使用大写字母，这是bash shell的标准惯例。
如果是自己创建的局部变量或是shell脚本，请使用小写字母。

**在涉及环境变量名时，什么时候使用$，什么时候不该使用$，是在让人摸不着头脑。记住一点就行了：如果要用到变量，使用$，如果要操作变量，不是用$。（这条规则的例外就是使用printenv显示某个变量的值）**
