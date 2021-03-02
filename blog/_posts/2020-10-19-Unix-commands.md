---
layout: post
title:  "常用*nix命令串讲"
date:   2020-10-19
categories: [Jekyll Paper]
tags: [Getting Start]
---

本文维护更新中！

# mutt

mutt是一个命令行里发邮件的工具。

# grep

本来想单独写一篇笔记总结grep，但是发现用的指令其实就那几个，所以就放这里串讲了，可能常用的命令部分，我会把字体加粗展现

grep = global regular expression print，grep主要的功能是搜索匹配，匹配到的过滤出来打印或者重定向

search input files for a search string, and print the lines that match it. Beginning at the first line in the file, grep copies a line into a buffer, compares it against the search string, and if the comparison passes, prints the line to the screen.
文件 a_file如下所示

boot
book
booze
machine
boots
bungie
bark
aardvark
broken$tuff
robots
```bash
grep -n "boo" a_file
```
**带上-n参数可以输出匹配的内容，同时还输出行号**

1:boot
2:book
3:booze
5:boots
带上-v，如-vn的参数，则会把不符合匹配内容的都返回；
带上 -c ，则不会打印匹配的内容，只会输出匹配的内容的行数；
**带上-l，当grep接的是多个文件，例如 grep -l "boo" * 时，会返回存在匹配字符串的当前文件夹下的所有文件名**；
-i：忽视匹配模式的大小写；
-x: 完全匹配
-An: 匹配成功并输出后续的n行，如-A2，会输出匹配行同时，输出后面的两行

# uniq

Linux uniq 命令用于检查及删除文本文件中重复出现的行列，一般与 sort 命令结合使用。

uniq 可检查文本文件中重复出现的行列。当重复的行并不相邻时，uniq 命令是不起作用的，所以uniq前面经常可以接sort

 sort  testfile1 | uniq
uniq所谓的重复是连续出现的相同记录，如果想对全局去重，用sort -u。或者将内容sort一下，保证相同的记录时连续出现，再去重即可。
uniq -c ：去重并将每条记录出现的次数计数，常用于wordcount
uniq -f 2，表示只关注第二列的去重，不关心第一列是否重复。 this allows the N fields to be skipped while comparing uniqueness of the lines. This option is helpful when the lines are numbered
当重复的行并不相邻时，uniq 命令是不起作用的，即若文件内容为以下时，uniq 命令不起作用：

```bash
$ cat testfile1      # 原有内容 
test 30  
Hello 95  
Linux 85 
test 30  
Hello 95  
Linux 85 
test 30  
Hello 95  
Linux 85 
```
这时我们就可以使用 sort：

$ sort  testfile1 | uniq
Hello 95  
Linux 85 
test 30
统计各行在文件中出现的次数：

```bash
$ sort testfile1 | uniq -c
   3 Hello 95  
   3 Linux 85 
   3 test 30
```
# sort

-b 忽略每行前面开始出的空格字符。
-c 检查文件是否已经按照顺序排序。
-d 排序时，处理英文字母、数字及空格字符外，忽略其他的字符。
-f 排序时，将小写字母视为大写字母。
-i 排序时，除了040至176之间的ASCII字符外，忽略其他的字符。
-m 将几个排序好的文件进行合并。
-M 将前面3个字母依照月份的缩写进行排序。
-n 依照数值的大小排序。
-u 意味着是唯一的(unique)，输出的结果是去完重了的。
-o<输出文件> 将排序后的结果存入指定的文件。
-r 以相反的顺序来排序。
-t<分隔字符> 指定排序时所用的栏位分隔字符。
+<起始栏位>-<结束栏位> 以指定的栏位来排序，范围由起始栏位到结束栏位的前一栏位。

```bash
   -k, --key=KEYDEF
          sort via a key; KEYDEF gives location and type
```
sort by key 是最常用的，key的定义是自己取field，

# wc

wc命令用于计算字数。利用wc指令我们可以计算文件的Byte数、字数、或是列数，若不指定文件名称、或是所给予的文件名为"-"，则wc指令会从标准输入设备读取数据。
-c或--bytes或--chars 只显示Bytes数。
-l或--lines 只显示行数。
-w或--words 只显示字数。
--help 在线帮助。
--version 显示版本信息。

```bash
wc testfile testfile_1 testfile_2   #统计三个文件的信息 
```
# paste

cat是按行合并，paste是按列合并
$ paste 1 2 3

```bash
-d<间隔字符>或--delimiters=<间隔字符> 　用指定的间隔字符取代跳格字符。
-s或--serial 　串列进行而非平行处理。
--help 　在线帮助。
--version 　显示帮助信息。
[文件…] 指定操作的文件路径
```
# head

head -10 a.txt > tmp
前10行输出或重定向

