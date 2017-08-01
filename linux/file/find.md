# find
## 基本格式
```bash
find   start_directory  test  options   criteria_to_match
action_to_perform_on_results
```
## 根据名字查找文件                         
在以下命令中， find 将开始在**当前目录（用“.”表示）**中查找任何扩展名为“java”的文件：
find . -name  "*.java"  
下面是该命令所找到的命令的缩略清单：
```bash
find . -name  "*.java"
```
以下命令将执行相同的操作。
```bash
find . -name  \*.java
```
在这两种情况下，您都需要对通配符进行转义以确保它传递到 find 命令并且不由 shell 解释。因此，请将您的**搜索字符串放到引号里，或者在它前面加上反斜线**

### 指定查找路径
尽管 find 的所有参数均为可选，但是如果您未指定从哪里开始搜索，**搜索默认将在当前目录中开始**。如果您不指定要匹配的测试连接、选项或值，您的结果将不完整或者无区别。 
  
运行以下三个 find 命令将得出同样的结果 — 当前目录和所有子目录中的所有文件（包括隐藏文件）的完整清单：
```bash
find 
find .
find . -print
```
如果您希望上述命令的输出包含完整的路径名（或许是为了备份），您将需要指定起始目录的完整路径：
```bash
find /home/bluher -name \*.java
```
/home/bluher/plsql/REGEXPvalidate/src/oracle/otnsamples/plsql/ConnectionManager.java

/home/bluher/plsql/REGEXPvalidate/src/oracle/otnsamples/plsql/DBManager.java/
...

### 指定多个查找路径
您还可以在搜索字符串中指定多个起始目录。如果以具有相应权限的用户身份运行，以下命令将下到 /usr、/home /tmp 目录查找所有 jar 文件：
```bash
find /usr /home  /tmp -name "*.jar"
```
但是，如果您没有相应的权限，您在开始浏览许多系统目录时将生成错误消息。以下是一个示例：
```bash
find:  /tmp/orbit-root: Permission denied
```
您可以通过附加您的搜索字符串来避免混乱的输出，如下所示：
```bash
find /usr /home  /tmp -name "*.jar" 2>/dev/null
```
这会将所有错误消息发送到空文件，因此提供清理器输出。

### 不区分大小写查找
默认情况下，find 是区分大小写的。**对于不区分大小写的 find，用-iname替换-name。**
```bash
find downloads  -iname "*.gif"
```
downloads/.xvpics/Calendar05_enlarged.gif

downloads/lcmgcfexsmall.GIF

## 根据类型搜索文件
**除文件名外，您还可以按类型搜索文件**。例如，您可以使用以下命令查找一个目录中的所有子目录：
```bash
find . -type d    
```
您可以使用以下命令查找您的/usr 目录中的所有符号链接：
```bash
find /usr -type l
```
以下的任何一个命令使用根权限运行都将列出 /usr 目录中的链接以及它所指向的文件：
```bash
find /usr/bin  -type l  -name "z*" -exec ls  -l {} \;
lrwxrwxrwx 1 root  root 8 Dec 12 23:17 /usr/bin/zsh -> /bin/zsh
lrwxrwxrwx 1 root  root 5 Dec 12 23:17 /usr/bin/zless -> zmore
lrwxrwxrwx 1 root  root 9 Dec 12 23:17 /usr/bin/zcat -> /bin/zcat

find /usr/bin -type  l  -name "z*" -ls
```
但是，第二个更短的命令将列出更多的文件，以及目录和 inode 信息。
### 可查找的类型
其他 find 可以找到的文件类型包括：

• b — 块（缓存）特殊 

• c — 字符（未缓存）特殊 

• p — 命名管道 (FIFO) 

• s — 套接字

使用根作为 find 命令的起点会极大地降低系统的速度。如果您必须运行这样一个命令，您可以在非高峰时段或晚上运行它。您可以使用以下语法将输出重定向到一个文件：
```bash
find  / -print > masterfilelist.out
```

### -exec
exec解释：

-exec  参数后面跟的是command命令，它的终止是以;为结束标志的，所以这句命令后面的分号是不可缺少的，考虑到各个系统中分号会有不同的意义，所以前面加反斜杠:**\;**

**{}   花括号代表前面find查找出来的文件名。**

**注意：{}和\;之间一定要有空格**

