---
title: linux查看日志常用命令
date: 2018-11-11 15:45:27
tags:
---

## 命令

### 必须掌握
1. cat
2. grep
3. awk
3. tail
4. head 
5. wc (word count)
6. less
7. sort[todo]
8. uniq[todo]

### 不太常用但是有用
1. sed
2. tac
3. nl



## 详细

更详细用法可以 `man command `

### 1. cat

1. 功能：
    1. 查看一个文本所有信息（经常和grep结合）
    2. 黏合文件
2. 用法：
    cat [-benstuv] [file ...]
3. 常用参数:
    共7个参数，常用的是-n
    -n  带上行数
4. demo
    1. 合并文件
        cat test1 test2 > test3 (> 表示覆盖写，创建的意思)
        cat test1 test2 >> test3 (>> 表示追加写)
    2. 带行数的查看文件
        cat -n filename

### 2. grep
1. 功能：
    正则匹配的查找文件内容
2. 用法：
    grep 
        [-abcdDEFGHhIiJLlmnOopqRSsUVvwxZ]
        [-A num]
        [-B num]
        [-C[num]]
        [-e pattern]
        [-f file]
        [--binary-files=value]
        [--color[=when]]
        [--colour[=when]]
        [--context[=num]]
        [--label]
        [--line-buffered]
        [--null]
        [pattern]
            [file ...]

3. 常用参数:
    -a   --text   #不要忽略二进制的数据。   
    -A<显示行数>   --after-context=<显示行数>   #除了显示符合范本样式的那一列之外，并显示该行之后的内容。   
    -b   --byte-offset   #在显示符合样式的那一行之前，标示出该行第一个字符的编号。   
    -B<显示行数>   --before-context=<显示行数>   #除了显示符合样式的那一行之外，并显示该行之前的内容。   
    -c    --count   #计算符合样式的列数。   
    -C<显示行数>    --context=<显示行数>或-<显示行数>   #除了显示符合样式的那一行之外，并显示该行之前后的内容。   
    -d <动作>      --directories=<动作>   #当指定要查找的是目录而非文件时，必须使用这项参数，否则grep指令将回报信息并停止动作。   
    -e<范本样式>  --regexp=<范本样式>   #指定字符串做为查找文件内容的样式。   
    -E      --extended-regexp   #将样式为延伸的普通表示法来使用。   
    -F   --fixed-regexp   #将样式视为固定字符串的列表。   
    -G   --basic-regexp   #将样式视为普通的表示法来使用。   
    -h   --no-filename   #在显示符合样式的那一行之前，不标示该行所属的文件名称。   
    -H   --with-filename   #在显示符合样式的那一行之前，表示该行所属的文件名称。   
    -i    --ignore-case   #忽略字符大小写的差别。   
    -l    --file-with-matches   #列出文件内容符合指定的样式的文件名称。   
    -L   --files-without-match   #列出文件内容不符合指定的样式的文件名称。   
    -n   --line-number   #在显示符合样式的那一行之前，标示出该行的列数编号。   
    -q   --quiet或--silent   #不显示任何信息。   
    -r   --recursive   #此参数的效果和指定“-d recurse”参数相同。   
    -s   --no-messages   #不显示错误信息。   
    -v   --revert-match   #显示不包含匹配文本的所有行。   
    -V   --version   #显示版本信息。   
    -w   --word-regexp   #只显示全字符合的列。   
    -x    --line-regexp   #只显示全列符合的列。   
    -y   #此参数的效果和指定“-i”参数相同。

4. 正则匹配规则

grep的规则表达式:

    ^  #锚定行的开始 如：'^grep'匹配所有以grep开头的行。    
    $  #锚定行的结束 如：'grep$'匹配所有以grep结尾的行。    
    .  #匹配一个非换行符的字符 如：'gr.p'匹配gr后接一个任意字符，然后是p。    
    *  #匹配零个或多个先前字符 如：'*grep'匹配所有一个或多个空格后紧跟grep的行。    
    .*   #一起用代表任意字符。   
    []   #匹配一个指定范围内的字符，如'[Gg]rep'匹配Grep和grep。    
    [^]  #匹配一个不在指定范围内的字符，如：'[^A-FH-Z]rep'匹配不包含A-R和T-Z的一个字母开头，紧跟rep的行。    
    \(..\)  #标记匹配字符，如'\(love\)'，love被标记为1。    
    \<      #锚定单词的开始，如:'\<grep'匹配包含以grep开头的单词的行。    
    \>      #锚定单词的结束，如'grep\>'匹配包含以grep结尾的单词的行。    
    x\{m\}  #重复字符x，m次，如：'0\{5\}'匹配包含5个o的行。    
    x\{m,\}  #重复字符x,至少m次，如：'o\{5,\}'匹配至少有5个o的行。    
    x\{m,n\}  #重复字符x，至少m次，不多于n次，如：'o\{5,10\}'匹配5--10个o的行。   
    \w    #匹配文字和数字字符，也就是[A-Za-z0-9]，如：'G\w*p'匹配以G后跟零个或多个文字或数字字符，然后是p。   
    \W    #\w的反置形式，匹配一个或多个非单词字符，如点号句号等。   
    \b    #单词锁定符，如: '\bgrep\b'只匹配grep

