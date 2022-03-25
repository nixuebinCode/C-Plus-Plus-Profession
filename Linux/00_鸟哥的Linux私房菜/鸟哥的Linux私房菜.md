# 第2章 主机规划与磁盘分区

## 2.1 各硬件设备在Linux中的文件名

在Linux系统中，每个设备都被当成一个文件来对待

 ![image-20220310192422227](\images\image-20220310192422227.png)

## 2.2 磁盘分区

### 2.2.1 MBR(MS-DOS)与GPT磁盘分区表

#### MBR(Master Boot Record, 主引导记录)分区表格式与限制

启动引导程序及录取与分区表通通放在磁盘的第一个扇区，这个扇区通常是512字节的大小，包括：

* 主引导记录：可以安装启动引导程序的地方，有446字节
* 分区表：记录整块硬盘分区的状态，有64字节

由于分区表所在区块仅有64 Bytes容量，因此最多仅能有四组记录区，每组记录区记录了该区段的启始与结束的柱面号码。注意分区的最小单位通常为柱面。

 ![](\images\1.jpg)

##### 扩展分区

既然分区表只有记录四组数据的空间，那么是否代表我一颗硬盘最多只能分区出四个分区？当然不是。

扩展分区的目的是使用额外的扇区来记录分区信息，扩展分区本身并不能被拿出来格式化。

 ![](\images\2.jpg)

L1~L5是由扩展分区继续切出来的分区，就被称为**逻辑分区**。

上述的分区在Linux系统中的设备文件名分别如下：
P1: /dev/sda1
P2: /dev/sda2
L1: /dev/sda5
L2: /dev/sda6
L3: /dev/sda7
L4: /dev/sda8
L5: /dev/sda9
前面四个号码都是保留给主要分区或扩展分区用的，所有逻辑分区的设备名称号码就由5号开始。

##### MBR主要分区、扩展分区、逻辑分区特性

* 主要分区与扩展分区最多可以有四个（硬盘的限制）
* 扩展分区最多只能有一个（操作系统的限制）
* 逻辑分区是由扩展分区持续划分出来的分区
* 能够被格式化后作为数据存取的分区是主要分区与逻辑分区。扩展分区无法格式化
* 逻辑分区的数量依操作系统而不同，在Linux系统中SATA硬盘已经可以突破63个以上的分区限制

#### GPT(GUID partition table)磁盘分区表

过去一个扇区大小就是512字节，不过目前已经有4K的扇区设计出现。为了兼容所有的磁盘，因此在扇区的定义上面，大多会使用所谓的逻辑区块地址（Logical Block Address, LBA）来处理。

GPT将磁盘的所有区块以此LBA来规划。
与 MBR 仅使用第一个 512Bytes 区块来纪录不同， GPT 使用了 34 个 LBA 区块来纪录分区信息。同时与过去 MBR 仅有一的区块，被干掉就死光光的情况不同， GPT 除了前面 34 个LBA 之外，整个磁盘的最后 34 个 LBA 也拿来作为另一个备份。

### 2.2.2 启动流程中的BIOS

#### BIOS 搭配 MBR/GPT 的启动流程

BIOS是一个写入到主板上的一个固件（固件就是写入到硬件上的一个软件程序）。这个BIOS就是在开机的时候，计算机系统会主动执行的第一个程序。接下来整个启动流程为：

1. BIOS：启动后主动执行的固件，会认识第一个可启动的设备；
2. MBR：第一个可启动设备的第一个扇区内的主引导记录块，内含启动引导代码；
3. 启动引导程序（boot loader）：一个可读取内核文件来执行的软件；
4. 内核文件：开始操作系统的功能。

其中boot loader的主要任务有：

1. 提供菜选项：用户可以选择不同的启动选项，这也是多重引导的重要功能；
2. 加载内核文件：直接指向可使用的程序区段来启动操作系统；
3. 转交其他loader：将启动管理功能转交给其他loader负责。

启动引导程序boot loader除了可以安装在MBR之外，还可以安装在每个分区的启动扇区。

### 2.2.3 Linux安装模式下，磁盘分区的选择

#### 目录树结构

所谓的目录树架构（directory tree）就是以根目录为主，然后向下呈现分支状的目录结构的一种文件架构。

整个目录树架构最重要的就是那个根目录（root directory），这个根目录的表示方法为一条斜线“/”， 所有的文件都与目录树有关。

 ![](\images\3.jpg)

#### 文件系统与目录树的关系（挂载）

我们现在知道整个Linux系统使用的是目录树架构，但是我们的文件数据其实是放置在磁盘分区当中的， 现在的问题是“如何结合目录树的架构与磁盘内的数据”呢？

所谓的挂载，就是利用一个目录当成进入点，将磁盘分区的数据放置在该目录下。也就是进入该目录就可以读取该分区的意思。

例如我们将硬盘分为两个区，分区1挂载到根目录，分区2挂载到/home这个目录。也就是说当我的数据放置在/home内的各次目录时，数据是放置到
分区2的，如果不是放在/home下面的目录， 那么数据就会被放置到分区1了。

 ![](\images\4.jpg)

当我们想知道/home/vbird/test这个文件在哪个分区时，反向追踪，由test --> vbird --> home --> /，看哪个“进入点”先被查到就可以了。所以test使用的是/home这个进入点而不是/。

#### distributions安装时，挂载点与磁盘分区的规划

* 初次接触Linux：只要划分“ / ”及“交换分区”即可

* 建议分区的方法：预留一个备用的剩余磁盘容量

  可以用这个剩余的容量划分出新的分区，并使用它来备份重要的文件。

## 2.3 安装Linux前的规划

 ### 2.3.1 选择适当的Linux发行版

CentOS 7.1版，下载 everything 版本即可：CentOS-7-x86_64-Everything-1503-01.iso

CentOS官方网站：http://mirror.centos.org/centos/7/isos/

中科大镜像站：http://centos.ustc.edu.cn/centos/7/isos/

# 第3、4章 安装CentOS7.x与登录

Linux中，只要文件名的开头是小数点，那么该文件名就不会在一般观察模式被显示出来。所以说，在Linux下面，隐藏文件并不是什么特殊的权限，单纯是因为文件名命名的处理方式不同。

## 4.1 首次登录系统

### 4.1.1 X Window与命令行模式的切换

Linux默认的情况下会提供六个终端来让用户登陆，切换的方式为使用【Ctrl+Alt+F1~F6】的组合键。

* [Ctrl] + [Alt] + [F2] ~ [F6] ：命令行登陆 tty2 ~ tty6 终端机；
* [Ctrl] + [Alt] + [F1] ：图形接口桌面。

### 4.1.2 在终端登录Linux

在终端正确输入账号名和密码之后，能够看到：

 ![image-20220313211512698](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220313211512698.png)

[dmtsai@study ~]$ _

是正确登陆之后才显示的讯息， 最左边的 dmtsai 显示的是“目前使用者的帐号”，而@之后接的 study 则是“主机名称”，至于最右边的~则指的是“目前所在的目录”，那个$则是我们常常讲的“提示字符“。

~ 符号代表的是“用户的家目录”的意思，它是个“变量”，举例来说root的家目录在/root，所以~ 就代表/root的意思。而dmtsai的家目录在/home/dmtsai，所以如果你以dmtsai登录时，~ 就会等价于/home/dmtsai。

在Linux中，默认root的提示字符为#，而一般身份用户的提示字符为$。

#### 离开系统

注销Linux很简单，直接输入`exit`就可以了

但是离开系统并不是关机，Linux本身已经有相当多的任务在进行，你的登录也仅是其中的一个任务而已，所以当你离开时，这次这个登录任务 就停止了，但此时Linux其他的任务还是在继续进行中。

## 4.2 命令行模式下命令的执行

命令行模式登陆后所取得的程序被称为壳（Shell），这是因为这支程序负责最外面跟使用者（我们）沟通，所以才被戏称为壳程序。

我们Linux的壳程序就是厉害的BASH。

### 4.2.1 开始执行命令

#### 命令的格式

`[dmtsai@study ~]$ command [-options] parameter1 parameter2 ...`

* 一行指令中第一个输入的部分绝对是“命令（command）”或“可执行文件（例如shell脚本）”
* 命令, 选项, 参数等这几个东西中间以空格来区分，不论空几格 shell 都视为一格。所以空格是很重要的特殊字符。
* 按下[Enter]按键后，该命令就立即执行。[Enter]按键代表着一行命令的开始启动。
* 命令太长的时候，可以使用反斜线 （\） 来转义回车键，使命令连续到下一行。

### 4.2.2 基础命令的操作

#### 显示日期的命令：date

```shell
[dmtsai@study ~]$ date
Fri May 29 14:32:01 CST 2015
[dmtsai@study ~]$ date +%Y/%m/%d
2015/05/29
[dmtsai@study ~]$ date +%H:%M
14:33
```

#### 显示日历的指令： cal

列出目前这个月的日历

```shell
[dmtsai@study ~]$ cal
May 2015
Su Mo Tu We Th Fr Sa
                1 2
3  4  5  6   7 8   9
10 11 12 13 14 15 16
17 18 19 20 21 22 23
24 25 26 27 28 29 30
31
```

显示整年的月历

```shell
[dmtsai@study ~]$ cal 2015
```

显示2015年10月的月历

```shell
[dmtsai@study ~]$ cal 10 2015
```

#### 简单好用的计算机： bc

在命令行输入bc后，屏幕会显示出版本信息， 之后就进入到等待指示的阶段。

bc默认仅输出整数，如果要输出小数点下位数，那么就必须要执行 scale=number ，那个number就是小数点位数。

要离开bc回到命令行界面时，务必要输入【quit】来离开bc的软件环境。

### 4.2.3 重要的几个热键

#### Tab按键

具有命令补全与文件补齐的功能。

例如在命令行输入ca后，连续按下**两次**【Tab】按键，所以以ca为开头的命令都被显示出来。

#### Ctrl-c按键

中断目前程序的按键。

#### Ctrl-d按键

* 表示键盘输入结束（End Of File）
* 也可以用来取代exit的输入，例如你想要直接离开命令行模式，可以直接按下Ctrl-d就能直接离开，相当于输入exit

#### [shift]+{[Page UP]|[Page Down]}按键

当某些命令的输出信息很长的时候，可能会导致前面的部分已经不在目前的屏幕中，可以使用这两个组合键来往前/往后翻页

## 4.3 Linux系统在线求助man page与info page

### 4.3.1 命令的 --help求助说明

使用命令的--help选项就能够将该命令的用法做一个大致的理解。

### 4.3.2 man page

除了【date --help】之外，还可以执行【man date】，就可以看到date的一大堆用法，出现的屏幕界面，就称它为man page。

 ![image-20220314145036675](\images\image-20220314145036675.png)

