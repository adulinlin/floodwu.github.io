# 《鸟哥的Linux私房菜——基础学习篇》读书笔记

## 基础知识
除了`man`之外，Linux中还提供另外一种查询方式，即`info`，使用方法和man差不多：
[root@test root]# info command

### 正常情况下，关机时需要注意下面几件事：
	①观察系统的使用状态：如果要看目前有谁在线，可以输入who指令，而如果要看网络的联机状态，可以输入netstat -a指令，而要看背景执行的程序可以执行ps -aux指令。使用这些指令可以让您稍微了解主机当前的使用状态。当然，就可以让您判断是否可以关机了（这些指令在后面会提及）。
	②通知在线用户关机的时刻：要关机前总得给在线用户一些时间用于结束他们的工作，所以，这个时候您可以使用shutdown特别指令达到这一目的。
	③正确的关机指令：例如`shutdown`与`reboot`两个指令。
/sbin/shutdown [-t 秒 ] [-arkhncfF] 时间 ] [ 警告信息]
/sbin/shutdown -h 10 'This server will shutdown after 10 mins'
它的参数有如下几个。
-t sec： -t后面跟秒数，亦即“过几秒后关机”的意思
-k： 不要真的关机，只是发送警告信息
-r： 在将系统的服务停掉之后就重新开机
-h： 将系统的服务停掉后，立即关机
-n： 不经过init程序，直接以shutdown功能关机
-f： 关机并开机之后，强制略过fsck工具的磁盘检查
-F： 系统重新开机之后，强制执行fsck磁盘检查
-c： 取消已经在进行的shutdown指令内容
shutdown -h now 立刻关机，其中now相当于时间为0
shutdown -h 20:25 系统在今天的20:25分关机
shutdown -h +10 系统再过十分钟后自动关机
shutdown -r now 系统立刻重新开机
shutdown -r +30 'The system will reboot'
再过三十分钟系统会重新开机，并显示后面的信息。
shutdown -k 'This system will reboot'
仅发出引号内的警告信息，系统不会关机

## Linux文件权限与目录配置
### Linux文件权限
-rwxrwxrwx 	    1 		  root 		  root 		      293 		   Oct 19 21:24 		test
[文件属性]	 [连接数]   [文件拥有者] [文件所属群组]    [文件大小] 	   [最后修改时间]      [文件名]
①文件属性：包含十个属性。
第一个字符代表这个文件的类型：
若为[ d ]，则是目录，例如上面的tmp/行；
若为[ - ]，则是文件，例如上面的.bashrc行；
若是[ l ]，则表示为链接文件（link file）；
若是[ b ]，则表示为设备文件中可供储存的接口设备；
若是[ c ]，则表示为设备文件中的串行端口设备，例如键盘、鼠标。
接下来的9个属性3个为一组，且均为“rwx”的组合形式。其中：
 [ r ]代表可读
 [ w ]代表可写
 [ x ]代表可执行
如果不具备某个属性，则相应字母会被删掉。
第一组[ rwx ]为“拥有者的权限，owner”；第二组[ rwx ]为“群组的权限，group”；第三组[ --- ]为“其他非本群组的用户的权限，others”。
 注意：x与目录的关系相当重要，如果您在该目录下不能执行任何指令，那么自然也就无法执行ls，cd等指令，所以，也就无法进入。因此，请特别注意，如果您想开放某个目录，请记得将该目录的x属性也开放。在Linux下，文件是否能执行，则是藉由是否具有x这个属性来决定，跟后缀名没有绝对的关系
②连接数：表示链接占用的节点（i-node），若为目录，通常与该目录下有多少子目录有关；
③文件名：如果文件名之前多一个“.”，则表明这个文件为“隐藏文件”

### 改变文件权限
①改变群组：chgrp
chgrp 群组名称 文件或目录
[root@test root]# chgrp users tmp

②改变拥有者：chown
chown [ -R ] 账号名称文件或目录
chown [ -R ] 账号名称:群组名称文件或目录
[root@test root]# chown test tmp
[root@test root]# chown –R root:root tmp

③改变权限：chmod
数字类型改变文件权限
使用数字代表各个属性，如下：r: 4、w: 2、x: 1；
将同一组数字相加结果为：
owner = rwx = 4+2+1 = 7
group = rwx = 4+2+1 = 7
others = --- = 0+0+0 = 0
所以，三组属性生成的数值就是770
chmod [-R] xyz 文件或目录
其中，xyz为同三组rwx属性数值的相加
[root@test root]# chmod 777 .bashrc

符号类型改变文件形态
		u	+（加入）		r
chmod 	g	-（除去）		w	文件或目录
		o	=（设定）		x
		a
[root@test root]# chmod u=rwx,og=rx .bashrc
[root@test root]# chmod a+w .bashrc
=与–的状态下，只要是没有涉及到的项，则该属性不会被变动。

## Linux目录配置
`/`为根目录，每个目录都是依附在“/”根目录下。
`/bin` 这是存放诸如ls，mv，rm，mkdir，rmdir，gzip，tar，telnet和ftp等常用执行文件的地方（这些执行文件的执行方法会在后面提到）。有时这个目录的内容与/usr/bin一样（有时甚至会使用链接文件），是专门用于放置一般用户使用的执行程序
`/boot` 这里是放置您的Linux核心与启动相关文件的地方，目录下的vmlinuz-xxx就是Linux的内核。如果您的启动管理程序选择grub，那么这个目录内还有/boot/grub子目录
`/dev` 存放与设备有关的文件。基本上，Unix或Linux系统均把设备当成文件，例如/dev/fd0代表软驱，相当于Windows系统下的A区，而/dev/cdrom则代表光驱。这个目录下的文件通常分为两种，分别是管理硬盘I/O的块文件与外设的字符文件
`/etc` 系统在启动过程中需要读取的文件均在这个目录下，例如Lilo的参数、用户账号与密码、系统的主要设定、http架站参数、您要启动的服务项等，所以在这个目录下工作的时候一定要记得备份，否则文件被意外修改后会很麻烦
`/home` 基本上，这是系统默认的用户根目录（home directory），在您新增一个一般用户的账号时，默认的用户根目录已在这里设定好
`/lib` 在Linux执行或编译某些程序时要用到的函数库（library）就在这个目录下
`/lost+found` 系统产生异常错误时，会将一些遗失的片段放置在此目录下，通常这个目录会自动出现在设备目录下。例如您在/disk中加装一块硬盘， 这个目录下就会自动产生目录/disk/lost+found
`/mnt` 软驱与光驱接默认装载点的地方。通常，软驱挂在/mnt/floppy下，光驱挂在/mnt/cdrom下，不过也不一定，只要您高兴，随便找一个地方装载也可以
`/proc` 用于放置系统核心与执行程序所需的一些信息，例如您的网络状态等问题。这个目录将在启动Linux的时候自动被挂上，而且该目录不会占用硬盘空间，因为里面都是内存中的数据
`/root` 系统管理员的根目录
/sbin 放置系统管理常用的程序，例如fdisk，mke2fs，fsck，mkswap和mount等。与/bin不太一样，这个目录下的程序通常是root等系统管理员使用的程序
`/tmp` 这是让一般用户存放临时文件的地方，例如您在安装Linux下的软件时，可能软件的默认安装目录就是/tmp，所以您要定期清理，当然，重要数据最好不要放在这里
`/usr` 这是最重要的一个目录，里面含有很多系统信息，其下包含许多子目录，用来存放程序与指令。这个目录有点类似Windows下的Program Files目录
`/usr/include` 一些套件的头文件。基本上，当我们以Tarball方式（*.tar.gz方式）安装某些数据时会用到的函数库都在这个目录下
`/usr/lib` 内含许多程序与子程序所需的函数库
`/usr/src` 是放置核心源代码的默认目录，未来我们要编译核心的时候，就必须到这个目录下
`/var` 这个目录也非常重要，所有服务的登录文件或错误信息文件（log files）都在/var/log下，此外，一些数据库如MySQL则在/var/lib下，还有，用户未读邮件的默认存放地点为/var/spool/mail

