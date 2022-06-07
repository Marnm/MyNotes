---
title: Shell脚本练习
tags:
  - shell
  - bash
---

**Hello World**
```sh
#!/bin/bash

function example {
    echo "Hello World"
}
example
```

-----

**用户猜数字**

```sh
#!/bin/bash

# 脚本生成一个100以内的随机数，提示用户猜数字，根据用户的输入，提示用户猜对了。
# 猜小了或猜大了，直至用户猜对 脚本结束

# RANDOM 为系统自带的系统变量，值为0-32767的随机数
# 使用取余算法将随机数变为1-100的随机数
num=$[ RANDOM%100+1 ]
echo "$num"

# 使用 read 提示用户猜数字
# 使用 if 判断用户猜数字的大小关系： 
# -eq（等于）, -ne（不等于），-gt（大于）,-lt小于，-le（小于等于）
while :
do
    read -p "计算机生成了一个1-100的随机数，你猜：" guess
    if [ $guess -eq $num ]
    then echo "恭喜，猜对了"
    elif [ $guess -gt $num ]
    then echo "Oops，猜大了"
    else echo "Oops, 猜小了"
    fi
done
```

-----

**剪刀石头布 游戏**
```sh
#!/bin/bash

game=(剪刀 石头 布)
num=$[ RANDOM%3 ]
computer=${ game[$sum] }

echo "请根据下列提示选择您的出拳手势"
echo "1. 石头"
echo "2. 剪刀"
echo "3. 布"

read -p "请选择 1-3:" person
case $person in
1)
    if [ $num -eq 0 ]
    then echo "平局"
    elif [ $num -eq 1 ]
    then echo "你赢了"
    else 
        echo "电脑赢"
    fi;;
2)
   if [ $num -eq 0 ] 
   then
    echo "电脑赢"
    elif [ $num -eq 1 ]
    then
        echo "平局"
    else
        echo "你赢了"
    fi;;
3)
    if [ $num -eq 0 ]
    then
        echo "你赢"
    elif [ $num -eq 1 ]
        echo "电脑赢"
    else
        echo "平局"
    fi;;
*)
    echo "必须输入1-3的数字"
esac
```

-----

**九九乘法表**
```sh
#!/bin/bash

for i in `seq 9`
do
    for j in `seq $i`
    do
        echo -n " $j*$i=$[i*j] "
    done
        echo
done
```

-----

**if运算表达式**
```sh
#!bin/bash

if [ $1 -eq 2 ]; then
    echo "wo ai wenmin"
elif [ $1 -eq 3 ]; then
    echo "wo ai wenxing"
elif [ $1 -eq 4 ]; then
    echo "wo de xin"
elif [ $1 -eq 5 ]; then
    echo "wo de ai"
fi
```

-----

**打印国际象棋棋盘**
```sh
#!/bin/bash

# 打印国际象棋棋盘
# 设置两个变量，i 和 j，一个代表行，一个代表列，国际象棋为 8x8棋盘
# i=1 是代表准备打印第一行棋盘，第1行棋盘有灰色和蓝色间隔输出，总共为8列
# i=1 ，j=1 代表第1行的第1列，i=2，j=3代表第2行的第3列
# 棋盘的规律是 i+j 如果是偶数，就打印蓝色色块，如果是技术就打印灰色色块
# 使用 echo -ne 打印色块，并且打印完成色块后不自动换行，在同一行继续输出其他色块

for i in {1..8}
do
    for j in {1..8}
    do
        sum=$[ i+j ]
        if [ $[sum%2] -eq 0 ]; then
            echo -ne "\033[46m \033[0m"
        else
            echo -ne "\033[47m \033[0m"
        fi
    done
    echo
done
```

------

