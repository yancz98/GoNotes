[toc]

## 一、安装 / 配置

### 1、安装 CentOS_7

- [VirtualBox 下载](https://www.virtualbox.org/)
- [CentOS 下载](https://www.centos.org/download/) （推荐使用阿里云镜像下载）
- [安装教程](https://www.cnblogs.com/Young-wind/p/5852180.html)

注：若安装虚拟机时，只有32位的可供选择，可能是 BIOS 没有开启虚拟技术。
```
# 启用虚拟机功能
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

### 2、网络配置

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
- 桥接网卡：这个网络完全可以共享主机网络，主机网络发生变化时，也跟随变化，ip也随之变动。
- 仅主机(Host-Only)网络：这个网络也可以用来主机访问虚拟机以及虚拟机上web服务，但是虚拟机不可访问外网。



## 二、目录结构

```
/：根目录
	/bin -> usr/bin：	程序的启动文件（所有命令）
	/boot：				启动目录
	/dev：				设备目录
	/etc：				可编辑文本配置
	/home：				用户目录
	/lib -> usr/lib：	库
	/lib64 -> usr/lib64：64位操作系统的库
	/lost+found：		非正常关机留下的文件（无）
	/media：				媒体
	/mnt：				安装临时文件
	/opt：				可选程序
	/proc：				虚拟文件系统目录
	/root：				root目录
	/run：
	/sbin -> usr/sbin：	超级用户的可执行程序
	/srv：				可访问数据库目录
	/sys：				sysfs文件系统的挂载点
	/tmp：				公用临时文件存储点
	/usr：				Unix系统资源，存放一些应用程序
	/var：				存放运行时需改变数据的文件，如：日志文件
	/selinux：			保证系统安全（无）
		getenforce：		获取状态
		setenforce = 1：	设置状态
		永久关闭：修改配置文件/etc/selinux/config -> SELINUX=disabled
```



## 三、常用命令

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

注：不指定选项和参数，默认1分钟后关闭电脑
```

### 2、工作目录

**（1）切换工作目录（change directory）**

```
cd [目录]
	
#		root的根目录（或普通用户的home目录）
-		打印并切换回上一个工作目录
~		当前用户的home目录
cd		不加参数，也是切换到home目录
.		当前目录
..		上级目录
/		根目录
/dir	切换到指定目录

# 打印当前工作目录
pwd
```

**（2）显示目录列表（list）**

```
ls [选项] [目录]

选项：
	-a	显示所有文件，包括隐藏文件（以./ 和 ../ 开头的文件）
	-l	列表显示文件，会显示文件的所有信息 <==> ll
	-lh	带单位显示文件大小
	
参数：
	默认打印当前目录列表
	
结果说明：
	文件类型及权限 | 引用数 | 用户 | 组 | 大小 | 月 | 日 | 年/时间 | 文件名
	
文件类型说明：
		-	普通文件
		d	目录
		b	块设备
		c	字符设备
		l	链接
		s	套接字
		p	管道	
		
# 以树形结构打印目录
tree [目录]
```

### 3、其它

```
自动补全（命令、参数）：tab
清屏：clear
显示历史命令：history
结束进程：Ctrl+c

重定向：
    将本该显示在终端上的内容 输出、追加到指定文件中
    >	表示输出到文件，会覆盖文件原有的内容
    >>	表示追加，追加到文件的末尾

    另一种创建文件的方式：
    echo text > text.txt
    touch 只能创建一个空文件，而重定向方式可以创建带内容的文件
    还可以将其他命令的输出结果追加到文件中：
    tree laravel >> laravel_directory_structure.txt

管道：|
    Linux允许将一个命令的输出通过管道作为另一个命令的输入。
    常与管道搭配使用的命令有：
    more
    grep

通配符：
    *
    ?	匹配1个字符
    []	字符组中的任何一个（如：[abc] [1-9]）
```



## 四、终端编辑工具

> 终端编辑工具有：vi、vim、emacs 等。系统默认自带 vi 编辑器。

### 1、安装

```
# 安装 vim 
yum install -y vim
```

### 2、使用

**（1）打开文件**

> 若文件存在，则直接打开。
>
> 若文件不存在，则新建文件，不修改则不会创建空文件。

```
vi filename
```

**（2）工作模式**

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
> 在正常模式下，输入':'即可进入命令模式。

```
:w		保存文件；
:q		退出软件；
:x		保存退出（:wq）；
:!		强制操作；
:e!		放弃修改；
```

- 可视模式（正常模式下操作）

```
v		可视字符模式（通过左右逐个选择字符）
V		可视行模式（通过上下选择行数）
Ctrl+v	可视块模式（通过上下左右(或 kjhl)选择行数和列数）

# 多行注释
① Esc 进入正常模式；
② Ctrl+v 进入可视化块模式；
③ 利用上下左右调整需要注释的行数及列数；
④ Shift+i 进入插入模式，插入注释符：“#”；
⑤ 再次按 Esc，即可完成多行注释。

# 取消多行注释
① Esc 进入正常模式；
② Ctrl+v 进入可视化块模式；
③ 利用上下左右调整需要注释的行数及列数；
④ 按 d 即可取消注释。
```

### 3、使用技巧

**（1）打开文件**

```shell
# 打开文件，光标定位到第n行
vi filename +n

# 打开文件，光标定位到最后一行
vi filename +
```

**（2）光标定位**（正常模式下操作）

```
gg		首行
GG		尾行
ngg		第n行
0		行首
^		行首第一个非空字符
$		行尾
kjhl	上下左右
```

**（3）复制粘贴**（正常模式下操作）

```
yy		复制光标所在行
dd		剪切光标所在行
p		粘贴缓冲区的内容
nyy		复制光标开始的n行
ndd		剪切光标开始的n行
```

**（4）操作回退**（正常模式下操作）

```
u			撤销操作
Ctrl+r		反撤销
shift + zz	保存编辑（:x）
```

**（5）查找替换**（命令模式下操作）

```
# 查找，回车查找，n向前查找，N向后查找
:?

# 替换，g代表全局替换，可选
:%s/查找内容/替换内容/[g]

# 替换从起始行到结束行查找到的内容
:起始行,结束行s/查找内容/替换内容/[g]	
```

**（6）基本配置**（命令模式下操作）

```
:set nu							显示行号
:set nonu						取消显示行号
:set tabstop=4					设置缩进字符数
:set fileeccodings=utf-8,gbk	设置字符集
```

**（7） 配置文件**

> 以上命令行配置仅针对本次打开文件有效，永久生效需添加配置文件： `~/.vimrc` 。

```
vi ~/.vimrc

set nu
```

> 立即生效：
>

```
source ~/.vimrc
```



## 五、文件与目录操作

### 1、文件内容查看

**（1）cat**

> 显示文件的所有内容

```
cat [选项] 文件名
    -b	对非空行进行编号
    -n	对所有行进行编号（包含空行）
    
tac [选项] 文件名：倒序显示文件的所有内容
```

**（2）head / tail**

> 显示文件开头 / 结尾的 n 行

```
head -n 文件名：显示文件开头的 n 行（默认 n=10）
tail -n 文件名：显示文件结尾的 n 行（默认 n=10）
```

**（3）more**

> 分页显示文件的内容（留痕浏览）

```
more [选项] 文件名

说明：
    当内容显示满屏时停止
    空格向下翻页
    回车向下显示一行
    q退出（结束分页）
	
其它命令 | more ：分页显示其它命令执行的结果
```

**（4）less**

> 与more命令功能一样，增加了一个上下翻行显示（无痕浏览）

```
less [选项] 文件名

其它命令 | less：分页显示其它命令执行的结果
```

### 2、文件整体操作

**（1）创建文件**

```
touch [选项] 文件1 文件2 ...

说明：
	文件不存在时，创建。
	文件存在时，更新最后修改时间。
```

**（2）拷贝文件**

```
cp [选项] 源文件 目标文件

选项：
	-i	覆盖前提示
	-r	递归复制目录及其子目录内的所有内容（拷贝目录时必须）
	-T	将目标目录视作普通文件
	
scp：远程拷贝文件
scp -P port 文件名 user@remote:Dir/文件名	
```

**（3）删除文件**

```
rm [选项] 文件1 文件2 ...

选项：
	-d	删除空目录
	-r	递归删除目录，即该目录下的所有文件都会删除（删除目录时必须）
	-f	强制删除目录，不会有错误信息
```

**（4）移动文件**（重命名文件或文件夹）

```
mv [选项] 源文件 目标文件

选项：
	-f, --force			覆盖前不询问
	-i, --interactive	覆盖前询问
	-n, --no-clobber	不覆盖已存在文件
	指定 -i、-f、-n 中的多个时，仅最后一个生效。
```

**（5）创建目录**

```
mkdir [选项] 目录

选项：
	-p	不报错错误，根据需要创建父目录
```

**（6）删除空的目录**

```
rmdir [选项] 目录1 目录2 ...
```

**（7）创建链接文件**

```
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

### 3、文件搜索定位

**（1）grep**

> 在每个 FILE 或是标准输入中查找 PATTERN，默认的 PATTERN 是一个基本正则表达式。

```
grep [选项] PATTERN [FILE ...]
-----
其他返回结果 | grep [选项] PATTERN

选项：
	-i	不区分大小写
	-v	显示不包含匹配文本的所有行（排除）
	-n	显示匹配行及行号
```

**（2）find**

```
find [-H] [-L] [-P] [-Olevel] [-D help|tree|search|stat|rates|opt|exec] [path...] [expression]

find [目录] [条件] [动作]

目录：所要搜索的目录及其所有子目录。默认为当前目录。

条件：所要搜索的文件特征。

动作：对搜索结果进行特定的处理。

选项：
    -name：指定文件名，可以通过*模糊匹配
    -type：指定文件类型（b/c/d/p/l/f）
    -size：指定文件大小，+表示大于，-表示小于
    -user：指定用户
    -group：指定组
    -mtime/atime/ctime：指定修改/访问/创建时间，单位为天，+表示几天前，-表示几天内
    -mmin/amin/cmin：同上，单位为分钟

例：
	find：查找当前目录下的所有文件。
	find -name '*.log' ：查找 .log 文件。
```

**（3）whereis**

```
whereis [选项] 文件

选项：
	-b         只搜索二进制文件
    -B <目录>  定义二进制文件查找路径
    -m         只搜索 man 手册
    -M <目录>  定义 man 手册查找路径
    -s         只搜索源代码
    -S <目录>  定义源代码查找路径
    -f         终止 <目录> 参数列表
    -u         搜索不常见记录
    -l         输出有效查找路径
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

### 4、文件压缩与解压

**（1）gzip 压缩文件**

> 只能压缩单个文件，不压缩目录

```
gzip [选项] [文件]

选项：
	-d	等价于 gunzip 解压文件
	
说明：
	执行命令会生成file.gz，删除原文件
```

**（2）gunzip 解压文件**

```
gunzip [选项] [文件]

说明：
	解压文件，删除原压缩文件
```

**（3）bzip2 / bunzip2：压缩、解压文件**

```
bzip2  [选项] [文件]
bunzip2 [选项] [文件]

选项：
	-k	压缩或解压后保留原文件

说明：
	用法与 gzip/gunzip 相同，只是多了一个 -k 选项；
	使用 bzip2 压缩文件的后缀为 bz2。
```

**（4）tar** 

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

### 5、文件权限管理

**（1）ls -lh 打印文件列表信息**

```
结果说明：
	文件类型及权限 | 引用数 | 用户 | 组 | 大小 | 月 | 日 | 年/时间 | 文件名
	
例如：	
	-rw-r--r--. 1 root root 7.4M 9月  23 16:19 tree.txt

文件类型及权限：[-rw-r--r--.]
	1		文件类型（-：文件；b：块设备；c：字符设备；d：目录；l：链接；s：套接字）
	2/3/4	文件所有者的读(r)/写(w)/执行(x)权限，-表示无权限
	5/6/7	文件所有者所属组的读/写/执行权限
	8/9/10	其它用户的读/写/执行权限
	.		表示启用了selinux，空表示没有开启

文件类型说明：
    -	普通文件
    d	目录
    b	块设备
    c	字符设备
    l	链接
    s	套接字
    p	管道	
```

**（2）chmod 修改文件权限**

```
chmod [选项] [模式|八进制模式] 文件

模式：
	r	可读
    w	可写
    x	可执行
	+	添加权限
    -	去除权限
    =	设置权限
    u	用户
    g	组
    o	其它
	777	八进制模式，777 => 111 111 111 这三组二进制分别代表了三个角色的三种权限
	
实例：
	chmod +x file				给文件添加可执行权限（所有者、组、其它）
	chmod u+x file				给文件所有者添加可执行权限
	chmod u=rwx,g=rx,o=- file	分别设置权限
	chmod 777 file				用八进制数来设置权限	
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
	i	表示忽略
	+	表示添加
	-	表示去掉
```

**（5）lsattr**

> 查看使用chattr设置的文件属性

```
lsattr [-RVadlv] [files...]
```



## 六、用户与用户组

### 1、用户管理

（1）查看当前登录用户

```
whoami
```

（2）查看所有用户

```
cat /etc/passwd

格式：
	root:x:0:0:root:/root:/bin/bash
说明：
	用户名:密码:uid:gid:home目录:命令解析文件
```

（3）新增用户

```
useradd 用户名 [选项]

选项：
	-d		指定用户home目录，不指定则在/home下创建一个与用户名相同的目录
	-u		指定用户ID，必须大于500
	-s		指定用户执行的shell，若用户已经创建，用以下方式禁止登录
	chsh 用户名 -s /sbin/nologin	
```

（4）删除用户

```
userdel 用户名

说明：
	彻底删除用户还需要删除 `/home/用户名` 和 `/var/mail/用户名` 目录
```

（5）设置用户密码

```
passwd [用户名]

说明：
	① 若不指定用户则修改当前登录用户的密码
	② 查看所有用户密码 /etc/shadow
```

（6）切换用户

```
su [用户名]

说明：
	不指定默认切换到root用户
```

（7）sudo

```
sudo 
	普通用户执行root用户的命令，不想切换用户，可以在命令前加sudo
```

（8）特殊标识

```
#	超级用户
$	普通用户
~	用户home目录
```

### 2、用户组管理

（1）查看系统中的所有组

```
cat /etc/group
```

（2）添加用户组

```
groupadd 组名
```

（3）删除用户组

```
groupdel 组名
```

（4）向用户组中添加或删除用户

```
gpasswd [选项] 用户名 组名

选项：
    -a	添加
    -d	删除
```

（5）设置/修改文件所属组

```
chgrp 组名 文件名
```

（6）设置/修改文件所属者[及组名]

```
chown 用户[:组名] 文件名

注：
    修改用户及组的时候，可以使用UID或GID；
    加上 -r 选项可以递归修改子目录的用户及组。
```



## 七、网络相关设置

### 1、查看网卡信息

（1）ping

> 通常用于检测网络设备的连通性。

```
ping IP/域名：

选项：
	-c	指定发送请求次数，不指定无限次发送
```

（2）ip addr / ifconfig

> 查看或设置网卡信息（IP）

```
ifconfig [-a] [-v] [-s] <interface> [[<AF>] <address>]

ifconfig eth0 down	关闭网卡，等价于 ifdown eth0
ifconfig eth0 up	开启网卡，等价于 ifup eth0

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



## 八、进程与服务

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



## 十一、Cron

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

```
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