## 文件与目录管理
### 目录与路径
绝对路径：路径的写法一定是从根目录“/”写起，例如：/usr/share/doc 目录。
相对路径：路径的写法不是由“/”写起，例如从/usr/share/doc转到/usr/share/man下时，可以写成cd ../man。这就是相对路径的写法。
常用的目录“符号”代表的意义：
`.` 代表当前层目录
`..` 代表上层目录
`~`代表自己的根目录
`~user` 代表到user这个人的根目录

### 目录与路径的几个常用指令
①`cd` 变换目录
一旦登入Linux系统，系统管理员的工作路径会自动切换到其根目录（即/root）下，而用户会自动转到/home/username下。
[root @test /root]# cd .. <==回到上一层目录
[root @test /root]# cd ../home <==相对路径的写法
[root @test /root]# cd /var/www/html <==绝对路径的写法
[root @test /etc]# cd <==回到用户的根目录
[root @test /etc]# cd ~ <==回到用户的根目录
[root @test /etc]# cd ~test <==回到test用户的根目录
②`pwd` 显示当前目录
[root @test root]# cd /home/test
[root @test test]# pwd
/home/test <==显示当前所在目录
③`mkdir` 建立一个新目录
[root @test /root]# mkdir test <==建立名为test的目录
目录需要一层一层地建立，假如您要建立一个目录为/home/bird/testing/test1，那么首先必须要有/home然后/home/bird，再后来是/home/bird/testing，这些必须都存在才可以建立test1这个目录，假如没有/home/bird/testing，就没有办法建立test1。
④`rmdir` 删除一个内容为空的空目录
[root @test /root]# rmdir test <==删除名为test的目录
目录需要一层一层地删除，而且被删除的目录中不能有其他的目录或文件。如果要将所有目录下的东西都删除，这个时候就必须使用rm -rf test命令，但使用rmdir的安全性较高一些。

### 环境变量PATH
当我们执行一个指令时，系统会依照PATH的设定到PATH定义的每个路径下搜寻文件，先搜寻到的指令文件先被执行
[root@test root]# echo $PATH
/sbin:/usr/sbin:/bin:/usr/bin:/usr/X11R6/bin:/usr/local/bin:/usr/local/sbin
这也就解释了可以在/root下执行/bin/ls这个文件的原因。但是为了安全起见，不建议将“.”加入PATH中。

### 文件与目录管理
①`ls` 	显示文件名称、属性等 # ls [-ailS]
参数说明：
-a :列出所有文件（连同隐藏文档）
-i :打印inode的值
-l :详细列出，连同文件大小、属性数据等；ll等价于ls –l
-S :以文件大小排序
--color=never :不显示颜色
--color=always :均显示颜色
--color=auto :由系统自行判断
②`cp` 	复制文件或目录 # cp [-drsu] [源文件] [目标文件]
参数说明：
-d ：进行复制时，如果是复制到链接文件，若不加任何参数，则默认情况下会将链接到的源文件
复制到目的地，若加-d，则链接文件可原封不动地将链接这个快捷方式复制到目的地。
-r ：可以进行目录的复制。
-s ：做成链接文件，与ln指令功能相同。
-u, --update：如果源文件较新，或者没有目标文件，才会进行复制动作。可用于备份操作。
 [root @test /root]# cp .bashrc bashrc
将.bashrc复制成bashrc文件！
[root @test /root]# cp -r /bin /tmp/bin
这个功能用来复制整个目录的参数！
[root @test /root]# cp -s .bashrc bashrc.cp
建立一个链接文件，文件名为bashrc.cp
[root @test /root]# cp -u /root/.bashrc /home/test/.bashrc
先检查/home/.bashrc与.bashrc是否相同，如果不同就复制一份；如果相同则不做任何动作！
③`rm` 	删除文件或目录 # rm [-fir] [文件名]
-i :提供用户确认（这是默认值）。
-r :循环删除，直到没有东西为止。
-f :force，就是强制删除。
④`mv` 	移动文件或目录 # mv [-u] [源文件] [目标文件]
-u :同样，为update的简写，当源文件比目标文件还新时才会动作！

查看文件内容
①`cat` 由第一行开始显示文件内容 # cat [-n]
-n: 显示时，连行号一起输出到屏幕上。
cat较不常见，毕竟在文件内容行数超过40行以上时根本来不及看，所以，配合more或者less执行比较好。
②`tac` 从最后一行开始显示，可以看出，tac是cat的倒写；# tac [文件名]
③`more` 一页一页地显示文件内容； # more [文件名]
 [root @test /root]# more ~/.bashrc <==一页一页地显示文件内容
[root @test /]# ls -al | more <==一页一页地将ls的内容显示出来
④`less` 与more类似，但其优点是可以使用[pageup]、[pagedown]等按键的功能向前向后翻看文件； 
less [文件名]
⑤`head` 只看头几行；  # head [-n number] [文件名]
⑥`tail` 只看末尾几行； # tail [-n number] [文件名]
⑦`nl` 显示时同时输出行号
⑧`od` 以二进制方式读取文件内容

## 链接文件
`inode`：Block是记录文件内容数据的区域，inode则是记录该文件的属性及其放置在哪个Block之内的信息。所以，每个文件都会占用一个inode。当Linux系统要查找某个文件时，它会先搜寻inode table找到这个文件的属性及数据放置地点，然后再查找数据存放的Block进而将数据取出
`硬链接`：硬链接就是再建立一个inode链接到文件放置的Block块。也就是说，进行硬链接时，实际上您的文件内容不会改变，只是在查询时，利用原来的inode与后来添加的inode均可指定到该文件放置的地点，因此，读取两个inode的结果都是存取同一个文件的内容。不过，这样一来就有个问题，因为inode会链接到Block块，而“目录”本身仅消耗inode，这样，硬链接就不能链接目录。所以，`硬链接有两个最大的限制`：
1. 不能跨文件系统，因为不同的文件系统有不同的inode table；
2. 不能链接目录。
`符号链接`：再建立一个独立文件，而这个文件会让数据读取操作指向它链接的那个文件。由于只是利用文件作为指向的动作，所以，当源文件被删除，符号链接的文件就打不开了，屏幕会显示“无法开启某文件”。
[root @test /root ]# ln [-s] [源文件] [目标文件]
-s :提供符号链接
:如果直接使用ln而不加任何参数，就属于硬链接
Linux的链接与Windows的快捷方式是不一样的。举个例子，当您在Windows建立一个快捷方式时，可以在这个快捷方式内修改任何数据，而原始数据不会跟着变。而当您修改Linux下的链接文件时，则更改的其实是原始文档。所以，不论这个原始文档被链接到哪里，只要修改了链接文件，原始文档就会跟着变。

### 文件与目录权限
①`chown`	改变文件的拥有者
②`chgrp` 	改变文件的所属群组
③`chmod` 改变文件的可写、可读、可执行等属性
④`umask`	改变预设的建立文件或目录时的属性
umask指定的是“该默认值需要取消的权限”
[root @test root]# umask
0022
[root@vbird test]# umask 002
[root@vbird test]# umask
0002 
⑤`chattr`	改变文件的特殊属性
⑥`lsattr` 	显示文件的特殊属性