其中DATE(1)后面的1表示的是date是一般用户可使用的命令，其它的如下：

1：用户在shell环境中可以操作的命令或可执行文件
5：配置文件或者是某些文件的格式
8：系统管理员可用的管理命令

#### man page中常用的按键

 ![image-20220314145745736](\images\image-20220314145745736.png)

### 4.3.3 info page

info与man的用途其实差不多，但是与man page一口气输出一堆信息不同的是，info page则是将文件数据拆成一个一个的段落，每个段落用自己的页面来编写，并且在各个页面中还有超链接来跳到各个不同的页面中，每个独立的页面也被称为一个节点。

不够查询的命令说明要具有info page功能的话，得用info page的格式来写成在线求助文件才行。至于以非info page格式写的说明文件，虽然也能够使用info来显示，不过其结果就会跟man相同。

## 4.4 超简单的文本编辑器：nano

在nano后面直接加上文件名就能够打开一个旧文件或新文件：`nano text.txt`，文件存在就打开旧文件，不存在就创建新文件。

进入nano后，下方会有组合键的功能，其中指数符号(^)代表的是键盘的[Ctrl]按键。下面列举出几个重要的组合键：

* [ctrl]-G：获得联机帮助（help）
* [ctrl]-X：离开nano软件，若有修改过文件会提示是否需要保存
* [ctrl]-O：保存文件，若你有权限的话就能够保存文件了
* [ctrl]-R：从其他文件读入数据，可以将某个文件的内容贴在本文件中
* [ctrl]-W：查找字符串
* [ctrl]-C：说明目前光标所在处的行数与列数等信息
* [ctrl]-_：可以直接输入行号，让光标快速移动到该行
* [alt]-Y：语法校验功能开启或关闭
* [alt]-M：可以支持鼠标来移动光标的功能

## 4.5 正确的关机方法

### 数据同步写入磁盘：sync

直接在命令行模式下输入sync，那么在内存中尚未被更新的数据，就会被写入硬盘中。所以，这个命令在系统关机或重新启动之前，最好多执行几次。

### 常用的关机命令：shutdown

`shutdown [-krhc] [时间] [警告讯息]`

#### 选项与参数：
-k ： 不要真的关机，只是发送警告信息出去
-r ： 在将系统的服务停掉之后就重新启动（常用）
-h ： 将系统的服务停掉后，立即关机。 （常用）
-c ： 取消已经在进行的 shutdown 命令内容
时间 ： 指定系统关机的时间。若没有这个项目，则默认 1 分钟后自动进行。

#### 使用范例

`[root@study ~]# shutdown -h now`
立刻关机，其中 now 相当于时间为 0 的状态
`[root@study ~]# shutdown -h 20:25`
系统在今天的 20:25 分会关机，若在21:25才下达此指令，则隔天才关机
`[root@study ~]# shutdown -h +10`
系统再过十分钟后自动关机
`[root@study ~]# shutdown -r now`
系统立刻重新开机
`[root@study ~]# shutdown -r +30 'The system will reboot'`
再过三十分钟系统会重新开机，并显示后面的讯息给所有在线上的使用者
`[root@study ~]# shutdown -k now 'This system will reboot'`
仅发出警告信件的参数！系统并不会关机啦！吓唬人！

### 重新启动，关机：reboot、halt、poweroff

#### 重新启动

`sync; sync; sync; reboot`

### 实际使用管理工具systemctl关机

上面谈到的halt、poweroff、reboot、shutdown等，其实都是调用systemctl这个命令。

```shell
[root@study ~]# systemctl [命令]
命令项目包括如下：
halt 进入系统停止的模式，屏幕可能会保留一些信息，这与你的电源管理模式有关
poweroff 进入系统关机模式，直接关机
reboot 直接重新启动
suspend 进入休眠模式
[root@study ~]# systemctl reboot # 系统重新启动
[root@study ~]# systemctl poweroff # 系统关机
```

# 第五章 Linux的文件权限与目录配置

## 5.1 用户与用户组

用户组最有用的功能之一，就是当你在团队进行协同工作的时候。

每个账号都可以有多个用户组的支持。

### Linux用户身份与用户组记录文件

在我们Linux系统当中，默认的情况下，所有的系统上的帐号与一般身份用户，还有那个root的相关信息， 都是记录在/etc/passwd这个文件内的。

至于个人的密码则是记录在/etc/shadow这个文件内。

 此外，Linux所有的组名都纪录在/etc/group中。

## 5.2 Linux文件权限概念

### 5.2.1 Linux文件属性

>使用【su -】切换身份成root
>
>离开su -则使用exit回到之前的身份

【ls -al】命令

选项【-al】: 列出所有的文件以及其详细的权限与属性（包含隐藏文件）

```shell
[dmtsai@study ~]$ su - # 先来切换一下身份看看
Password:
Last login: Tue Jun 2 19:32:31 CST 2015 on tty2
[root@study ~]# ls -al
total 48
dr-xr-x---. 5 root root 4096 May 29 16:08 .
dr-xr-xr-x. 17 root root 4096 May 4 17:56 ..
-rw-------. 1 root root 1816 May 4 17:57 anaconda-ks.cfg
```

 ![](\images\20220320212750.png)

**第一个字符代表文件的类型**

* 当为[ d ]则是目录
* 当为[ - ]则是常规文件
* 若是[ l ]则表示为链接文件
* 若是[ b ]则表示为设备文件里面的可供储存的周边设备
* 若是[ c ]则表示为设备文件里面的串行端口设备，例如键盘、鼠标
* 若是[ s ]则为数据接口文件
* 若是[ p ]则为数据输送文件

**接下来的字符中，以三个为一组，且均为【rwx】的三个参数的组合。其中，[ r ]代表可读（read）、[ w ]代表可写（write）、[ x ]代表可执行（execute）。这三个权限的位置不会改变，如果没有权限，就会用[ - ]当占位符。**

* 第一组为文件拥有者可具备的权限
* 第二组为加入此用户组的帐号的权限
* 第三组为非本人且没有加入本用户组的其他帐号的权限

**第二栏表示有多少文件名链接到此节点（inode）**

**第三栏表示这个文件的拥有者账号**

**第四栏表示这个文件的所属用户组**

**第五栏为这个文件的容量大小，默认单位为Bytes**

**第六栏为这个文件的创建日期或者是最近的修改日期**

**第七栏为这个文件的文件名**

### 5.2.2 如何修改文件属性与权限

#### 修改所属用户组, chgrp

```shell
[root@study ~]# chgrp [-R] dirname/filename ...
选项与参数：
-R : 进行递归（recursive）修改，亦即连同子目录下的所有文件、目录都更新成为这个用户组之意。常常用在修改某一目录内所有的文件之情况。
范例：
[root@study ~]# chgrp users initial-setup-ks.cfg
[root@study ~]# ls -l
-rw-r--r--. 1 root users 1864 May 4 18:01 initial-setup-ks.cfg
```

#### 修改文件拥有者, chown

chown还可以顺便直接修改用户组的名称。

```shell
[root@study ~]# chown [-R] 帐号名称 文件或目录
[root@study ~]# chown [-R] 帐号名称:用户组名称 文件或目录
选项与参数：
-R : 进行递归（recursive）修改，亦即连同子目录下的所有文件都修改
范例：将 initial-setup-ks.cfg 的拥有者改为bin这个帐号：
[root@study ~]# chown bin initial-setup-ks.cfg
[root@study ~]# ls -l
-rw-r--r--. 1 bin users 1864 May 4 18:01 initial-setup-ks.cfg
范例：将 initial-setup-ks.cfg 的拥有者与群组改回为root：
[root@study ~]# chown root:root initial-setup-ks.cfg
[root@study ~]# ls -l
-rw-r--r--. 1 root root 1864 May 4 18:01 initial-setup-ks.cfg
```

#### 修改权限，cha\mod

##### 数字类型修改文件权限

Linux文件的基本权限有9个，分别是拥有者，用户组，其他人三种身份各自的read、write、execute。我们可以用数字来代表各个权限：

```shell
r:4
w:2
x:1
```

每种身份（owner、group、others）各自的三个权限（r、w、x）数字是需要累加的，例如当权限为： [-rwxrwx---] 数字则是：

```text
owner = rwx = 4+2+1 = 7
group = rwx = 4+2+1 = 7
others= --- = 0+0+0 = 0
```

所以该文件的权限数字就是770

```shell
[root@study ~]# chmod [-R] xyz 文件或目录
选项与参数：
xyz : 就是刚刚提到的数字类型的权限属性，为 rwx 属性数值的相加。
-R : 进行递归（recursive）修改，亦即连同次目录下的所有文件都会修改

[root@study ~]# ls -al .bashrc
-rw-r--r--. 1 root root 176 Dec 29 2013 .bashrc
[root@study ~]# chmod 777 .bashrc
[root@study ~]# ls -al .bashrc
-rwxrwxrwx. 1 root root 176 Dec 29 2013 .bashrc
```

##### 符号类型修改文件权限

借由u, g, o来代表三种身份(user、group、others)，a则代表all亦即全部的身份。

* 设置一个文件的权限成为“-rwxr-xr-x”

  ```shell
  [root@study ~]# chmod u=rwx,go=rx .bashrc
  # 注意！那个 u=rwx,go=rx 是连在一起的，中间并没有任何空格
  [root@study ~]# ls -al .bashrc
  -rwxr-xr-x. 1 root root 176 Dec 29 2013 .bashrc
  ```

* 如果我不知道原先的文件属性，而我只想要增加.bashrc这个文件的每个人均可写入的权限

  ```shell
  [root@study ~]# ls -al .bashrc
  -rwxr-xr-x. 1 root root 176 Dec 29 2013 .bashrc
  [root@study ~]# chmod a+w .bashrc
  [root@study ~]# ls -al .bashrc
  -rwxrwxrwx. 1 root root 176 Dec 29 2013 .bashrc
  ```

* 拿掉全部人的可执行权限

  ```shell
  [root@study ~]# chmod a-x .bashrc
  [root@study ~]# ls -al .bashrc
  -rw-rw-rw-. 1 root root 176 Dec 29 2013 .bashrc
  ```

### 5.2.3 目录与文件的权限意义

#### 权限对文件的意义

文件是实际含有数据的地方。

* r：可读取此文件的实际内容，如读取文本文件的文字内容等

* w：可以编辑、新增或者是修改该文件的内容（但不含删除该文件）

* x：该文件具有可以被系统执行的权限

  在Windows下面一个文件是否具有执行的能力是借由“ 扩展名 ”来判断的， 例如：.exe, .bat, .com 等等，但是在Linux下面，我们的文件是否能被执行，则是借由是否具有“x”这个权限来决定，跟文件名是没有绝对的关系的。

