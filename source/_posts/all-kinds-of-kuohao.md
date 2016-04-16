---
title: shell中各种括号的作用()、(())、[]、[[]]、{} todo
date: 2016-04-15 22:37:45
tags: [linux,shell]
categories: [linux,shell]
---

[原文地址](http://blog.csdn.net/taiyang1987912/article/details/39551385)

## 一、 小括号、圆括号（）(())

### 单小括号 ()

* 命令组。括号中的命令将会新开一个子 shell 顺序执行，所以括号中的变量不能够被脚本余下的部分使用。括号中多个命令之间用分号隔开，最后一个命令可以没有分号，各命令和括号之间不必有空格。

	* 一串的命令执行()和{}，()和{}都是对一串的命令进行执行，但有所区别
		1. ()只是对一串命令重新开一个子shell进行执行
		2. {}对一串命令在当前shell执行
		3. ()和{}都是把一串的命令放在括号里面，并且命令之间用;号隔开
		4. ()最后一个命令可以不用分号
		5. {}最后一个命令要用分号
		6. {}的第一个命令和左括号之间必须要有一个空格
		7. ()里的各命令不必和括号有空格
		8. ()和{}中括号里面的某个命令的重定向只影响该命令，但括号外的重定向则影响到括号里的所有命令

```bash
#!/bin/sh

var=test

(var=test01; echo $var)     ## 变量var值为test01,在子shell中有效

echo $var                   ## 父shell中值仍为test

{ var=test02; echo $var;}   ## 注意左括号和var之间有一个空格

echo $var                   ## 父shell中的var变量的值变为test02
```

```
test01
test
test02
test02
```

* 命令替换。等同于 `cmd`，shell扫描一遍命令行，发现了 $(cmd) 结构，便将 $(cmd) 中的 cmd 执行一次，得到其标准输出，再将此输出放到原来命令。

```bash
$ ls 
a b c 

$ echo $(ls) 
a b c 

$ echo `ls` 
a b c
```

* 用于初始化数组。如：array=(a b c d)。



### 双小括号 (())

* 只要括号中的运算符、表达式符合c语言运算规则，都可用在$((exp))中，甚至是三目运算符。作不同进位(如二进制、八进制、十六进制)运算时，输出结果全都自动转化成了十进制。如：echo $((16#5f)) 结果为95 (16进位转十进制)

* 单纯用 (()) 也可重定义变量值，比如 a=5; ((a++)) 可将 $a 重定义为6；

* 常用于算术运算比较，双括号中的变量可以不使用 $ 符号前缀，也可以使用 $。括号内支持多个表达式用逗号分开。只要括号中的表达式符合c语言运算规则，比如可以直接使用 for((i=0;i<5;i++))， 如果不使用双括号, 则为 for i in `seq 0 4` 或者 for i in {0..4}。再如可以直接使用 if (($i<5)) 或者 if ((i<5)), 如果不使用双括号, 则为if [ $i -lt 5 ]。

```bash
#!/bin/sh

for i in `seq 0 4`
do
    echo $i
done

for i in {0..4}
do
    echo $i
done
```

```
0
1
2
3
4
0
1
2
3
4
```

```bash
#!/bin/sh

i=5

if ((i<10)); then
    echo "less than 10" 
fi

if (($i<10)); then
    echo "less than 10"
fi

if [ $i -lt 10 ]; then
    echo "less than 10"
fi
```

```
less than 10
less than 10
less than 10
```


## 二、中括号，方括号[]

  1、单中括号 []
    ①bash 的内部命令，[和test是等同的。如果我们不用绝对路径指明，通常我们用的都是bash自带的命令。if/test结构中的左中括号是调用test的命令标识，右中括号是关闭条件判断的。这个命令把它的参数作为比较表达式或者作为文件测试，并且根据比较的结果来返回一个退出状态码。if/test结构中并不是必须右中括号，但是新版的Bash中要求必须这样。
    ②Test和[]中可用的比较运算符只有==和!=，两者都是用于字符串比较的，不可用于整数比较，整数比较只能使用-eq，-gt这种形式。无论是字符串比较还是整数比较都不支持大于号小于号。如果实在想用，对于字符串比较可以使用转义形式，如果比较"ab"和"bc"：[ ab \< bc ]，结果为真，也就是返回状态为0。[ ]中的逻辑与和逻辑或使用-a 和-o 表示。
    ③字符范围。用作正则表达式的一部分，描述一个匹配的字符范围。作为test用途的中括号内不能使用正则。
    ④在一个array 结构的上下文中，中括号用来引用数组中每个元素的编号。
 2、双中括号[[ ]]
    ①[[是 bash 程序语言的关键字。并不是一个命令，[[ ]] 结构比[ ]结构更加通用。在[[和]]之间所有的字符都不会发生文件名扩展或者单词分割，但是会发生参数扩展和命令替换。
    ②支持字符串的模式匹配，使用=~操作符时甚至支持shell的正则表达式。字符串比较时可以把右边的作为一个模式，而不仅仅是一个字符串，比如[[ hello == hell? ]]，结果为真。[[ ]] 中匹配字符串或通配符，不需要引号。
    ③使用[[ ... ]]条件判断结构，而不是[ ... ]，能够防止脚本中的许多逻辑错误。比如，&&、||、<和> 操作符能够正常存在于[[ ]]条件判断结构中，但是如果出现在[ ]结构中的话，会报错。比如可以直接使用if [[ $a != 1 && $a != 2 ]], 如果不适用双括号, 则为if [ $a -ne 1] && [ $a != 2 ]或者if [ $a -ne 1 -a $a != 2 ]。
    ④bash把双中括号中的表达式看作一个单独的元素，并返回一个退出状态码。