### 搜寻文件或目录
①`which` 查看可执行文件的位置
[root @test /root ]# which [文件名称]
[root @test /root]# which passwd
/usr/bin/passwd
which的基本功能是通过PATH环境变量到该路径内寻找可执行文件，所以基本的功能在于寻找可执行文件。
②`whereis` 查看文件的位置
[root @test /root ]# whereis [-bmsu] [目录名称]
-b :只找二进制文件
-m :只找在说明文件manual路径下的文件
-s :只找source源文件
-u :没有说明文档的文件！
③`locate` 配合数据库查看文件位置
[root @test /root ]# locate [目录名称]
 [root @test /root]# locate root
……一大堆带有root字符串的文件都会显示出来
[root @test /root]# updatedb
立刻更新数据库
Linux系统会将系统内的所有文件都记录在一个数据库文件中，当使用whereis或locate时，都会以此数据库文件的内容为准，使用locate查找数据特别快，这是因为locate是从已建立的数据库/var/lib/slocate中查找数据，不用直接在硬盘中存取数据，所以自然很快。正因为它是通过数据库来搜寻而数据库的更新默认是每周执行一次，所以，在数据库更新之前新建的文件就会找不到，必须要在更新数据库之后。
④`find` 实际搜寻硬盘查询文件名称
[root @test /root ]# find [路径] [参数]
参数说明：
1. 时间：
-atime n :将n*24小时内被存取过的文件列出来
-ctime n :将n*24小时内被改变、新增的文件或目录列出来
-mtime n :将n*24小时内被修改过的文件列出来
-newer file :把比file还要新的文件列出来
2. 使用名称：
-gid n :寻找群组ID为n的文件
-group name :寻找群组名称为name的文件
-uid n :寻找拥有者ID为n的文件
-user name :寻找用户名称为name的文件
-name file :寻找文件名为file的文件名称（可以使用通配符）
-type type :寻找文件属性为type的文件，type包含了b, c, d, p, l, s，
这些与前一章的属性相同。例如l为Link而d为目录
范例：
[root @test /root]# find / -name testing <==寻找文件名为testing
[root @test /root]# find / -name 'test*' <==寻找文件名包含test的
[root @test /root]# find . -ctime 1
寻找当前目录下一天内新增的目录或文件
[root @test /root]# find /home/test –newer .bashrc
寻找/home/test目录下比.bashrc还要新的文件

SUID与SGID
`UID`代表用户代号，`GID`则是群组代号。
[test@test test]$ ls -l /usr/bin/passwd
-r-s--x--x 1 root root 13476 Aug 7 2001 /usr/bin/passwd
在原来x的位置有一个s属性，这个就是所谓的`SUID`。如果是-r-xr-s--x，那么s就成为所谓的`SGID`。当一个文件具有SUID时，同时others群组具有可执行权限，那么当others群组执行该程序时，others将拥有该文件的owner权限。Set UID（SUID）的主要功能是在某个文件执行其间具有文件拥有者的权限，因此，s可以替代上面提到的x可执行属性的位置。

`Sticky bit`：具有sticky bit属性的目录，其下的文件或目录只有文件拥有者及root才有权删除。
[test@test test]$ ls -l /
drwxrwxrwt 2 root root 4096 Jul 18 13:08 tmp
最末位用t取代x

`file`命令
[root @test /root ]# file [文件名]
[root @test /root]# file ~/.bashrc
/root/.bashrc: ASCII text <==表示这个文件是ASCII纯文本文件
file指令可以用来查看这个文件的类型，例如ASCII文档或二进制文件等，还可以用来查看文件是否被加入SUID等信息。

## vi文本处理器
	Linux与Unix系统中的参数文件几乎都是ASCII码的纯文本文件，因此，利用简单的文本编辑软件可以立刻修改Linux的参数文档。vi是Unix默认的字处理软件，当然，也是Linux默认的字处理软件。vi分为3种模式，分别是“一般模式”、“编辑模式”与“命令行模式”：
	①一般模式：以vi处理文件时，一进入该文件就是一般模式了。在这个模式中，可以使用上下左右按键来移动光标，可以使用“删除字符”或“删除整行”来处理文件内容，也可以使用“复制”、“粘贴”来处理文件数据。在一般模式中按下:wq，保存后退出vi。如果文件权限不对，例如为-r--r--r--，那么可能无法写入，可以使用强制写入方式，即使用:wq!，多加一个惊叹号即可。不过，需要特别注意，这只有在您的权限可以改变的情况下才能成立。
	②编辑模式：在一般模式下可以处理删除、复制、粘贴等动作，但是却无法编辑。在您按下i，I，o，O，a，A，r，R等字母之后才会进入编辑模式。注意，通常在Linux中，按下上述字母后，在画面的左下方会出现INSERT或REPLACE字样，这才可以输入任何字符写入您的文件中。如果要回到一般模式，必须按下Esc键，才可退出编辑模式。
	③命令行模式：在一般模式中，输入“:”或“/”就可以将光标移动到最末一行。在这个模式中，您可以搜寻数据，读取、存盘、大量字符替换、退出vi、显示行号等动作也是在此模式中完成。

### 常用指令
①一般模式
	`Ctrl + f` 屏幕向前翻动一页
	`Ctrl + b` 屏幕向后翻动一页
	0 （这是数字0）移动到这一行的第一个字符处
	$ 移动到这一行的最后一个字符处
	G 移动到这个文件的最后一行
	n<Enter> 光标向下移动n行
	/word 在光标之后查找一个名为word的字符串
	:n1,n2s/word1/word2/g 在第n1与n2行之间查找word1这个字符串，并将该字符串替换为word2
	:1,$s/word1/word2/g 从第一行到最后一行查找word1字符串，并将该字符串替换为word2
	:1,$s/word1/word2/gc 从第一行到最后一行查找word1字符串，并将该字符串替换为word2，且在替换前显示提示符让用户确认（conform）
	x, X x为向后删除一个字符，X为向前删除一个字符
	dd 删除光标所在的那一整列
	ndd 删除光标所在列的向下n列，例如，20dd则是删除20列
	yy 复制光标所在行
	nyy 复制光标所在列的向下n列，例如，20yy则是复制20列
	p, P p为复制的数据粘贴在光标下一行，P则为粘贴在光标上一行
	u 恢复前一个动作
②编辑模式
	i, I 插入：在当前光标所在处插入输入的文字，已存在的字符会向后退
	a, A 添加：由当前光标所在处的下一个字符开始输入，已存在的字符会向后退
	o, O 插入新的一行：从光标所在处的下一行行首开始输入字符
	r, R 替换：r会替换光标所指的那一个字符；R会一直替换光标所指的文字，直到按下Esc为止
	Esc 退出编辑模式，回到一般模式
③命令行模式
	:w 将编辑的数据写入硬盘文件中
	:w! 若文件属性为只读，强制写入该文件
	:q 退出vi
	:q! 若曾修改过文件，又不想保存，使用!为强制退出不保存文件
	:wq 保存后退出，若为:wq!，则为强制保存后退出
	:w [filename] 将编辑数据保存为另一个文件（类似另存新文档）