对于文件的rwx来说，主要是针对文件的内容而言。

#### 权限对目录的意义

目录主要的内容在记录文件名列表

* r：表示具有读取目录结构列表的权限，表示你可以查询该目录下的文件名数据，即可以利用【ls】这个命令将目录的内容列表显示出来。
* w：表示具有改动该目录结构列表的权限
  * 建立新的文件与目录
  * 删除已存在的文件与目录
  * 将已存在的文件或目录进行更名
  * 移动该目录内的文件、目录位置
* x：表示用户能否进入该目录称为工作目录，所谓的工作目录就是你目前所在的目录，即表示用户能否通过【cd】命令变换到该目录。

### 5.2.4 Linux文件种类与扩展名

#### 文件种类

* 常规文件

  * 纯文本文件（ASCII）

    内容为我们人类可以直接读到的数据。可以执行`cat`就可以看到该文件的内容。

  * 二进制文件（binary）

    Linux当中的可执行文件。

  * 数据文件（data）

    比如我们的Linux在用户登陆时，都会将登录的数据记录在 /var/log/wtmp这个文件内，该文件是一个数据文件，他能够通过`last`这个命令读出来。但是使用cat时，会读出乱码。

* 目录

* 链接文件

* 设备与设备文件：与系统周边及存储相关的一些文件

  * 区块设备文件

    就是一些储存数据， 以提供系统随机存取的接口设备，举例来说，硬盘和软盘等就是

  * 字符设备文件

    一些串行端口的接口设备，例如鼠标、键盘等。

* 数据接口文件（sockets）：通常被用在网络上的数据交换。

* 数据输送文件（FIFO，pipe）：主要目的是解决多个程序同时读写一个文件所造成的错误问题。

#### Linux文件扩展名

基本上，Linux的文件是没有所谓的“扩展名”的，一个Linux文件能不能被执行，与它的第一栏的十个属性有关， 与文件名根本一点关系也没有。只要你的权限当中具有x的话，即代表这个文件据具有可以被执行的能力。 但是x仅仅代表这个文件具有可执行的能力，但是能不能执行成功，就得要看该文件的内容。

虽然如此，通常我们还是会以适当的扩展名来表示该文件是什么种类的：

* *.sh

  脚本或批处理文件

* Z, .tar, .tar.gz, .zip, *.tgz

  经过打包的压缩文件

* .html, .php

  网页相关的文件

#### Linux文件名长度的限制

单一文件或目录的最大容许文件名为 255字节，以一个 ASCII 英文占用一个字节来说，则大约可达 255 个字符长度。若是以每个汉字占用2字节来说， 最大文件名就是大约在 128 个汉字之间

## 5.3 Linux目录配置

### 5.3.1 Linux目录配置的依据——FHS（Filesystem Hierarchy Standard）

FHS将目录定义为四种交互作用的形态：

* 可分享：可以分享给其他系统挂载使用的目录，所以包括可执行文件与用户的邮件等数据， 是能够分享给网络上其他主机挂载用的目录
* 不可分性：自己机器上面运行的设备文件或者是与程序有关的socket文件等， 由于仅与自身机器有关，所以当然就不适合分享给其他主机
* 不变：有些数据是不会经常变动的。例如函数库、文件说明、系统管理员所管理的主机服务配置文件等等
* 可变动：经常修改的数据，例如日志文件、一般用户可自行接收的新闻组等

 ![image-20220322211008291](\images\image-20220322211008291.png)

FHS针对目录树架构仅定义出三层目录下面应该放置什么数据：

* / （root, 根目录）：与启动系统有关
* /usr （unix software resource）：与软件安装/执行有关
* /var （variable）：与系统运行过程有关

#### 根目录 （/） 的意义与内容

根目录是整个系统最重要的一个目录，因为不但所有的目录都是由根目录衍生出来的，同时根目录也与启动/还原/系统修复等操作有关。

FHS标准建议：根目录（/）所在分区应该越小越好， 且应用程序所安装的软件最好不要与根目录放在同一个分区内，保持根目录越小越好。 如此不但性能较佳，根目录所在的文件系统也较不容易发生问题。

#### /usr 的意义与内容

/usr里面放置的数据属于可分享的与不可变动的。

usr是Unix Software Resource的缩写， 也就是“Unix操作系统软件资源”所放置的目录，而不是使用者的数据。

FHS建议所有软件开发者，应该将他们的数据合理地分别放置到这个目录下的子目录，而不要自行建立该软件自己独立的目录。

#### /var 的意义与内容

/var目录主要针对经常性变动的文件，包括缓存（cache）、日志文件（log file）以及某些软件运行所产生的文件， 包括程序文件（lock file, run file），或者例如
MySQL数据库的文件等。

### 5.3.2 目录树（directory tree）

#### 目录树的特性

* 目录树的起始点为根目录（/, root）

* 每一个目录不止能使用本地分区的文件系统，也可以使用网络上的文件系统，比如用NFS服务挂载某特定目录。

* 每一个文件在此目录树中的文件名（包含完整路径）都是独一无二的

### 绝对路径与相对路径

* 绝对路径：由根目录（/）开始写起的文件名或目录名称， 例如 /home/dmtsai/.bashrc
* 相对路径：相对于目前路径的文件名写法。 例如 ./home/dmtsai 或 ../../home/dmtsai/ 等。反正开头不是 / 就属于相对路径的写法
  * **.**    代表当前的目录，也可以使用 ./ 来表示
  * **..**   代表上一层目录，也可以使用 ../ 来表示

# 第六章 Linux文件与目录管理

## 6.1 目录与路径

### 6.1.2 目录的相关操作

#### 特殊的目录

```
.				代表此层目录
..				代表上一层目录
-				代表前一个工作目录
~				代表“目前使用者身份”所在的主文件夹
~account 		代表 account 这个使用者的主文件夹（account是个帐号名称）
```

#### 常见的处理目录的命令

* **cd （change directory, 切换目录）**

  ```shell
  [dmtsai@study ~]$ su - # 先切换身份成为 root 看看。
  [root@study ~]# cd [相对路径或绝对路径]
  # 最重要的就是目录的绝对路径与相对路径，还有一些特殊目录的符号。
  [root@study ~]# cd ~dmtsai
  # 代表去到 dmtsai 这个使用者的家目录，亦即 /home/dmtsai
  [root@study dmtsai]# cd ~
  # 表示回到自己的家目录，亦即是 /root 这个目录
  [root@study ~]# cd
  # 没有加上任何路径，也还是代表回到自己家目录的意思喔，即等价于 cd ~
  [root@study ~]# cd ..
  # 表示去到目前的上层目录，亦即是 /root 的上层目录的意思
  [root@study /]# cd -
  # 表示回到刚刚的那个目录，也就是 /root
  [root@study ~]# cd /var/spool/mail
  # 这个就是绝对路径的写法。直接指定要去的完整路径名称
  [root@study mail]# cd ../postfix
  # 这个是相对路径的写法，我们由/var/spool/mail 去到/var/spool/postfix 就这样写。
  ```

* **pwd （Print Working Directory，显示目前所在的目录）**

  ```shell
  [root@study ~]# pwd [-P]
  选项与参数：
  -P ：显示出真正的路径，而非使用链接 （link） 路径。
  
  范例：单纯显示出目前的工作目录：
  [root@study ~]# pwd
  /root <== 显示出目录
  
  范例：显示出实际的工作目录，而非链接文件本身的目录名而已
  [root@study ~]# cd /var/mail <==注意，/var/mail是一个链接文件
  [root@study mail]# pwd
  /var/mail <==列出目前的工作目录
  [root@study mail]# pwd -P
  /var/spool/mail <==怎么回事？有没有加 -P 差很多
  
  [root@study mail]# ls -ld /var/mail
  lrwxrwxrwx. 1 root root 10 May 4 17:51 /var/mail -> spool/mail
  # 看到这里应该知道为啥了吧？因为 /var/mail 是链接文件，链接到 /var/spool/mail
  # 所以，加上 pwd -P 的选项后，不会显示链接文件的路径，而是显示正确的完整路径。
  ```

* **mkdir （创建新目录）**

  ```shell
  [root@study ~]# mkdir [-mp] 目录名称
  选项与参数：
  -m ：设置文件的权限.直接设置，不使用默认权限 （umask）
  -p ：帮助你直接将所需要的目录（包含上层目录）递归创建
  
  范例：请到/tmp下面尝试创建数个新目录看看：
  [root@study ~]# cd /tmp
  [root@study tmp]# mkdir test <==建立一名为 test 的新目录
  
  [root@study tmp]# mkdir test1/test2/test3/test4
  mkdir: cannot create directory ‘test1/test2/test3/test4’: No such file or directory
  # 话说，系统告诉我们，不可能建立这个目录，就是没有目录才要建立的，见鬼嘛
  [root@study tmp]# mkdir -p test1/test2/test3/test4
  # 原来是要建 test4 上层没先建 test3 的原因，加了这个 -p 的选项，可以自行帮你建立多层目录。
  
  范例：创建权限为rwx--x--x的目录
  [root@study tmp]# mkdir -m 711 test2
  [root@study tmp]# ls -ld test*
  drwxr-xr-x. 2 root root 6 Jun 4 19:03 test
  drwxr-xr-x. 3 root root 18 Jun 4 19:04 test1
  drwx--x--x. 2 root root 6 Jun 4 19:05 test2
  # 仔细看上面的权限部分，如果没有加上 -m 来强制设置属性，系统会使用默认属性。
  # 那么你的默认属性是什么？这要通过下面介绍的 [umask]才能了解。
  ```

* **rmdir （删除“空”的目录）**

  ```shell
  [root@study ~]# rmdir [-p] 目录名称
  选项与参数：
  -p ：连同上层的“空的”目录也一起删除
  
  范例：将于mkdir范例中创建的目录（/tmp下面）删除掉
  [root@study tmp]# ls -ld test* <==看看有多少目录存在？
  drwxr-xr-x. 2 root root 6 Jun 4 19:03 test
  drwxr-xr-x. 3 root root 18 Jun 4 19:04 test1
  drwx--x--x. 2 root root 6 Jun 4 19:05 test2
  [root@study tmp]# rmdir test <==可直接删除掉，没问题
  [root@study tmp]# rmdir test1 <==因为尚有内容，所以无法删除
  rmdir: failed to remove ‘test1’: Directory not empty
  [root@study tmp]# rmdir -p test1/test2/test3/test4
  [root@study tmp]# ls -ld test* <==您看看，下面的输出中test与test1不见了！
  drwx--x--x. 2 root root 6 Jun 4 19:05 test2
  # 使用 -p 这个选项，立刻可将 test1/test2/test3/test4 一次删除
  # 不过要注意，这个 rmdir 仅能“删除空目录”——被删除的目录里面必定不能存在其他的目录或文件
  ```

  如果要将所有目录下的东西都删除，就必须使用【rm -r test】。

