**Shell 行排序**，对文本文件进行排序是一项常见的任务。`sort`命令能够对文本文件和`stdin`进行排序。它可以配合其他命令来生成所需要的输出。`uniq`经常与`sort`一同使用，提取不重复（或重复）的行。本章将演示`sort`和`uniq`命令的常见用法。

![Shell行排序](image/15679523319565675.png)

文章目录

- [1 预备知识](https://geek-docs.com/shell/shell-examples/shell-line-sort.html#i)
- [2 实战演练](https://geek-docs.com/shell/shell-examples/shell-line-sort.html#i-2)
- [3 工作原理](https://geek-docs.com/shell/shell-examples/shell-line-sort.html#i-3)
- [4 补充内容](https://geek-docs.com/shell/shell-examples/shell-line-sort.html#i-4)

## 预备知识

`sort`和`uniq`命令可以从特定的文件或`stdin`中获取输入，并将输出写入`stdout`。

## 实战演练

(1) 可以按照下面的方式排序一组文件（例如file1.txt和file2.txt）：

```shell
$ sort file1.txt file2.txt > sorted.txt
```

Shell

或是

```shell
$ sort file1.txt file2.txt -o sorted.txt
```

Shell

(2) 按照数字顺序排序：

```shell
$ sort -n file.txt
```

Shell

(3) 按照逆序排序：

```shell
$ sort -r file.txt
```

Shell

(4) 按照月份排序（依照一月、二月、三月……）：

```shell
$ sort -M months.txt
```

Shell

(5) 合并两个已排序过的文件：

```shell
$ sort -m sorted1 sorted2
```

Shell

(6) 找出已排序文件中不重复的行：

```shell
$ sort file1.txt file2.txt | uniq
```

Shell

(7) 检查文件是否已经排序过：

```shell
#!/bin/bash
#功能描述：排序
sort -C filename ;
if [ $? -eq 0 ]; then
   echo Sorted;
else
   echo Unsorted;
fi
```

Shell

将`filename`替换成你需要检查的文件名，然后运行该脚本。

## 工作原理

`sort`命令包含大量的选项，能够对文件数据进行各种排序。如果使用`uniq`命令，那`sort`更是必不可少，因为前者要求输入数据必须经过排序。

`sort`和`uniq`可以应用于多种场景。让我们来看一下这些命令的各种选项及用法。

要检查文件是否排序过，可以利用以下事实：如果文件已经排序，`sort`会返回为0的退出码（`$?`），否则返回非0。

```shell
if sort -c fileToCheck ; then echo sorted ; else echo unsorted ; fi
```

Shell

## 补充内容

我们已经介绍了`sort`命令的基本用法。下面来看看如何利用`sort`来完成一些复杂的任务。

1. **依据键或列排序**
   如果输入数据的格式如下，我们可以按列排序：

```shell
$ cat data.txt
1  mac    2000
2  winxp    4000
3  bsd    1000
4  linux    1000
```

Shell

有很多方法可以对这段文本排序。目前它是按照序号（第一列）来排序的。我们也可以依据第二列和第三列来排序。
`-k`指定了排序所依据的字符。如果是单个数字，则指的是列号。`-r`告诉`sort`命令按照逆序进行排序。例如：

```shell
# 依据第1列，以逆序形式排序
$ sort -nrk 1  data.txt
4  linux    1000
3  bsd    1000
2  winxp    4000
1  mac    2000
# -nr表明按照数字顺序，采用逆序形式排序

# 依据第2列进行排序
$ sort -k 2  data.txt
3  bsd    1000
4  linux    1000
1  mac    2000
2  winxp    4000
```

Shell

> 一定要留意用于按数字顺序进行排序的选项`-n`。`sort`命令对于字母表排序和数字排序有不同的处理方式。因此，如果要采用数字顺序排序，就应该明确地给出`-n`选项。

`-k`后的整数指定了文本文件中的某一列。列与列之间由空格分隔。如果需要将特定范围内的一组字符（例如，第2列中的第4~5个字符）作为键，应该使用由点号分隔的两个整数来定义一个字符位置，然后将该范围内的第一个字符和最后一个字符用逗号连接起来：

```shell
$ cat data.txt
　
　
1 alpha 300
2 beta 200
3 gamma 100
$ sort -bk 2.3,2.4 data.txt ;    # 按照m、p、t的顺序排序
3 gamma 100
1 alpha 300
2 beta 200
```

Shell

把作为排序依据的字符写成数值键。为了提取出这些字符，用其在行内的起止位置作为键的书写格式（在上面的例子中，起止位置是2和3）。
用第一个字符作为键：

```shell
$ sort -nk 1,1 data.txt
```

Shell

为了使`sort`的输出与以`\0`作为终止符的`xargs`命令相兼容，采用下面的命令：

```shell
$ sort -z data.txt | xargs -0
# 终止符\0用来确保安全地使用xargs命令
```

Shell

有时文本中可能会包含一些像空格之类的多余字符。如果需要忽略标点符号并以字典序排序，可以使用：

```shell
$ sort -bd unsorted.txt
```

Shell

其中，选项`-b`用于忽略文件中的前导空白行，选项`-d`用于指明以字典序进行排序。

1. **`uniq`**
   `uniq`命令可以从给定输入中（`stdin`或命令行参数指定的文件）找出唯一的行，报告或删除那些重复的行。
   `uniq`只能作用于排过序的数据，因此，`uniq`通常都与`sort`命令结合使用。
   你可以按照下面的方式生成唯一的行（打印输入中的所有行，但是其中重复的行只打印一次）：

```shell
$ cat sorted.txt
bash
foss
hack
hack

$ uniq sorted.txt
bash
foss
hack
```

Shell

或是

```shell
$ sort unsorted.txt | uniq
```

Shell

只显示唯一的行（在输入文件中没有重复出现过的行）：

```shell
$ uniq -u sorted.txt
bash
foss
```

Shell

或是

```shell
$ sort unsorted.txt | uniq -u
```

Shell

要统计各行在文件中出现的次数，使用下面的命令：

```shell
$ sort unsorted.txt | uniq -c
  1 bash
  1 foss
  2 hack
```

Shell

找出文件中重复的行：

```shell
$ sort unsorted.txt  | uniq -d
hack
```

Shell

我们可以结合`-s`和`-w`选项来指定键：

- `-s` 指定跳过前N个字符；
- `-w` 指定用于比较的最大字符数。

这个对比键可以作为`uniq`操作时的索引：

```shell
$ cat data.txt
u:01:gnu
d:04:linux
u:01:bash
u:01:hack
```

Shell

为了只测试指定的字符（忽略前两个字符，使用接下来的两个字符），我们使用`-s 2`跳过前两个字符，使用`-w 2`选项指定后续的两个字符：

```shell
$ sort data.txt | uniq -s 2 -w 2
d:04:linux
u:01:bash
```

Shell

我们将命令输出作为`xargs`命令的输入时，最好为输出的各行添加一个0值字节（zero-byte）终止符。使用`uniq`命令的输入作为`xargs`的数据源时，同样应当如此。如果没有使用0值字节终止符，那么在默认情况下，`xargs`命令会用空格来分割参数。例如，来自`stdin`的文本行“this is a line”会被`xargs`视为4个不同的参数。如果使用0值字节终止符，那么`\0`就被作为定界符，此时，包含空格的行就能够被正确地解析为单个参数。
`-z`选项可以生成由0值字节终止的输出：

```shell
$ uniq -z file.txt
```

Shell

下面的命令将删除所有指定的文件，这些文件的名字是从files.txt中读取的：

```shell
$ uniq -z file.txt | xargs -0 rm
```

Shell

如果某个文件名出现多次，`uniq`命令只会将这个文件名写入`stdout`一次，这样就可以避免出现`rm: cannot remove FILENAME: No such file or directory`。