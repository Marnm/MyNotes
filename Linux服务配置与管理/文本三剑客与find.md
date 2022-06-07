---
title: 文本三剑客与find
tags:
  - awk
  - grep
  - sed
  - find
---

## awk

```shell
line1 f2 f3
line2 f4 f5
line3 f6 f7
echo -e "line1 f2 f3\nline2 f4 f5\nline3 f6 f7" | awk '{print "Line No:"NR", No of fields:"NF, "$0="$0, "$1="$1, "$2="$2, "$3="$3}' 
Line No:1, No of fields:3 $0=line1 f2 f3 $1=line1 $2=f2 $3=f3
Line No:2, No of fields:3 $0=line2 f4 f5 $1=line2 $2=f4 $3=f5
Line No:3, No of fields:3 $0=line3 f6 f7 $1=line3 $2=f6 $3=f7

```
### 增删改查
#### 添加操作
#### 删除操作
#### 修改操作
#### 查询操作

### 替换
#### 替换字符
#### **替换数字**
#### **替换英文**
#### **替换空行**

### 正则表达式
#### **标准**
#### **扩展**

### 条件
### 列操作
### 行操作
打印第一行
```shell
awk 'NR==1{print}' /tmp/tmp.txt
```



## sed
### 行操作

**添加内容到首行**
```shell
# 基本语法，（空格可以省略）
# i在当前行插入内容
# a在当前行追加内容
sed '作用范围行 插入行前/后 插入内容' 文件名
# 在file.txt文件第1行前面插入hello
sed -i '1 i hello' file.txt

# 在file.txt文件第1行后面插入world
sed -i '1 a world' file.txt
```

**添加内容到指定行**
```shell
# 在第3行前面插入内容
sed -i '3i hello ' file.txt
```

**在最后一行插入内容**
```shell
# $表示最后一行
sed -i '$a hello' sample.txt
```

### 列操作
### 正则表达式

## grep
### 正则表达式

## find