### 6.1.3 关于执行文件路径的变量：$PATH

查看文件属性的命令 ls 完整文件名为：/bin/ls（这是绝对路径），但是我们可以在任何地方执行/bin/ls这个命令。这是因为当我们在执行一个命令的时候，举例来说 ls 好了，系统会依照PATH的设置去每个PATH定义的目录下查找文件名为ls的可执行文件。

PATH（一定是大写）这个变量的内容是由一堆目录所组成的，每个目录中间用冒号（:）来隔开， 每个目录是有顺序之分的。

可以执行【echo $PATH】来查看PATH的内容，echo有“显示、打印”的意思，而\$表示后面接的是变量

```shell
范例：用root的身份列出查找的路径是什么？
[root@study ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin

范例：用dmtsai的身份列出查找的路径是什么？
[root@study ~]# exit # 由之前的 su - 离开，变回原本的帐号
[dmtsai@study ~]$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/dmtsai/.local/bin:/home/dmtsai/bin
# 记不记得我们前一章说过，目前 /bin 是链接到 /usr/bin 当中
```

* 不同身份用户默认的PATH不同，默认能够随意执行的命令也不同（如root与dmtsai）

* PATH是可以修改的

  例如我们将“/root”加入到PATH中

  ```shell
  [root@study ~]# PATH="${PATH}:/root"
  ```

* 使用绝对路径或相对路径直接指定某个命令的文件名来执行，会比查找PATH来的正确

* 命令应该要放置到正确的目录下，执行才会比较方便

* 本目录（.）最好不要放到PATH当中

## 6.2 文件与目录管理

### 6.2.1 文件与目录的查看：ls

```shell
[root@study ~]# ls [-aAdfFhilnrRSt] 文件名或目录名称
[root@study ~]# ls [--color={never,auto,always}] 文件名或目录名称
[root@study ~]# ls [--full-time] 文件名或目录名称
选项与参数：
'-a ：全部的文件，连同隐藏文件（ 开头为 . 的文件） 一起列出来（常用）
-A ：全部的文件，连同隐藏文件，但不包括 . 与 .. 这两个目录
'-d ：仅列出目录本身，而不是列出目录内的文件数据（常用）
-f ：直接列出结果，而不进行排序 （ls 默认会以文件名排序）
-F ：根据文件、目录等信息，给予附加数据结构，例如：
	*代表可可执行文件； /代表目录； =代表 socket 文件； |代表 FIFO 文件；
-h ：将文件容量以人类较易读的方式（例如 GB, KB 等等）列出来；
-i ：列出 inode 号码，inode 的意义下一章将会介绍；
'-l ：详细信息显示，包含文件的属性与权限等等数据；（常用）
-n ：列出 UID 与 GID 而非使用者与用户组的名称 （UID与GID会在帐号管理提到）
-r ：将排序结果反向输出，例如：原本文件名由小到大，反向则为由大到小；
-R ：连同子目录内容一起列出来，等于该目录下的所有文件都会显示出来；
-S ：以文件容量大小排序，而不是用文件名排序；
-t ：依时间排序，而不是用文件名。
--color=never ：不要依据文件特性给予颜色显示；
--color=always ：显示颜色
--color=auto ：让系统自行依据设置来判断是否给予颜色
--full-time ：以完整时间模式 （包含年、月、日、时、分） 输出
--time={atime,ctime} ：输出 access 时间或改变权限属性时间 （ctime）而非内容修改时间 （modification time）
```

当你只有执行 ls 时，默认显示的只有：非隐藏文件的文件名、 以文件名进行排序及文件名代表的颜色显示如此而已。

### 6.2.2 复制、删除与移动： cp, rm, mv

* **cp （复制文件或目录）**

  ```shell
  [root@study ~]# cp [-adfilprsu] 源文件（source） 目标文件（destination）
  [root@study ~]# cp [options] source1 source2 source3 .... directory
  选项与参数：
  '-a ：相当于 -dr --preserve=all 的意思，至于 dr 请参考下列说明（常用）
  -d ：若源文件为链接文件的属性（link file），则复制链接文件属性而非文件本身；
  -f ：为强制（force）的意思，若目标文件已经存在且无法开启，则删除后再尝试一次；
  '-i ：若目标文件（destination）已经存在时，在覆盖时会先询问操作的进行（常用）
  -l ：进行硬链接（hard link）的链接文件建立，而非复制文件本身；
  '-p ：连同文件的属性（权限、用户、时间）一起复制过去，而非使用默认属性（备份常用）；
  '-r ：递归复制，用于目录的复制操作；（常用）
  -s ：复制成为符号链接文件 （symbolic link），亦即“快捷方式”文件；
  -u ：源文件比目标文件新才更新目标文件，或目标文件不存在的情况下才复制。常用于备份的工作当中。
  --preserve=all ：除了 -p 的权限相关参数外，还加入 SELinux 的属性, links, xattr 等也复制
  最后需要注意的，如果来源文件有两个以上，则最后一个目的文件一定要是“目录”才行
  ```

  在默认的条件中，cp的源文件与目标文件的权限是不同的，目标文件的拥有者通常会是命令操作者本身。

  因此我们在进行备份的时候，某些需要特备注意的特殊权限文件，例如密码文件，就要加上 -a 或是 -p 等可以完整复制文件权限的选项才行。

  范例：

  ```shell
  范例一：用root身份，将加目录下的 .bashrc 复制到 /tmp 下，并更名为 bashrc
  [root@study ~]# cp ~/.bashrc /tmp/bashrc
  [root@study ~]# cp -i ~/.bashrc /tmp/bashrc
  cp: overwrite `/tmp/bashrc'? n <==n不覆盖，y为覆盖
  # 重复作两次操作，由于 /tmp 下面已经存在 bashrc 了，加上 -i 选项后，
  # 则在覆盖前会询问使用者是否确定，可以按下 n 或者 y 来二次确认
  
  范例二：变换目录到/tmp，并将/var/log/wtmp复制到/tmp且观察属性：
  [root@study ~]# cd /tmp
  [root@study tmp]# cp /var/log/wtmp . <==想要复制到目前的目录，最后的 . 不要忘
  [root@study tmp]# ls -l /var/log/wtmp wtmp
  -rw-rw-r--. 1 root utmp 28416 Jun 11 18:56 /var/log/wtmp
  -rw-r--r--. 1 root root 28416 Jun 11 19:01 wtmp
  # 注意上面的特殊字体，在不加任何选项的情况下，文件的某些属性/权限会改变；
  # 这是个很重要的特性,要注意,还有，连文件创建的时间也不一样了
  # 那如果你想要将文件的所有特性都一起复制过来该怎办？可以加上 -a ,如下所示：
  [root@study tmp]# cp -a /var/log/wtmp wtmp_2
  [root@study tmp]# ls -l /var/log/wtmp wtmp_2
  -rw-rw-r--. 1 root utmp 28416 Jun 11 18:56 /var/log/wtmp
  -rw-rw-r--. 1 root utmp 28416 Jun 11 18:56 wtmp_2
  # 整个数据特性完全一模一样,这就是 -a 的特性
  
  范例三：复制 /etc/ 这个目录下的所有内容到 /tmp 下面
  [root@study tmp]# cp /etc/ /tmp
  cp: omitting directory `/etc' <== 如果是目录则不能直接复制，要加上 -r 的选项
  [root@study tmp]# cp -r /etc/ /tmp
  # 还是要再次的强调, -r 是可以复制目录，但是，文件与目录的权限可能会被改变
  # 所以，也可以利用[cp -a /etc /tmp]来执行命令，尤其是在备份的情况下。
  
  范例四：将范例一复制的 bashrc 创建一个符号链接文件 （symbolic link）
  [root@study tmp]# ls -l bashrc
  -rw-r--r--. 1 root root 176 Jun 11 19:01 bashrc <==先观察一下文件情况
  [root@study tmp]# cp -s bashrc bashrc_slink
  [root@study tmp]# cp -l bashrc bashrc_hlink
  [root@study tmp]# ls -l bashrc*
  -rw-r--r--. 2 root root 176 Jun 11 19:01 bashrc <==与原始文件不太一样了
  -rw-r--r--. 2 root root 176 Jun 11 19:01 bashrc_hlink
  lrwxrwxrwx. 1 root root 6 Jun 11 19:06 bashrc_slink -> bashrc
  # bashrc_slink 是一个“快捷方式”，这个快捷方式会链接到bashrc。所以你会看到文件名右侧会有个指向（->）的符号
  # bashrc_hlink文件与bashrc的属性与权限完全一模一样，与尚未进行链接前的差异则是第二栏的link数由1变成2了
  
  范例五：若 ~/.bashrc 比 /tmp/bashrc 新才复制过来
  [root@study tmp]# cp -u ~/.bashrc /tmp/bashrc
  # 这个 -u 的特性，是在目标文件与来源文件有差异时，才会复制的。
  # 所以，比较常被用于“备份”的工作当中
  
  范例六：将范例四造成的 bashrc_slink 复制成为 bashrc_slink_1 与bashrc_slink_2
  [root@study tmp]# cp bashrc_slink bashrc_slink_1
  [root@study tmp]# cp -d bashrc_slink bashrc_slink_2
  [root@study tmp]# ls -l bashrc bashrc_slink*
  -rw-r--r--. 2 root root 176 Jun 11 19:01 bashrc
  lrwxrwxrwx. 1 root root 6 Jun 11 19:06 bashrc_slink -> bashrc
  -rw-r--r--. 1 root root 176 Jun 11 19:09 bashrc_slink_1 <==与原始文件相同
  lrwxrwxrwx. 1 root root 6 Jun 11 19:10 bashrc_slink_2 -> bashrc <==是链接文件！
  # 原本复制的是链接文件，但是却将链接文件的实际文件复制过来了
  # 也就是说，如果没有加上任何选项时，cp复制的是原始文件，而非链接文件的属性。
  # 若要复制链接文件的属性，就得要使用 -d 的选项了，如 bashrc_slink_2 所示。
  
  范例七：将家目录的 .bashrc 及 .bash_history 复制到 /tmp 下面
  [root@study tmp]# cp ~/.bashrc ~/.bash_history /tmp
  # 可以将多个数据一次复制到同一个目录，最后面一定是目录
  ```

