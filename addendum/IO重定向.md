# Unix Shell I/O重定向（包括管道）

> [270_redirection (idallen.com)](http://teaching.idallen.com/cst8207/12w/notes/270_redirection.txt)
>
> 伊恩！D.艾伦 - idallen@idallen.ca - www.idallen.com

在下面的例子中，我使用了元字符";"在一个shell命令行上输入多个命令：date ; who ; echo hi ; pwd。这些行为就像你在单独的行中输入了它们一样。

## 1.输出重定向-标准输出和标准错误

在输出重定向中，shell 程序通常会将出现在屏幕上的大部分命令输出转移（重定向）到其他地方，或者进入另一个命令的输入（使用管道元字符 '|'）或者进入一个文件（使用文件重定向元字符 '>' ）。

- 重定向首先由shell在找到命令之前完成；shell不知道该命令是否存在或是否会产生任何输出。

- 你只能重定向你能看到的输出。 如果没有重定向就没有可见的输出，添加重定向就不会产生任何

- 重定向只能转到一个地方。不能使用多个重定向将输出发送到多个位置。（请参见“tee”命令。）

- 默认情况下，错误消息（称为“标准错误”或“标准错误”）不会被重定向；只有“正常输出”（称为“标准输出”或“标准输出”）被重定向（但您也可以使用更多语法重定向stderr）。

## 1.1 输出重定向到文件

shell 元字符 “>” 表示命令行上的下一个参数是一个输出文件（不是程序），应该创建或截断该文件（设置为空），并准备好接收命令的标准输出：

```shell
date >outfile
```

在'>'和文件名之间的空格是可选的。

```shell
date   >   outfile
```

文件总是在 shell 找到并运行该命令之前创建或截断为空，除非使用以下方式追加文本信息。

```shell
date >> outfile   # 输出是*添加到outfile上的，没有截断。
```

将输出重定向到一个文件的例子：

```shell
 echo hello                   # 打印 hello 到终端（屏幕）
    hello

    echo hello >file          # 擦除“file”文件内容;发送输出到文件中
    cat file                  # 查看文件内容
    hello

    echo there >>file         # 追加输出到文件的末尾
    cat file                  # 查看文件内容
    hello
    there
```

是shell创建或截断了文件并设置了重定向，而不是被重定向的命令。 命令对重定向一无所知--在找到并执行命令之前，重定向的语法已经从命令行中删除。   

```shell
 echo one two three                   # echo has three arguments
 one two three
 echo one two three >out              # echo still has three arguments
 cat out
 one two three
```

shell在寻找要运行的命令名之前会处理重定向。实际上，即使找不到命令或根本没有命令，也可以重定向：

```shell
    nosuchcommandxxx >out                # file "out" is created empty
    sh: nosuchcommandxxx: command not found
    wc out
     0 0 0 out                             # shell created an empty file

    >out                                 # file "out" is created empty
    wc out
     0 0 0 out                             # shell created an empty file
```

shell创建或截断文件“out”为空，然后尝试查找并运行不存在的命令，但失败。空文件仍然存在。任何现有文件的内容都将被删除：

```shell
    echo hello >out ; cat out
    hello
    nosuchcommandxxx >out
    sh: nosuchcommandxxx: command not found
    wc out
      0 0 0 out                             # shell truncated the file
```

重定向是由shell在运行命令之前完成的。

```shell
    mkdir empty
    cd empty
    ls -l
    total 0                               # no files found

    ls -l >out                            # shell creates "out" first
    cat out                               # display output
    total 0
    -rw-r--r--  1 idallen idallen 0 Sep 21 06:02 out

    date >out
    ls -l
    total 4
    -rw-r--r--  1 idallen idallen 29 Sep 21 06:04 out

    ls -l >out                            # shell empties "out" first
    cat out                               # display output
    total 0
    -rw-r--r--  1 idallen idallen 0 Sep 21 06:06 out
```

shell在运行 "ls "命令之前创建或清空了 "out "文件。
解释一下这个命令的顺序。

```shell
    date
    Wed Feb  8 03:01:11 EST 2012

    date >a
    cat a
    Wed Feb  8 03:01:21 EST 2012

    cp a b
    cat b
    Wed Feb  8 03:01:21 EST 2012

    cp a b >a
    cat b
                                  # why is file b empty?
```

Shell 并不关心你在命令行的什么地方进行文件的重定向。文件重定向由shell完成，然后在调用命令之前从命令行中删除重定向语法。实际运行的命令没有看到重定向语法的任何部分；参数的数量不受影响。
下面的所有命令行都相当于shell；在每种情况下，echo命令只看到三个参数，三个命令行参数“hi”、“there”和“mom”都被重定向到“file”：

```shell
    echo hi there mom >file                   # echo has three arguments
    echo hi there >file mom                   # echo has three arguments
    echo hi >file there mom                   # echo has three arguments
    echo >file hi there mom                   # echo has three arguments
    >file echo hi there mom                   # echo has three arguments
```

在命令运行之前，重定向语法会被shell删除。所以，重定向语法永远不会被算作命令的参数。
例如：

```shell
    echo hello there
      - shell调用有两个参数的 "echo"==> echo(hello,there)
      - "echo "在标准输出上呼应两个参数
      - 输出出现在默认位置（标准输出是你的屏幕）。

    echo hello there >file
      - shell创建了 "file "并将标准输出转入其中。
      - shell从命令行中删除语法">file"。
      - shell调用有两个参数的 "echo"==> echo(hello,there)
       (注意 "echo "的参数与前面的例子没有变化)
      - "echo "在标准输出上呼出两个参数
      - 标准输出被捕获在输出 "文件 "中，而不是在你的屏幕上。

    >file echo hello there
      - 这和上面的例子是一样的（shell并不关心你把重定向放在命令行的什么地方)
      - 标准输出被捕获在输出 "file"中，而不是在你的屏幕上
      - 你可以把重定向放在命令行中的任何地方 
```

重定向是由shell首先完成的，甚至在找到命令之前。

- shell会创建一个新的空文件或者截断（清空）一个现有的文件
- 在做完重定向，并从命令行中删除语法后，shell会找到并执行命令（如果有的话）行中的语法，shell会找到并执行命令（如果有的话）。

解释一下这一系列的命令:

```shell
    rm
    rm: missing operand

    touch file
    rm >file
    rm: missing operand               # why doesn't rm remove "file"?

    rm nosuchfile
    rm: cannot remove `nosuchfile': No such file or directory

    rm nosuchfile >nosuchfile
                                    # why is there no rm error message here?
```

你只能重定向你能看到的输出! *只有*你所看到的!

- 重定向不会发明新的输出! *只有你看到的！*

- 如果你没有看到一个命令的任何输出，添加重定向将只是让shell创建一个空文件（没有输出）。
  
  例如：cp /etc/passwd x # 在标准输出上没有输出

```shell
    cp /etc/passwd x >out   # file "out" is created empty

    cd /tmp                 # no output on standard output
    cd /tmp >out            # file "out" is created empty

    touch x ; rm x          # no output from rm on standard output
    touch x ; rm x >out     # file "out" is created empty
```

重定向只能去一个地方。

- 最右边的文件重定向会获得输出（其他文件创建空文件）
  例如：`date >a >b >c` ， 输出进入文件c；a和b是空的。

重定向到一个文件胜过重定向到一个管道。

- 详情见下面关于使用"|"管道重定向到程序的部分
- 如果你重定向到一个文件和一个管道，那么管道什么也得不到。
  例子：`date >a | cat` ， 输出进入文件 "a"；cat没有显示任何内容

重定向的输出文件被清空（截断），除非你通过">>"添加内容

- 在shell寻找和运行命令之前，该文件就被清空了
- 不要把输出的重定向文件作为同一命令的输入。 坏例子：sort a >a # 错！文件 "a "被截断为空

### 标准输出（"stdout"）和标准错误（"stderr"）

大多数命令有两个独立的输出 流，编号为1和2。

1. stdout - 第1单元 - 标准输出（正常输出）
2. stderr - 第2单元 - 标准错误输出（错误和警告信息）

你屏幕上的正常（非错误）"单元1 "输出来自于 命令的 "标准输出"（"stdout"）。Stdout是指在C和C++程序中，"printf "和 "cout "语句的输出，以及从"System.print "和 "System.println "在Java中。这是预期的。

命令的通常输出。

在你的屏幕上输出的错误信息 "unit 2 "来自于 "标准错误输出"（"stderr"）的命令。Stderr是来自"fprintf(stderr) "和 "cerr "语句在C和C++程序中的输出，以及从"System.err.print "和Java中的 "System.err.println"。程序只在此输出上打印错误信息。

stdout和stderr混合在你的终端屏幕上。它们在屏幕上看起来是一样的，所以你无法通过观察你的屏幕来判断什么是来自stdout的程序，什么是来自stderr的程序。

为了说明stdout和stderr同时出现在屏幕上的一个简单例子，使用 "ls "命令，给它一个存在的文件名和一个不存在的文件名（因此导致显示错误信息）。

```shell
ls -l /etc/passwd nosuchfile
ls: nosuchfile: No such file or directory             # standard error
-rw-r--r-- 1 root root 2209 Jan 19 20:39 /etc/passwd  # standard output
```

stderr(错误信息)的输出常常先于stdout出现，这是由于命令为stdout使用了内部I/O缓冲区。

通常，stdout和stderr一起出现在你的终端上。shell可以将这两个输出单独或一起重定向到文件或其他程序中。默认的输出重定向类型（无论是重定向到文件还是重定向到使用管道的程序）只重定向标准输出，而让标准错误进入你的终端，不被触及。

下面是一些使用shell文件重定向元字符">"的例子。

```shell
ls /etc/passwd nosuchfile                 # no redirection used
ls: nosuchfile: No such file or directory   # this on screen from stderr
/etc/passwd                                 # this on screen from stdout

ls /etc/passwd nosuchfile >out            # shell redirects only stdout
ls: nosuchfile: No such file or directory   # only stderr appears on screen

cat out
/etc/passwd
```

你可以使用'>'元字符前的单元号将stdout和stderr分别重定向到文件中：

- stdout总是单元1

- stderr总是单元2（stdin是单元0） - 将单元号立即放在'>'元字符前（没有空格）。

">foo"（前面没有单元号）是shell对 "1>foo "的简写。
">foo "只重定向默认单位1（stdout），不重定向stderr
">foo "和 "1>foo "是相同的。

你也可以告诉shell将标准错误，即单元2，重定向到一个文件。

```shell
ls /etc/passwd nosuchfile 2>errors        # shell redirects only stderr
/etc/passwd                                 # only stdout appears on screen

cat errors
ls: nosuchfile: No such file or directory
```

你可以把stdout重定向到一个文件，把stderr重定向到另一个文件。

```shell
ls /etc/passwd nosuchfile >out 2>errors   # shell redirects each one
                                          # nothing appears on screen

cat out
/etc/passwd

cat errors
ls: nosuchfile: No such file or directory
```

你需要一个特殊的语法 "2>&1 "来将stdout和stderr一起安全地重定向到Bourne shells中的一个文件。将该语法理解为
"2>&1 "是指 "将单元2发送到单元1的同一个地方"。

```shell
ls /etc/passwd nosuchfile >both 2>&1      # redirect both into same file
                                          # nothing appears on screen

cat both
ls: nosuchfile: No such file or directory
/etc/passwd
```

`>both`和`2>&1`在命令行中的顺序很重要!

`>both`的stdout重定向必须在stderr "2>&1 "的左边，因为你必须在发送stderr（单元1）到 "与单元1相同的地方 "之前，设置stdout（单元1）的去向。不要颠倒这些!

你必须使用特殊的语法">both 2>&1 "来把stdout和stderr都放到同一个文件里。不要使用下面的方法，这是不一样的。

```shell
ls /etc/passwd nosuchfile >wrong 2>wrong  # WRONG! DO NOT DO THIS!

cat wrong
/etc/passwd
ccess nosuchfile: No such file or directory
```

上面这个错误的例子会导致stderr和stdout相互覆盖，结果是输出文件被篡改。
互相覆盖，其结果是输出文件被篡改；不要这样做。

现代的Bourne shells现在有一个特殊的简短语法，可以将stdout和stderr重定向到同一个输出文件中。
重定向到同一个输出文件。

```shell
ls /etc/passwd nosuchfile &>both          # redirect both into same file
                                          # nothing appears on screen

cat both
ls: nosuchfile: No such file or directory
/etc/passwd
```

现在你可以使用"&>both "或">both 2>&1"，但只有后者在每个版本的Bourne shell中都有效（可以追溯到1960年代！）。当编写shell脚本时，使用">both 2>&1 "版本以获得最大的可移植性。

输出重定向总结。

重定向是由shell完成的。事情按照这个顺序发生:

- 首先：所有的重定向（和文件截断）都由shell完成。
  shell会从命令行中删除所有的重定向语法。
  这种重定向和截断即使在没有命令执行的情况下也会发生。
  命令将不知道它的输出被重定向了。

- 第二：命令（如果有的话）被执行并可能产生输出。
  shell在做完所有的重定向后，会执行命令。
  (如果重定向失败，shell不会运行任何命令）。

- 第三：命令的输出（如果有的话）会发生，并且会进入指定的重定向输出文件。
  指定的重定向输出文件。这是最后发生的。
  如果该命令没有产生输出，输出文件将是空的。
  添加重定向永远不会产生输出。

---

### 1.2. 使用/dev/null丢掉输出

在每个Unix系统中都有一个特殊的文件，你可以将你不想保留或看到的输出重定向到该文件中：/dev/null

下面的命令产生了一些我们不愿意看到的错误输出。

```shell
cat * >/tmp/out
 cat: course_outlines: Is a directory # errors print on STDERR
 cat: jclnotes: Is a directory # errors print on STDERR
 cat: labs: Is a directory # errors print on STDERR
 cat: notes: Is a directory # errors print on STDERR
```

我们可以把错误（stderr, unit 2）扔到/dev/null中。

```shell
cat * >/tmp/out 2>/dev/null
```

文件/dev/null永远不会被填满，它只是吃掉了输出。当作为一个输入路径名时，它看起来总是空的。

```shell
wc /dev/null
 0 0 0 /dev/null
```

系统管理员：不要养成扔掉所有命令的错误输出的习惯! 你也会丢掉合法的错误信息，没有人会知道这些命令是失败的。

---

### 1.3 要避免的输出重定向错误

首先，这里总结一下正确使用重定向的方法。

`date >out`

- shell首先截断文件 "out" - 文件现在是空的
- shell将命令 "date "的标准输出重定向到文件 "out"。
- shell从命令行中删除了">输出 "的语法。
- shell找到并运行 "date "命令
- date命令的标准输出到标准输出（1行）。
  标准输出已被shell重定向到文件 "out "中。

结果：文件 "out "包含 "date "的一行输出。

#### Unix重定向错误 #1

不要将一个重定向文件同时作为程序或管道的输出和输入! 下面使用 sort 命令演示--任何读取文件和产生输出的程序都有风险。

`sort a >a` #错误! 重定向输出文件被用作排序输入文件!

- shell首先截断了 "a "文件 - 文件现在是空的
- "a"的原始内容丢失了 - 被截断了 - 消失了! - 在shell寻找 "sort "命令运行之前
- shell将sort的标准输出重定向到空文件 "a "中。
- shell找到并运行带有一个文件名参数 "a "的 "sort "命令 ==> 即 sort(a)
- sort命令打开空的参数文件 "a "以供阅读
- 标准输出已被shell重定向到文件 "a "中。
  对一个空文件进行排序不会产生任何输出；文件 "a "仍然为空
  

结果:文件 "a "始终是空的，不管之前有什么。
正确的方法（使用两个命令）：sort a >tmp && mv tmp a
正确的方法（使用特殊的排序输出选项）：sort -o a a

下面是另一个不正确的例子，使用相同的输出文件作为输入。

date >out wc out >out # 错! 重定向输出文件被用作排序输入文件!

- shell首先截断了文件 "out" - 文件现在是空的
  "out "的原始内容丢失了--被截断了--消失了! - 在shell甚至还没有去寻找要运行的 "wc "命令!
- shell将wc的标准输出重定向到空文件 "a "中。
- shell找到并运行带有一个文件名参数 "out "的 "wc "命令==> 即wc(out)
- wc命令打开空的参数文件 "out "进行读取
- 标准输出被shell重定向到文件 "out "中。

计算一个空文件在标准输出上产生1行 "0 0 0 0 out"。
结果:一行wc输出 "0 0 0 out "被放入文件 "out"。
文件 "out "现在只有一行，是一个空文件的字数。out "的原始内容在步骤1中被shell截断了，而且从未使用过。

正确的方法（使用两个命令）：wc out >tmp && mv tmp out

其他不正确的重定向例子，不工作。

```shell
head file >file           # ALWAYS creates an EMPTY FILE
tail file >file           # ALWAYS creates an EMPTY FILE
uniq file >file           # ALWAYS creates an EMPTY FILE
cat  file >file           # ALWAYS creates an EMPTY FILE
grep 'foo' file >file     # ALWAYS creates an EMPTY FILE
sum  file >file           # ALWAYS checksums an EMPTY FILE
......
```

千万不要在输入和输出中使用相同的文件名--shell会在命令读取文件之前将其截断。

#### Unix重定向的错误 #2

不要使用[通配符/GLOB文件模式](https://www.cnblogs.com/savorboard/p/glob.html)，因为这种模式会拾取输出重定向文件的名称，并导致它成为一个非预期的输入文件。

Bourne shells（例如BASH）会在创建重定向文件之前进行通配符扩展。 C Shell 先做重定向文件的创建，这可能引发更多的意外问题。

nl（数字行）程序被用作这里的例子程序，任何读取文件和产生输出的程序都有风险。

```shell
cp /etc/passwd bar   # 创建一个大于磁盘块的文件
touch foo
nl * >foo  # 错了! GLOB * 输入文件与重定向输出文件相匹配!
^C           # 在磁盘被填满之前立即中断这个命令 ！
ls -l
-rw-rw-r--  1 idallen idallen    194172 Feb 15 05:19 bar
-rw-r--r--  1 idallen idallen 289808384 Feb 16 05:20 foo
```

下面是使输出文件 "foo "永远增长的情况。

- Shell扩展 "*"以匹配所有的路径名，即 "bar "和 "foo"。
- Shell截断了>foo并使其准备好接收命令的stdout。
- nl打开第一个文件 "bar "并将输出发送到stdout（进入foo）。
- nl打开下一个文件 "foo"，并开始从文件的顶部读取，将输出写入文件的底部。 这个过程永远不会结束，文件 "foo "不断增加，直到所有的磁盘空间被用完。

结果：一个无限循环，随着 "foo "越来越大，磁盘驱动器也被填满了。
变得越来越大。

修复#1：使用通配符，但不匹配的隐藏文件名。

```shell
nl * >.z
- 使用了一个没有被shell "*"通配符匹配的隐藏文件名。
- nl命令没有给出".z "作为参数，所以没有发生循环。
```

修复#2（两种方法）。使用其他目录中的文件。

```shell
nl * >../z
nl * >/tmp/z
- 将输出重定向到一个不在当前目录下的文件中。
  这样就不会被nl命令读取，也不会发生循环。
```

## 2. 输入重定向 - 标准输入

如果命令行中给出了文件路径名，许多Unix命令都会从文件中读取输入。如果没有给出文件名，这些命令通常从标准输入（"stdin"）读取，它通常与你的键盘相连。(你可以发送EOF来让命令停止读取）。

cat 命令从文件中读取，然后在没有提供文件时读取stdin的例子。

```shell
cat /etc/passwd      # cat从文件/etc/passwd中读取内容。
[...many lines print here...]

cat                  # 没有文件；cat读取标准输入（你的键盘）。
you type lines here
^D                     # 你可以通过输入^D(CTRL-D)发出键盘EOF信号。
you type lines here    # 这是cat的输出结果
```

其他可能从路径名或标准输入中读取的命令的例子。

```shell
less, more, cat, head, tail, sort, wc, grep, nl, uniq
```

诸如上述的命令可以读取标准输入。只有当命令行上没有要读取的路径名时，它们才会读取你的键盘，并且不涉及输入重定向。

```shell
wc foo # wc打开并读取文件 "foo"；wc完全忽略了stdin wc # wc打开并读取标准输入=你的键盘

cat foo # cat打开并读取文件 "foo"；cat完全无视stdin cat # cat打开并读取标准输入=你的键盘

tail foo # tail打开并读取 "foo"；tail完全无视stdin tail # tail打开并读取标准输入=你的键盘

[...etc. for all commands that can read from stdin...]
```

要告诉命令停止读取你的键盘，向它发送一个EOF（End-Of-File）指示，通常是通过输入^D（Control-D）。如果你中断命令（例如通过键入^C），你可能会杀死该命令，该命令可能根本不产生任何输出。

### 2.1 不是所有的命令都能读取标准输入

不是所有的命令都从标准输入中读取数据，因为不是所有的命令都从命令行上提供的文件中读取数据。常见的Unix命令的例子是不从文件或标准输入读取任何数据。

```shell
 ls, date, who, pwd, echo, cd, hostname, ps # 从不读取stdin
```

上述所有的命令都有一个共同的事实，即它们从不在命令行上打开任何文件进行读取。如果一个命令从不从任何文件中读取任何数据，它也不会从你的键盘上读取数据，也不会从标准输入中读取任何数据。

Unix的复制命令 "cp "显然是从文件中读取内容，但它从不从标准输入中读取文件数据，因为按照规定，它总是必须有一个源路径名和目标路径名的参数。它从不读取stdin。

### 2.2 shell重定向的标准输入

shell的元字符'<'表示命令行上的下一个字是一个输入文件（不是一个程序），应该提供给标准输入的命令。

使用shell元字符'<'，你可以告诉shell使用输入重定向来改变标准输入的来源，这样它就不是来自你的键盘，而是来自一个输入文件。

你只能在一个本来会读取你的键盘的命令上使用标准输入重定向。如果该命令在没有重定向的情况下不读取你的键盘（标准输入），添加重定向就没有任何作用，会被忽略。只有在没有重定向的情况下，该命令才会读取你的键盘，重定向才会起作用。

如果（也只有当！）一个命令从标准输入读取，重定向的标准输入将导致程序从shell附加到标准输入的任何内容中读取。下面是使用shell将文件附加到都在读取标准输入的命令的例子:

```shell
cat food # reads from file "food" cat # reads from stdin (from your keyboard)
cat <food # reads from stdin (now from the file "food")

head food # reads from file "food" head # reads from stdin (from your keyboard)
head <food # reads from stdin (now from the file "food")

sort food # reads from file "food" sort # reads from stdin (from your keyboard)
sort <food # reads from stdin (now from the file "food")

[ 用于所有可以从stdin读取的命令...]
```

shell不知道哪些命令会真正从标准输入中读取输入；你可以将标准输入中的文件附加到任何命令中。一个忽略标准输入的命令将忽略附加的文件。

如果一个命令没有从标准输入读取，重定向输入到该命令将被忽略，并且什么也做不了。shell不能强迫一个命令从标准输入中读取。

例如，date命令和sleep命令从不从标准输入读取，你也不能通过添加重定向来强迫它们。

```shell
date # date never reads stdin
 Thu Feb 16 05:48:13 EST 2012 date <file # date never reads stdin and ignores <file
 Thu Feb 16 05:48:15 EST 2012

echo 30 >file # first, put the number 30 into a file sleep 10 # sleep never reads stdin
 sleep 10
```

许多其他常见的命令从不读取标准输入，因此在这些命令中加入输入重定向并没有什么作用。

```shell
ls -l /bin # show pathnames under /bin ls -l /bin <input # no difference; ls never reads stdin

cd /bin # change to the /bin directory cd /bin <input # no difference; cd never reads stdin

cp foo bar # cp reads data from foo and writes to bar; ignores stdin cp foo bar <file # no change; cp never reads file data from stdin
```

如果命令行上有任何路径名，可能接受路径名参数的命令也会忽略标准输入。如果提供了路径名参数，这些命令总是读取路径名而忽略标准输入。

下面是更多的例子，这些例子并不像输入重定向那样有效，因为在添加重定向时，命令并没有从标准输入中读取。以下命令行都忽略了标准输入，因为所有的命令都被赋予了文件名参数来代替读取。c

```shell
cat food

[...etc. for all commands that take pathnames or read from stdin...]
```

如果命令行上有路径名，stdin就不会被使用。在上述所有不正确的例子中，shell将打开文件 "file "并附加它，使其在stdin上为命令做好准备；命令本身将忽略stdin并从命令行上提供的 "food "路径名读取。在标准输入上附加"<file" 会被忽略。

命令从来不会同时读取路径名和标准输入；两者缺一不可，而且命令参数的路径名总是用来代替stdin。

那么，如果一个文件可以作为命令行路径名提供，或者通过标准输入附加到命令中，有什么区别呢？注意 "cat food "和 "cat <food "之间的区别。

```shell
cat food

- the cat command has a pathname argument, which means it ignores stdin

- the "cat" command is opening the file argument "food", not the shell
  
  - any errors will come from the "cat" command and will mention the file
    name "food", e.g. cat: food: Permission denied

- the cat command reads data from the file it opened itself
  
  cat <food

- the cat command has no arguments, which means it will read standard input

- the shell is performing standard input redirection from file "food",
  which means standard input for "cat" will come from file "food"

- the shell itself is opening the file "food", not the "cat" command
  
  - any errors will come from the shell, not from the "cat" command,
    e.g. bash: food: Permission denied

- the cat command reads data from standard input, opened by the shell
```shell
对于在输出中显示其输入路径名的命令来说，上述区别更为显著。如果在命令行中没有提供路径名，而且所有的数据都来自标准输入，那么命令就没有文件名可以在输出中显示:

```shell
wc -l /etc/passwd
 44 /etc/passwd
```

- wc将文件名"/etc/passwd "作为一个命令行路径名参数，所以wc必须自己打开该文件。
  
- wc知道文件名，所以它在输出中打印了这个文件名。

```shell  
  wc -l </etc/passwd
  44
```

- shell打开标准输入重定向文件"/etc/passwd"，并将其附加到标准输入的命令中，该命令是 "wc"

- wc没有得到任何文件参数，所以从标准输入中读取数据；wc不知道文件名；只有shell知道文件名，所以wc没有打印任何文件名。

上述输入重定向技巧对于只获得文件中的行数，而不同时获得文件名是很有用的。

```shell
echo "The number of lines is:" ; wc -l /etc/passwd
 The number of lines is:
 44 /etc/passwd # wrong - "44 /etc/passwd" is not a number

echo "The number of lines is:" ; wc -l </etc/passwd
 The number of lines is:
 44 # correct - just the number, no name
```

警告：下面的命令行如何处理--当命令完成后，"myfile "中的内容是什么？

```shell
cat  <myfile >myfile                            # WRONG!
sort <myfile >myfile                            # WRONG!
head <myfile >myfile                            # WRONG!
```

鉴于上述情况，为什么在以下情况下 "myfile "不留空？

```shell
wc <myfile >myfile                              # WRONG!
```

## 3. 重定向到程序（管道）中

由于shell可以重定向程序的输出和输入，它可以将一个程序的输出连接（重定向）到另一个程序的输入。这被称为 "管道"，使用 "管道 "元字符'|'（shift-\\），例如：date | wc

### 3.1. 管道的规则

1. 管道重定向由shell首先完成，然后才是文件重定向。
2. 管道左边的命令必须产生一些标准输出。
3. 管道右边的命令必须能读取标准输入。

shell元字符"|"（"管道"）预示着命令行上另一个命令的开始。在"|"左边的命令的标准输出（只有stdout，没有stderr）被连接到右边的命令的标准输入（"管道"）。

```shell
date
 Mon Feb 27 06:37:52 EST 2012 date | wc
 1 6 29
```

(注意，行末的换行符是由wc计算的）。

在使用第二条命令之前，你可以使用一个临时文件进行中间存储，来近似于管道的一些行为。

```shell
date >out ; wc <out # output of date is saved and given to input of wc
 1 6 29
```

如果你使用了一个临时文件，在shell运行右边的命令读取文件之前，左边的命令必须完成并将*所有*的输出放到临时文件中。如果左边的命令没有完成，右边的命令就不会运行。管子就没有这个问题。输出立即开始流经管道，因为*两个*命令实际上都是*同时运行的：

```shell
find / >out ; less out  # huge output of find has to finish first (slow)
find / | less           # huge output of find goes directly into "less"
```

管道不需要临时文件，因此一旦管道左边的命令开始产生标准输出，它就会直接进入右边的命令的标准输入。如果左边的命令没有完成，右边的命令将继续等待更多的输入，并在出现时对其进行处理。如果左边的命令完成了，右边的命令会在管道（其标准输入）上看到一个EOF（文件结束）。与文件的EOF一样，EOF通常意味着右边的命令将完成处理，产生其最后的输出，并退出。

在进行文件重定向之前，首先识别管道并将命令行分割成管道命令。文件重定向发生在第二步（在管道拆分之后），如果存在，则优先于管道重定向。(文件重定向是在管道拆分之后进行的，所以它总是胜出，没有给管道留下任何东西）。

```shell
ls -l | wc # correct - output of ls goes into the pipe
 2 11 57

ls -l >out | wc # WRONG! - output of ls goes into the file
 0 0 0 # wc reads an empty pipe and outputs zeroes
```

shell首先分割管道上的行，将左边的命令的输出重定向到右边的命令的输入：

- shell处理左边 "ls "的标准输出文件重定向，并将 "ls "的标准输出改为 "out "文件。
- 最后，shell发现并同时运行两个命令

所有来自 "ls "的标准输出都进入 "out "文件；没有任何东西可以进入管道。

wc从管道中计算出一个空的输入并输出。0 0 0

与文件输出重定向一样，你只能将你能看到的标准输出重定向到管道中；重定向绝不会产生输出。

```shell
cp /etc/passwd x        # no output visible on standard output
cp /etc/passwd x | cat  # no output is passed to "cat"

cd /tmp                 # no output visible on standard output
cd /tmp | head          # no output is passed to "head"

touch x ; rm x          # no output from rm on standard output
touch x ; rm x | wc     # no output is passed to "wc"
  0 0 0                   # wc counts an empty input from the pipe
```

与文件重定向一样，你需要使用特殊的语法 "2>&1 "来将stdout和stderr同时重定向到管道中。记得 "2>&1 "的意思是 "将标准错误重定向到与标准输出相同的地方"，所以如果标准输出已经进入管道，"2>&1 "也将把标准错误送到那里。

```shell
ls /etc/passwd nosuchfile            # no redirection used
    ls: cannot access nosuchfile: No such file or directory   # STDERR unit 2
    /etc/passwd                                               # STDOUT unit 1

ls /etc/passwd nosuchfile | wc       # only stdin is redirected to "wc"
    ls: cannot access nosuchfile: No such file or directory   # STDERR unit 2
    1 1 12                               # stdout went into the pipe to "wc"

ls /etc/passwd nosuchfile 2>&1 | wc  # both stdin and stderr redirected
    2 10 68                              # wc counts both lines from pipe
```

请记住。重定向只能去*一个地方，而文件重定向总是胜过管道，因为它是在管道分割之后进行的。

```shell
ls /bin >out # all output from ls goes into file "out" 
ls /bin >out | wc # WRONG! output goes into "out", not into pipe
 0 0 0 # wc counts an empty input from the pipe
```

### 3.2. Using commands as Filters

Note that many Unix commands can be made to act as "filters" - reading from stdin and writing to stdout, all supplied by the shell, without opening any pathnames themselves. With no file names on the command line,the commands read from standard input and write to standard output. The shell provides redirection for both standard input and standard
output:

```shell
grep "/bin/sh" /etc/passwd | sort | head -5
```

The "grep" command above is reading from the filename argument /etc/passwd given on the command line. (When reading from files, commands do not read from standard input. File names take priority over standard input.)

The "sort" and "head" commands have no file names to read; this means they read from standard input, which is set up to be pipes by the shell.Both "sort" and "head" are acting as filters; they are reading from stdin and writing to stdout. (The "grep" command is technically not a filter - it is reading from the supplied argument pathname, not from stdin.)

Remember: if file names are given on the command line, the commands ignore standard input and only operate on the file names. Look at this small change to the above pipeline:

```shell
grep "/bin/sh" /etc/passwd | sort | head -5 /etc/passwd    # WRONG!
```

Above is the same command line as the previous example, except the "head" command is now ignoring standard input and is reading directly from its /etc/passwd filename argument. The "grep" and "sort" commands are doing a lot of work for nothing, since "head" is not reading the utput of sort coming down the pipe. The head command is reading from the supplied file name argument /etc/passwd instead. File names take precedence over standard input.

如果命令给定了要读取的文件名，就会忽略标准输入。

If a command does read from file names supplied on the command line,it is more efficient to let it open its own file name than to use "cat" to open the file and feed the data to the command on standard input.(There is less data copying done!)

Do this:

```shell
head /etc/passwd
sort /etc/passwd
```

Do not do this (wasteful of processes and I/O):

```shell
cat /etc/passwd | head     # DO NOT DO THIS - INEFFICIENT
cat /etc/passwd | sort     # DO NOT DO THIS - INEFFICIENT
```

Advice: Let commands open their own files; don't feed them with "cat".

### 3.3 Examples of pipes

Problem: Display only lines 6-10 of the password file:

```shell
head /etc/passwd | tail -n 5 # last five lines of first ten: lines 6-10
```

Problem: Display only the second-last line of the password file:

```shell
tail -n 2 /etc/passwd | head -n 1 # first line of last two lines
```

Problem: Which five files in current directory are largest:

```shell
ls -s | sort -nr | head -n 5
ls -la | sort -k 5,5nr | head -n 5
```

- the sort command is sorting by the fifth field, numerically, in reverse

- Problem: "Count the number of each kind of shell in /etc/passwd."

```shell
cut -d : -f 7 /etc/passwd | sort | uniq -c
```

- the cut command picks out colon-delimited field 7 in the password file
- the sort command puts all the shell names in order
- the uniq command counts the adjacent names

Problem: "Count the number of each kind of shell in /etc/passwd and display the results sorted in descending numeric order."

```shell
cut -d : -f 7 /etc/passwd | sort | uniq -c | sort -nr
```

- use the previous pipeline and add to it:
- sort the above output numerically and in reverse

Problem: "Count the number of each kind of shell in /etc/passwd and display the top two results sorted in descending numeric order."

```shell
cut -d : -f 7 /etc/passwd | sort | uniq -c | sort -nr | head -n 2
```

- use the previous pipeline and add to it:
- pick off only the top two lines of the above output
Problem: Which ten IP addresses are trying most often to break into my machine:

```shell
 grep 'refused connect' /var/log/auth.log \
  | awk '{print $NF}' \
  | sort | uniq -c | sort -nr | head
```

- the grep command picks off the sshd lines containing the IP address
- the awk command is displaying just the last field on each input line
- the first (leftmost) sort command puts all the IP addresses in order
- the uniq command is counting how many adjacent addresses are the same
- the second sort command is sorting the count numerically, in reverse
- the head picks off only the top ten addresses

问题：显示课程说明中的练习测试和每周文件日期。

```shell
alias ee='elinks -dump -no-numbering -no-references'
ee 'http://teaching.idallen.com/cst8207/12w/notes/' | grep 'practice'
ee 'http://teaching.idallen.com/cst8207/12w/notes/' | grep 'week'
```

Problem: Display the dates of the Midterm tests from the Home Page:

```shell
ee 'http://teaching.idallen.com/cst8207/12w/' | grep 'Midterm'
```

问题：显示当前渥太华的天气温度和预报。

```shell
ee 'http://text.www.weatheroffice.gc.ca/forecast/city_e.html?on-118' \
| grep -A1 'Temp' ee 'http://text.www.weatheroffice.gc.ca/forecast/city_e.html?on-118' \
| grep -A2 'Today:'

ee 'http://text.www.weatheroffice.gc.ca/forecast/city_e.html?on-118' \
| grep -A2 'Tonight:'
```

Problem: Display Ottawa tomorrow weather forecast:

```shell
ee 'http://text.www.weatheroffice.gc.ca/forecast/city_e.html?on-118' \
| grep -A8 'Tonight:' | tail -n 5
```

Problem: Display the first top story from the BBC:

```shell
ee 'http://www.bbc.co.uk/' | grep -A9 'Top stories'
```

Problem: Display first top story in each subject from the BBC:

```shell
ee 'http://www.bbc.co.uk/' | grep -A3 'Top Story'
```

Problem: Display the current BBC weather for Vancouver:

```shell
ee 'http://www.bbc.co.uk/weather/6173331' | grep -A19 'Observations' | tail -n 20
```

Problem: Display the current Space Weather forecast for Canada:

```shell
ee 'http://www.spaceweather.gc.ca/index-eng.php' | grep -A10 'ISES Regional Warning Centre'
```

Problem: Display the current phase of the Moon:

```shell
ee 'http://www.die.net/moon/' | grep -A2 'Moon Phase' | head -n 3 | tail -n 1
```

### 3.4 误用重定向到程序中

人们经常被误导，以为在一个命令中添加重定向会产生在添加重定向之前不存在的输出。事实并非如此。

管子的规则。

1. 管道重定向首先由shell完成，然后才是文件重定向。
2. 管道左边的命令必须产生一些标准输出。
3. 管道右侧的命令必须想要读取标准输入。

如果一个能够打开并读取路径名内容的Unix命令没有得到任何路径名来打开，它通常会从标准输入（stdin）中读取输入行。

```shell
wc /etc/passwd  # wc reads /etc/passwd, ignores stdin and your keyboard
wc              # without a file name, wc reads stdin (your keyboard)
```

如果命令被赋予了一个路径名，它就会从路径名中读取，并且*总是*忽略标准输入，即使你试图向它发送什么。
```shell
wc               # without a file name, wc reads standard input (keyboard)
date | wc        # wc opens and reads standard input, counts date output

wc foo           # wc reads foo; wc does not read stdin
date | wc foo    # WRONG! wc opens and reads foo; wc ignores stdin
```
上述规定适用于每一个读取文件内容的命令，例如。
```shell
date | head foo  # WRONG! head opens and reads foo; head ignores stdin
date | less foo  # WRONG! less opens and reads foo; less ignores stdin
```
如果你想让一个命令读取 stdin，你不能给它任何文件名参数。有文件名参数的命令会忽略标准输入。
带有文件名参数的命令会忽略标准输入；它们不应该被用在管道的右边。

忽略标准输入的命令（因为它们是在命令行上打开和读取路径名）。
忽略标准输入的命令（因为它们在命令行上打开并读取路径名）将始终忽略标准
输入，无论你试图在标准输入上向它们发送什么东西。

```shell
echo hi | head /etc/passwd   # WRONG: head has a pathname and ignores stdin
echo hi | tail /etc/group    # WRONG: tail has a pathname and ignores stdin
echo hi | wc .vimrc          # WRONG:   wc has a pathname and ignores stdin
sort a | cat b               # WRONG:  cat has a pathname and ignores stdin
cat a | sort b               # WRONG: sort has a pathname and ignores stdin
```
Standard input is thrown away if it is sent to a command that ignores it.
The shell *cannot* make a command read stdin; it's up to the command.
The command must *want* to read standard input, and it will *only* want to read standard input if you *leave off all the file names*.

Commands that do not open and process the *contents* of files usually ignore standard input, no matter what silly things you try to send them on standard input. All these commands will never read standard input:
```
echo hi | ls          # NO: ls doesn't open files - always ignores stdin
echo hi | pwd         # NO: pwd doesn't open files - always ignores stdin
echo hi | cd          # NO: cd doesn't open files - always ignores stdin
echo hi | date        # NO: date doesn't open files - always ignores stdin
echo hi | chmod +x .  # NO: chmod doesn't open files - always ignores stdin
echo hi | rm foo      # NO: rm doesn't open files - always ignores stdin
echo hi | rmdir dir   # NO: rmdir doesn't open files - always ignores stdin
echo hi | echo me     # NO: echo doesn't open files - always ignores stdin
echo hi | mv a b      # NO: mv doesn't open files - always ignores stdin
echo hi | ln a b      # NO: ln doesn't open files - always ignores stdin
```
Some commands *only* operate on file name arguments and never read stdin:
```
echo hi | cp a b # NO: cp opens arguments - always ignores stdin
```
Standard input is thrown away if it is sent to a command that ignores it.The shell *cannot* make a command read stdin; it's up to the command.

Commands that might read standard input will do so only if *no* file name arguments are given on the command line. The presence of any file arguments will cause the command to ignore standard input and process
the file(s) instead, and that means they cannot be used on the right side of a pipe to read standard input. File name arguments always win over standard input.

Example of mis-used redirection:

The very long sequence of pipes below is pointless - the last (rightmost) command ("head") has a pathname and will open and read it, ignoring all the standard input coming from all the pipes on the left:
```shell
head /etc/passwd | sort | tail | sort -r | head /etc/passwd
```
The above mal-formed pipeline is equivalent to this (same output):
```shell
head /etc/passwd
```
If you give a command a file to process, it will ignore standard input,and so a command with a file name must not be used on the right side of any pipe.


## 4.独特的STDIN和STDOUT


There is only one standard input and one standard output for each command.Each can only be redirected to *one* other place. You cannot redirect standard input from two different places, nor can you redirect standard output into two different places.

The Bourne shells (including bash) do not warn you that you are trying to redirect the input of a command from two or more different places (and that only one of the redirections will work - the others will be ignored):

```
wc <a <b <c <d <e
```

- the "wc" stdin comes from file "e" *only*
- the other file names must exist (the shell will open every one of them), but they will be ignored because only the final redirection wins

```shell
bashdate | cat <food
```

- the "date" output vanishes; cat reads stdin from file "food"(file redirection is done second and always wins over pipe redirection)

The Bourne shells (including bash) do not warn you that you are trying to redirect the output of a command to two or more different places and that only one of the redirections will work - the others will be ignored:

```
bashdate >a >b >c >d >e
```

- the "date" output goes into file "e"; the other four output files are each created and truncated by the shell but they are all left empty because only the final redirection into "e" wins

```shell
date >out | wc
      0 0 0
```
- the "date" output goes into file "out"; nothing goes into the pipe  (file redirection is done second and always wins over pipe redirection) Some shells (including the "C" shells, but not the Bourne shells) will try to warn you about silly shell redirection mistakes:

```
csh% date <a <b <c <d
Ambiguous input redirect.

csh% date | cat <food
Ambiguous input redirect.

csh% date >a >b >c
Ambiguous output redirect.

csh% date >a | wc
Ambiguous output redirect.
```

The C shells tell you that you can't redirect stdin or stdout to/from more than one place at the same time. Bourne shells do not tell you - they simply ignore the "extra" redirections and do only the last one of each.

## 5.tr 不接受路径名的命令

Unix的 "tr"命令是少数（或许是唯一的）不接受路径的命令，它读取标准输入，但*不*允许在命令行上有任何路径名。

- 你必须始终*在标准输入上向 "tr "提供输入。

```shell
tr 'a-z' 'A-Z' file1 file2 >out         # *** WRONG - ERROR ***
  tr: too many arguments
cat file1 file2 | tr 'a-z' 'A-Z' >out   # correct for multiple files
tr 'a-z' 'A-Z' <tmp >out                # correct for a single file
```

Note: System V versions of "tr" demand that character ranges appear inside square brackets, e.g.: tr '[a-z]' '[A-Z]' Berkeley Unix and Linux do not use the brackets.No version of "tr" accepts pathnames on the command line.All versions of "tr" *only* read standard input.

Here is an incorrect example using the same output file as input:

```shell
$ date >out
$ tr ' ' '_' <out >out # WRONG! Redirection output file is used as input file!
```

1. shell opens the file "out" as standard input to "tr" (due to "<out")
2. shell next truncates the file "out" - file is now empty (due to ">out")
   - original contents of "out" are lost - truncated - GONE! - before the shell even goes looking for the "tr" command to run!
3. shell finds and runs command "tr" with two string arguments ==> i.e. tr(' ','_')
4. command "tr" reads from standard input (tr always reads stdin)
   - standard input is attached to "out", now an empty file
5. standard output has been redirected by the shell to appear in file "out"
   - translating an empty input file produces no output in "out" Result: The file "out" is always empty. File "out" gets a translated copy of an empty input file; the file "out" is always left empty. RIGHT WAY (use two commands): tr ' ' '_' tmp && mv tmp out

Problem: convert lower-case to upper-case from the "who" command:

```shell
who | tr 'a-z' 'A-Z'
```

Shell question: Are the single quotes required around the two arguments? (Are there any special characters in the arguments that need protection?)

Using redirection, you can use a similar command to convert a lower-case file of text into upper-case.

EXPERIMENT: Why doesn't this convert the file "myfile" to upper-case?

```shell
$ date >myfile
$ tr 'a-z' 'A-Z' <myfile >myfile                  # WRONG!
$ wc myfile
  0 0 0 myfile                 # what happened?
```

Why is the file "myfile" empty after this command is run?

The following command line doesn't work because the programmer doesn't understand the "tr" command syntax:

```shell
$ tr 'a-z' 'A-Z' myfile >new                      # WRONG!
```

Why does this generate an error message from "tr"? (The "tr" command is unusual in its handling of command line pathnames. RTFM) The following command line redirection is faulty (input file is also output file); however, it sometimes works for small files:

```shell
$ cat foo bar | tr 'a' 'b' | grep "lala" | sort | head >foo   # WRONG!
```

There is a critical race between the first "cat" command trying to read the data out of "foo" before the shell truncates it to zero when launching the "head" command at the end of the pipeline. Depending on the system load and the size of the file, "cat" may or may not get out all the data before the "foo" file is truncated or altered by the shell in the redirection at the end of the pipeline.

Don't depend on long pipelines saving you from bad redirection! Never redirect output into a file that is being used as input in the same command or anywhere in the command pipeline.

## 6. 不要重定向全屏程序，如VIM

全屏键盘交互式程序，如VIM文本编辑器，如果你重定向它们的输入或输出，它们就不会表现得很好--它们真的想与你的键盘和屏幕对话；不要重定向它们或
试图用"&"在后台运行它们。如果你试图这样做，你会让你的终端机挂掉。

## 7. 仅将stderr重定向到一个管道中（高级）

你如何只将stderr重定向到管道，而让stdout进入终端？这很棘手；在管道的左边，你必须将stdout（连接到管道）和stderr（连接到终端）交换。你需要一个临时输出单元（使用 "3"）来记录并记住终端的位置（将单元3重定向到与单元2相同的地方："3>&2"），然后将stderr重定向到管道中（将单元2重定向到与单元1相同的地方："2>&1"），然后将stdout重定向到终端（将单元1重定向到与单元3相同的地方："1>&3"）。

```shell
$ ls /etc/passwd nosuchfile 3>&2 2>&1 1>&3 | wc  # switch STDOUT and STDERR
    /etc/passwd                                    # STDOUT appears on terminal
    1 9 56                                         # STDERR goes into the pipe
```

你很少需要做这种高级技巧，即使是在脚本里面。

---
| Ian! D. Allen - [idallen@idallen.ca](mailto:idallen@idallen.ca) - Ottawa, Ontario, Canada
| Home Page: [http://idallen.com/](http://idallen.com/) Contact Improv: [http://contactimprov.ca/](http://contactimprov.ca/) | College professor (Free/Libre GNU+Linux) at: [http://teaching.idallen.com/](http://teaching.idallen.com/) | Defend digital freedom: [http://eff.org/](http://eff.org/) and have fun: [http://fools.ca/](http://fools.ca/)