**使用脚本对输入的三个整数进行排序**
```sh
#!/bin/bash

# 一次提示用户输入3个整数，脚本根据数字大小依次排序输出3个数字
read -p "请输入一个整数：" num1
read -p "请输入一个整数：" num2
read -p "请输入一个整数：" num3

# 不管谁大谁小，最后都打印 echo "$num1, $num2, $num3"
# num1 中永远存最小的值，num2 中永远存中间值， num3永远存最大值
# 如果输入的不是这样的顺序，则改变数的存储书序，如：可以将num1和num2的值对调

tmp=0
# 如果num1 大于 num2 ，就把num1和num2的值交换，确保num1变量中存的是最小值
if [ $num1 -gt $num2 ]; then
    tmp=$num1
    num1=$num2
    num2=tmp
fi

# 如果num1大于num3，就把num1和num3的值交换，确保num1的变量中存的是最小值
if [ $num1 -gt $num3 ]; then
    tmp=$num1
    num1=$num3
    num3=$tmp
fi

# 如果num2大于num3，就把num2和num3的值交换，确保num2的变量中存的是最小值
if [ $num2 -gt $num3 ]; then
    tmp=$num2
    num2=$num3
    num3=$tmp
fi
echo "排序后的数据（从小到大）为：$num1,$num2,$num3"
```

-----

**for循环判断**
```sh
#!/bin/bash

s=0;
for((i=1;i<100;i++))
do
    s=$[$s+$i]
done

echo $s

r=0;
a=0;
b=0;
for((x=1; x<9; x++))
do
    a=$[$a+$x]
    echo $x
done

for ((y=1; y<9; y++))
do
    b=$[$b+$y]
    echo $y
done

echo $r=$[$a+$b]
```

```sh
#!/bin/bash

for i in "$*"
do
    echo "wenmin xihua $i"
done

for j in "$@"
do
    echo "wenmin xihuan $j"
done
```

------

**使用函数 求和运算**
```sh
#!/bin/bash

function sum() {
    s=0;
    s=$[$1+$2]
    echo $s
}

read -p "input your parameter " p1
read -p "input your parameter " p2

sum $p1 $p2

function multi() {
    r=0;
    r=$[$1/$2]
    echo $r
}
read -p "input your parameter " x1
read -p "input your parameter " x2

multi $x1 $x2

v1=1
v2=2
let v3=$v1+$v2
echo $v3
```

------

**case——esac分支结构**
```sh
#!/bin/bash

case $1 in
1)
    echo "wenmin "
    ;;
2)
    echo "wenxing "
    ;;
3)
    echo "wemchang "
    ;;
4)
    echo "yijun"
    ;;
5)
    echo "sinian"
    ;;
6)
    echo "sikeng"
    ;;
7)
    echo "yanna"
    ;;
*)
    echo "danlian"
    ;;
esac
```

-----

**对变量的传入与获取个数及打印**
```sh
#!/bin/bash

echo "$0 $1 $2 $3"  //传入四个参数
echo $# // 获取传入参数的数量
echo $@ //打印获取传入参数
echo $* //打印获取传入参数
```

------

**脚本定义while循环语句**
```sh
#!/bin/bash

s=0
i=1
while [ $i -le 100 ]
do
    s=$[$s + $i ]
    i=$[$i + 1]
done

echo $s
echo $i
```

------

**读取控制台传入参数**
```sh
#!/bin/bash

read -t 7 -p "input your name " NAME
echo $NAME

read -t 11 -p "input you age " AGE
echo $AGE

read -t 15 -p "input your friend " FRIEND
echo $FRIEND

read -t 16 -p "input your love " LOVE
echo $LOVE
```

**打印字母数小于8的单词**
> **描述**  
> 写一个bash脚本以统计一个文本文件nowcoder.txt中字母数小于8的单词  
> 示例：  
> 假设nowcoder.txt内容如下：  
> how they are implemented and applied in computer  
> 你的脚本应当输出：  
> how  
> they  
> are  
> and  
> applied  
> in  
> 注意：不要担心你输出的空格以及换行的问题



```shell
awk 'BEGIN{FS="";RS=" ";ORS="\n"}{if(NF<8)print$0}' nowcoder.txt
```