## Bash
什么是shell:基本上，替我们工作的是硬件，而控制硬件的是核心，用户是利用Shell控制kernel提供的工具（Utility）来操控硬件正确工作。进一步来说，由于kernel听不懂人类语言，而人类也没有办法直接使用kernel的语言，所以两者的沟通就得藉由shell
BASH是GNU计划中重要的工具软件之一，目前也是GNU操作系统中标准的shell，它主要兼容于sh，并依据一些用户需求而加强，可以说目前几乎所有的Linux版本都是使用bash作为管理核心的主要shell.
BASH主要的优点:
①命令编辑能力: 它能记忆使用过的指令，您只要在命令行中按上下键就可以找到输入的前一个指令. 记录的文件在根目录的.bash_history下。记录的是上一次登入以前执行过的指令，至于这一次登入执行的指令都被暂存在内存中.
②补全功能:
指令补全：用在执行命令时不想按太多按键的时候。例如指令pcprofiledump够长，如果在输入pcprofile之后按下Tab键，bash马上会自动将后面的dump接上来。如果有重复的指令,按下两次Tab将会把所有重复的指令都列出来。
文件名称补全：如果您用vi读取某个文件，例如/etc/man.config，可以在输入vi /etc/man.之后直接按下Tab键，那么该文件名称会被自动补全。
③命令别名: 要实现自定义命令可以使用alias，在命令行输入alias就可以知道当前的命令别名都有哪些。也可以直接输入下列命令来设定别名：
	alias lm='ls -al'
④作业控制、前景背景控制;
⑤Shell scripts 的强大功能;

### 输入指令的方式:
[root@test /root]# command [-options] parameter1 parameter2 ...
指令 选项 参数(1) 参数(2)
1. command为指令的名称，例如变换路径的指令为cd
2. 中括号[]并不存在于实际的指令中，在加入参数时，通常为“-”符号，有时候完整名称会输入“--”符号。例如ls --help
3. parameter1 parameter2为依附在option之后的参数，或者是command的参数
4. command, -options, parameter1这几项之间以空格分隔，不论空几格shell都视为一格
5. 指令太长的时候，可以使用 \ 符号来跳转Enter，使指令连续到下一行
实例：
[root@test /root]# ls -al /root
以ls列出/root目录中的隐藏文件与相关的属性参数
[root@test /root]# ls –al /root \
> /
上面这两行实际上是一行指令，但是加上 \ 跳转符号后，指令可以连续到下一行。请注意，\字符之后直接按下enter就可以，不可在后面接空格符
[root@test /root]# ls -al /root
这个指令与第一个相同，空格符不论几个，仅视为一个来处理

## 变量与变量的设定
Linux系统预设变量名称前会加一个$符号:
[root @test root]# echo $PATH
/bin:/sbin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/local/bin:/bin:/usr/bin:/usr/X11R6/bin
env指令：基本上，在Linux默认情况下，使用{大写字母}设定的变量一般都是系统的预设变量, 简单地使用env指令就可以列出所有环境变量。
set指令：set的使用方法就是直接输入set。它除了会显示当前的环境变量，也会显示您的自定义变量。
?变量：如果您上一个命令执行过程中没有发生错误，那么这个变量会被设为0，如果上个命令出现错误信息，那么这个变量会变成1。
	echo $?
$变量：当前shell的PID；

### 设定变量的规则：
1. 变量与变量内容以等号“=”连结； 
2. 等号两边不能直接接空格符； 
3. 变量名称只能是英文字母与数字，其中数字不能是开头字符； 
4. 若有空格符，可以使用双引号或单引号将变量内容结合起来，但要特别留意，双引号内的特殊字符可以保留变量特性，单引号内的特殊字符则仅为一般字符； 
5. 必要时以转义字符“\”将特殊符号（如Enter，$，\，空格符，'等）变成一般符号； 
6. 在一串指令中，还需要借助其他指令提供的信息，这时可以使用quote“` command`”； 
7. 若该变量为扩增变量内容时，则需以双引号及$变量名称（如"$PATH":/home）继续累加内容； 
8. 若该变量需要在其他子程序执行，则以export使变量可以动作，如export PATH； 
9. 通常大写字符为系统预设变量，自定义变量可以使用小写字符，方便判断（纯粹依照用户兴趣与嗜好）； 
10. 取消变量的方法为：unset 变量名称。 
一般变量设定：
[root @test root]# 12name=VBrid <==错误。因为变量开头不能是数字
[root @test root]# name = VBird <==错误。因为等号两边不能直接接空格符
[root @test root]# name=VBird <==正确。echo $name显示VBird
[test @test test]# name=VBird name <==错误。需要加双引号
[test @test test]# name="VBird name" <==正确。
[test @test test]# name="VBird's name" <==正确。
[test @test test]# name=’VBird’s name’ <==错误。因为有3个单引号
[test @test test]# name=VBird\’s\ name <==正确。
变量累加设定：
[test @test test]# name=$nameisme <==错误。需要用双引号将原变量圈起来
[test @test test]# name="$name"isme <==正确。
[test @test test]# PATH="$PATH":/home/test <==正确。
[test @test test]# PATH="$PATH:/home/test" <==正确。这个形式对于PATH来说也是正确的格式
变量延伸到下一个子程序：
[test @test test]# name="VBird's name" <==设定name变量
[test @tset test]# echo $name <==显示name变量的指令
[test @test test]# VBird's name
[test @test test]# /bin/bash <==另开一个bash的子程序
[test @tset test]# echo $name <==显示name变量
[test @tset test]# <==会显示空字符串，因为name这个变量不能用在子程序
[test @test test]# exit <==退出子程序bash shell
[test @test test]# export name <==正确。如此则$name可以用于下一个子程序中
指令中的指令：
[test @test test]# cd /lib/modules/`uname –r`/kernel
上式中，会先执行`uname –r`这个内嵌的指令，然后输出结果附加在/lib/module… 中，所以，执行这个指令可以完成几个附指令程序
取消变量设定：
[test @test test]# unset name
单引号与双引号的最大不同在于双引号仍然可以保留变量的内容，但单引号内仅能是一般字符，而不会有特殊符号
[root @test root]# name=VBird
[root @test root]# echo $name
VBird
[root @test root]# myname="$name its me"
[root @test root]# echo $myname
VBird its me
[root @test root]# myname='$name its me'
[root @test root]# echo $myname
$name its me