* **rm （删除文件或目录）**

  ```shell
  [root@study ~]# rm [-fir] 文件或目录
  选项与参数：
  -f ：就是 force 的意思，忽略不存在的文件，不会出现警告讯息；
  -i ：互动模式，在删除前会询问使用者是否操作
  -r ：递归删除 最常用于目录的删除，这是非常危险的选项
  
  范例一：将刚刚在 cp 的范例中创建的 bashrc 删除掉！
  [root@study ~]# cd /tmp
  [root@study tmp]# rm -i bashrc
  rm: remove regular file `bashrc'? y
  # 如果加上 -i 的选项就会主动询问，避免你删除到错误的文件名
  
  范例二：通过通配符*的帮忙，将/tmp下面开头为bashrc的文件名通通删除：
  [root@study tmp]# rm -i bashrc*
  # 注意那个星号，代表的是 0 到无穷多个任意字符，很好用的东西
  
  范例三：将 cp 范例中所创建的 /tmp/etc/ 这个目录删除掉！
  [root@study tmp]# rmdir /tmp/etc
  rmdir: failed to remove '/tmp/etc': Directory not empty <== 删不掉，因为这不是空的目录
  [root@study tmp]# rm -r /tmp/etc
  rm: descend into directory `/tmp/etc'? y
  rm: remove regular file `/tmp/etc/fstab'? y
  rm: remove regular empty file `/tmp/etc/crypttab'? ^C <== 按下 [crtl]+c 中断
  .....（中间省略）.....
  # 因为身份是 root ，默认已经加入了 -i 的选项，所以你要一直按 y 才会删除！
  # 如果不想要继续按 y ，可以按下“ [ctrl]-c ”来终止 rm 的工作。
  # 这是一种保护的操作，如果确定要删除掉此目录而不要询问，可以这样做：
  [root@study tmp]# \rm -r /tmp/etc
  # 在指令前加上反斜线，可以忽略掉 alias 的指定选项，至于 alias 我们在bash再谈。
  
  范例四：删除一个带有 - 开头的文件
  [root@study tmp]# touch ./-aaa- <== touch这个命令可以建立空文件
  [root@study tmp]# ls -l
  -rw-r--r--. 1 root root 0 Jun 11 19:22 -aaa- <==文件大小为0，所以是空文件
  [root@study tmp]# rm -aaa-
  rm:invalid option -- 'a' <== 因为 "-" 是选项嘛，所以系统误判了
  Try 'rm ./-aaa-' to remove the file `-aaa-'. <== 新的 bash 有给建议的, 即在前面加上本目录./
  Try 'rm --help' for more information.
  [root@study tmp]# rm ./-aaa-
  ```

* **mv （移动文件与目录，或重命名）**

  ```shell
  [root@study ~]# mv [-fiu] source destination
  [root@study ~]# mv [options] source1 source2 source3 .... directory
  选项与参数：
  -f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖；
  -i ：若目标文件 （destination） 已经存在时，就会询问是否覆盖
  -u ：若目标文件已经存在，且 source 比较新，才会更新 （update）
  
  范例一：复制一文件，创建一目录，将文件移动到目录中
  [root@study ~]# cd /tmp
  [root@study tmp]# cp ~/.bashrc bashrc
  [root@study tmp]# mkdir mvtest
  [root@study tmp]# mv bashrc mvtest
  # 将某个文件移动到某个目录去，就是这样做
  
  范例二：将刚刚的目录名称更名为 mvtest2
  [root@study tmp]# mv mvtest mvtest2 <== 这样就更名了。
  # 其实在 Linux 下面还有个有趣的指令，名称为 rename ，
  # 该指令专职进行多个文件名的同时重命名，并非针对单一文件名修改，与mv不同。请man rename。
  
  范例三：再创建两个文件，再全部移动到 /tmp/mvtest2 当中
  [root@study tmp]# cp ~/.bashrc bashrc1
  [root@study tmp]# cp ~/.bashrc bashrc2
  [root@study tmp]# mv bashrc1 bashrc2 mvtest2
  # 注意到这边，如果有多个来源文件或目录，则最后一个目标文件一定是目录
  # 意思是说，将所有的文件移动到该目录的意思！
  ```

### 6.2.3 获取路径的文件名与目录名称

```shell
[root@study ~]# basename /etc/sysconfig/network
network <== 很简单,就取得最后的文件名
[root@study ~]# dirname /etc/sysconfig/network
/etc/sysconfig <== 取得的变成目录名
```

## 6.3 文件内容查看

### 6.3.1 直接查看文件内容

* **cat (concatenate)**

  ```shell
  [root@study ~]# cat [-AbEnTv]
  选项与参数：
  -A ：相当于 -vET 的整合选项，可列出一些特殊字符而不是空白而已；
  -b ：列出行号，仅针对非空白行做行号显示，空白行不标行号;
  -E ：将结尾的断行字符 $ 显示出来；
  '-n ：打印出行号，连同空白行也会有行号，与 -b 的选项不同；
  -T ：将 [tab] 按键以 ^I 显示出来；
  -v ：列出一些看不出来的特殊字符;
  
  范例一：查看 /etc/issue 这个文件的内容
  [root@study ~]# cat /etc/issue
  \S
  Kernel \r on an \m
  
  范例二：承上题，如果还要打印行号？
  [root@study ~]# cat -n /etc/issue
  1 \S
  2 Kernel \r on an \m
  3
  # 所以这个文件有三行，看到了吧！可以列出行号。这对于大文件要找某个特定的行时，有点用处。
  # 如果不想要显示空白行的行号，可以使用“cat -b /etc/issue”
  
  范例三：将 /etc/man_db.conf 的内容完整的显示出来（包含特殊字符）
  [root@study ~]# cat -A /etc/man_db.conf
  # $
  ....（中间省略）....
  MANPATH_MAP^I/bin^I^I^I/usr/share/man$
  MANPATH_MAP^I/usr/bin^I^I/usr/share/man$
  MANPATH_MAP^I/sbin^I^I^I/usr/share/man$
  MANPATH_MAP^I/usr/sbin^I^I/usr/share/man$
  .....（下面省略）.....
  # 基本上，在一般的环境中，使用 [tab] 与空格键的效果差不多，都是一堆空白。我们无法知道两者的差别。
  # 此时使用 cat -A 就能够发现那些空白的地方是啥鬼东西了，[tab]会以 ^I 表示，
  # 换行符则是以 $ 表示，所以你可以发现每一行后面都是 $ 。
  # 不过换行符在Windows/Linux则不太相同，Windows的换行符是 ^M$。
  ```

* **tac （反向列示）**

  ```shell
  [root@study ~]# tac /etc/issue
  Kernel \r on an \m
  \S
  # 与刚刚上面的范例一比较，是由最后一行先显示。
  ```

* **nl （添加行号打印）**

  ```shell
  [root@study ~]# nl [-bnw] 文件
  选项与参数：
  -b ：	指定行号指定的方式，主要有两种：
  		-b a ：表示不论是否为空行，也同样列出行号（类似 cat -n）；
  		-b t ：如果有空行，空的那一行不要列出行号（默认值）；
  -n ：	列出行号表示的方法，主要有三种：
  		-n ln ：行号在屏幕的最左方显示；
  		-n rn ：行号在自己字段的最右方显示，且不加 0 ；
  		-n rz ：行号在自己字段的最右方显示，且加 0 ；
  -w ：行号字段的占用的字符数。
  
  范例一：用 nl 列出 /etc/issue 的内容
  [root@study ~]# nl /etc/issue
  1 \S
  2 Kernel \r on an \m
  # 注意看，这个文件其实有三行，第三行为空白（没有任何字符），
  # 因为他是空白行，所以 nl 不会加上行号。如果确定要加上行号，可以这样做：
  [root@study ~]# nl -b a /etc/issue
  1 \S
  2 Kernel \r on an \m
  3
  # 那么如果要让行号前面自动补上 0 呢？
  [root@study ~]# nl -b a -n rz /etc/issue
  000001 \S
  000002 Kernel \r on an \m
  000003
  # 自动在自己字段的地方补上 0 了，默认字段是六位数，如果想要改成 3 位数？
  [root@study ~]# nl -b a -n rz -w 3 /etc/issue
  001 \S
  002 Kernel \r on an \m
  003
  # 变成仅有 3 位数
  ```

### 6.3.2 可翻页查看

前面提到的 nl 与 cat, tac 等等，都是一次性的将数据一口气显示到屏幕上面。

* **more （一页一页翻动）**

  ```shell
  [root@study ~]# more /etc/man_db.conf
  #
  #
  This file is used by the man-db package to configure the man and cat paths.
  # It is also used to provide a manpath for those without one by examining
  # their PATH environment variable. For details see the manpath（5） man page.
  # .....（中间省略）.....
  --More--（28%） <== 重点在这一行,你的光标也会在这里等待你的命令
  ```

  最后一行会显示出目前显示的百分比，而且还可以在最后一行输入一些有用的命令：

  * 空白键 （space）：代表向下翻一页
  * Enter ：代表向下翻一行
  * /字符串 ：代表在这个显示的内容当中，向下查找“字符串”这个关键词，而重复查找同一字符串，可以直接按下n即可
  * :f ：立刻显示出文件名以及目前显示的行数
  * q ：代表立刻离开 more ，不再显示该文件内容
  * b 或 [ctrl]-b ：代表往回翻页，不过这动作只对文件有用，对管道无用

* **less （一页一页翻动）**

  ```shell
  [root@study ~]# less /etc/man_db.conf
  #
  #
  This file is used by the man-db package to configure the man and cat paths.
  # It is also used to provide a manpath for those without one by examining
  # their PATH environment variable. For details see the manpath（5） man page.
  # .....（中间省略）.....
  : <== 这里可以等待你输入命令
  ```

  在 more 的时候，我们并没有办法向前面翻， 只能往后面看，但若使用了 less 时，就可以使用 [pageup] [pagedown] 等按键的功能来往前往后翻看文件。

  除此之外，less里面拥有更多的查找功能，不止可以向下查找，也可以向上查找

  * 空格键 ：向下翻动一页
  * [pagedown]：向下翻动一页
  * [pageup] ：向上翻动一页
  * /字符串 ：向下查找“字符串”的功能
  * ?字符串 ：向上查找“字符串”的功能
  * n ：重复前一个查找（与 / 或 ? 有关）
  * N ：反向的重复前一个查找 （与 / 或 ? 有关）
  * g ：前进到这个数据的第一行
  * G ：前进到这个数据的最后一行 （注意大小写）
  * q ：离开 less 这个程序

### 6.3.3 数据截取

* **head （取出前面几行）**

  ```shell
  [root@study ~]# head [-n number] 文件
  选项与参数：
  -n ：后面接数字，代表显示几行的意思
  
  [root@study ~]# head /etc/man_db.conf
  # 默认的情况中，显示前面十行,若要显示前 20 行，就得要这样：
  [root@study ~]# head -n 20 /etc/man_db.conf
  
  范例：如果后面100行的数据都不打印，只打印/etc/man_db.conf的前面几行，该如何是好？
  [root@study ~]# head -n -100 /etc/man_db.conf
  # -n后面如果接的是负数，如上面的-100，代表列出前面所有行，但不包括后面100行
  ```

* **tail （取出后面几行）**

  ```
  [root@study ~]# tail [-n number] 文件
  选项与参数：
  -n ：后面接数字，代表显示几行的意思
  -f ：表示持续刷新显示文件中的内容，要等到按下[ctrl]-c才会结束
  
  [root@study ~]# tail /etc/man_db.conf
  # 默认的情况中，显示最后的十行。若要显示最后的 20 行，就得要这样：
  [root@study ~]# tail -n 20 /etc/man_db.conf
  
  范例一：如果不知道/etc/man_db.conf有几行，却只想列出100行以后的数据时？
  [root@study ~]# tail -n +100 /etc/man_db.conf
  # +100表示该文件从100行以后都会被列出来
  
  范例二：持续检测/var/log/messages的内容
  [root@study ~]# tail -f /var/log/messages
  	<==要等到输入[crtl]-c之后才会结束执行tail这个命令
  ```

假如我想要显示 /etc/man_db.conf 的第 11 到第 20 行？

```shell
head -n 20 /etc/man_db.conf | tail -n 10
# 这两个命令中间有个管道 （|） 的符号存在，这个管线的意思是：“前面的命令所输出的信息，通过管道交由后续的命令继续使用。
```

### 6.3.4 非纯文本文件：od

上面提到的都是在查看纯文本文件的内容，由于执行文件通常是二进制文件，使用上面的命令会产生乱码，可以利用od这个命令来读取执行文件

```shell
[root@study ~]# od [-t TYPE] 文件
选项或参数：
-t ：	后面可以接各种“类型 （TYPE）”的输出，例如：
		a ：利用默认的字符来输出；
		c ：使用 ASCII 字符来输出
		d[size] ：利用十进制（decimal）来输出数据，每个整数占用 size Bytes ；
		f[size] ：利用浮点数值（floating）来输出数据，每个数占用 size Bytes ；
		o[size] ：利用八进位（octal）来输出数据，每个整数占用 size Bytes ；
		x[size] ：利用十六进制（hexadecimal）来输出数据，每个整数占用 size Bytes ；
		
