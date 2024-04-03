---
title: sort |unique-排序
date: 2023-10-27 14:39:22
tags: sort linux 排序
categories: linux
---


# Linux sort 排序命令

Linux sort 命令用于将文本文件内容加以排序。  
sort 可针对文本文件的内容，以行为单位来排序。       
<!-- more -->
## 语法  
```bash 
sort [-bcdfimMnr][-o<输出文件>][-t<分隔字符>][+<起始栏位>-<结束栏位>][--help][--verison][文件][-k field1[,field2]]  
```
    
## 参数说明：

|参数|说明|
|---- | ---- |
|-b |忽略每行前面开始出的空格字符。|
|-c| 检查文件是否已经按照顺序排序。|
|-d| 排序时，处理英文字母、数字及空格字符外，忽略其他的字符。|
|-f| 排序时，将小写字母视为大写字母。|
|-i| 排序时，除了040至176之间的ASCII字符外，忽略其他的字符。|
|-m| 将几个排序好的文件进行合并。|
|-M| 将前面3个字母依照月份的缩写进行排序。|
|-n| 依照数值的大小排序。|
|-u| 意味着是唯一的(unique)，输出的结果是去完重了的。｜
|-o|<输出文件> 将排序后的结果存入指定的文件。｜
|-r| 以相反的顺序来排序。｜
|-t<分隔字符>| 指定排序时所用的栏位分隔字符。｜
|+<起始栏位>-<结束栏位> |以指定的栏位来排序，范围由起始栏位到结束栏位的前一栏位。｜
|--help |显示帮助。｜
|--version |显示版本信息。｜
|[-k field1[,field2]] |按指定的列进行排序。｜

常用的参数有：
|参数|描述|
|----|----|
|-u| 排序去重|
|-r| 反向排序|
|-t| 指定分隔符|
|-k| 指定列排序|



## 实例
在使用 sort 命令以默认的式对文件的行进行排序，使用的命令如下：  
```bash
sort testfile 
```
sort 命令将以默认的方式将文本文件的第一列以 ASCII 码的次序排列，并将结果输出到标准输出。    
使用 cat 命令显示 testfile 文件可知其原有的排序如下：
```bash
$ cat testfile      # testfile文件原有排序  
test 30  
Hello 95  
Linux 85 
```
使用 sort 命令重排后的结果如下：
```bash
$ sort testfile # 重排结果  
Hello 95  
Linux 85  
test 30 
```
使用 -k 参数设置对第二列的值进行重排，结果如下：
```bash
$ sort testfile -k 2
test 30  
Linux 85 
Hello 95  
```

使用t参数以,为分隔符，参数k指定按照第二列的数据进行排序：
```bash
sort -t ',' -k 2 testfile2
```




# Linux uniq 排序命令
可检查文本文件中重复出现的行列

##语法： 
```bash
uniq [-cdu][-f<栏位>][-s<字符位置>][-w<字符位置>][--help][--version][输入文件][输出文件] 
```
##参数：
|参数 |说明 |
|----|----|
|-c或--count |在每列旁边显示该行重复出现的次数|
|-d或--repeated |仅显示重复出现的行列|
|-f<栏位>或--skip-fields=<栏位>| 忽略比较指定的栏位|
|-s<字符位置>或--skip-chars=<字符位置> |忽略比较指定的字符|
|-u或--unique |仅显示出一次的行列|
|-w<字符位置>或--check-chars=<字符位置> |指定要比较的字符|
|--help| 显示帮助|
|--version |显示版本信息|
|[输入文件] |指定已排序好的文本文件。如果不指定此项，则从标准读取数据|
|[输出文件] |指定输出的文件。如果不指定此选项，则将内容显示到标准输出设备（显示终端）|


# 补充：sort与unique的区别

    两者都可去重，但uniq去除的是连续出现的相同记录，sort -u 则可以去除连续或者不连续的相同记录.

实例：
```bash
[root@mysql ~]# cat test.txt 
a
123
123
a
a
ff
12
fff
a
ff
[root@mysql ~]# sort -u test.txt 
12
123
a
ff
fff
[root@mysql ~]# uniq test.txt 
a
123
a
ff
12
fff
a
ff
```

# 总结
如果想把文本里所有的重复项去掉，可用 `sort -u`

如果想统计重复项的次数并排序，需结合`uniq`，先排序再去重再排序：
```bash
[root@mysql ~]# sort test.txt |uniq -c| sort -r
      4 a
      2 ff
      2 123
      1 fff
      1 12
```
实际中可用来统计连接数最多的ip：

```bash
netstat -na|grep ESTABLISHED|awk '{print $5}'|awk -F: '{print $1}'|sort|uniq -c|sort -n
```
