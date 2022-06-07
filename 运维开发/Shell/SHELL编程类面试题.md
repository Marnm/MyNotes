---
title: SHELL编程类面试题
tags:
  - shell
  - bash
---

### 1. 有一个test.txt文本(内容如下)，要求将所有域名截取出来，并统计重复域名
> 出现的次数。
> https://www.baidu.com/index.html
> https://www.jaking.com/index.html
> https://www.sina.com.cn/1024.html
> https://www.jaking.com/2048.html
> https://www.sina.com.cn/4096.html
> https://www.jaking.com/8192.html

**参考答案:**

```bash
cat test.txt | cut -d "/" -f 3 | sort | uniq -c | sort -nr

# cut -d 按照分隔符分割
# -f 3截取第三列
# uniq -c 显示改行重复次数
# sort -nr 按照数字从大到小排列
```

### 2. 统计当前服务器正在连接的IP地址，并按连接次数排序。
**参考答案**

```bash
netstat -ntup | grep -o [0-9].* | awk '{print $3}' | cut -d: -f 1 | sort | uniq -c | sort -rn
# netstat -ntup 显示TCP和UDP的连接状况，且直接显示IP地址；
# grep -o [0-9].* 只输出匹配到的部分，
# awk '{print $3}' 选择匹配搜索到的第3列
# cut -d: -f1 指定分隔符 : 指定显示第1个字段
# sort 默认升序排序
# uniq -c 统计各行出现的次数
# sort -rn  根据数字进行倒序排序
```

### 3. 使用循环在/jaking目录下创建10个txt文件，要求文件名称由6位随机小。

> 写字母加固定字符串（_nb）组成，例如：jk16ok_nb.txt。

**参考答案**
```bash
#!/bin/bash
# 检查jaking是否存在并且是一个目录
if [ ! -d /jaking ]	
then
	mkdir /jaking
fi
cd /jaking	# 进入测试目录

#循环10次，每次循环建立6位随机数文件
for ((i=1;i<=10;i++))	
do
	filename=$(tr -dc 'A-Za-z0-9' < /dev/urandom | head -c 6)
	touch "$filename"_gg.txt
done

# /dev/urandom 与 /dev/random
# /dev/urandom非阻塞随机数发生器，读取操作不会产生阻塞
# /dev/random存储着系统当前运行环境的实时数据，是阻塞的随机数发生器，读取有时需要等待
# 通常使用urandom生成随机数，使用urandom避免阻塞
```
### 4. 用Shell写99乘法表。
**参考答案**
```bash
#方式一
#!/bin/bash

n=1
while [ $n -lt 10 ]
do
      for (( m=1;m<=$n;m++ ))
      do
        echo -n -e "${m}x${n}=$[m*n]\t"
      done
echo
n=$((n+1))
done
#-----------------或者---------------------------
n=9
while [ $n -ge 1 ]
do
    for((i=1;i<=$n;i++))
    do
        echo -n -e "${i}x${n}=$[ n*i ]\t"
    done
echo
#   echo $n
    n=$((n-1))
done
########################################

# 方式二
#!/bin/bash

for j in {1..9}
do
      for i in `seq $j`
      do        echo -e -n "${i}x${j}=$[ $i * $j ]\t"
      done
echo
done
# ---------------或者----------------------------
for i in {9..1}
do
    for((j=1;j<=$i;j++))
    do
        echo -e -n "${j}x${i}=$[ i*j ]  "
    done
    echo
done
```

### 5. 写出1000以内的水仙花数。水仙花数：水仙花数是指一个 3 位数，它的每个位上的数字的 3次幂之和等于它本身（例如：1^3 + 5^3+ 3^3 = 153）。
**参考答案**
>   所谓"水仙花数"是指一个三位数，其各位数字立方和等于该本身。 例如：153是一个水仙花数，因为153=1^3+5^3+3^3。

```bash
#!/bin/bash
# 水仙花数是指一个3位数，其各位数字立方和等于该本身，所以直接从100开始找：
for i in {100..1000}
do
      a=${i:0:1}
      b=${i:1:1}
      c=${i:2}
      num=$[$a**3+$b**3+$c**3]
      [ $num -eq $i ] && echo "water flower number is: $i"
done
```