# cut

Linux cut命令用于提取显示每行从开头算起 num1 到 num2 的文字。
看这个就行，主要还是文本提取的命令 详情 请看
https://www.cnblogs.com/dong008259/archive/2011/12/09/2282679.html

# tee

将程序的输出结果重定向，使得我们可以同时显示和保存结果，例如，添加一个新的条目到hosts文件中:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201019175232745.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4MzM5ODMy,size_16,color_FFFFFF,t_70#pic_center)

```
echo "127.0.0.1 foobar" | sudo tee -a /etc/hosts
```
结合前一篇博文的重定向，说说这两个的关系

2>&1 >output.log 
means first start sending all file handle 2 stuff (standard error) to file handle 1 (standard output) then send that to the file output.log. In other words, send standard error and standard output to the log file.

2>&1 | tee output.log 
is the same with the 2>&1 bit, it combines standard output and standard error on to the standard output stream. It then pipes that through the tee program which will send its standard input to its standard output (like cat) and also to the file. So it combines the two streams (error and output), then outputs that to the terminal and the file.

后者输入进文件的同时，也会输出到console。前者则都被重定向到文件，不会在console中展示。
如图所示，tee其实是一种T字形的管道，他在输出给标准stdout的同时，也会重定向给文件流。

# shuf

就是shuffle了。

```bash
seq 3 | shuf
```
将1、2、3按行乱序输出。
也可以shuf一个文件，如shuf file

# wait

略

# xargs

xargs（英文全拼： eXtended ARGuments）是给命令传递参数的一个过滤器，也是组合多个命令的一个工具。
可以看http://www.ruanyifeng.com/blog/2019/08/xargs-tutorial.html   xargs主要是和标准输入输出有关系。xargs命令的作用，是将标准输入转为命令行参数。

$ echo "hello world" |  echo
这个不会有任何输出

$ echo "hello world" | xargs echo
hello world
上面的代码将管道左侧的标准输入，转为命令行参数hello world，传给第二个echo命令。xargs接Linux的命令作为参数，用该命令去处理标准输出如输出。真正执行的命令，紧跟在xargs后面，接受xargs传来的参数。

xargs的作用在于，大多数命令（比如rm、mkdir、ls）与管道一起使用时，都需要xargs将标准输入转为命令行参数。xargs后面的命令默认是echo。

$ echo "one two three" | xargs mkdir  等同于 mkdir one two three
大多数时候，xargs命令都是跟管道一起使用的。但是，它也可以单独使用。

输入xargs按下回车以后，命令行就会等待用户输入，作为标准输入。你可以输入任意内容，然后按下Ctrl d，表示输入结束，这时echo命令就会把前面的输入打印出来。
默认情况下，xargs将换行符和空格作为分隔符，把标准输入分解成一个个命令行参数。

$ echo "one two three" | xargs mkdir
上面代码中，mkdir会新建三个子目录，因为xargs将one two three分解成三个命令行参数，执行mkdir one two three。

-d参数可以更改分隔符。

$ echo -e "a\tb\tc" | xargs -d "\t" echo
a b c
上面的命令指定制表符\t作为分隔符，所以a\tb\tc就转换成了三个命令行参数。echo命令的-e参数表示解释转义字符，即转义字符生效。

# tr

主要用于压缩重复字符，删除文件中的控制字符以及进行字符转换操作。

```bash
echo "Hello World I Love You" |tr -t [a-z] [A-Z]  #小写转大写
echo "Hello World I Love You" |tr -t [:lower:] [:upper:] #小写转大写
```
-s 压缩重复字符 replace each input sequence of  a  repeated  character  that  is listed in SET1 with a single occurrence of that character (就是去重的意思)

echo "aaabbbaacccfddd" | tr -s [abcdf] // abacfd
删除文件中的空白行

cat 文件名 |tr -s ‘\n'   cat b.txt | tr -s ["\n"]
-d ：delete，删除SET1中指定的所有字符，不转换

xiaosi@Qunar:~/test$ echo "a12HJ13fdaADff" | tr -d "[a-z][A-Z]"
1213
xiaosi@Qunar:~/test$ echo "a1213fdasf" | tr -d [adfs]
1213
-t：truncate，将SET1中字符用SET2对应位置的字符进行替换，一般缺省为-t

 echo "a1213fdasf" | tr -t [afd] [AFO] // A1213FOAsF
 echo "Hello World I Love You" |tr -t [a-z] [A-Z]
上述代码将a转换为A，f转换为F，d转换为O。