quote：在一串指令中，在 ` 之内的指令将被首先执行，而其执行结果将作为外部的输入信息。
cd /lib/modules/`uname –r`
export 变量：在引用他人的文件或其他程序时，export相当重要，尤其在需要两三个文件互相引用时，如果忘记设定export，那么不同文件中的相同变量值将需要一再地重复设定。所以，只要在头一个文件使用export，那么后续的文件引用该变量时，将会自动读取该变量内容。

`history`指令配合“!”的用法：
[test @test test]# history
[test @test test]# [!number] [!command] [!!]
number ：第几个指令
command ：指令的开头几个字母
! ：上一个指令
[test @test test]# history <==底下列出的就是(1)历史指令的编号；(2)指令的内容
66 man rm
67 alias
68 man history
69 history
[test @test test]# !66 <==执行第 66 个历史指令
[test @test test]# !! <==执行上一个指令（在本例中，就是 !66指令）
[test @test test]# !al <==执行最近一次以 al 为开头的指令内容，就是第 67 个指令

### bash shell 的配置文件
在命令行输入的变量也好、命令别名也罢，都只是针对该次登入的设定，所以，只要您注销，那么上次的设定值就会不见了。
系统设定值
`/etc/profile`：这个文件设定了几个重要变量，例如PATH、USER、MAIL、HOSTNAME、HISTSIZE、umask等，也同时规划出/etc/inputrc 这个针对键盘热键设定的文件数据内容；
`/etc/bashrc`：这个文件用于规划umask，同时规划提示符的内容；
`/etc/man.config`：这个文件指定了输入man时查看数据的路径设定；
通常设定完这几个文件之后，需要先注销再登录后才会启动设定。
个人设定值
`~/.bash_profile`：里面定义了个人路径（PATH）与环境变量的文件名称。可以在这里修改个人路径，当然，也可以在 ~/.bashrc这个个人设定的变量中修改。
`~/.bashrc`：这个文件对于个人喜好的bash设定是最重要的，因为我都是在这里设定我的个人变量，例如命令别名的设定，路径的重新定义等。
`~/.bash_history`：这个文件用于将您曾经用过的命令记录下来，而当您再次以上下
键搜寻或者直接以history搜寻时，就可以找到曾经用过的指令。
`~/.bash_logout`：这个文件则是在您注销shell的时候BASH为您所做的事情。通常默认是只有清除屏幕这件事；
不注销而直接读入变量配置文件使用source即可！
[root @test root]# source 变量配置文件

### 通配符与特殊符号
在bash中常会用到一些通配符，这些通配符搭配一些特殊符号可以更好地利用指令；
* 通配符，代表任意字符（0到多个）
? 通配符，代表一个字符
# 注释，这个最常用在脚本中，视为说明
\ 跳转符号，将特殊字符或通配符还原成一般字符
| 分隔两个管线命令的界定
; 连续性命令的界定（注意，与管线命令不同）
~ 用户的根目录
$ 即变量前需要加的变量值
& 将指令变成在背景下工作
! 逻辑运算中的“非”（not）
/ 路径分隔符号
>, >> 输出导向，分别为“取代”与“累加”
' 单引号，不具有变量置换功能
" 具有变量置换功能
` ` 两个“`”中间为可以先执行的指令
( ) 中间为子shell的起始与结束
[ ] 中间为字符组合
{ } 中间为命令区块组合
[test @test test]# ls test* <==*代表后面不论接几个字符都予以接受(没有字符也接受)
[test @test test]# ls test? <==?代表后面一定要接一个字符
[test @test test]# ls test??? <==???代表一定要接3个字符
[test @test test]# cp test[1-5] /tmp <==test1, test2, test3, test4, test5若存在，就将其复制到/tmp 下
[test @test test]# cd /lib/modules/`uname -r`/kernel/drivers <==被 ` ` 括起来的内容会先执行

### 组合键 
Ctrl + C 终止当前命令
Ctrl + D 输入结束（EOF），例如邮件结束的时候
Ctrl + M 就是Enter
Ctrl + S 暂停屏幕的输出
Ctrl + Q 恢复屏幕的输出
Ctrl + U 在提示符下，将整行命令删除
Ctrl + Z 暂停当前命令

### 连续输入指令的方式
command1; command2 不论 command1 执行结果为何，command2 都会被执行
command1 && command2 当command1执行结果返回值为0时，也就是没有错误时，command2才会开始执行
command1 || command2 当command1 有错误信息时，command2才会执行

### 命令重定向
基本的指令书写方式为：
		>
指令		2>	设备或文件
		>>
		<
左边一定是指令，右边则可能是设备或文件。
命令重定向里几个常用的符号与设备：
< ：由 < 的右边读入参数文件；
> ：将原本由屏幕输出的正确数据输出到 > 右边的file（文件名称）或device（设备，如printer）；
>> ：将原本由屏幕输出的正确数据输出到 >> 右边，与 > 不同的是，该文件将不会被覆盖，而新的数据将以累加方式添加到文件的最后面；
2> ：将原本应该由屏幕输出的错误数据输出到2>的右边；
/dev/null ：可以视为垃圾设备。
[test @test test]# ls -al > list.txt
将显示结果输出到 list.txt 文件中，若该文件已存在则予以取代
[test @test test]# ls -al >> list.txt
将显示结果累加到list.txt 文件中，该文件为累加的，旧数据保留！
[test @test test]# ls -al 1> list.txt 2> list.err
将显示数据正确输出到 list.txt，错误的数据输出到list.err
[test @test test]# ls -al 1> list.txt 2>&1
将显示数据不论正确或错误均输出到list.txt中。注意，错误与正确信息输出到同一个文件中，则必须以上面的方法来写，不能写成其他格式
[test @test test]# ls -al 1> list.txt 2> /dev/null
将显示的数据，正确的输出到list.txt，错误的数据予以丢弃！
1. 完全由键盘输入数据：
[root @test test]# mail -s "test" root <==-s表示标题，root为收件者
I am root! <==以下的数据都是由键盘输入的
That's OK
. <==要结束键盘输入时，需要在一行的最前面加上“.”
CC. <==是否需要有密件副本？不需要的话，直接按下 Enter
EOF <==表示送出
2. 由文件代替输入
[test @test tset]# mail -s "test" root < /root/.bashrc
将.bashrc内容寄给root

### 管线命令
管线命令“|”仅能处理经由前一个指令传来的正确信息，也就是标准输出（Stdout）信息，对于标准错误信息并没有直接处理能力。
① `cut`
[root @test /root ]# cut -d "分隔字符" [-cf] fields
-d ：后面接的是分隔字符，默认是空格符
-c ：后面接的是第几个字符
-f ：后面接的是第几个区块
[root @test /root]# cat /etc/passwd | cut -d ":" -f 1
将 passwd文件中每一行里的“:”用作分隔符，列出第一个区块，也就是姓名所在
[root @test /root]# last | cut -d " " -f1
以空格符作为分隔，并列出第一个区块
[root @test /root]# last | cut -c1-20
将 last 之后的数据，每一行的1~20个字符取出来
② `sort`
[root @test /root ]# sort [-t 分隔符] [(+起始)(-结束)] [-nru]
-t 分隔符：使用分隔符隔开不同区块，默认是 tab
+start -end：由第 start 区块排序到end区块
-n： 使用纯数字排序（否则会以字母方式排序）
-r： 反向排序
-u： 相同出现的一行，只列出一次
[root @test /root]# cat /etc/passwd | sort
将列出来的个人账号排序！
[root @test /root]# cat /etc/passwd | sort -t: +2n
将个人账号以用户ID排序（以 : 作分隔符，第三个为ID，但第一个代号为 0）
[root @test /root]# cat /etc/passwd | sort -t: +2nr
反相排序
③ `wc`
[root @test /root ]# wc [-lmw]
-l ：多少行
-m ：多少字符
-w ：多少字
[root @test /root]# cat /etc/passwd | wc -l
这个文件里有多少行
[root @test /root]# cat /etc/passwd | wc -w
这个文件里有多少字
④ `uniq`
[root @test /root ]# uniq
[root @test /root]# last | cut -d" " -f1 | sort | uniq
这个指令用来删除重复的行从而只显示一个，举个例子，您要知道这个月登入您主机的用户有谁，而不在乎他的登入次数，那么就使用上面的范例，(1)先列出所有的数据；(2)将人名独立列出来；(3)排序；(4)只显示一个。由于这个指令是将重复的东西减少，所以需要配合排序过的文件来处理。
⑤ `split`
[root @test /root ]# split [-bl] 输入文件输出文件前导字符
-b ：以文件 size 来分
-l ：以行数来分
 [root @test /root]# split -l 5 /etc/passwd test
会产生 testaa, testab, testac等文件