使用find时，只要把想要的操作写在一个文件里，就可以用exec来配合find查找，很方便的。在有些操作系统中只允许-exec选项执行诸如ls或ls -l这样的命令。大多数用户使用这一选项是为了查找旧文件并删除它们。建议在真正执行rm命令删除文件之前，最好先用ls命令看一下，确认它们是所要删除的文件。 exec选项后面跟随着所要执行的命令或脚本，然后是一对{ }，一个空格和一个\，最后是一个分号。为了使用exec选项，必须要同时使用print选项。如果验证一下find命令，会发现该命令只输出从当前路径起的相对路径及文件名。


实例1：ls -l命令放在find命令的-exec选项中

命令：find . -type f -exec ls -l {} \;

说明：find命令匹配到了当前目录下的所有普通文件，并在-exec选项中使用ls -l命令将它们列出。

```bash

[root@localhost test]# find . -type f -exec ls -l {} \;   
-rw-r--r-- 1 root root 127 10-28 16:51 ./log2014.log  
-rw-r--r-- 1 root root 0 10-28 14:47 ./test4/log3-2.log  
-rw-r--r-- 1 root root 0 10-28 14:47 ./test4/log3-3.log  
-rw-r--r-- 1 root root 0 10-28 14:47 ./test4/log3-1.log  
-rw-r--r-- 1 root root 33 10-28 16:54 ./log2013.log  
-rw-r--r-- 1 root root 302108 11-03 06:19 ./log2012.log  
-rw-r--r-- 1 root root 25 10-28 17:02 ./log.log  
-rw-r--r-- 1 root root 37 10-28 17:07 ./log.txt  
-rw-r--r-- 1 root root 0 10-28 14:47 ./test3/log3-2.log  
-rw-r--r-- 1 root root 0 10-28 14:47 ./test3/log3-3.log  
-rw-r--r-- 1 root root 0 10-28 14:47 ./test3/log3-1.log  
[root@localhost test]#  
```

实例2：在目录中查找更改时间在n日以前的文件并删除它们

命令：find . -type f -mtime +14 -exec rm {} \;

说明：在shell中用任何方式删除文件之前，应当先查看相应的文件，一定要小心！当使用诸如mv或rm命令时，可以使用-exec选项的安全模式。它将在对每个匹配到的文件进行操作之前提示你。


实例3：在目录中查找更改时间在n日以前的文件并删除它们，在删除之前先给出提示

命令：find . -name "*.log" -mtime +5 -ok rm {} \;

说明：在上面的例子中， find命令在当前目录中查找所有文件名以.log结尾、更改时间在5日以上的文件，并删除它们，只不过在删除之前先给出提示。 按y键删除文件，按n键不删除。


实例4：-exec中使用grep命令

命令：find /etc -name "passwd*" -exec grep "root" {} \;

说明：任何形式的命令都可以在-exec选项中使用。在上面的例子中我们使用grep命令。find命令首先匹配所有文件名为“passwd*”的文件，例如passwd、passwd.old、passwd.bak，然后执行grep命令看看在这些文件中是否存在一个"spp"关键词。


实例5：查找文件并移动到指定目录  

命令：find . -name "*.log" -exec mv {} .. \;


实例6：用exec选项执行cp命令  

命令：find . -name "*.log" -exec cp {} test3 \;


## 中止查找
如果您错误地输入一个 find 命令，生成大量不必要的输出，只需按 CTRL-C 中断该命令，这将停止最近执行的命令。

**************************************不常用**************************************
## 按时间查找
find 命令有几个用于根据您系统的时间戳搜索文件的选项。这些时间戳包括

• mtime — 文件内容上次修改时间 

• atime — 文件被读取或访问的时间 

• ctime — 文件状态变化时间

mtime 和 atime 的含义都是很容易理解的，而 ctime 则需要更多的解释。由于 inode 维护着每个文件上的元数据，因此，如果与文件有关的元数据发生变化，则 inode 数据也将变化。这可能是由一系列操作引起的，包括创建到文件的符号链接、更改文件权限或移动了文件等。由于在这些情况下，文件内容不会被读取或修改，因此 mtime 和 atime 不会改变，但 ctime 将发生变化。

这些时间选项都需要与一个值 n 结合使用，指定为 -n、n 或 +n。

• -n 返回项小于 n 

• +n 返回项大于 n 

• n 返回项正好与 n 相等


下面，我们来看几个例子，以便于理解。

以下命令将查找在**最近1小时内**修改的所有文件：
```bash
find . -mtime -1
```
./plsql/FORALLSample

./plsql/RegExpDNASample

/plsql/RegExpSample