4. demo
    1. 查看进程 `ps -ef|grep svn`
    2. 统计进程数 `ps -ef|grep svn -c`
    3. 管道查看关键字的行 `cat test.txt | grep -F keyword`
    4. 管道查看关键字的行并且打印行号`cat test.txt | grep -nf keyword`
    5. 直接根据关键字查看行 `grep keyword filename`
    6. 多文件 `grep keyword filename1 filename2`
    6. 所有文件 `grep keyword *`
    7. 查看以**开头的行 `cat test.txt |grep ^u`
    8. 查看以**结尾的行 `cat test.txt |grep hat$`
    9. 查看不以**开头的行 `cat test.txt |grep ^[^u]`
    10. 查看有\*\*或者\*\*的行`cat test.txt |grep -E "ed|at"`

### awk
1. 功能：
    awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。
    特别适合一行文本特别长的时候进行显示处理
    
2. 用法：
    awk '{pattern + action}' {filenames}
    其中action支持多种操作，强大又复杂
3. 常用参数:
    -F：分割符
4. demo
    1. 查看某一行的前面部分：`cat filename | awk -F '|' '{print $1, $2}'`

### tail/head
1. 功能：
    tail 显示文件末尾/开始的内容
2. 用法：
    - tail [-F | -f | -r] [-q] [-b number | -c number | -n number] [file ...]
    - head [-n count | -c bytes] [file ...]
3. 常用参数:
    -f：循环读取
    -n<行数>： 显示行数
4. demo
    1. 监控日志：`tail -f filename`
    2. 从第n行开始查看文件：`tail -n +100 filename` (必须有`+`)
    3. 查看前n行的内容：`head -n 20 filename`
    4. 查看n-m行之间的内容：`cat -n  info.log | tail -n +140 | head -n 2`

### wc
1. 功能：
    wc -- word, line, character, and byte count
2. 用法：
    wc [-clmw] [file ...]
3. 常用参数:
    -c 统计字节数。
    -l 统计行数。
    -m 统计字符数。这个标志不能与 -c 标志一起使用。
    -w 统计字数。一个字被定义为由空白、跳格或换行字符分隔的字符串
4. demo
    1. 统计行数: `wc -l filename`
    2. 统计当前目录下的文件数: `ls -l | wc -l`

### less
less 命令（分页查看文件内容）分页查看日志，但是中文有乱码
`less error.log`
直接定位到第100行
`less +100g xx.log`
定位到最后一行
`less +GG xx.log`
查找并高亮关键字
`less fis.log.2018-05-20  | grep 2018052019004984219071028 -A 5 --color=auto`
移动日志
    G ：到日志最后
    g ：到日志最前面
    j/↑ ：向前移动一行
    k/↓ ：向后移动一行
    pgup ：向上翻页
    pgdn ：向下翻页


### sed

查看n-m行之间的内容： `sed -n '5,10p' filename `

### tac
    
和cat反着来的

### nl
    
加强版 `cat -n`,使用参考 `man nl`


## 常用组合

1. 实时监控日志：
    
        tail -f 20 filename

2. 显示一个文件的某几行

        1. `cat -n  info.log | tail -n +140 | head -n 2`
        2. `sed -n "10,20p" filename`

3. 统计行数：

        1. wc -l filename

4. 现在有一万多条记录，其中包含重复的记录，每条记录占一行，问如何从这些记录中找到数量排名前10的记录:

        sort data | uniq -c | sort -k 1 -n -r | head 10

    > 1) sort data
    >    表示对data文件中的内容进行排序。sort命令是对于每一行的内容根据字典序（ASCII码）进行排序，这样可以保证重复的记录时相邻的。
    > 2) sort data | uniq -c
    >    这里，通过管道（|）将左边部分的命令的输出作为右边部分的输入。uniq -c 表示合并相邻的重复记录，并统计重复数。因为uniq -c 只会合并相邻的记录，所以在使用该命令之前需要先排序。
    > 3) sort data | uniq -c | sort -k 1 -n -r
    >    经过uniq -c 处理之后的数据格式形如"2 data"，第一个字段是数字，表示重复的记录数；第二个字段为记录的内容。我们将对此内容进行排序。sort -k 1表示对于每行的第一个字段进行排序，这里即指代表重复记录数的那个字段。因为sort命令的默认排序是按照ASCII，这就会导致按从大到小进行排序时，数值2会排在数值11的前面，所以需要使用-n 参数指定sort命令按照数值大小进行排序。-r 表示逆序，即按照从大到小的顺序进行排序。
    > 4) sort data | uniq -c | sort -k 1 -n -r | head 10
        head 命令表示选取文本的前x行。通过head 10 就可以得到排序结果中前十行的内容
    > 
    > 来自[blog](http://eriol.iteye.com/blog/870641)