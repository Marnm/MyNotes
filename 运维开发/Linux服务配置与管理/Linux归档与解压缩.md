---
title: Linux归档与解压缩
tags:
  - tar
  - bz
  - bz2
  - bzip2
  - uncompress
  - compress
  - rpm2cpio
---

### Linux解压缩工具（命令）
> tar 、gzip、bzip2、zip、xz、gunzip、


### tar
```bash
-c 创建打包
-x 展开归档文件
-v 显示过程信息
-f 指定文件/目录
-O 将文件解开到标准输出
-C 指定路径
-j bz2格式文件
-J xz格式文件
-z gzip格式文件
```
> 解包：tar -xvf filename.tar
> 打包：tar -cvf filename.tar

### gzip格式文件
```bash
tar -zxvf filename.tar.gz   # 解压gzip格式压缩文件
tar -zcvf filename.tar.gz compress_dir  # 创建gzip格式压缩文件
################################33
gunzip filename.gz  # 解压gzip格式文件
gzip -d filename.gz # 解压gzip格式文件
gzip filename.gz    # 创建gzip格式文件
```

### bz
```bash
bzip2 -d filename.bz    # 解压bz格式文件
bunzip2 filename.bz     # 解压bz格式文件
################3
tar jxvf    # 解压bz格式文件
```

### bz2
```bash
bzip2 -d filename.bz2   # 解压bz2格式文件
bunzip2 filename.bz2    # 解压bz2格式文件
bzip2 -z filename   # 创建bz2格式文件
####################
tar jxvf filename.tar.bz2   # 解压bz2格式文件
tar jcvf file_dir   # 创建bz2格式文件
```

### z
```bash
uncompress filename.z   # 解压z格式文件
compress    filename    # 创建z格式文件
##################
tar Zxvf    # 解压z格式文件
tar Zcvf    # 创建z格式文件
```

### zip
```bash
unzip filename.zip  # 解压zip文件
zip filename.zip file_dir   # 创建zip文件
```

### rpm包的解包
```bash
rpm2cpio filename.rpm | cpio -div
```

### deb包的解包
```bash
ap p filename.deb data.tar.gz | tar zxvf -
```