```c
//参考C的写法
//方法一
#include<stdio.h>
#include<stdlib.h>
#include<stdbool.h>
int cube(const int n){
    return n*n*n;
}
bool
isNarcissistic(const int n){//判断是不是水仙花数
    int hundreds=n/100;
    int tens=n/10-hundreds*10;
    int ones=n%10;
    return cube(hundreds)+cube(tens)+cube(ones)==n;
}
int main(void){
    int i;
    for(i=100;i<1000;++i){
        if(isNarcissistic(i))
        printf("%d\n",i);
    }
    return 0;
}
//=============================================//
//方法二
#include <stdio.h> 
#include <stdlib.h>
int main() 
{ 
    int i,j,k,n; 
    printf("'water flower'number is:"); 
    for(n=100;n<1000;n++) 
    { 
        i=n/100;/*分解出百位*/ 
        j=n/10%10;/*分解出十位*/ 
        k=n%10;/*分解出个位*/ 
        if(n==i*i*i+j*j*j+k*k*k) 
        { 
            printf("%-5d",n); 
        } 
    } 
    printf("\n");
    return 0;
} 
```


### 6. 获取随机8位字符串。
**参考**
```bash

#使用date 生成随机字符串
date +%s%N | md5sum | head -c 8

#使用 /dev/urandom 生成随机字符串
cat /dev/urandom | head -n 10 | md5sum | head -c 8
```
拓展：
> 生成随机数
> 参考：[https://doobom.me/random-string-number-in-shell](https://doobom.me/random-string-number-in-shell)


### 7. Shell批量转换1-9成为01-09。
**参考答案**
```bash
# 方式一
# 匹配每一个单词
sed 's:\w\+:0&:g'
# 改成配以任意数字
sed 's/[0-9]/0&/g'  #只能匹配1-9，

# 扩展
# 如果需要匹配9以上的数字，可只匹配行首
sed 's/^[0-9]/0&/g'

for i in {1..9}; do echo $i; done | sed 's/[0-9]/0&/g'

```

### 8. Shell打印自定义正等腰三角形。
**参考答案**
```bash
#!/bin/bash
# 简单粗暴法
echo '    *';
echo '   ***';
echo '  *****';
echo ' *******';
echo '*********';

# for
#!/bin/bash

for((i=1;i<=8;i++))
do
     for((k=8;k>i;k--))
    do
         echo -n " "
   done
    for((y=1;y<=i;y++))
    do
         echo -n "* "
   done
    echo ''
done  | sed 's/ $//g'
###################################
#!/bin/bash

for((i=1;i<=10;i++))
do
    for((j=10-i;j>=1;j--))
    do
        printf " "
    done
    for((k=1;k<=i;k++))
    do
        printf "* "
    done
printf "\n"
done

###############other######################
a='          *';for i in `seq 1 10` ; do echo "$a";a=$a' *';a=${a:1}; done;

################awk######################
awk 'BEGIN{
n=8;
l=n*2-1;
for(i=1;i<=n;i++){
    for(j=1;j<=n-i;j++){
        printf " "
    }
    for(k=1;k<=i;k++){
        printf "* "
    }
    print x
    }
 }'
```

参考C格式
```c
#include "stdio.h"
int main(){
    int a,x,y,z;
    scanf("%d",&a); //输入需要打印的等腰三角形的行数
    for(x=1;x<=a;x++)   //打印列
    {
        for(y=1;y<=a-x;y++) //打印空格
        {
            printf(" ");
        }
        for(z=1;z<=2*x-1;z++)   //打印*
            printf("*");
        printf("\n");
    }
}
```

### 9. Shell打印自定义倒等腰三角形。
**参考答案**
```bash
# 简答粗暴法
echo '*********';
echo ' *******';
echo '  *****';
echo '   ***';
echo '    *';
##########################
#!/bin/bash

for(( i=0; i<10; i++ ))
do
    for(( j=0; j<i; j++))
    do
        printf " ";
    done
    for(( j=10-j; j>0; j-- ))
    do
        printf "* ";
    done
    printf "\n";
done
```


### 10. 批量检查多个网站是否可以正常访问，要求使用she1l数组实现，检测策略尽量模拟用户真实访问模式。
> https://www.jakingchuanqi.com
> https://www.163.com
> https://www.baidu.com

**参考答案**
```bash
#!/bin/bash
web=(
https://www.jakingchuanqi.com
https://www.163.com
https://www.baidu.com
1.1.1.1)	#定义数组

for i in ${web[*]} #按照数组中值的个数循环，每次循环把数组中值赋予变量i
do
	code=$( curl -o /dev/null -s --connect-timeout 5 -w '%{http_code}' $i | grep -E "200|302")	#检测curl状态码
then
	echo "$i is ok" >> /root/ok.log
else	# 变量$code值为空，则休眠10秒，重新检测
	sleep 10
	code=$( curl -o /dev/null -s --connect-timeout 5 -w '%{http_code}' $i | grep -E "200|302")
if [ "$code" != "" ]
then
	echo "$i is ok" >> /root/ok.log
else
	echo "$i is error" >> /root/error.log
fi
fi
done
```
