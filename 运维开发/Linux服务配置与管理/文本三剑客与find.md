---
title: 文本三剑客与find
tags:
  - awk
  - grep
  - sed
  - find
---

## awk

```bash
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



## sed
### 行操作
### 列操作
### 正则表达式

## grep
### 正则表达式

## find