范例一：请将/usr/bin/passwd的内容使用ASCII方式来展现！
[root@study ~]# od -t c /usr/bin/passwd
0000000 177 E L F 002 001 001 \0 \0 \0 \0 \0 \0 \0 \0 \0
0000020 003 \0 > \0 001 \0 \0 \0 364 3 \0 \0 \0 \0 \0 \0
0000040 @ \0 \0 \0 \0 \0 \0 \0 x e \0 \0 \0 \0 \0 \0
0000060 \0 \0 \0 \0 @ \0 8 \0 \t \0 @ \0 035 \0 034 \0
0000100 006 \0 \0 \0 005 \0 \0 \0 @ \0 \0 \0 \0 \0 \0 \0
.....（后面省略）....
# 最左边第一列是以 8 进位来表示Bytes数。
# 以上面范例来说，第二行0000020代表开头是第 16 个 byes （2x8） 的内容之意。

范例二：请将/etc/issue这个文件的内容以8进位列出储存值与ASCII的对照表
[root@study ~]# od -t oCc /etc/issue
0000000 134 123 012 113 145 162 156 145 154 040 134 162 040 157 156 040
		\ 	S 	\n 	K 	e 	r 	n 	e 	l 	\ 	r 	o 	n
0000020 141 156 040 134 155 012 012
		a n \ m \n \n
0000027
# 如上所示，可以发现每个字符可以对应到的数值是什么
# 例如 S 对应的记录数值为 123 ，转成十进制：1x8^2+2x8+3=83。
```

我不想找 google，想要立刻找到 password 这几个字的 ASCII 对照，该如何通过 od来判断？

```shell
echo password | od -t oCc
# echo 可以在屏幕上面显示任何信息，而这个信息不由屏幕输出，而是传给 od 去继续处理
```

### 6.3.5 修改文件时间或创建新文件： touch

Linux中文件的主要时间参数：

* **修改时间（modification time，mtime）**

  当该文件的“内容数据”变更时，就会更新这个时间，内容数据指的是文件的内容，而不是文件的属性或权限

* **状态时间（status time，ctime）**

  当该文件的“状态 （status）”改变时，就会更新这个时间，举例来说，像是权限与属性被更改了，都会更新这个时间

* **读取时间（access time，atime）**

  当“该文件的内容被读取”时，就会更新这个读取时间。举例来说，我们使用 cat 去读取 /etc/man_db.conf ， 就会更新该文件的atime 了

```shell
[root@study ~]# date; ls -l /etc/man_db.conf ; ls -l --time=atime /etc/man_db.conf ; \
> ls -l --time=ctime /etc/man_db.conf # 这两行其实是同一行,用分号隔开
Tue Jun 16 00:43:17 CST 2015 # 目前的时间
-rw-r--r--. 1 root root 5171 Jun 10 2014 /etc/man_db.conf # 在 2014/06/10 建立的内容（mtime）
-rw-r--r--. 1 root root 5171 Jun 15 23:46 /etc/man_db.conf # 在 2015/06/15 读取过内容（atime）
-rw-r--r--. 1 root root 5171 May 4 17:54 /etc/man_db.conf # 在 2015/05/04 更新过状态（ctime）
```

**在默认的情况下，ls 显示出来的是该文件的 mtime ，也就是这个文件的内容上次被修改的时间。**

#### 使用【touch】修改文件的时间

```shell
[root@study ~]# touch [-acdmt] 文件
选项与参数：
-a ：仅修订 access time；
-m ：仅修改 mtime ；
-c ：仅修改文件的时间，若该文件不存在则不建立新文件；
-d ：后面可以接欲自定义的日期而不用目前的日期，也可以使用 --date="日期或时间"
-t ：后面可以接欲自定义的时间而不用目前的时间，格式为[YYYYMMDDhhmm]

范例一：新建一个空的文件并观察时间
[dmtsai@study ~]# cd /tmp
[dmtsai@study tmp]# touch testtouch
[dmtsai@study tmp]# ls -l testtouch
-rw-rw-r--. 1 dmtsai dmtsai 0 Jun 16 00:45 testtouch
# 注意到，这个文件的大小是 0 。在默认的状态下，如果 touch 后面有接文件，
# 则该文件的三个时间 （atime/ctime/mtime） 都会更新为目前的时间。若该文件不存在，
# 则会主动的创建一个新的空的文件！

范例二：将 ~/.bashrc 复制成为 bashrc，假设复制完全的属性，检查其日期
[dmtsai@study tmp]# cp -a ~/.bashrc bashrc
[dmtsai@study tmp]# date; ll bashrc; ll --time=atime bashrc; ll --time=ctime bashrc # [ll] 这个命令其实就是[ls -l]
Tue Jun 16 00:49:24 CST 2015 <==这是目前的时间
-rw-r--r--. 1 dmtsai dmtsai 231 Mar 6 06:06 bashrc <==这是 mtime
-rw-r--r--. 1 dmtsai dmtsai 231 Jun 15 23:44 bashrc <==这是 atime
-rw-r--r--. 1 dmtsai dmtsai 231 Jun 16 00:47 bashrc <==这是 ctime

范例三：修改案例二的 bashrc 文件，将日期调整为两天前
[dmtsai@study tmp]# touch -d "2 days ago" bashrc
[dmtsai@study tmp]# date; ll bashrc; ll --time=atime bashrc; ll --time=ctime bashrc
Tue Jun 16 00:51:52 CST 2015
-rw-r--r--. 1 dmtsai dmtsai 231 Jun 14 00:51 bashrc
-rw-r--r--. 1 dmtsai dmtsai 231 Jun 14 00:51 bashrc
-rw-r--r--. 1 dmtsai dmtsai 231 Jun 16 00:51 bashrc
# 跟上个范例比较看看，本来是 16 日变成 14 日了 （atime/mtime）,不过， ctime 并没有跟着改变