用 1 取代 -1 运行同一命令将查找恰好在**1小时以前**修改的所有文件：
```bash
find . -mtime 1
```
上述命令不会生成任何结果，因为它要求完全吻合。以下命令查找**1个多小时以前**修改的所有文件：
```bash
find . -mtime +1
```
默认情况下，-mtime、-atime 和 -ctime 指的是最近 24 小时。但是，如果它们前面加上了开始时间选项，则 24 小时的周期将从当日的开始时间算起。您还可以使用 mmin、amin 和 cmin 查找在不到 1 小时的时间内变化了的时间戳。

如果您在登录到您的帐户后立即运行以下命令，您将找到在不到1分钟以前读取的所有文件：
```bash
find . -amin -1
```
./.bashrc
/.bash_history
./.xauthj5FCx1
应该注意的是，使用 find 命令查找文件本身将更改该文件的访问时间作为其元数据的一部分。

您还可以使用 -newer、-anewer 和 –cnewer 选项查找已修改或访问过的文件与特定的文件比较。这类似于 -mtime、-atime 和 –ctime。 
  
• -newer 指内容最近被修改的文件 

• -anewer 指最近被读取过的文件 

• -cnewer 指状态最近发生变化的文件


要查找您的主目录中自上一个 tar 文件以来以某种方式编辑过的所有文件，使用以下命令：
```bash
find . -newer  backup.tar.gz
```

## 按大小查找文件
-size 选项查找满足指定的大小条件的文件。要查找所有大于 5MB 的用户文件，使用
```bash
find / -size  +5000000c 2> /dev/null
```
/var/log/lastlog
/var/log/cups/access_log.4
/var/spool/mail/bluher
结尾的“c”以字节为单位报告我们的结果。默认情况下，find 以 512 字节块的数量报告大小。如果我们将“c”替换为“k”，我们还会看到以千字节的数量报告的结果，如果使用“w”，则会看到以两字节字的数量报告的结果。

-size 选项经常用于搜索所有零字节文件并将它们移至 /tmp/zerobyte 文件夹。以下命令恰好可以完成这一任务：
```bash
find test -type f  -size 0 -exec mv {} /tmp/zerobyte \;
```
**-exec 操作允许 find 在它遇到的文件上执行任何 shell 命令。在本文的后面部分，您将看到其用法的更多示例。大括号允许移动每个空文件。**

## 查找空文件
选项 -empty 还可用于查找空文件：
```bash
find test -empty        
```
test/foo

test/test

## 按权限和所有者查找
要监视您的系统安全离不开 find 命令。您可以使用符号或八进制表示法查找面向广大用户开放的文件，如下所示：
```bash
find . -type f  -perm a=rwx -exec ls -l {} \;
```
或者
```bash
find . -type f  -perm 777 -exec ls -l {} \;
```
-rwxrwxrwx 1 bluher  users 0 May 24 14:14 ./test.txt

在这一部分中，在上面和下面的命令中，我们使用了 -exec ls -l 操作，因此，您可以看到返回的文件的实际权限。以下命令将查找可由“other”和组写入的文件：
```bash
find plsql -type f  -perm -ug=rw -exec ls -l {} \; 2>/dev/null
```
或者
```bash
find plsql -type f  -perm -220 -exec ls -l {} \; 2>/dev/null 
```
-rw-rw-rw- 1 bluher users 4303  Jun  7   2004 plsql/FORALLSample/doc/otn_new.css

-rw-rw-rw- 1 bluher users 10286 Jan  12  2005  plsql/FORALLSample/doc/readme.html

-rw-rw-rw- 1 bluher users 22647 Jan  12  2005  plsql/FORALLSample/src/config.sql

..

下一个命令将查找由用户、组或二者共同写入的文件：  
```bash
find plsql -type f  -perm /ug=rw -exec ls -l {} \; 2>/dev/null, or,
find plsql -type f  -perm /220 -exec ls -l {} \; 2>/dev/null 
```
-rw-r--r-- 1 bluher users 21473  May  3 16:02 plsql/regexpvalidate.zip

-rw-rw-rw- 1 bluher users 4303  Jun  7   2004 plsql/FORALLSample/doc/otn_new.css

-rw-rw-rw- 1 bluher users 10286 Jan  12  2005  plsql/FORALLSample/doc/readme.html

-rw-rw-rw- 1 bluher users 22647 Jan  12  2005  plsql/FORALLSample/src/config.sql

您可能会看到以下命令在 Web 和较早的手册中引用过：
```bash
find . -perm +220  -exec ls -l {} \; 2> /dev/null
```
\+ 符号的作用与 / 符号相同，但是现在新版 GNU findutils 中不支持使用该符号。