## 压缩指令与正规表达法
压缩指令：compress、gzip、bzip、tar；
除了正规表示法之外，我们还可以通过搭配通配符进行字符串的搜索与其他应用。下面列出几个常见的通配符及其含义：
^ 句首字符相符
$ 句尾相同的字符
? 任何一个单一字符
* 随意几个任意字符
\ 跳转符号，将特殊字符变成普通字符
[list] 列表中的字符
[range] 列表中范围内的字符
[^list] 反向选择，与 [list] 相反
[^range] 反向选择，与 [range]相反
\{n\} 与前一个相同字连续n个
\{n,m\} 与前一个相同字连续n-m个
在/etc/下查询某个内容中第一行以boot做为开头的文件：grep ^boot /etc/*
要列出在/etc下含有XYZ三个字符中任何一个字符的那一行：grep [XYZ] /etc/*
在/etc里，只要句首是w-z的就将它列出：grep ^[w-z] /etc/*

## Shell script
bash据以判断执行脚本的步骤为：
	①如果读取到一个Enter符号（CR），就尝试开始执行该行命令；
	②如同前面bash command提到的，指令间的多个空白会被忽略；
	③空白行也将被忽略，并且Tab也不会被理会；
	④如果一行内容太多，则可以使用\延伸至下一行；
	⑤此外，使用最多的#可做为注释。任何加在#后的字，将全部被视为注释文字而被忽略。
执行的方法：
	①一个是将该文件改成可以执行的属性，如chmod 755 scripts.file，然后执行该文件；
	②另一种则是直接以sh这个执行文件来执行脚本内容，如sh scripts.file。

[root @test /root]# mkdir test; cd test
建立一个新的目录，所有的脚本都暂存在此！
[root @test test]# vi test01-hello.sh
#!/bin/bash <==在 # 之后加上 ! 与 shell 的名称，用来宣告使用的 shell
# 这个脚本的用途在于在屏幕上显示Hello ! How are you
# 创建日期： 2002/05/20
hello=Hello\ \!\ How\ are\ you\ \? <==这就是变量
echo $hello
[root @test test]# sh test01-hello.sh
Hello ! How are you ? <==输出结果显示在屏幕上！
这里注意：
	①所有在脚本里的东西，基本规则（如变量设定规则）需要与在命令行下相同；
	②脚本的后缀名最好为.sh，以方便他人识别；
	③并非加上.sh就是可执行文件，还需要查看其属性中是否有x属性。

声明变量使用declare指令：
`declare`
 [test @test test]# declare [-afirx]
-a ：定义为数组 array
-f ：定义为函数 function
-i ：定义为整数 integer
-r ：定义为只读
-x ：定义为通过环境输出变量
 [test @test test]# declare -i a=3
[test @test test]# declare -i b=5
[test @test test]# declare -i c=$a*$b
[test @test test]# echo $c
15 <==变成数字了。
如果没有定义某个变量，则该变量默认呈现字符串的形式：
[root @test test]# vi test03-declare.sh
#!/bin/bash
# This program is used to "declare" variables
number1=2*3+5*13-32+25
declare -i number2=2*3+5*13-32+25
echo "Your result is ==> $number1"
echo "Your result is ==> $number2"
[root @test test]# sh test03-declare.sh
Your result is ==> 2*3+5*13-32+25
Your result is ==> 64

### 交互式脚本
[root @test test]# read name
VBird <==这是您直接在键盘上输入的内容
[root @test test]# echo $name
VBird
应用到脚本中：
[root @test test]# vi test04-read.sh
#!/bin/bash
# This program is used to "read" variables
echo "Please keyin your name, and press Enter to start."
read name
echo "This is your keyin data ==> $name"

[root @test test]# sh test04-read.sh
Please keyin your name, and press Enter to start.
VBird Tsai
This is your keyin data ==> VBird Tsai

### 脚本的参数代号
[root @test test]# 	myscript 	opt1 	opt2 	opt3 	opt4
						$0 		$1 		$2 		$3 		$4
上面的意思是说，在这个脚本（myscript）中，只要变量名称为$0，就表示为myscript，
也就是说：
· $0 : myscript，亦即脚本的文件名
· $1 : opt1，亦即第一个附加的参数
· $2 : opt2
· $3 : opt3
 [root @test test]# vi test05-0123
#!/bin/bash
# This program will define what is the parameters
echo "This script's name => $0"
echo "parameters $1 $2 $3"

[root @test test]# sh test05-0123 pa1 pa2 pa3
This script's name => test05-0123
parameters pa1 pa2 pa3

### 脚本逻辑判断式与表达式
逻辑卷标含义
1. 关于文件与目录的逻辑卷标
	-f 常用！检测文件是否存在		-d 常用！检测目录是否存在		-b 检测是否为一个block文件
	-c 检测是否为一个character文件		-S 检测是否为一个socket标签文件	-L 检测是否为一个符号链接文件
	-e 检测某个东西是否存在！可以是任何东西
2. 关于程序的逻辑卷标
	-G 检测是否由GID所执行的程序拥有
	-O 检测是否由UID所执行的程序拥有
	-p 检测是否为程序间传送信息的name pipe或FIFO
3. 关于文件的属性检测
	-r 检测是否为可读属性		-w 检测是否为可写入属性		-x 检测是否为可执行属性
	-s 检测是否为非空白文件		-u 检测是否具有SUID属性		-g 检测是否具有SGID属性
	-k 检测是否具有sticky bit属性
4. 两个文件之间的判断与比较。例如test file1 -nt file2
	-nt 第一个文件比第二个文件新
	-ot 第一个文件比第二个文件旧
	-ef 第一个文件与第二个文件为同一个文件（诸如链接文件）
5. 逻辑与（and）和或（or）
	&& 逻辑与
	|| 逻辑或

### 运算符
= 等于	!= 不等于		< 小于	> 大于	-eq 等于	-ne 不等于
-lt 小于	-gt 大于		-le 小于或等于		-ge 大于或等于
-a 双方都成立（and）	-o 单方成立（or）	-z 空字符串
-n 非空字符串

### 条件判断
if [ 条件判断一 ] && (||) [ 条件判断二 ]; then <==if 是起始，后面可以接若干个判断式，使用 && 或 ||执行判断
elif [ 条件判断三 ] && (||) [ 条件判断四 ]; then <==第二段判断，如果第一段不符合要求就转到此搜寻条件，执行第二段程序
else <==当前两段都不符合时，就执行这段内容
fi <==结束 if then 的条件判断
注：①在 [ ] 中，只能有一个判断式；
	②在 [ ] 与 [ ] 间，可以使用&&或| |结合判断式；
	③每一个独立的组件之间需要用空格键隔开。

### case... esac
case 种类方式(string) in <==开始阶段，种类方式可分成两种类型，
通常使用 $1 这种直接输入类型
种类方式一)
程序执行段
;; <==种类方式一的结束符号
种类方式二)
程序执行段
;;
*)
echo "Usage: {种类方式一|种类方式二}" <==列出可以利用的参数值
exit 1
esac <== case结束处
种类方式（string）的格式主要有两种：
[root @test test]# vi test10-case.sh
#!/bin/bash
# program: Using case mode
# content: I will use this program to study the case mode!
# 1. print this program
echo "Press your select one, two, three"
read number
case $number in
one)
echo "your choice is one"
;;
two)
echo "your choice is two"
;;
three)
echo "your choice is three"
;;
*)
echo "Usage {one|two|three}"
exit 1
esac
[root @test test]# sh test10-case.sh
Press your select one, two, three
two <==这一行是您输入的内容
your choice is two

### 循环
①for ((条件1; 条件2; 条件3))
②for variable in variable1 variable2 .....
③while [ condition1 ] && { | | } [ condition2 ] ...
④until [ condition1 ] && { | | } [ condition2 ] ...
计算1 + 2+ 3 +... + 100：
[test @test test]# vi test11-loop.sh
#!/bin/bash
# Using for and loop
declare -i s # <==变量声明
for (( i=1; i<=100; i=i+1 ))
do
s=s+i
done
echo "The count is ==> $s"
[test @test test]# sh test11-loop.sh
The count is ==> 5050
----------------------------------------------------------
[test @test test]# vi test14-for.sh
#!/bin/bash
# using for...do ....done
LIST="Tomy Jony Mary Geoge"
for i in $LIST
do
echo $i
done
[test @test test]# sh test5.sh
Tomy
Jony
Mary
Geoge
----------------------------------------------------------
[test @test test]# vi test12-loop.sh
#!/bin/bash
# Using while and loop
declare -i i
declare -i s
while [ "$i" != "101" ]
do
s=s+i
i=i+1
done
echo "The count is ==> $s"

### 如何调试脚本
[test @test test]# sh [-nvx] scripts
-n ：不执行脚本，查询脚本内的语法，若有错误则列出
-v ：在执行脚本之前，先将脚本的内容显示在屏幕上；
-x ：将用到的脚本内容显示在屏幕上，与-v稍微不同

## 账号管理
用户ID与群组ID
Linux并不认识账号名称，它认识的其实是账号ID。即Linux只认识代表身份的号码，而对应的号码与账号则是记录在/etc/passwd中。

### 登入Linux主机
在输入账号与密码之后，Linux会：
①先查找/etc/passwd中是否有这个账号，如果没有则跳出，如果有则将该账号对应的UID（User ID）与GID（Group ID）读出来。另外，该账号的根目录与shell设定也一并读出。
②然后核对密码表。这时Linux会进入/etc/shadow中找出对应的账号与UID，然后核对您刚刚输入的密码与其密码是否相符。
③一切妥当之后，进入Shell。

### UID、GID、SUID 与SGID
每个文件都会有所谓的拥有者ID与拥有群组ID，亦即UID与GID，然后系统依据/etc/passwd的内容，将该文件的拥有者与群组名称、使用账号的形式显示出来； UID为0时，就是root。
/etc/passwd 文件：在这个文件中，每一行代表一个账号，有几行就代表在您的系统中有几个账号。
/etc/shadow 文件：存放账号对应的密码。
/etc/group：这个文件可以让您直接将账号所要支持的群组加进来。
/etc/gshadow：存放群组密码。

### 增加用户的一般步骤
添加群组：`groupadd`
[root @test /root ]# groupadd [-g GID] groupname
-g GID：自行设定GID的大小
 [root @test /root]# groupadd -g 55 testing <==设定一个群组，GID为55
也可以直接修改/etc/group与/etc/gshadow这两个文件，根本不需要使用这个指令，使用vi修改上面两个文件更简单。如果要新增的用户所在群组并不存在于系统中，那么在增加用户账号之前，必须先新增群组。
删除群组：`groupdel`
[root @test /root ]# groupdel groupname
[root @test /root]# groupdel testing
在删除群组之前，请先将该群组的Primary用户删除。Primary就是/etc/passwd中其GID设定为这个群组的GID
的用户。
添加用户：`useradd`
[root @test /root ]# useradd [-u UID] [-g GID] [-d HOME] [-mM] [-s shell] username
-u ：直接给出一个UID
-g ：直接给出一个GID（此GID必须已经存在于/etc/group中）
-d ：直接将其根目录指向已经存在的目录（系统不会再建立）
-M ：不建立根目录
-s ：定义其使用的shell
 [root @test /root]# useradd testing
直接以默认数据建立一个名为testing的账号
[root @test /root]# useradd -u 720 -g 100 -M -s /bin/bash testing
以自己的设定建立账号
删除用户：`userdel`
[root @test /root ]# userdel [-r] username
-r ：将该账号的[home directory]与[/var/spool/mail/username]一并删除
 [root @test /root]# userdel testing
只删除/etc/passwd与/etc/shadow中该账号的内容
[root @test /root]# userdel -r testing
连同该账号的/home/testing与/var/spool/mail/testing一起删除

`chsh`
[root @test /root ]# chsh [-l] [-s shellname]
-l ：列出当前机器上能用的shell名称
-s ：将当前的shell变为shellname
 [test @test /root]# chsh -l <==列出本机上所有能用的shell名称
/bin/sh
/bin/bash
/bin/ash
/bin/bsh
/bin/csh
[test @test /root]# chsh -s /bin/csh <==test用户自行改变自己的默认shell

密码管理与设定：`passwd`
root可以设定任何样式的密码，而且，root可以帮助user设定他们的密码，而user仅能修改自己的密码。修改密码使用passwd这个命令。passwd必须具有SUID才能让一般用户修改其密码。
[root @test /root]# passwd [username]
[test @test /root]# passwd <==一般用户自行修改密码
[root @test /root]# passwd test <==root帮助修改密码
Changing password for user test
New password: <==输入密码
BAD PASSWORD: it is based on a dictionary word
Retype new password: <==再次输入
passwd: all authentication tokens updated successfully

### 用户身份切换
将一般用户变成root：
	①以su直接将身份变成root。这个指令需要root的密码，也就是说，如果您要以su变成root，您的一般用户必须有root的密码。
	[root @test /root ]# su
	[test@test test]$ su
	Password: <==输入root的密码
	[root@test test]# <==身份变成root
	虽然您已经是root，但是您的环境还是属于当初登入的那个用户。例如，我以test登入Linux，再以su切换身份成为root，但是我的mail，PATH及其他相关的环境变量，都还是test这个身份的。单纯使用su变换成root身份，最大的好处是可以直接输入我们惯用的指令
	②如果多人同时管理一台主机，那么，root的密码就有很多人知道了，这样不好。所以，不想将root的密码透露出去，可以使用sudo。
	[root @test /root ]# sudo [-u username] [command]
	-u ：将身份变成username的身份
 	[test@test test]$ sudo mkdir /root/testing
	Password: <==输入test的密码
	[root@test test]# sudo -u test touch test
	root可以执行test用户的指令，建立test的文件
	不需要root的密码仍可以执行root的工具，这时就可以使用sudo。由于执行root身份的工作时，输入的密码是用户的密码，而不是root的密码，所以可以减少root密码外泄的可能性。

### 用户查询
可以直接到/etc/passwd及/etc/group中查看，但还有更简单的方法，就是使用简单的指令工具：
	①id 查询用户的UID，GID及所拥有的群组；
	[root @test root]# id [username]
 	[root @test root]# id
	uid=0(root) gid=0(root) groups=0(root)
	[root @test root]# id test
	uid=501(test) gid=501(test) groups=501(test)
	②groups 查询用户能够支持的群组：直接输入groups就可以显示当前用户所属的群组，Primary与其他相关群组都会显示出来。
	③finger 查询用户的一些相关信息，如电话号码等。
	[root @test root]# finger [-s] username
	-s ：完整列出



## Linux多用户多任务环境
Linux默认提供了6个文字界面登入窗口，以及一个图形界面，您可以使用Alt+F1~F7来切换不同的终端界面，而且每个终端界面的登入者可以是不同的人，这就不同于Windows一次只能在屏幕前登入一个人的情况。这个特性很有用，尤其在某个程序死掉的时候。可以随意按下Alt+F1~F7切换到其他终端界面，然后以ps -aux找出刚刚的错误程序，然后杀掉它，转到前面的终端界面，就又恢复正常了。

### 背景工作管理
`&`
[root @test /root ]# command &
[root @test /root]# find / -name testing & <==将该执行程序丢到背景执行
[root @test /root]# fg <==将该程序拉回屏幕前执行
也可以使用Ctrl+z将当前正在进行中的工作丢到背景下。放在背景下执行最大的好处就是不怕被Ctrl+c这个中断指令中断。

### 程序与资源管理
[root @test /root ]# ps -aux
a : 选择列出所有的程序
u : 列出所有用户的程序
x : 列出所有tty的程序

[root @test /root ]# `top`
在执行top的程序中，可以输入下面的字符进行排序
A ：以age亦即执行的先后顺序进行排序
T ：由启动的时间排序
M ：以所占的内存大小排序
P ：以所耗用的CPU资源排序
使用top可以用动态（每5秒钟更新一次）的方式检测程序的运行。

[root @test /root ]# `free`
-k ：以KBytes显示内存
-m ：以MBytes显示内存

[root @test /root ]# kill -signal PID
-signal跟上面的kill一样：
-1 ：让该PID重新读取它的配置文件
-9 ：杀掉该程序
-15 ：停止该程序
[root @test /root]# kill -9 2380

[root @test /root ]# uname [-apnr]
-a ：列出所有的系统信息
-p ：列出CPU信息
-n ：列出主机名
-r ：列出核心版本信息

### 程序的优先级
[root @test /root ]# ps -l
F S UID PID PPID C PRI NI ADDR SZ WCHAN TTY TIME CMD
100 S 0 5624 5606 0 70 0 - 608 wait4 pts/0 00:00:00 bash
000 R 0 6944 5624 0 76 0 - 769 - pts/0 00:00:00 ps
注意，上面的信息中：
· UID代表执行者的身份；
· PID代表这个程序的代号；
· PPID代表这个程序由哪个程序发展而来，即父程序；
· PRI代表这个程序可被执行的优先级，越小就越早被执行；
· NI代表这个程序的nice值，nice值是系统可被执行的修正数值。如前面所说，PRI越小就越先被执行，在我们加入nice值之后，将使PRI变为：
	PRI(new) = PRI(old) + nice
这样，当nice值为负值时，该程序会提前执行，即调整了程序处理的优先顺序。只有具有root权限的用户可以将程序的nice值调为负值，所以，对于nice值有如下约定：
	①一般用户可用的nice值为0 ~ 19；
	②root 管理员可用的nice值为-20 ~ 19。
[root @test /root ]# nice [-n number] command
-n ：后面那个number即为nice值
[root @test /root]# nice -n -5 find / -name core > /tmp/core

[root @test /root ]# renice [number] PID
[root @test /root]# ps -aux
[root @test /root]# renice 5 234
renice的功能不太一样，由于renice用于改变一个正在进行中的程序的优先级，所以必须先取得该程序的PID。

### 信息管理
[root @test /root ]# `dmesg`
[root @test /root]# dmesg | more
dmesg提供了系统信息，例如CPU的形式、硬盘、光盘型号及硬盘分割表等。

[root @test /root ]# `uptime`
[root @test /root]# uptime
11:27pm up 9 days, 7:12, 1 user, load average: 0.07, 0.12, 0.14
系统显示当前时间是11:27pm，系统已经开机了9天零7个小时12分，当前有一个用户在线，过去的1，5，15分钟系统平均负载为0.07，0.12，0.14。

[root @test /root]# `who`
root pts/0 Aug 2 20:43
[root @test /root]# w
8:48pm up 4 days, 5:08, 1 user, load average: 0.00, 0.00, 0.00
USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT
root pts/0 192.168.1.2 8:43pm 0.00s 0.38s ? -
[root @test /root]# whoami
test

[root @test /root ]# `last`
-number ：number为数字，如果您的登入信息太多，可以使用这个指令！
[test @test /root]# last -5
test pts/0 192.168.1.2 Tue Apr 9 20:34 - 20:35 (00:01)
test pts/0 192.168.1.2 Tue Apr 9 20:14 - 20:30 (00:15)
test ftpd21546 192.168.1.2 Tue Apr 9 02:55 - 03:06 (00:10)
test ftpd15813 192.168.1.2 Tue Apr 9 01:20 - 01:21 (00:00)
test pts/0 192.168.1.2 Mon Apr 8 20:14 - 00:27 (04:13)
wtmp begins Tue Apr 2 01:12:26 2002

## 开机关机流程与多重启动
### Linux开机流程
①加载BIOS的硬件信息；
	主机的CPU数据、启动顺序、硬盘大小、芯片组工作状态、PnP的开启与否、内存的时钟，等等。
②读取MBR的Kernel Loader引导信息；
	在读完BIOS之后，会先读取第一个引导硬盘的第一个扇区（就是主引导扇区记录，MBR），这个扇区主要记录引导信息。
③加载内核的操作系统核心信息；
④内核执行init程序并取得运行信息；
	由内核执行的第一个程序就是/sbin/init，而这个程序第一个目标当然是确定主机以怎样的模式登入。
	/etc/inittab文件：
	# 0 - halt (Do NOT set initdefault to this) 关机
	# 1 - Single user mode 单用户模式（系统有问题时的登入模式）
	# 2 - Multiuser, without NFS (The same as 3, if you do not have networking)多用户但无网络
	# 3 - Full multiuser mode 从文字界面登入的多用户系统
	# 4 - unused 系统保留
	# 5 - X11 X-Windows图形界面登入的多用户系统
	# 6 - reboot (Do NOT set initdefault to this) 重新启动
	#
	id:3:initdefault: <=默认的登录模式
⑤init执行/etc/rc.d/rc.sysinit文件将主机的信息读入Linux系统，包括默认路径、文件系统等等。
⑥启动核心的外挂式模块（/etc/modules.conf）；
⑦init执行运行一级的各个批处理文件（Scripts）；
⑧init执行/etc/rc.d/rc.local文件：执行您的Linux主机的个性化设定；
⑨执行/bin/login程序；
⑩登入之后开始以Shell控管主机。

### 变换默认的登入模式
直接修改/etc/inittab文件的内容即可：以vi或其他文本编辑软件进入/etc/inittab文件，文件的内容类似下面这样：
Default runlevel. The runlevels used by Mandrake Linux are:
0 - halt (Do NOT set initdefault to this)
1 - Single user mode
2 - Multiuser, without NFS (The same as 3, if you do not have networking)
3 - Full multiuser mode
4 - unused
5 - X11
6 - reboot (Do NOT

id:3:initdefault: <=默认的登录模式

## X-Window的架构
`X server`：就是用来管理Linux中关于显示的一些硬件与驱动程序。
`X client`：X server主要的功能只是管理显示的驱动程序与硬件，但是将整个屏幕前显示给用户，并且经由用户移动鼠标或键盘来启动一些事件，以响应给X server，并进一步处理一些信息，则需要一些X的软件，这些关于X的软件，我们就称为X client。
Window manager：窗口管理员可以简单看做是一个X client，这个Window manager主要是做为您和整个X Window系统的接口，所以，所有的X软件（就是上面讲到的X client）都归它管。目前最热门的两个窗口管理员就是KDE与GNOME。

## 简易连接Internet的方法介绍
Linux网络相关配置文件
①/etc/sysconfig/network
这个文件主要的功能用于设定默认的GATEWAY，修改主机名称（HOSTNAME），是否启动Network。
②/etc/sysconfig/network-scripts/ifcfg-ethn
这个文件的内容即是设定网卡的参数文件，里面可以设定network，IP，netmask，broadcast，gateway，开机时取得IP协议的方式（DHCP，static），是否在开机时启动等，其中的n 是数字。如果是第一块网卡，则文件名称为 ifcfg-eth0 ，第二块网卡为 ifcfg-eth1，依此类推。
③/etc/modules.conf
这个文件只在找不到网卡芯片组的时候才会用到，亦即开机时系统一些核心模块的加载文件。
④/etc/resolv.conf
这个是设定DNS（域名服务器）的文件，常常有人提到，我已经可以 ping 到远程计算机的公共IP了，为何输入网址却无法联机？通常发生错误的就是这个文件。
⑤/etc/hosts
这个文件可以记录计算机的IP对应主机的名称或者主机的别名。通常如果想要改善联机速度，尤其是在内部私有IP的情况下，由于缺乏DNS反查信息，所以这里需要将私有IP写入这个文件中，这样内部网域对于具有公共IP的主机的联机速度才会有明显改善。