范例四：将上个范例的 bashrc 日期改为 2014/06/15 2:02
[dmtsai@study tmp]# touch -t 201406150202 bashrc
[dmtsai@study tmp]# date; ll bashrc; ll --time=atime bashrc; ll --time=ctime bashrc
Tue Jun 16 00:54:07 CST 2015
-rw-r--r--. 1 dmtsai dmtsai 231 Jun 15 2014 bashrc
-rw-r--r--. 1 dmtsai dmtsai 231 Jun 15 2014 bashrc
-rw-r--r--. 1 dmtsai dmtsai 231 Jun 16 00:54 bashrc
# 注意看看，日期在 atime 与 mtime 都改变了，但是 ctime 则是记录目前的时间
```

touch这个命令最常被使用的情况是：

* 建立一个空文件
* 将某个文件日期自定义为目前（mtime与atime）

## 6.4 文件与目录的默认权限与隐藏权限

### 6.4.1 文件默认权限：umask

当你建立一个新的文件或目录时，它的默认权限与umask有关，umask就是指定目前用户在建立文件或目录的时候的权限默认值。

* 若用户建立为文件，则默认没有可执行（ x ）权限，即只有 rw 这两个项目，也就是最大为 666 ，默认权限如下： -rw-rw-rw-
* 若用户建立为目录，则由于 x 与是否可以进入此目录有关，因此默认为所有权限均开放，亦即为 777 ，默认权限如下： drwxrwxrwx

umask的数字值得是该默认值需要减掉的权限。因为 r、w、x 分别是 4、2、1 ，所以，也就是说，当要拿掉能写的权限，就是输入 2 ；而如果要拿掉能读的权
限，也就是4；而要拿掉读与写的权限，也就是 6 。

使用umask和umask -S查看umask：

```shell
[root@study ~]# umask
0022 <==与一般权限有关的是后面三个数字！
# umask为022，所以 user 并没有被拿掉任何权限，不过 group 与 others 的权限被拿掉了 2 （也就是 w 这个权限）
[root@study ~]# umask -S
u=rwx,g=rx,o=rx
```

设置umask：直接在umask后面输入数字

```shell
[root@study ~]# umask 002
[root@study ~]# touch test3
[root@study ~]# mkdir test4
[root@study ~]# ll -d test[34] # 中括号 [ ] 代表中间有个指定的字符，而不是任意字符的意思
-rw-rw-r--. 1 root root 0 6月 16 01:12 test3
drwxrwxr-x. 2 root root 6 6月 16 01:12 test4
```

### 6.4.2 文件隐藏属性

要先强调的是，下面的chattr命令只能在ext2/ext3/ext4的 Linux 传统文件系统上面完整生效， 其他的文件系统可能就无法完整的支持这个命令了

* chattr （设置文件隐藏属性）

  ```shell
  [root@study ~]# chattr [+-=][ASacdistu] 文件或目录名称
  选项与参数：
  + ：增加某一个特殊参数，其他原本存在参数则不动。
  - ：移除某一个特殊参数，其他原本存在参数则不动。
  = ：直接设置参数，且仅有后面接的参数
  A ：当设置了 A 这个属性时，若你在存取此文件（或目录）时，他的存取时间 atime 将不会被修改，
     可避免 I/O 较慢的机器过度的读写磁盘。（目前建议使用文件系统挂载参数处理这个项目）
  S ：一般文件是非同步写入磁盘的，如果加上 S 这个属性时，当你进行任何文件的修改，该更动会“同步”写入磁盘中。
  'a ：当设置 a 之后，这个文件将只能增加数据，而不能删除也不能修改数据，只有root才能设置这属性
  c ：这个属性设置之后，将会自动的将此文件“压缩”，在读取的时候将会自动解压缩，
  	但是在储存的时候，将会先进行压缩后再储存（看来对于大文件似乎蛮有用的）
  d ：当 dump 程序被执行的时候，设置 d 属性将可使该文件（或目录）不会被 dump 备份
  'i ：这个 i 可就很厉害了，他可以让一个文件“不能被删除、改名、设置链接也无法写入或新增数据”
  	'对于系统安全性有相当大的助益，只有 root 能设置此属性
  s ：当文件设置了 s 属性时，如果这个文件被删除，他将会被完全的移除出这个硬盘空间，
  	所以如果误删了，完全无法恢复
  u ：与 s 相反的，当使用 u 来设置文件时，如果该文件被删除了，则数据内容其实还存在磁盘中，可以使用来恢复该文件
  注意1：属性设置常见的是 a 与 i 的设置值，而且很多设置值必须要是 root 才能设置
  注意2：xfs 文件系统仅支持 AadiS 而已
  
  范例：请尝试到/tmp下面创建文件，并加入 i 的参数，尝试删除看看。
  [root@study ~]# cd /tmp
  [root@study tmp]# touch attrtest <==创建一个空文件
  [root@study tmp]# chattr +i attrtest <==给予 i 的属性
  [root@study tmp]# rm attrtest <==尝试删除看看
  rm: remove regular empty file `attrtest'? y
  rm: cannot remove `attrtest': Operation not permitted
  # 看到了吗？连 root 也没有办法将这个文件删除，赶紧取消参数设置。
  范例：请将该文件的 i 属性取消！
  [root@study tmp]# chattr -i attrtest
  ```

  最重要的当属 +i 与 +a 这个属性了。+i 可以让一个文件无法被修改，对于需要强烈的系统安全的人来说， 真是相当的重要。
  此外，如果是 log file 这种的日志文件，就更需要 +a 这个可以增加但是不能修改旧数据与删除的参数。

* lsattr （显示文件隐藏属性）

  ```Shell
  [root@study ~]# lsattr [-adR] 文件或目录
  选项与参数：
  -a ：将隐藏文件的属性也显示出来；
  -d ：如果接的是目录，仅列出目录本身的属性而非目录内的文件名；
  -R ：连同子目录的数据也一并列出来。
  [root@study tmp]# chattr +aiS attrtest
  [root@study tmp]# lsattr attrtest
  --S-ia---------- attrtest
  ```

### 6.4.3 文件特殊权限： SUID, SGID, SBIT

* Set UID

  当 s 这个标志出现在文件拥有者的 x 权限上时，例如 /usr/bin/passwd 这个文件的权限状态：“-rwsr-xr-x”，此时就被称为 Set UID，简称为 SUID 的特殊权限。

  它具有如下功能：

  * SUID 权限仅对二进制程序（binary program）有效
  * 执行者对于该程序需要具有 x 的可执行权限
  * 本权限仅在执行该程序的过程中有效 （run-time）
  * 执行者将具有该程序拥有者 （owner） 的权限

  举例来说，Linux 系统中，所有帐号的密码都记录在 /etc/shadow 这个文件里面，这个文件仅有root可读且仅有root可以强制写入，但是一般使用者却可以修改自己的密码。明明 /etc/shadow 不能让一般用户去存取的，为什么还能够修改这个文件内的密码呢？ 这就是 SUID 的功能：

  * 一般用户对于 /usr/bin/passwd 这个程序来说是具有 x 权限的，表示其能执行passwd
  * passwd 的拥有者是 root 这个帐号
  * 一般用户执行 passwd 的过程中，会“暂时”获得 root 的权限
  * /etc/shadow 就可以被一般用户所执行的 passwd 所修改

  注意，SUID仅可用在二进制程序上，不能够用在shell脚本上面，**shell脚本只是将很多的二进制执行文件调用执行而已**。

  另外，SUID对于目录也是无效的。

* Set GID

  当 s 标志在用户组的 x 时则称为 Set GID, 简称为SGID 。

  与 SUID 不同的是，SGID 可以针对文件或目录来设置。

  * 如果是对文件来说， SGID 有如下的功能：
    * SGID 对二进制程序有用
    * 程序执行者对于该程序来说，需具备 x 的权限
    * 执行者在执行的过程中将会获得该程序用户组的支持
  * 如果是对目录来说， SGID 有如下的功能：
    * 用户若对于此目录具有 r 与 x 的权限时，该使用者能够进入此目录
    * 用户在此目录下的有效用户组（effective group）将会变成该目录的用户组
    * 用途：若用户在此目录下具有 w 的权限（可以新建文件），则使用者所创建的新文件，该新文件的用户组与此目录的用户组相同。

* Sticky Bit

  Sticky Bit, SBIT 目前只针对目录有效，对于文件已经没有效果了。SBIT 对于目录的作用是：

  * 当用户对于此目录具有 w, x 权限，即具有写入的权限
  * 当用户在该目录下创建文件或目录时，仅有自己与 root 才有权力删除该文件

  举例来说，我们的 /tmp 本身的权限是“drwxrwxrwt”， 在这样的权限内容下，任何人都可以在/tmp 内新增、修改文件，但仅有该文件/目录的创建者与 root 能够删除自己的目录或文件。

* SUID/SGID/SBIT 权限设置

  在用数字形式更改权限的方式中再加一个数字，最前面的数字就代表这几个权限了

  * 4 为 SUID
  * 2 为 SGID
  * 1 为 SBIT

  假设要将一个文件权限改为“-rwsr-xr-x”时，由于 s 在使用者权限中，所以是 SUID ，因此，在原先的 755 之前还要加上 4：

  ```Shell
  chmod 4755 filename
  ```

  范例：

  ```Shell
  [root@study ~]# cd /tmp
  [root@study tmp]# touch test <==创建一个测试用空文件
  [root@study tmp]# chmod 4755 test; ls -l test <==加入具有 SUID 的权限
  -rwsr-xr-x 1 root root 0 Jun 16 02:53 test
  [root@study tmp]# chmod 6755 test; ls -l test <==加入具有 SUID/SGID 的权限
  -rwsr-sr-x 1 root root 0 Jun 16 02:53 test
  [root@study tmp]# chmod 1755 test; ls -l test <==加入 SBIT 的功能！
  -rwxr-xr-t 1 root root 0 Jun 16 02:53 test
  [root@study tmp]# chmod 7666 test; ls -l test <==具有空的 SUID/SGID 权限
  -rwSrwSrwT 1 root root 0 Jun 16 02:53 test
  #注意这里出现了大写的S和T，我们执行的是7666，也就是说usr，group，others都没有x这个可执行的标志，所有S和T就代表空。
  ```

  也可以通过符号法来处理，其中SUID为u+s，SGID为g+s，SBIT则是o+t

  ```shell
  # 设置权限成为 -rws--x--x 的模样：
  [root@study tmp]# chmod u=rwxs,go=x test; ls -l test
  -rws--x--x 1 root root 0 Jun 16 02:53 test
  # 承上，加上 SGID 与 SBIT 在上述的文件权限中！
  [root@study tmp]# chmod g+s,o+t test; ls -l test
  -rws--s--t 1 root root 0 Jun 16 02:53 test
  ```

### 6.4.4 观察文件类型：file

  通过这个命令，我们可以简单地先判断这个文件地格式是什么。

  ```shell
  [root@study ~]# file ~/.bashrc
  /root/.bashrc: ASCII text <==告诉我们是 ASCII 的纯文本文件
  [root@study ~]# file /usr/bin/passwd
  /usr/bin/passwd: setuid ELF 64-bit LSB shared object, x86-64, version 1 （SYSV）, dynamically
  linked （uses shared libs）, for GNU/Linux 2.6.32,
  BuildID[sha1]=0xbf35571e607e317bf107b9bcf65199988d0ed5ab, stripped
  # 可执行文件的数据可就多的不得了,包括这个文件的 suid 权限、相容于 Intel x86-64 等级的硬件平台
  # 使用的是 Linux 核心 2.6.32 的动态函数库链接等
  [root@study ~]# file /var/lib/mlocate/mlocate.db
  /var/lib/mlocate/mlocate.db: data <== 这是 data 文件
  ```

  ## 6.5 命令与文件的查找

### 6.5.1 脚本文件的查找

* **which （查找“执行文件”）**

  ```Shell
  [root@study ~]# which [-a] command
  选项或参数：
  -a ：将所有由 PATH 目录中可以找到的命令均列出，而不止第一个被找到的命令名称
  
  范例一：查找 ifconfig 这个命令的完整文件名
  [root@study ~]# which ifconfig
  /sbin/ifconfig
  
  范例二：用 which 去找出 which 的文件名是什么
  [root@study ~]# which which
  alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
  	/bin/alias
  	/usr/bin/which
  # 竟然会有两个 which ，其中一个是 alias ，那是啥？那就是所谓的“命令别名”
  # 意思是输入 which 会等于后面接的那串命令，更多的内容我们会在 bash 章节中再来谈
  
  范例三：请找出 history 这个命令的完整文件名
  [root@study ~]# which history
  /usr/bin/which: no history in （/usr/local/sbin:/usr/local/bin:/sbin:/bin:
  /usr/sbin:/usr/bin:/root/bin）
  [root@study ~]# history --help
  -bash: history: --: invalid option
  history: usage: history [-c] [-d offset] [n] or history -anrw [filename] or history -ps arg
  # 什么？怎么可能没有 history ，我明明就能够用 root 执行 history
  ```

  这个命令是根据“PATH”这个环境变量所规范的路径，去搜寻“可执行文件”的文件名。所以，重点是找出执行文件而已。若加上 -a 选项，则可以列出所有的可以找到的同名可执行文件，而非仅显示第一个而已。

### 6.5.2 文件的查找

* **whereis （由一些特定的目录中查找文件）**

  ```Shell
  [root@study ~]# whereis [-bmsu] 文件或目录名
  选项与参数：
  -l :可以列出 whereis 会去查询的几个主要目录
  -b :只找 binary 格式的文件
  -m :只找在说明文件 manual 路径下的文件
  -s :只找 source 源文件
  -u :查找不在上述三个项目当中的其他特殊文件
  
  范例一：请找出 ifconfig 这个文件名
  [root@study ~]# whereis ifconfig
  ifconfig: /sbin/ifconfig /usr/share/man/man8/ifconfig.8.gz
  
  范例二：只找出跟 passwd 有关的“说明文件”文件名（man page）
  [root@study ~]# whereis passwd # 全部的文件名通通列出来
  passwd: /usr/bin/passwd /etc/passwd /usr/share/man/man1/passwd.1.gz /usr/share/man/man5/passwd.5.gz
  [root@study ~]# whereis -m passwd # 只有在 man 里面的文件名才显示出来
  passwd: /usr/share/man/man1/passwd.1.gz /usr/share/man/man5/passwd.5.gz
  ```

  whereis 搜寻的速度会比 find 快很多，这是因为whereis 只找几个特定的目录而已，并没有全系统去查询。

* **locate / updatedb**

  ```shell
  [root@study ~]# locate [-ir] keyword
  选项与参数：
  -i ：忽略大小写的差异；
  -c ：不输出文件名，仅计算找到的文件数量
  -l ：仅输出几行的意思，例如输出五行则是 -l 5
  -S ：输出 locate 所使用的数据库文件的相关信息，包括该数据库纪录的文件/目录数量等
  -r ：后面可接正则表达式的显示方式
  
  范例一：找出系统中所有与 passwd 相关的文件名，且只列出 5 个
  [root@study ~]# locate -l 5 passwd
  /etc/passwd
  /etc/passwd-
  /etc/pam.d/passwd
  /etc/security/opasswd
  /usr/bin/gpasswd
  
  范例二：列出 locate 查询所使用的数据库文件的文件名与各数据数量
  [root@study ~]# locate -S
  Database /var/lib/mlocate/mlocate.db:
  8,086 directories # 总纪录目录数
  109,605 files # 总纪录文件数
  5,190,295 Bytes in file names
  2,349,150 Bytes used to store database
  ```

  使用locate 来寻找数据的时候特别的快， 这是因为 locate 寻找的数据是由“已建立的数据库 /var/lib/mlocate/” 里面的数据所查找到的，所以不用直接在去硬盘当中存取数据。

  但是数据库的建立默认是在每天执行一次 ，所以当你新建立起来的文件， 却还在数据库更新之前查找该文件，那么 locate 会告诉你“找不到！”

  但是我们可以使用【updatedb】来手动更新数据库，updatedb会根据 /etc/updatedb.conf 的设置去查找系统硬盘内的文件，并更新/var/lib/mlocate 内的数据库文件。

* **find**

  ```shell
  [root@study ~]# find [PATH] [option] [action]
  选项与参数：
  1. 与时间有关的选项：共有 -atime, -ctime 与 -mtime ，以 -mtime 说明
  -mtime n ：n 为数字，意义为在 n 天之前的“一天之内”被修改过内容的文件；
  -mtime +n ：列出在 n 天之前（不含 n 天本身）被修改过内容的文件
  -mtime -n ：列出在 n 天之内（含 n 天本身）被修改过内容的文件
  -newer file ：file 为一个存在的文件，列出比 file 还要新的文件
  
  范例一：将过去系统上面 24 小时内有修改过内容 （mtime） 的文件列出
  [root@study ~]# find / -mtime 0
  # 那个 0 是重点。0 代表目前的时间，所以，从现在开始到 24 小时前，
  # 有变动过内容的文件都会被列出来，那如果是三天前那一天的 24 小时内？
  # find / -mtime 3 有变动过的文件都被列出的意思
  
  范例二：寻找 /etc 下面的文件，如果文件日期比 /etc/passwd 新就列出
  [root@study ~]# find /etc -newer /etc/passwd
  # -newer 用在分辨两个文件之间的新旧关系是很有用的。
  ```

  find相关的时间参数的意义：

   ![image-20220325214217902](images\image-20220325214217902.png)

  ```shell
  选项与参数：
  2. 与使用者或用户组名称有关的参数：
  -uid n ：n 为数字，这个数字是使用者的帐号 ID，亦即 UID ，这个 UID 是记录在/etc/passwd 里面与帐号名称对应的数字
  -gid n ：n 为数字，这个数字是群组名称的 ID，亦即 GID，这个 GID 记录在/etc/group
  -user name ：name 为使用者帐号名称。例如 dmtsai
  -group name：name 为用户组名称，例如 users
  -nouser ：查找文件的拥有者不存在 /etc/passwd中
  -nogroup ：查找文件的用户组不存在于 /etc/group 的文件。当你自行安装软件时，很可能该软件的属性当中并没有文件拥有者，这是可能的。在这个时候，就可以使用 -nouser 与 -nogroup 搜寻。
  
  范例三：搜寻 /home 下面属于 dmtsai 的文件
  [root@study ~]# find /home -user dmtsai
  # 这个东西也很有用的，当我们要找出任何一个用户在系统当中的所有文件时，
  # 就可以利用这个命令将属于某个用户的所有文件都找出来。
  
  范例四：搜寻系统中不属于任何人的文件
  [root@study ~]# find / -nouser
  # 通过这个指令，可以轻易的就找出那些不太正常的文件。如果有找到不属于系统任何人的文件时，
  # 不要太紧张，那有时候是正常的，尤其是你曾经以源代码自行编译软件时
  ```

  ```shell
  选项与参数：
  3. 与文件权限及名称有关的参数：
  -name filename：查找文件名称为 filename 的文件；
  -size [+-]SIZE：查找比 SIZE 还要大（+）或小（-）的文件。这个 SIZE 的规格有：
  	c: 代表 Byte， k: 代表 1024Bytes。所以，要找比 50KB还要大的文件，就是“ -size +50k ”
  -type TYPE ：查找文件的类型为 TYPE 的，类型主要有：一般正规文件 （f）, 设备文件 （b, c）,
  	目录 （d）, 链接文件 （l）, socket （s）, 及 FIFO （p） 等属性。
  -perm mode ：查找文件权限“刚好等于” mode 的文件，这个 mode 为类似 chmod的属性值，举例来说， -rwsr-xr-x 的属性为 4755 
  -perm -mode ：查找文件权限“必须要全部囊括 mode 的权限”的文件，举例来说，我们要搜寻 -rwxr--r-- ，亦即 0744 的文件，使用 -perm -0744，
  	当一个文件的权限为 -rwsr-xr-x ，亦即 4755 时，也会被列出来，因为 -rwsr-xr-x 的属性已经囊括了 -rwxr--r-- 的属性了。
  -perm /mode ：查找文件权限“包含任一 mode 的权限”的文件，举例来说，我们搜寻 -rwxr-xr-x ，亦即 -perm /755 时，但一个文件属性为 -rw-------
  	也会被列出来，因为他有 -rw.... 的属性存在！
  	
  范例五：找出文件名为 passwd 这个文件
  [root@study ~]# find / -name passwd
  范例五-1：找出文件名包含了 passwd 这个关键字的文件
  [root@study ~]# find / -name "*passwd*"
  # 利用这个 -name 可以查找文件名啊，默认是完整文件名，如果想要找关键字，
  # 可以使用类似 * 的任意字符来处理
  
  范例六：找出 /run 目录下，文件类型为 Socket 的文件名有哪些？
  [root@study ~]# find /run -type s
  # 这个 -type 的属性也很有帮助，尤其是要找出那些怪异的文件，
  # 例如 socket 与 FIFO 文件，可以用 find /run -type p 或 -type s 来找
  
  范例七：查找文件当中含有 SGID 或 SUID 或 SBIT 的属性
  [root@study ~]# find / -perm /7000
  # 所谓的 7000 就是 ---s--s--t ，那么只要含有 s 或 t 的就列出，所以当然要使用 /7000，
  # 使用 -7000 表示要同时含有 ---s--s--t 的所有三个权限。而只需要任意一个，就是 /7000
  ```

  find后面可以接多个目录来进行查找。另外， find 本来就会查找子目录，这个特色也要特别注意。

  ```shell
  选项与参数：
  4. 额外可进行的操作：
  -exec command ：command 为其他命令，-exec 后面可再接额外的命令来处理查找到的结果。
  -print ：将结果打印到屏幕上，这个操作是默认动作
  
  范例八：将上个范例找到的文件使用 ls -l 列出来
  [root@study ~]# find /usr/bin /usr/sbin -perm /7000 -exec ls -l {} \;
  # 注意到，那个 -exec 后面的 ls -l 就是额外的命令，命令不支持命令别名，
  # 所以仅能使用 ls -l 不可以使用 ll
  # {} 代表的是find找到的内容
  # -exec 一直到 \；是关键词，代表find额外擦偶作的开始到结束，在这中间的就是find命令内的额外操作。在本例中就是"ls -l {}"
  # 因为";"在bash环境下是有特殊意义的，因此利用反斜杠来转义
  ```

  注意由于find在查找数据的时候相当消耗硬盘资源，所以没事不要使用find。

## 6.6 权限与命令间的关系

一、让用户能进入某目录成为可工作目录

* 可使用的命令：例如 cd 等变换工作目录的命令
* 目录所需权限：**用户对这个目录至少需要具有 x 的权限**
* 额外需求：如果用户想要在这个目录内利用 ls 查看文件名，则用户对此目录还需要 r的权限

二、用户在某个目录内读取一个文件

* 可使用的命令：例如 car、more、less等
* 目录所需权限：用户对这个目录至少需要具有 x 的权限
* 文件所需权限：**用户对这个文件至少需要具有 r 的权限**

三、让用户可以修改一个文件

* 可使用的命令：例如 nano 或 vi 编辑器等
* 目录所需权限：用户对这个目录至少需要具有 x 的权限
* 文件所需权限：**用户对这个文件至少需要具有 r、w 的权限**

四、让一个用户可以建立一个文件

* 目录所需权限：用户对这个目录至少需要具有 w、x 的权限，重点在于w

五、让用户进入某目录并执行该目录下的某个命令

* 目录所需权限：用户对这个目录至少需要具有 x 的权限
* 文件所需权限：用户对这个文件至少需要具有 x 的权限