要查找您的系统上所有人都可以写入的所有文件，使用以下命令：
```bash
find / -wholename  '/proc' -prune  -o  -type f -perm -0002 -exec ls -l {} \;
```
-rw-rw-rw- 1 bluher users 4303  Jun  7   2004/home/bluher/plsql/FORALLSample/doc/otn_new.css

-rw-rw-rw- 1 bluher users 10286 Jan  12  2005  /home/bluher/plsql/FORALLSample/doc/readme.html

...

第 4 个权限将在稍后进行讨论，但最后一个字段中的“2”是文件权限中的“other”字段，也称为写入位。我们在权限模式 0002 前面使用了破折号，以指明我们希望看到为 other 设置了写权限的文件，无论其他权限设置为什么。

上述命令还引入了三个新概念。针对文件模式“/proc”使用 -wholename 测试，如果该模式已找到，-prune 可防止 find 下到该目录中。布尔类型“-o”使 find 可以针对其他目录处理该命令的其余部分。由于每个表达式之间有一个假设的隐式 and 运算符 (-a)，因此，如果左侧的表达式计算结果为 false， and 之后的表达式将不进行计算；因此需要 -o 运算符。Find 还支持布尔类型 -not、!，就像使用括号强行优先一样。

系统管理员经常使用 find 通过用户或组的名称或 ID 搜索特定用户或组的常规文件：

```bash
[root] $  find / -type f -user bluher -exec ls -ls {}  \;
```
下面是这样一个命令的高度精简的输出示例：

4 -rw-r--r-- 1 bluher users 48  May  1 03:09  /home/bluher/public_html/.directory

4 -rw-r--r-- 1 bluher users 925  May  1 03:09 /home/bluher/.profile

您还可以使用 find 按组查找文件：

```bash
[root] $ find /  -type f -group users
find / -type d -gid  100
```
该命令将列出由 ID 为 100 的组拥有的目录。要找到相应的 uid 或 gid，您可以针对 /etc/passwd 或 /etc/group 文件运行 more 或 cat 命令。

除了查找特定已知用户和组的文件外，您还会发现它对于查找没有这些信息的文件也很有用。下一个命令将识别未列在 /etc/passwd 或 /etc/group 文件中的文件：
```bash
find / -nouser -o  -nogroup
```
上述命令可能不会在您的系统上生成实际的结果。但是，它可用于识别或许在经常移动后没有用户或组的文件。

好了，现在我们可以解决本部分开始时提到的格外重要的权限了。

SGID 和 SUID 是特殊访问权限标志，可以分配给基于 UNIX 的操作系统上的文件和目录。设置它们是为了允许访问计算机系统的普通用户使用临时提升的权限执行二进制可执行文件。
```
find /  \( -perm -2000 -o -perm -4000 \) -ls
bash
```
167901   12 -rwsr-xr-x   1 root     root         9340 Jun 16  2006 /usr/bin/rsh

167334   12 -rwxr-sr-x   1 root     tty         10532 May  4  2007 /usr/bin/wall

在上述命令中，您可以看到转义括号的使用。您还可以看到权限的不同。第一个文件设置了 SGID 权限，第二个文件设置了 SUID 权限。上述命令中的最后的操作与带 -exec ls -dils 操作的 find 效果类似。

## 控制 find
与 Linux 中的许多命令不同，find 不需要 -r 或 -R 选项即可下到子目录中。它默认情况下就这样操作。但是，有时您可能希望限制这一行为。因此，选项 -depth、-maxdepth 和 -mindepth 以及操作 -prune 就派上用场了。

我们已经看到了 -prune 是多么有用，下面让我们来看看 -depth、-maxdepth 和 -mindepth 选项。

-maxdepth 和 -mindepth 选项允许您指定您希望 find 搜索深入到目录树的哪一级别。如果您希望 find 只在目录的一个级别中查找，您可以使用 maxdepth 选项。

通过运行以下命令在目录树的前三个级别中查找日志文件，您可以看到 -maxdepth 的效果。使用该选项较之不使用该选项所生成的输出要少得多。
```bash
find / -maxdepth 3  -name "*log"
```
您还可以让 find 在至少下至目录树三个级别的目录中查找：
```bash
find / -mindepth 3  -name "*log"
```
-depth 选项确保先在一个目录中进行查找，然后才在其子目录中进行查找。以下命令提供了一个示例：
```bash
find -name "*test*" -depth
./test/test
./test
./localbin/test
./localbin/test_shell_var
./localbin/test.txt
./test2/test/test
./test2/test
./test2
```
