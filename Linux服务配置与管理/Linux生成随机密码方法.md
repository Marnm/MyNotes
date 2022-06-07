---
title: Linux生成随机密码方法
tags:
  - gpg
  - cracklib
  - openssl
---

----

### gpg生成随机字符
```bash
# 
gpg --gen-random --armor 2 12
# --gen-random 随机生成字符
# -armor 生成ASCII字符
# 2 质量级别，0、1、2三个选项
# 12    字符长度
```
**过滤特殊字符**
```bash
gpg --gen-random --armor 2 12 | sed 's/[^a-zA-Z0-9]//g'
```

**检测密码强度**
> 可以通过安装`cracklib`工具来检查密码强度

```bash
yum install cracklib

echo "123545" | cracklib-check
```

### openssl生成随机字符

```bash
openssl rand -base64 15
```
**过滤特殊字符**
```bash
openssl rand -base64 15 | sed 's/[a-zA-Z0-9]//g'
```