```bash
\NNN 八进制值的字符 NNN (1 to 3 为八进制值的字符)
\\ 反斜杠
\a Ctrl-G 铃声
\b Ctrl-H 退格符
\f Ctrl-L 走行换页
\n Ctrl-J 新行
\r Ctrl-M 回车
\t Ctrl-I tab键
\v Ctrl-X 水平制表符
CHAR1-CHAR2 从CHAR1 到 CHAR2的所有字符按照ASCII字符的顺序
[CHAR*] in SET2, copies of CHAR until length of SET1
[CHAR*REPEAT] REPEAT copies of CHAR, REPEAT octal if starting with 0
[:alnum:] 所有的字母和数字
[:alpha:] 所有字母
[:blank:] 水平制表符，空白等
[:cntrl:] 所有控制字符
[:digit:] 所有的数字
[:graph:] 所有可打印字符，不包括空格
[:lower:] 所有的小写字符
[:print:] 所有可打印字符，包括空格
[:punct:] 所有的标点字符
[:space:] 所有的横向或纵向的空白
[:upper:] 所有大写字母
```
# split

#cat hello
Hello, World1
Hello, World2
Hello, World3
Hello, World4
Hello, World5
使用命令：

#split -2 hello split1_
split命令会将文件以两行为单位进行切割，每两行组成一个新文件，5行就有三个文件，名称会分别为：

split1_aa , split1_ab , split_ac
# nl

nl命令其它和cat输出文件内容 命令很像，只不过它会打上行号。

统计你使用的最高频命令的top10

history | awk '{print $2}' | sort | uniq -c | sort -k1,1nr | head -10
或者			tr '\n' '' < file | awk '{print $2}' | sort | uniq -c | sort -k1,1nr | head -10

同样，可以给别的文件打上行号：

```bash
nl file1 >file2
```
# find

find -name *.txt  #搜索当前目录下所有txt文件
find -name *.txt  #搜索当前目录下所有txt文件 
find的使用实例：
　　$ find . -name 'my*'

搜索当前目录（含子目录，以下同）中，所有文件名以my开头的文件。

　　$ find . -name 'my*' -ls

搜索当前目录中，所有文件名以my开头的文件，并显示它们的详细信息。

　　$ find . -type f -mmin -10

搜索当前目录中，所有过去10分钟中更新过的普通文件。如果不加-type f参数，则搜索普通文件+特殊文件+目录。

2. locate

locate命令其实是"find -name"的另一种写法，但是要比后者快得多，原因在于它不搜索具体目录，而是搜索一个数据库（/var/lib/locatedb），这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，所以使用locate命令查不到最新变动过的文件。为了避免这种情况，可以在使用locate之前，先使用updatedb命令，手动更新数据库。

locate命令的使用实例：

　　$ locate /etc/sh

搜索etc目录下所有以sh开头的文件。

　　$ locate ~/m

搜索用户主目录下，所有以m开头的文件。

　　$ locate -i ~/m

搜索用户主目录下，所有以m开头的文件，并且忽略大小写。-i ignore case

3. whereis

whereis命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。

whereis命令的使用实例：

　　$ whereis grep 
　　$ whereis python3

4. which

which命令的作用是，在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。

which命令的使用实例：

　　$ which grep

5. type

type命令其实不能算查找命令，它是用来区分某个命令到底是由shell自带的，还是由shell外部的独立二进制文件提供的。如果一个命令是外部命令，那么使用-p参数，会显示该命令的路径，相当于which命令。

type命令的使用实例：

　　$ type cd

系统会提示，cd是shell的自带命令（build-in）。

　　$ type grep

系统会提示，grep是一个外部命令，并显示该命令的路径。

　　$ type -p grep

加上-p参数后，就相当于which命令。

# nohup

用途：不挂断地运行命令。忽略所有挂断（SIGHUP）信号

语法：nohup Command [ Arg … ] [　& ]

　　无论是否将 nohup 命令的输出重定向到终端，输出都将附加到当前目录的 nohup.out 文件中。

nohup /usr/local/node/bin/node /www/im/chat.js >> /usr/local/node/output.log 2>&1 &
这里的‘&’表示此命令会在终端后台工作；反之，如果没有‘&’，则表示此命令会在终端前台工作。
最后的&符表示在后台运行

常见的区别可见：

https://blog.csdn.net/hl449006540/article/details/80216061
linux命令 nohup命令行后面的& 符合代表什么意思？？ - 王奥的回答 - 知乎
https://www.zhihu.com/question/40910876/answer/932403152

使用&后台运行程序：

结果会输出到终端

使用Ctrl + C发送SIGINT信号，程序免疫

关闭session发送SIGHUP信号，程序关闭

使用nohup运行程序：

结果默认会输出到nohup.out

使用Ctrl + C发送SIGINT信号，程序关闭

关闭session发送SIGHUP信号，程序免疫

平日线上经常使用nohup和&配合来启动程序：同时免疫SIGINT和SIGHUP信号
