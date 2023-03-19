## 一、安装 / 配置

### 1、概述

Unix 的由来：20 世纪 70 年代，由贝尔实验室的 Ken tompsom 和 Dennis richres 共同创造的 C 语言编写而成。近期 Ken tompsom 又创造了 Go 语言。

GNU 计划：对互联网发展有巨大影响的计划（开源），由 Richard  Stallman 发起。

Linux 之父：Linus Torvalds，Git 创作者。

Linux 主要发行版本：Ubuntu、RedHat、CentOS、Debian、Fedora、SuSE、OpenSUSE。

[Linux 内核源码](https://www.kernel.org/)

### 2、安装 CentOS_7

- [VirtualBox 下载](https://www.virtualbox.org/)
- [CentOS 下载](https://www.centos.org/download/) （推荐使用阿里云镜像下载）
- [安装教程](https://www.cnblogs.com/Young-wind/p/5852180.html)

注：若安装虚拟机时，只有32位的可供选择，可能是 BIOS 没有开启虚拟技术。
```
# 启用虚拟机功能
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

### 3、网络配置

#### （1）在虚拟机中无法 ping 通外网

① VirtualBox 配置：设置 > 网络 > 网卡 1 > 连接方式：网络地址转换（NAT）。

② 修改网络配置文件： `ifcfg-enp0s3`  （网络地址转换(NAT) 对应的配置）：

```
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

```shell
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=6ae4e71b-23d5-42e5-8e05-48e2e6b36d21
DEVICE=enp0s3
# ONBOOT=no
ONBOOT=yes   # 修改为 yes
```

③ 重启网络服务

```
service network restart
```

④ ping

```
# 请求5次后停止，不带 -c 选项时，一直发送，需要 Ctrl+C 停止
ping www.baidu.com -c 5
```

#### （2）主机无法连接到虚拟机

① 用 `ip addr` 命令查看网卡信息，是否存在 `enp0s8` 网卡，默认没有。

② VirtualBox 配置：设置 > 网络 > 网卡 2 > 启用网络连接 =>  连接方式：仅主机（Host-Only）网络。

③ 修改网络配置文件： `ifcfg-enp0s8`，同上。

④ 再次用 `ip addr` 命令查看网卡信息：

```
enp0s8: ...
	...
	inet 192.168.56.102/24 ...
	...
```

⑤ 用 SSH 连接到虚拟机：

```
ssh root@192.168.1.159
```

#### （3）连接模式配置

|                       | NAT  | Bridged Adapter |   Internal   |   Host-Only    |
| --------------------: | :--: | :-------------: | :----------: | :------------: |
|     **虚拟机 → 主机** |  √   |        √        |      ×       | 默认不能需设置 |
|     **主机 → 虚拟机** |  ×   |        √        |      ×       |       √        |
| **虚拟机 → 其他主机** |  √   |        √        |      ×       | 默认不能需设置 |
| **其他主机 → 虚拟机** |  ×   |        √        |      ×       | 默认不能需设置 |
|        **虚拟机之间** |  ×   |        √        | 同网段下可以 |       √        |

**各模式的特点：**

- 网络地址转换(NAT)：连接这个网络可以访问外部网络，但是外部网络不可访问虚拟机。

  原理：在主机上创建一个虚拟网卡，与虚拟机的 IP 同网段，主机与虚拟机之前形成一个网络。

- 桥接网卡：这个网络完全可以共享主机网络，主机网络发生变化时，也跟随变化，IP 也随之变动。

  优点：局域网内的其它主机可以访问到虚拟机。

  缺点：会占用局域网中同网段的一个 IP，易造成 IP 冲突。

- 仅主机(Host-Only)网络：这个网络也可以用来主机访问虚拟机以及虚拟机上web服务，但是虚拟机不可访问外网。

  原理：相当于一个独立网络。



### 4、目录结构

> 在 Linux 的世界里，一切皆文件。

```
/：根目录
    /bin -> usr/bin：    程序的启动文件（所有命令）
    /boot：              启动目录
    /dev：               设备目录
    /etc：               可编辑文本配置
    /home：              普通用户目录
    /lib -> usr/lib：    库
    /lib64 -> usr/lib64：64位操作系统的库
    /lost+found：        非正常关机留下的文件（无）
    /media：             媒体
    /mnt：               安装临时文件
    /opt：               放置程序安装包
    /proc：              虚拟文件系统目录
    /root：              root用户目录
    /run：
    /sbin -> usr/sbin：  超级用户的可执行程序
    /srv：               可访问数据库目录
    /sys：               sysfs文件系统的挂载点
    /tmp：               公用临时文件存储点
    /usr：               Unix系统资源，存放一些应用程序
    /var：               存放运行时需改变数据的文件，如：日志文件
    /selinux：           保证系统安全（无）
        getenforce：        获取状态
        setenforce = 1：    设置状态
        永久关闭：修改配置文件/etc/selinux/config -> SELINUX=disabled

注：
    软件安装目录：/usr/local
```



## 二、常用命令

> 命令格式：命令 [选项] [参数]
>

### 1、关机、重启命令

```
shutdown [选项] [时间] 

# 立即重启（或 reboot）
shutdown -r now

# 立即关机（或 poweroff 或 halt）
shutdown -h now

# 取消关机计划
shutdown -c

注：不指定选项和参数，默认1分钟后关闭电脑（shutdown -h 1）
```

### 2、工作目录

**（1）切换工作目录（change directory）**

```
cd [目录]
	
    #     root的根目录（或普通用户的home目录）
    -     打印并切换回上一个工作目录
    ~     当前用户的home目录
    cd    不加参数，也是切换到home目录
    .     当前目录
    ..    上级目录
    /     根目录
    /dir  切换到指定目录

# 打印当前工作目录
pwd
```

**（2）显示目录列表（list）**

```
ls [选项] [目录]

选项：
    -a    显示所有文件，包括隐藏文件（以./ 和 ../ 开头的文件）
    -l    列表显示文件，会显示文件的所有信息 <==> ll
    -lh   带单位显示文件大小
	
参数：
    默认打印当前目录列表
	
结果说明：
    文件类型及权限 | 引用数 | 用户 | 组 | 大小 | 月 | 日 | 年/时间 | 文件名
	
文件类型说明：
    -    普通文件
    d    目录
    b    块设备
    c    字符设备
    l    链接
    s    套接字
    p    管道	
		
# 以树形结构打印目录
tree [目录]
```

### 3、vi-vim 编辑器

> 终端编辑工具有：vi、vim、emacs 等。
>
> 系统默认自带 vi 编辑器。
>
> vim 是 vi 的增加版，如：语法高亮、代码补全、编译及错误跳转等功能。

#### （1）打开文件

> 若文件存在，则直接打开。
>
> 若文件不存在，则新建文件，不修改则不会创建空文件。

```shell
vi filename

# 打开文件，光标定位到第n行
vi filename +n

# 打开文件，光标定位到最后一行
vi filename +
```

#### （2）工作模式

- 正常模式

> 主要用来浏览或修改文本内容；
>
> 使用 vi 打开文件的默认工作模式；
>
> 在任意模式下按 ESC 键即可进入正常模式。

- 编辑模式（插入模式）

> 主要用来编辑文本，正常模式下输入以下字符进入编辑模式。

```
i：在光标所在字符前开始输入文本；
I：在行首第一个非空白字符处开始输入文本；
a：在光标所在字符后开始输入文本；
A：在行尾开始输入文本；
o：在光标所在行的下面新增一行来开始输入文本；
O：在光标所在行的上面新增一行来开始输入文本；
s：删除光标所在字符开始输入；
S：删除光标所在行开始输入；
```

- 命令模式（单行模式）

> 主要用来管理文件或设置 vi，如：保存、退出、放弃等。
>
> 在正常模式下，输入 `:` 即可进入命令模式。

```
:w    保存文件；
:q    退出软件；
:x    保存退出（:wq）；
:!    强制操作；
:e!   放弃修改；
```

- 可视模式（正常模式下操作）

> 可视模式主要用作批量操作

```
v        可视字符模式（通过左右逐个选择字符）
V        可视行模式（通过上下选择行数）
Ctrl+v   可视块模式（通过上下左右(或 kjhl)选择行数和列数）

# 例1：多行注释
① Esc 进入正常模式；
② Ctrl+v 进入可视化块模式；
③ 利用上下左右调整需要注释的行数及列数；
④ Shift+i 进入插入模式，插入注释符：“#”；
⑤ 再次按 Esc，即可完成多行注释。

# 例2：取消多行注释
① Esc 进入正常模式；
② Ctrl+v 进入可视化块模式；
③ 利用上下左右调整需要注释的行数及列数；
④ 按 d 即可取消注释。
```

#### （3）光标定位（正常模式下操作）

```
gg    定位到首行
GG    定位到末行
ngg   定位到第 n 行
0     定位到行首
^     定位到行首第一个非空字符
$     定位到行尾
kjhl  上下左右
```

#### （4）复制粘贴（正常模式下操作）

```
yy    复制光标所在行
dd    剪切光标所在行
p     粘贴缓冲区的内容
nyy   复制光标开始的 n 行
ndd   剪切光标开始的 n 行
```

#### （5）操作回退（正常模式下操作）

```
u           撤销操作
Ctrl+r      反撤销
shift + zz  保存编辑（:x）
```

#### （6）查找替换（命令模式下操作）

```
# 查找，回车查找，n向前查找，N向后查找
?
/

# 替换，g代表全局替换，可选
%s/查找内容/替换内容/[g]

# 替换从起始行到结束行查找到的内容
起始行,结束行s/查找内容/替换内容/[g]	
```

#### （7）vi-vim 配置项

- 一次性配置（命令模式下操作，仅针对本次打开文件有效）

```
:set nu							显示行号
:set nonu						取消显示行号
:set tabstop=4					设置缩进字符数
:set fileeccodings=utf-8,gbk	设置字符集
```

- 配置文件： `~/.vimrc`（永久生效）

```shell 
# 编辑配置文件
vi ~/.vimrc

​``````````````````````````````````````````````````
set nu
set ...

" Indent & Tab
set autoindent      " 自动保留上一行的缩进
set smartindent     " 以 { 结尾的行，在新行会触发缩进
set tabstop=4       " Tab 宽度，4 空格
set shiftwidth=4    " 自动缩进时的宽度，4 空格
set expandtab       " 将 Tab 转换为空格，而不是 \t（注意不能打开 Makefile 文件）
​``````````````````````````````````````````````````

# 立即生效
source ~/.vimrc
```

#### （8）键盘图

![vi / vim 键盘图](Images/Linux_vi-vim键盘图.jpg)

### 4、其它

```shell
# 自动补全（命令、参数）：tab
# 清屏：clear
# 结束进程：Ctrl+c

# 重定向：
    将本该显示在终端上的内容 输出、追加到指定文件中
    >    表示输出到文件，会覆盖文件原有的内容
    >>   表示追加，追加到文件的末尾

    另一种创建文件的方式：
    echo text > text.txt
    touch 只能创建一个空文件，而重定向方式可以创建带内容的文件
    还可以将其他命令的输出结果追加到文件中：
    tree laravel >> laravel_directory_structure.txt

# 管道：|
    Linux允许将一个命令的输出通过管道作为另一个命令的输入。
    常与管道搭配使用的命令有：
    more
    grep

# 通配符：
    *
    ?	匹配1个字符
    []	字符组中的任何一个（如：[abc] [1-9]）

# 帮助指令：
    man <command>  获得帮助信息
    help <command> 获得Shell内置命令的帮助信息

# 查看已经执行过的历史命令
history     # 显示全部
history 10  # 显示最近10条
!15         # 执行历史第15条命令

# ---------------
#  日期时间
# ---------------

# 显示日期
$ date
2023年 03月 19日 星期日 01:09:11 CST

$ date '+%Y-%m-%d %H:%M:%S'
2023-03-19 01:10:41

# 设置日期
$ date -s "0000-00-00 00:00:00"
# 显示日历：
cal [选项] [[[日] 月] 年]

选项：
    -1, --one        只显示当前月份(默认)
    -3, --three      显示上个月、当月和下个月
    -s, --sunday     周日作为一周第一天
    -m, --monday     周一用为一周第一天
    -j, --julian     输出儒略日
    -y, --year       输出整年

```

### 5、运行级别

```
# 查看系统默认运行级别
systemctl get-default

multi-user.target    # 3

# 设置系统默认运行级别
systemctl set-default multi-user.target

# 通过 init 来切换不同的运行级别
init [0123456]

说明：
    0 关机
    1 单用户【找回密码】
    2 多用户状态没有网络服务
    3 多用户状态有网络服务：multi-user.target
    4 系统未使用，保留给用户
    5 图形界面：graphical.target
    6 系统重启
    
# 运行级别 1 的应用：[找回 root 密码]（www.baidu.com）
```



## 三、文件 & 目录

### 1、查看文件内容

```shell
# 将<文件>或标准输入组合输出到标准输出
cat [Options] <文件, ...>

Options:
    -b  对非空行进行编号
    -n  对所有行进行编号（包含空行）
    
注：
    如果没有指定文件，或者文件为 `-`，则从标准输入读取。
    与 more 结合，翻页显示 cat <File> | more

# 倒序输出 cat
tac [Options] <文件, ...>

# 分页显示文件内容（留痕）
more [Options] <文件 ,...>

说明：
    more 指令是一个基于 VI 编辑器的文本过滤器，它以全屏幕的方式按页显示文本内容。
    【其它命令 | more】：分页显示其它命令执行的结果。
    more 内置了若干快捷键（交互指令）：
    |--------|------------------|
    | space  | 向下翻一页         |
    | Enter  | 向下翻一行         |
    | q      | 退出（结束分页）    |
    | Ctrl+F | 向下滚动一屏       |
    | Ctrl+B | 返回上一屏         |
    | =      | 输出当前行号       |
    | :f     | 输出文件名和当前行号 |
    |--------|------------------|
    
# 与 more 功能一样（动态加载）
less [选项] 文件名

说明：
    其它命令 | less：分页显示其它命令执行的结果

# 输出内容到控制台
echo [Options] <Content>

例：
    输出环境变量：echo $PATH 
    输出内容并写入文件：echo 'Hello World' > hello

# 打印每个文件的【前|后】 n 行（默认10行）
head [Options] <文件, ...>

Options：
    -c, --bytes   打印前 c 字节
    -n, --lines   打印前 n 行

tail [Options] <文件, ...>

Options：
    -c, --bytes   打印前 c 字节
    -n, --lines   打印前 n 行
    -f, --follow  监听文件的更新
    -F            等同 --follow=name --retry
      --max-unchanged-stats=N  with -f，重新打开一个在迭代 N 次（默认5）后没有更改大小的 FILE，检查它是否已被取消链接或重命名（旋转日志）。
      --pid=PID                with -f，在 PID 终止后进程终止
    --retry       若文件无法访问，重试
```

### 2、文件操作

```shell
# 更新文件的访问和修改时间
# 当文件不存在时，创建一个空文件
touch [Options] <文件, ...>

Options:
    -a  更新访问时间
    -m  更改修改时间

# 删除文件 & 目录
rm [Options] <文件, ...>

Options:
    -f, --force  强制删除，忽略不存在的文件和参数，从不提示
    -r, -R, --recursive  递归删除目录及其内容
    -d, --dir    删除空目录

注：默认 rm 不会删除目录，使用 rm -r 删除目录及其内容

# 拷贝文件 & 目录
cp [Options] <源文件> <目标文件>

Options:
    -i  覆盖前提示
    -r  递归复制目录及其子目录内的所有内容（拷贝目录时必须）
    -T  将目标目录视作普通文件

# 拷贝到远程
scp -P port <文件> user@host:File

例：scp my.ini root@192.168.0.159:/usr/mysql

# 重命名或移动文件 & 文件夹
mv [选项] 源文件 目标文件

选项：
    -f, --force			覆盖前不询问
    -i, --interactive	覆盖前询问
    -n, --no-clobber	不覆盖已存在文件
    
注：指定 -i、-f、-n 中的多个时，仅最后一个生效。
```

### 3、目录操作

```shell
# 创建目录
mkdir [选项] 目录

选项：
    -p  不报错误，根据需要创建父目录

# 删除空的目录
rmdir <Options> 

# 创建链接文件
ln [-s] 源文件 目标文件：

硬链接：
    不加 -s 选项时，简单理解为一个文件有多个名字
    ① 不占用实际空间
    ② 不允许给目录创建
    ③ 不能跨文件系统
	
软链接：
    添加 -s 选项时，简单理解为一个文件的内容是另一个文件的路径
    ① 类似于Windows的快捷方式
    ② 可以对目录创建
    ③ 可以跨文件系统
```

### 4、查找文件

**（1）grep**

> 在每个 FILE 或是标准输入中查找 PATTERN，默认的 PATTERN 是一个基本正则表达式。

```
grep [选项] PATTERN <FILE ...>

选项：
    -i	不区分大小写查找
    -v	显示不包含匹配文本的所有行（排除）
    -n	显示匹配行及行号

管道：其他命令返回结果 | grep [选项] PATTERN
```

**（2）find**

```
find [-H] [-L] [-P] [-Olevel] [-D help|tree|search|stat|rates|opt|exec] [path...] [expression]

find [目录] [条件] [动作]

目录：所要搜索的目录及其所有子目录。默认为当前目录。

条件：所要搜索的文件特征。

动作：对搜索结果进行特定的处理。

选项：
    -name     指定文件名，可以通过*模糊匹配
    -type     指定文件类型（b/c/d/p/l/f）
    -size     指定文件大小，+表示大于，-表示小于
    -user     指定用户
    -group    指定组
    -mtime/atime/ctime  指定修改/访问/创建时间，单位为天，+表示几天前，-表示几天内
    -mmin/amin/cmin     同上，单位为分钟

例：
    find：查找当前目录下的所有文件。
    find -name '*.log' ：查找 .log 文件。
```

**（3）whereis**

```
whereis [选项] 文件

选项：
	-b        只搜索二进制文件
    -B <目录>  定义二进制文件查找路径
    -m        只搜索 man 手册
    -M <目录>  定义 man 手册查找路径
    -s        只搜索源代码
    -S <目录>  定义源代码查找路径
    -f        终止 <目录> 参数列表
    -u        搜索不常见记录
    -l        输出有效查找路径
```

**（4）which**

> 将命令的完整路径写入标准输出，在 $PATH 变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果，使用which可以检查某个命令是否存在，以及执行的到底是哪个命令。

```
which [options] [--] COMMAND [...]
```

**（5）PATH**

> PATH：环境变量（与Windows类似）

打印：

```
echo $PATH
```

设置：

```
方法1：一次性的设置

export PATH=$PATH:dir1[:dir2]

方法2：永久性的设置，所有用户有效，需重启或使用 source 命令生效

将方法1的配置添加到文件 `/etc/profile` 的末尾。

方法3：永久性的设置，针对一个用户有效，需重启或使用 source 命令生效

将方法1的配置添加到文件 `~/.bashrc` 的末尾。
```

### 5、文件压缩与解压

**（1）gzip / gunzip：压缩、解压文件**

> 只能压缩单个文件，不压缩目录

```
# 压缩文件：生成 FILE.gz，删除原文件
gzip [选项] [文件]

选项：
    -d  等价于 gunzip 解压文件

# 解压文件，删除原压缩文件
gunzip [选项] [文件]
```

**（2）bzip2 / bunzip2：压缩、解压文件**

```
bzip2  [选项] [文件]
bunzip2 [选项] [文件]

选项：
    -k	压缩或解压后保留原文件

说明：
    用法与 gzip/gunzip 相同，只是多了一个 -k 选项；
    使用 bzip2 压缩文件的后缀为 bz2。
```

**（3）tar** 

> 可以将多个文件或目录进行压缩

```
tar [选项...] [FILE]...

选项：
    -c	压缩
    -x	解压
    -z	使用 gzip
    -j	使用 bzip2
    -f	指定处理文件
    -v	显示（压缩解压过程的）详细信息
    -C	指定解压后存放文件的目录
    
例：
    tar -zxvf php-7.3.30.tar.gz
```

### 6、文件权限管理

**（1）ls -lh 打印文件列表信息**

```
结果说明：
    文件类型及权限 | 引用数 | 文件所有者 | 文件所属组组 | 大小 | 月 | 日 | 年/时间 | 文件名
	
例如：	
    -rw-r--r--. 1 root root 7.4M 9月  23 16:19 tree.txt

文件类型及权限：[-rw-r--r--.]
    1        文件类型（-：文件；b：块设备；c：字符设备；d：目录；l：链接；s：套接字）
    2/3/4    文件所有者的读(r)/写(w)/执行(x)权限，-表示无权限
    5/6/7    文件所有者所属组的读/写/执行权限
    8/9/10   其它用户的读/写/执行权限
    .        表示启用了selinux，空表示没有开启

注：
    所属组的权限将被组中成员继承；
    对文件拥有写（权限）不代表可以删除文件，同时还需要对该目录的写权限才可删除文件。

文件类型说明：
    -    普通文件
    d    目录
    b    块设备：硬盘
    c    字符设备：鼠标、键盘
    l    链接
    s    套接字
    p    管道	
```

**（2）chmod 修改文件权限**

```
chmod [选项] [模式|八进制模式] 文件

模式：
    r    可读
    w    可写
    x    可执行
    +    添加权限
    -    去除权限
    =    设置权限
    u    所有者
    g    所属组
    o    其它组
    777	 八进制模式，777 => 111 111 111 这三组二进制分别代表了三个角色的三种权限
	
实例：
    chmod +x file     给文件添加可执行权限（所有者、组、其它）
    chmod u+x file    给文件所有者添加可执行权限
    chmod 777 file    用八进制数来设置权限
    chmod u=rwx,g=rx,o=- file    分别设置权限
```

**（3）umask** 

> 查看或设置 umask 的值，用来确定创建文件的默认权限。

```
umask [-p] [-S] [模式]

说明：
    默认umask为0022 => 000 010 010
    创建目录文件的权限 rwx r-x r-x
    创建普通文件的权限 rw- r-- r--
	
    创建目录时，取反即得新建目录的操作权限
    创建普通文件时，没有可执行权限，其余位取反即得新建文件的操作权限
	
配置文件（永久生效）：
    所有用户：/etc/profile
    单个用户：~/.profile 或 ~/.bash_profile
```

**（4）chattr**

> 修改文件属性，提高系统的稳定性

```
chattr [-RVf] [-+=aAcCdDeijsStTu] [-v version] files...

选项：
    i  表示忽略
    +  表示添加
    -  表示去掉
```

**（5）lsattr**

> 查看使用chattr设置的文件属性

```
lsattr [-RVadlv] [files...]
```

**（6）修改文件所有者及组**

```shell
# 设置/修改文件所属组
chgrp 组 文件,...

# 设置/修改文件所属者[及组]
chown 所属者[:组] 文件,...

Options:
    -R, --recursive 递归操作文件和目录

注：
    修改所属者及组时，可以使用UID或GID；
```



## 四、用户与用户组

### 1、用户管理

```shell
# 显示当前登录用户
# 与 id -un 相同
whoami

# 新增用户
useradd [Options] <username>

Options：
    -d, --home-dir 新账户的home目录，不指定则在/home下创建一个与用户名相同的目录
    -g, --gid      新账户主组的名称或ID
    -G, --groups   新账户的附加组列表
    -p, --password 新账户的密码
    -s, --shell    新账户登录的shell，若用户已经创建，则用 `chsh 用户名 -s /sbin/nologin` 禁止登录
    -u, --uid      指定用户ID，必须大于500

# 删除用户
userdel [Options] <username>

Options:
    -f, --force        强制删除
    -r, --remove       同时删除 `/home/username` 目录和邮件池 `/var/mail/username`
    -Z, --selinux-user 为用户删除所有的 SELinux 用户映射

# 修改用户密码
passwd [Options] <username>

Options:
    -k, --keep-tokens       保持身份验证令牌不过期
    -d, --delete            删除已命名帐号的密码（仅限 root 用户）
    -l, --lock              锁定指名帐户的密码（仅限 root 用户）
    -u, --unlock            解锁指名账户的密码（仅限 root 用户）
    -e, --expire            终止指名帐户的密码（仅限 root 用户）
    -f, --force             强制执行操作

注：
    1.只有 root 用户可以指定 username，普通用户不能指定，默认修改当前登录用户的密码
    2.用户密码文件 `/etc/shadow`

# 查看所有用户
cat /etc/passwd

# 切换用户
su <username>

注：不指定 USER 默认切换到 root 用户

# 注销用户
# （仅能在Linux服务器执行，不能在Xshell执行）
logout

# 以 root 身份执行命令
sudo <command>

" 特殊标识
    #  超级用户
    $  普通用户
    ~  用户home目录
```

### 2、用户组管理（角色）

```shell
# 添加用户组
groupadd [Options] <Group>

# 删除用户组
groupdel [Options] <Group>

# 向用户组中添加或删除用户
gpasswd [Options] <Group>

Options:
    -a, --add USER          向组中添加用户 USER
    -d, --delete USER       从组中删除用户 USER
    -M, --members USER,...  设置组的成员列表（批量）

# 查看系统中的所有组
cat /etc/group

# 查看用户及组信息
id <用户名>

uid=0(root) gid=0(root) 组=0(root)
```

### 3、相关文件

> /etc/passwd 用户文件

```
cat /etc/passwd

root:x:0:0:root:/root:/bin/bash
========================================
用户名:密码:uid:gid:注释性描述:home目录:命令解析文件
```

> /etc/shadow 密码文件

```
cat /etc/shadow

root:$6$/Cbjf5XNFuA/ePr8$Ry.nB1tjFfE3yKV9BUdxi0YBRNiZKAc2hVQoRKkZrqWTSm8cPftKzjTdMJb3XgIUiXlYPv.0KypCDb2hr2O7K.::0:99999:7:::
========================================
用户名:密码:最后一次修改时间:最小时间间隔:最大时间间隔:警告时间:不活动时间:失效时间:标志
```

> /etc/group 用户组文件

```
cat /etc/group

root:x:0:
====================
组名:密码:gid:组内用户列表（隐藏）
```

## 五、定时任务 Cron

### 1、cron 服务

```
# 查看 cron 状态
service cron status

# 操作 cron 服务
/etc/init.d/cron start
/etc/init.d/cron stop
/etc/init.d/cron restart
```

### 2、crontab 用法

```
# 修改 crontab 文件，如果文件不存在会自动创建。 
crontab –e

# 显示 crontab 文件
crontab –l

# 删除 crontab 文件
crontab -r

# 删除 crontab 文件前提醒用户
crontab -ir
```

### 3、crontab 格式

```shell
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12)
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7)
# |  |  |  |  |
# *  *  *  *  * <command>

# 特殊符号
    * 代表所有的取值范围内的数字
    / 代表每的意思，"/5" 表示每5个单位
    - 代表从某个数字到某个数字
    , 分开几个离散的数字
	
# 例
    * * * * *		# 每分钟执行
    0 6 * * *		# 每天 6:00 执行
    */30 * * * *	# 每 30 分钟执行
    * 9-18/1 * * *	# 9:00-18:00 每小时执行
    * 0,8,16 * * *	# 每天的 0:00,8:00,16:00 执行
```



## 六、磁盘管理

### 1、磁盘分区机制

> 硬盘分类：
>
> - IDE 硬盘：驱动器标识符为 `hdx~` ，其中 hd 表明分区所在设备类型（IDE 硬盘），x 为盘号（a：基本盘，b：基本从属盘，c：辅助主盘，d：辅助从属盘），~ 代表分（1~4：主分区或从属分区，从 5 开始就是逻辑分区）。
> - SCSI 硬盘：标识符为 `sdx~`，其中 sd 表示分区所在设备的类型（SCSI 硬盘），其余同上。

```shell
# 查看所有设备挂载情况
$ lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0    8G  0 disk               # SCSI 硬盘 a盘
├─sda1            8:1    0    1G  0 part /boot         # a盘 1号分区
└─sda2            8:2    0    7G  0 part               # a盘 2号分区
  ├─centos-root 253:0    0  6.2G  0 lvm  /
  └─centos-swap 253:1    0  820M  0 lvm  [SWAP]
sr0              11:0    1 1024M  0 rom

# 输出文件系统信息
lsblk -f
NAME            FSTYPE   LABEL UUID                                   MOUNTPOINT
sda
├─sda1          xfs            ac8726ac-c47a-444d-89fa-787167a2145f   /boot
└─sda2          LVM2_mem       mqDLcn-V2Ru-po0v-J9yZ-Oe8I-LSwc-JHcC17
  ├─centos-root xfs            e8fa9679-32ed-49f2-9e0c-ccc0088164d7   /
  └─centos-swap swap           909cb6a9-2262-4f6e-b970-75cb6dc00e25   [SWAP]
sr0
```

### 2、增加磁盘

```shell
# ------------------
#  添加磁盘的步骤
# ------------------
# 1 在虚拟机中添加一个磁盘

# 2 分区
fdisk

# 3 格式化磁盘
mkfs

# 4 挂载
# 注：命令行模式下的挂载是临时的，重启后失效
# 永久挂载：vi /etc/fstab
mount

# 5 卸载
umount
```

### 3、查看磁盘使用情况

```shell
$ df -h
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                 484M     0  484M    0% /dev
tmpfs                    496M     0  496M    0% /dev/shm
tmpfs                    496M  6.8M  489M    2% /run
tmpfs                    496M     0  496M    0% /sys/fs/cgroup
/dev/mapper/centos-root  6.2G  1.5G  4.8G   24% /
/dev/sda1               1014M  137M  878M   14% /boot
tmpfs                    100M     0  100M    0% /run/user/0

# 查看指定目录的磁盘占用情况（默认当前目录）
$ du -h <目录>

Options：
    -s  指定目录占用大小汇总
    -h  带计量单位
    -a  含文件
    --max-depth=1 子目录深度
    -c  列出明细的同时，增加汇总值
```



## 七、网络管理

### 1、查看网卡信息

（1）ping

> 通常用于检测网络设备的连通性。

```
ping IP/域名：

选项：
    -c  指定发送请求次数，不指定无限次发送
```

（2）ip addr / ifconfig

> 查看或设置网卡信息（IP）

```
ifconfig [-a] [-v] [-s] <interface> [[<AF>] <address>]

ifconfig eth0 down  关闭网卡，等价于 ifdown eth0
ifconfig eth0 up    开启网卡，等价于 ifup eth0

注：ifconfig 需安装工具才能使用（yum -y install net-tools）
```

### 2、配置网卡信息

（1）网卡配置文件目录：`/etc/sysconfig/network-scripts/`


（2）添加域名服务器（DNS）地址：`/etc/resolv.conf`

```
8.8.8.8
114.114.114.114
```

（3）添加本地域名解析服务：`/etc/hosts`

```
127.0.0.1    www.test.com
```

### 3、网络服务管理

```
/etc/init.d/network start|stop|restart
或
service network start|stop|restart
```

### 4、防火墙管理

#### （1）防火墙的开启、关闭、禁用

① 设置开机启用防火墙：

```
systemctl enable firewalld.service
```

② 设置开机禁用防火墙：

```
systemctl disable firewalld.service
```

③ 启动防火墙：

```
systemctl start firewalld
```

④ 关闭防火墙：

```
systemctl stop firewalld
```

⑤ 启动防火墙：

```
systemctl restart firewalld
```

⑥ 检查防火墙状态：

```
systemctl status firewalld 
```

#### （2）使用 firewall-cmd 配置端口

① 查看防火墙状态：

```
firewall-cmd --state
```

② 重新加载配置：

```
firewall-cmd --reload
```

③ 查看开放的端口：

```
firewall-cmd --list-ports
```

④ 开启防火墙端口：

```
firewall-cmd --zone=public --add-port=9200/tcp --permanent

命令含义：
    –zone #作用域
    –add-port=9200/tcp #添加端口，格式为：端口/通讯协议
    –permanent #永久生效，没有此参数重启后失效
```

**注意：添加端口后，必须用命令 `firewall-cmd --reload` 重新加载一遍才会生效。**

⑤ 关闭防火墙端口：

```
firewall-cmd --zone=public --remove-port=9200/tcp --permanent
```

**（3）CentOS 7 以前版本**

```
service iptables stop|start		关闭、开启防火墙
service iptables status			查看防火墙状态
```



## 八、进程管理

### 1、vmstat：系统整体信息

```
结果：
    procs -----------memory---------- ---swap-- ----io---- --system-- ------cpu------
     r  b   swpd   free   buff  cache   si   so  bi    bo   in   cs   us sy id  wa st
     2  0      0 642528   2116 156468    0    0  7     1    32   57   0  0  100  0  
说明：
    procs 进程
        r 表示运行队列
        b 表示阻塞进程数
    memory 内存
        swpd	虚拟内存已使用的大小，大于0表示你的机器物理内存不足
        free	空闲的物理内存的大小
        buff	用来存储，目录里的内容、权限等的缓存
        cache	直接用来记忆打开的文件，给文件做缓存
    swap 交换
        si	每秒从磁盘读入虚拟内存的大小
        so	每秒虚拟内存写入磁盘的大小
    io 块设备
        bi	块设备每秒接收的块数量
        bo	块设备每秒发送的块数量
    system 系统
        in	每秒CPU的中断次数，包括时间中断
        cs	每秒上下文切换次数
    cpu 中央处理器
        us	用户CPU时间
        sy	系统CPU时间
        id	空闲CPU时间，id+us+sy = 100
        wa	等待IO的CPU时间
        st 虚拟机占用的时间百分比
```

### 2、w：系统正在做什么（what）

```
结果：
    17:50:24 up  5:55,  1 user,  load average: 0.00, 0.01, 0.05
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
    root     pts/0    192.168.245.1    13:07    0.00s  0.19s  0.03s w
说明：
    第一行：系统当前时间 开机持续时间 登录用户个数 平均负载
    其它行：用户|终端|来源|登录时间|空闲时间|使用时间|当前进程时间|正在做
```

### 3、top：详情

> w 的详细显示，每3秒刷新一次，shift+m 按所占内存排序，q退出检测

```
结果： 
    PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND

说明：
```

### 4、free：查看内存使用

```
选项：
    -h 单位人性化
```

### 5、ps：查看进程	

```
ps [选项]

选项：
       -a 显示控制终端的所有进程
       -u 显示用户信息
       -x 显示没有控制终端的进程

ps -aux

| USER | PID | %CPU | %MEM | VSZ | RSS | TTY | STAT | START | TIME | COMMAND |
```

> STAT 代表进程状态：

R——Runnable（运行）：正在运行或在运行队列中等待

S——sleeping（中断）：休眠中，受阻，在等待某个条件的形成或接收到信号

D——uninterruptible sleep(不可中断)：收到信号不唤醒和不可运行，进程必须等待直到有中断发生

Z——zombie（僵死）：进程已终止，但进程描述还在，直到父进程调用wait4()系统调用后释放

T——traced or stoppd(停止)：进程收到SiGSTOP,SIGSTP,SIGTOU信号后停止运行

> 状态后缀：

<：优先级高的进程

N：优先级低的进程

L：有些页被锁进内存

s：进程的领导者（在它之下有子进程）

l：ismulti-threaded (using CLONE_THREAD, like NPTL pthreads do)

+：位于后台的进程组 

### 6、kill：结束进程

```
kill [选项] 进程
	
选项：
    -9 强制结束
```



## 九、软件安装方式

### 1、Yum

> yum 是 Redhat 系列发行版的软件仓库（yum 源），Debian 系列的是 apt-get。
>
> yum 安装方式自动下载软件包所依赖的包，无需额外担心。
>
> 国内知名的镜像源：
>
> - 网易：https://mirrors.163.com/.help/centos.html
>
> - 阿里：https://mirrors.aliyun.com/

**（1）更换 yum 源**

> yum 源默认为国外镜像。
>
> yum 源的配置文件在：`/etc/yum.repos.d/` 目录下。

```
# 进入目录
cd /etc/yum.repos.d

# 备份原文件
mv CentOS-Base.repo CentOS-Base.repo.bak
		
# 下载网易镜像文件
curl -O http://mirrors.163.com/.help/CentOS7-Base-163.repo

# 修改名称
mv CentOS7-Base-163.repo CentOS-Base.repo

# 清除缓存
yum clean all
		
# 生成缓存
yum makecache
```

**（2）下载命令格式**

```
yum <操作> [选项]
```

> 常用操作：

```
清空所有缓存：clean all
重新生成包信息缓存：makecache
安装指定软件：install
安装一组软件：groupinstall
更新指定软件：update
卸载指定软件：remove
卸载一组软件：groupremove

# 使用 Yum 查找软件包 
yum search

# 列出所有可安装的软件包 
yum list 

# 列出所有可更新的软件包 
yum list updates 

# 列出所有已安装的软件包 
yum list installed 

# 列出所有已安装但不在 Yum Repository 内的软件包 
yum list extras 

# 使用 Yum 获取软件包信息 
命令：yum info 
```

> 常用选项：

```
默认确定操作：-y
只下载不安装：--downloadonly
指定下载目录：--downloaddir=/dir
```

### 2、RPM

> PRM（Redhat Pakage Manager） 也是 Redhat 系列发行版的软件包管理。
>
> 使用此方法安装的软件大多有依赖关系问题，通常一个软件需要依赖几个安装包。

格式：rpm [选项] 包名

选项：-ivh 可视化安装

> 常用选项

```
rpm -qa | grep vim 		查看软件是否安装
rpm -ql vim-*      		列出软件包安装的文件
rpm -qal | grep mysql 	查看mysql所有安装包的文件存储位置
```

**例 1：安装 vim**

① 下载 vim 的依赖包：

```
yum install vim -y --downloadonly --downloaddir=.
```

- vim-filesystem-7.4.629-8.el7_9.x86_64.rpm
- vim-common-7.4.629-8.el7_9.x86_64.rpm
- vim-enhanced-7.4.629-8.el7_9.x86_64.rpm

② 逐个安装包：

```
rpm -ivh vim-filesystem-7.4.629-5.el6.×86_64.rpm
rpm -ivh vim-common-7.4.629-5.el6.×86_64.rpm
rpm -ivh vim-enhanced-7.4.629-5.el6.×86_64.rpm
```

### 3、源码安装

```
配置：configure
编译：make
安装：make install
```



## 十、LNMP 环境搭建

> 查看系统版本：cat /etc/redhat-release
>
> 约定：
>
> - lnmp 软件源码存放地址：/tmp/lnmp
> - lnmp 软件安装地址：/usr/local/软件名

准备：

安装一些必要依赖

```
yum -y install libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel curl curl-devel openssl openssl-devel
```

```
yum -y install gcc
yum -y install gcc-c++
yum -y install libxslt-devel*
yum -y install mod_ssl
yum -y install libtool-ltdl*
yum -y install perl* 
yum -y install autoconf
```

### 1、安装 Nginx

切目录

```
mkdir /tmp/lnmp
cd /tmp/lnmp
```

下载 Nginx 源码包 [官网下载](http://nginx.org/en/download.html) 

```
curl -O http://nginx.org/download/nginx-1.20.1.tar.gz
```

解压

```
tar -zxvf nginx-1.20.1.tar.gz
```

进去

```
cd nginx-1.20.1
```

配置

> 此时会生成 `./Makefile` 文件，否则无法编译

```
./configure --with-http_stub_status_module --with-http_ssl_module
```

编译 & 安装

> 此时会生成 `/usr/local/nginx` 目录，代表安装成功。
>
> `/usr/local/nginx/sbin/nginx` 就是 nginx 服务，此时对 nginx 服务的操作很不方便，需进一步配置。

```
make && make install
```

添加 nginx 服务脚本

> vi /etc/init.d/nginx

```shell
#!/bin/bash
# nginx Startup script for the Nginx HTTP Server
# it is v.0.0.2 version.
# chkconfig: - 85 15
# description: Nginx is a high-performance web and proxy server.
#              It has a lot of features, but it's not for everyone.
# processname: nginx
# pidfile: /usr/local/nginx/logs/nginx.pid
# config: /usr/local/nginx/conf/nginx.conf
nginxd=/usr/local/nginx/sbin/nginx
nginx_config=/usr/local/nginx/conf/nginx.conf
nginx_pid=/usr/local/nginx/logs/nginx.pid
RETVAL=0
prog="nginx"
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ "${NETWORKING}" = "no" ] && exit 0
[ -x $nginxd ] || exit 0
# Start nginx daemons functions.
start() {
if [ -e $nginx_pid ];then
   echo "nginx already running...."
   exit 1
fi
   echo -n $"Starting $prog: "
   daemon $nginxd -c ${nginx_config}
   RETVAL=$?
   echo
   [ $RETVAL = 0 ] && touch /var/lock/subsys/nginx
   return $RETVAL
}
# Stop nginx daemons functions.
stop() {
        echo -n $"Stopping $prog: "
        killproc $nginxd
        RETVAL=$?
        echo
        [ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx /usr/local/nginx/logs/nginx.pid
}
# reload nginx service functions.
reload() {
    echo -n $"Reloading $prog: "
    #kill -HUP `cat ${nginx_pid}`
    killproc $nginxd -HUP
    RETVAL=$?
    echo
}
# See how we were called.
case "$1" in
start)
        start
        ;;
stop)
        stop
        ;;
reload)
        reload
        ;;
restart)
        stop
        start
        ;;
status)
        status $prog
        RETVAL=$?
        ;;
*)
        echo $"Usage: $prog {start|stop|restart|reload|status|help}"
        exit 1
esac
exit $RETVAL
```

设置权限

```
chmod 755 /etc/init.d/nginx
```

加入开启自启 

```
vi /etc/rc.local

# 在末尾增加一行
/usr/local/nginx/sbin/nginx
```

nginx 服务开机自启

```
chkconfig nginx on
```

管理 nginx 服务

```
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl reload nginx
systemctl restart nginx
```

开启防火墙端口（否则无法访问）

```
firewall-cmd --zone=public --add-port=80/tcp --permanent
```

查看防火墙端口列表

```
firewall-cmd --list-ports
```

重新加载配置

```
firewall-cmd --reload
```

### 2、安装 MySQL

安装 MySQL 源

```
yum localinstall -y http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
```

安装 MySQL 

```
yum install mysql-community-server
```

启动 MySQL 

```
systemctl start mysqld
```

获取初始密码

```
grep 'temporary password' /var/log/mysqld.log
```

得到这行A temporary password is generated for root@localhost: Jqqskhz1Wr （root@localhost:后面就是默认密码 只需复制 下一步输入密码的时候粘贴即可）

连接 MySQL

```
mysql -uroot -p
```

修改密码

```
SET PASSWORD = PASSWORD('123456//ZZZjjj'); 
# 密码必须复杂 需包含大小写特殊符号，否则无法修改成功
```

**Navicat 连接数据库**

① SSH 通道连接

② 通过主机连接

- 开放远程连接

```
use mysql;
update user set host = '%' where user = 'root';
```

- 刷新权限，使权限立即生效

```
flush privileges;
```

- 开放端口

```
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```



### 3、安装 PHP

切目录

```
cd /tmp/lnmp
```

下载

```
curl -O https://www.php.net/distributions/php-7.3.30.tar.gz
```

解压

```
tar -zxvf php-7.3.30
```

进去

```
cd php-7.3.30
```

配置

> --prefix=安装目录

```
./configure --prefix=/usr/local/php-7.3.30 --with-curl --with-freetype-dir --with-gd --with-gettext --with-iconv-dir --with-kerberos --with-libdir=lib64 --with-libxml-dir --with-mysqli --with-openssl --with-pcre-regex --with-jpeg-dir --with-freetype-dir --with-pdo-mysql --with-pdo-sqlite --with-pear --with-png-dir --with-xmlrpc --with-xsl --with-zlib --enable-fpm --enable-bcmath -enable-inline-optimization --enable-mbregex --enable-mbstring --enable-opcache --enable-pcntl --enable-shmop --enable-soap --enable-sockets --enable-sysvsem --enable-xml --enable-zip --enable-pcntl --with-curl --with-fpm-user=nginx --enable-ftp --enable-session --enable-xml --without-pear --disable-phar
```

编译 & 安装

```
make && make install
```

> 安装完成，会生成 `/usr/local/php-7.3.30` 目录。此时还不能使用 php 命令，继续配置。

添加环境变量

```
vi /etc/profile

# 在文件最后加入
PATH=$PATH:/usr/local/php-7.3.30/bin
export PATH
```

立即生效

```
source /etc/profile
```

查看PHP版本

```
php -v 
```

生成必要配置文件

```
cp php.ini-production /usr/local/php-7.3.30/etc/php.ini
cp sapi/fpm/php-fpm /usr/local/php-7.3.30/etc/php-fpm
cp /usr/local/php-7.3.30/etc/php-fpm.conf.default /usr/local/php-7.3.30/etc/php-fpm.conf
cp /usr/local/php-7.3.30/etc/php-fpm.d/www.conf.default /usr/local/php-7.3.30/etc/php-fpm.d/www.conf
```

### 4、配置 PHP 与 Nginx 协同工作

配置 nginx.conf

```
vi /usr/local/nginx/conf/nginx.conf
```

这一段都是包在 server{} 之中，如要配置多个域名，则复制粘贴多个server{}代码块。

① 重写url，隐藏 index.php 

② 解除 location ~ \.php$ {} 块的注释，并将其中的 /scripts 修改为 $document_root。

```
server {
    listen       80;
    server_name  www.abc.com abc.com;
    root /var/www/abc;
    location / {
            if (!-e $request_filename) {
                 rewrite ^/index.php(.*)$ /index.php?s=$1 last;
                 rewrite ^(.*)$ /index.php?s=$1 last;
             }
        index  index.html index.htm index.php;
    }
    location ~ \.php$ {
        root           /var/www/abc;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

配置 php-fpm

```
vi /usr/local/php-7.3.30/etc/php-fpm.d/www.conf
```

把里面的 user、group 两行，改为 nobody 或者是系统中存在的用户

```
user = nobody
group = nobody
```

启动php-fpm，载入php.ini

```
/usr/local/php-7.3.30/sbin/php-fpm -c /usr/local/php-7.3.30/etc/php.ini
```

 

注意：如果修改了 php.ini 则每次需要杀掉 php-fpm 进程再重新启动 php-fpm，PHP 的解析执行靠的是这家伙，不靠nginx。

```
ps -ef | grep php-fpm
kill -9 # 9：上一条命令查到的PID
```

### 5、负载均衡服务器

> vi /usr/local/nginx/conf/nginx.conf

```
http {
	upstream name {				# 连接池，存放提供 web 服务的服务器地址
		server 192.168.56.102 weight=5;	# 一台web服务器地址，权重 5/6
		server 192.168.56.103 weight=1;	# 一台web服务器地址，权重 1/6
	}
	
	server {
		localtion / {
			proxy_pass http://name;							# 指定代理连接池
			proxy_set_header Host $host;					# 转发请求头信息
			proxy_set_header X-Forward-For $remote_addr;	# 转发请求IP地址
		}
	}
}
```

> service nginx restart



### 6、主从服务器

① 主从服务器的配置

```
vi /etc/my.cnf

# 主从数据库的唯一标识
server-id = 1
# 主从服务的核心 log-bin 日志
log-bin = mysql-bin
```

重启服务器

```
service mysqld 
```

② 主从服务器中的表结构要保持一致。

③ 主服务器配置

创建一个专门用来同步数据的账号

```
grant replication slave on *.* to 'sync'@'%' identified by 'Pwd-123456';
```

查看状态

```
show master status;
```

| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
| :--------------- | -------- | ------------ | ---------------- | ----------------- |
| mysql-bin.000001 | 433      |              |                  |                   |

④ 从服务器配置	

> 要与 show master status; 的结果一致		

```sql
change master to master_host='192.168.56.103', master_user='sync', master_password='Pwd-123456', master_log_file='mysql-bin.000001', master_log_pos=433;
```

开启从服务

```
start slave;
```

查看从服务状态

```
show slave status;

# 以下值为 YES 则配置成功
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

⑤ 测试在主服务器上插入数据，OK。

### 7、Smaba 服务

> Windows 与 Linux 共享文件服务

```
1、关闭防火墙
systemctl stop firewalld
	
2、关闭SELinux
查看SELinux的状态：sestatus
关闭SELinux：setenforce 0

3、安装samba和samba-client
yum install -y samba samba-client

4、添加用户
useradd readonly
pdbedit -a readonly

5、启动samba服务
添加开机启动：chkconfig smb on
立即启动samba：service smb start

6、测试
在Windows系统中的文件搜索中输入：\\IP

7、自定义共享目录
(1)	创建共享目录，并修改权限
	mkdir -p /var/www/php
	chomd -R 777 /var/www/php
(2)	添加配置文件：/etc/samba/smb.conf
	[php]					#共享目录名称
		path = /var/www/php	#共享目录位置
		browseable = yes	#是否可以浏览
		writable = yes		#是否可以写入
		public = no			#是否公开
```

### 8、FTP 服务

> 利用 SFTP 协议利用 SSH 可以直接连接，无需搭建 FTP 服务。



