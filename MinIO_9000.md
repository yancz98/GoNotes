## 一、概述

### 1、Install & Deploy

```shell
curl -O https://dl.min.io/server/minio/release/linux-amd64/minio
mv minio /usr/local/bin/
chmod +x minio
```

#### （1）单节点单驱动（SNSD）

```shell
> minio server /data/minio --address :9000 --console-address :9001

...
Status:         1 Online, 0 Offline. 
API: http://172.16.56.107:9000  http://127.0.0.1:9000     
RootUser: minioadmin 
RootPass: minioadmin 
Console: http://172.16.56.107:41685 http://127.0.0.1:41685   
RootUser: minioadmin 
RootPass: minioadmin 
...
```

说明：直接访问 http://127.0.0.1:9000 也可以，API 内置了 Console 页面。

#### （2）单节点多驱动（SNMD）

> MinIO 强烈建议使用 XFS 格式磁盘的直连 JBOD 阵列以获得最佳性能。
>
> 确保 MinIO 的所有服务器驱动都是相同类型（NVMe、SSD、HDD）且具有相同的容量（如：1 TB）。

##### 先决条件

~~~shell
# 1 添加 [b, c, d, e] 四块磁盘

# 2 列出块设备信息
> lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   50G  0 disk 
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   49G  0 part 
  ├─centos-root 253:0    0   47G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0   10G  0 disk 
sdc               8:32   0   10G  0 disk 
sdd               8:48   0   10G  0 disk 
sde               8:64   0   10G  0 disk 
sr0              11:0    1 1024M  0 rom   

# 3 格式化磁盘
mkfs.xfs -f /dev/sdb -L DISK1
mkfs.xfs -f /dev/sdc -L DISK2
mkfs.xfs -f /dev/sdd -L DISK3
mkfs.xfs -f /dev/sde -L DISK4
...
meta-data=/dev/sde               isize=512    agcount=4, agsize=655360 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2621440, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
...

# 4 磁盘挂载（保证重启自动挂载）
> vi /etc/fstab

`
# <file system>  <mount point>  <type>  <options>         <dump>  <pass>
  LABEL=DISK1    /mnt/disk1     xfs     defaults,noatime  0       2
  LABEL=DISK2    /mnt/disk2     xfs     defaults,noatime  0       2
  LABEL=DISK3    /mnt/disk3     xfs     defaults,noatime  0       2
  LABEL=DISK4    /mnt/disk4     xfs     defaults,noatime  0       2
`

# 不需重启生效的方法（需预先创建好挂载点）
mkdir -p /mnt/disk1
mkdir -p /mnt/disk2
mkdir -p /mnt/disk3
mkdir -p /mnt/disk4

# 挂载 fstab 中的所有文件系统
> mount -a

# 5 列出块设备信息
> lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   50G  0 disk 
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   49G  0 part 
  ├─centos-root 253:0    0   47G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0   10G  0 disk /mnt/disk1
sdc               8:32   0   10G  0 disk /mnt/disk2
sdd               8:48   0   10G  0 disk /mnt/disk3
sde               8:64   0   10G  0 disk /mnt/disk4
sr0              11:0    1 1024M  0 rom 
~~~

##### 启动

```shell
# 使用扩展符号 /mnt/disk{1...4} 来表示一系列连续的驱动器
> minio server /mnt/disk{1...4}  --address :9000 --console-address :9001
...
Status:         4 Online, 0 Offline. 
API: http://172.16.56.101:9000 http://127.0.0.1:9000
RootUser: minioadmin 
RootPass: minioadmin 
Console: http://172.16.56.101:35994 http://127.0.0.1:35994
RootUser: minioadmin 
RootPass: minioadmin
...
```

#### （3）多节点多驱动（MNMD）

> MNMD 部署提供企业级性能、可用性和可扩展性，是生产环境推荐的部署。
>
> MNMD 部署支持[纠删码](https://min.io/docs/minio/linux/operations/concepts/erasure-coding.html#minio-ec-parity)配置，可以容忍部署中多达一半的节点或驱动器丢失，同时继续为**读取**操作提供服务。（不能写）
>
> MinIO 强烈建议为部署中的所有节点选择基本相似的硬件配置。确保硬件（CPU、内存、主板、存储适配器）和软件（操作系统、内核设置、系统服务）在所有节点上保持一致。

##### 先决条件

- 网络和防火墙：

  - 每个节点都应具有对部署中每个其他节点的完全双向网络访问权限。
  - 打开默认的 MinIO 服务器 API 端口 `9000`。
  - 部署中的所有 MinIO 服务器必须使用相同的监听端口。
  - MinIO 强烈建议使用负载均衡器来管理与集群的连接。

- 顺序主机名：

  - MinIO 需要在创建服务器池时使用扩展符号 `{x...y}` 来表示一系列连续的 MinIO 主机。

  - 创建 DNS 主机名映射：

    ~~~shell
    # 每台服务器均添加 hosts 配置
    > vi /etc/hosts
    
    ```
    192.168.56.101  minio1.com
    192.168.56.102  minio2.com
    192.168.56.103  minio3.com
    192.168.56.104  minio4.com
    ```
    ~~~

- 具有顺序挂载的本地 JBOD 存储：
  
  - 每个节点都按 SNMD 配置好磁盘挂载。

##### 部署分布式 MinIO

- 在每个节点上安装 MinIO，并做好相关配置。

- 运行集群：

  ```shell
  # 每台服务器均执行
  # 账号密码为各节点之间通讯的凭证（需保持一致）
  export MINIO_ROOT_USER=admin
  export MINIO_ROOT_PASSWORD=password
  minio server http://minio{1...2}.com/mnt/disk{1...4} --console-address :9001
  
  # 注意：
  # 以上启动方式才是【集群】的正确启动方式
  # 以下启动方式为【扩容】
  minio server http://192.168.56.101:9000/mnt/disk{1...4} http://192.168.56.102:9000/mnt/disk{1...4}
  ```

##### Nginx 负载均衡

```shell
# ...
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile           on;
    keepalive_timeout  65;
    # 默认 1M，特殊值 0 不限制
    client_max_body_size 0;

    upstream minio_api {
        server 192.168.56.104:9000;
        server 192.168.56.105:9000;
	}

    upstream minio_console {
        server 192.168.56.104:9001;
        server 192.168.56.105:9001;
 	}
 
 	server {
 	    listen       9000;
 	    server_name  localhost;
 
 	    location / {
            proxy_set_header Host $http_host; # S3 API 请求必须
            
            proxy_pass http://minio_api; 
 	    }
    }

    server {
        listen  9001;
        server_name  localhost;

        location / {
            proxy_set_header Host $http_host;
            
            # To support websockets in MinIO versions released after January 2023
            
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        
            proxy_pass http://minio_console; 
        }
    }
}
```



#### （4）以系统服务启动

##### ① 创建 `systemd` 服务文件

> `vi /etc/systemd/system/minio.service`

```shell
[Unit]
Description=MinIO
Documentation=https://min.io/docs/minio/linux/index.html
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio

[Service]
WorkingDirectory=/usr/local

#User=minio-user
#Group=minio-user
ProtectProc=invisible

EnvironmentFile=-/etc/default/minio
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES

# Let systemd restart this service always
Restart=always

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65536

# Specifies the maximum number of threads this process can create
TasksMax=infinity

# Disable timeout logic and wait until process is stopped
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target

# Built for ${project.name}-${project.version} (${project.name})
```

##### ② 创建服务环境文件

> `vi /etc/default/minio`

```shell
# Set the hosts and volumes MinIO uses at startup
# The command uses MinIO expansion notation {x...y} to denote a
# sequential series.
#
# The following example covers four MinIO hosts
# with 4 drives each at the specified hostname and drive locations.
# The command includes the port that each MinIO server listens on
# (default 9000)

# SNSD
# MINIO_VOLUMES="/data/minio"

# SNMD
# MINIO_VOLUMES="/mnt/disk{1...4}"

# MNMD
MINIO_VOLUMES="http://minio{1...2}.com/mnt/disk{1...4}"

# Set all MinIO server options
#
# The following explicitly sets the MinIO Console listen address to
# port 9001 on all network interfaces. The default behavior is dynamic
# port selection.

MINIO_OPTS="--console-address :9001"

# Set the root username. This user has unrestricted permissions to
# perform S3 and administrative API operations on any resource in the
# deployment.
#
# Defer to your organizations requirements for superadmin user name.

MINIO_ROOT_USER=admin

# Set the root password
#
# Use a long, random, unique string that meets your organizations
# requirements for passwords.

MINIO_ROOT_PASSWORD=password

# Set to the URL of the load balancer for the MinIO deployment
# This value *must* match across all MinIO servers. If you do
# not have a load balancer, set this value to to any *one* of the
# MinIO hosts in the deployment as a temporary measure.
#MINIO_SERVER_URL="https://minio.example.com:9000"
```

##### ③ 运行 MinIO 服务器进程

```shell
sudo systemctl start minio.service

# 查看日志
journalctl -f -u minio.service
```

### 2、管理已有的 MinIO 部署

#### （1）扩容

#### （2）升级

#### （3）退役



## 二、核心概览

### 1、纠删码（Erasure Coding）

### 2、擦除集（Erasure Sets）

### 3、纠删码奇偶校验（Erasure Code Parity (EC:N)）

### 4、多副本与纠删码对比

## 三、访问管理（策略管理）

### 1、Policies 文件结构

```json
{
    "Version" : "2012-10-17",
    "Statement" : [
        {
            "Effect" : "Allow",                     // 效果：允许
            "Action" : [ "s3:<ActionName>", ... ],  // 可执行的操作
            "Resource" : "arn:aws:s3:::*",          // 可操作的资源（桶/对象）
            "Condition" : { ... }                   // 条件
        },
        {
            "Effect" : "Deny",                      // 效果：拒绝
            "Action" : [ "s3:<ActionName>", ... ],  // 可执行的操作
            "Resource" : "arn:aws:s3:::*",          // 可操作的资源（桶）
            "Condition" : { ... }                   // 条件
        }
    ]
}
```

说明：

- `Deny` 覆盖 `Allow`：如果一个 Action/Resource 同时定义了 Deny 和 Allow 规则，则 Deny 规则覆盖 Allow 规则（Deny 规则优先）。

- Amazon 资源名称（ARN）格式：`arn:partition:service:region:namespace:relative-id`。
  - arn：Amazon 资源名称。
  - partition：aws（标准区域），aws-cn（中国区域），aws-us-gov（AWS GovCloud (US) 区域）。
  - service：s3。
  - relative-id：`bucket-name` 或 `bucket-name/object-key` （可以使用通配符）。
  - 通配符：星号 (`*`) 表示 0 个或多个字符的任意组合，问号 (`?`) 表示任何单个字符。
  - ARN 格式可以简写成：`arn:aws:s3:::bucket_name/key_name`。



### 2、S3 策略 Action

##### `s3:*` 所有操作

#### （1）桶操作 API

##### `s3:CreateBucket` 创建桶

S3 桶命名规则：

- 3 ~ 63 字符。
- 只能由小写字母、数字、句点 (.) 和连字符 (-) 组成。
- 必须以字母或数字开头和结尾。
- 不得包含两个相邻的句点 (.)，如：`*..*` 或 `*.-*` 都不行。
- 不得采用 IP 地址格式（例如，192.168.5.4）。
- 不得以前缀 `xn--` 开头。
- 不得以后缀 `-s3alias` 结尾。
- 在分区内必须是唯一的。

##### `s3:DeleteBucket` 删除桶

注：只能删除空桶（没有对象）。

##### `s3:ForceDeleteBucket`  强制删除桶

注：可以删除非空的桶。

##### `s3:GetBucketLocation` 获取桶所在的区域

##### `s3:ListAllMyBuckets` 获取我的所有桶的列表

##### `s3:DeleteObject` 删除对象

注：删除一个对象的空版本（如果有的话）并插入一个删除标记，它成为该对象的最新版本。如果没有空版本，Amazon S3 不会删除任何对象，但仍会响应命令成功。

##### `s3:GetObject` 获取对象

##### `s3:ListBucket` 获取桶中的对象列表

注：返回存储桶中的部分或全部（最多 1,000 个）对象。

##### `s3:PutObject` 将对象添加到桶

##### `s3:PutObjectTagging` 设置对象标签

##### `s3:GetObjectTagging` 获取对象标签

##### `s3:DeleteObjectTagging` 删除对象标签

#### （2）桶配置 API

##### `s3:GetBucketPolicy` 获取桶策略

##### `s3:PutBucketPolicy` 设置桶策略

##### `s3:DeleteBucketPolicy` 删除桶策略

##### `s3:GetBucketTagging` 获取桶标签

##### `s3:PutBucketTagging` 设置桶标签

#### （3）分段上传 API

##### `s3:AbortMultipartUpload` 中止分段上传

##### `s3:ListMultipartUploadParts` 分段上传已上传的部分

##### `s3:ListBucketMultipartUploads` 列出正在进行的分段上传

#### （4）版本控制和保留

##### `s3:PutBucketVersioning` 设置桶的版本控制状态

- Enabled：为存储桶中的对象启用版本控制。添加到存储桶中的所有对象都会收到一个唯一的版本 ID。
- Suspended（不会自动删除文件版本）：中止存储桶中对象的版本控制。添加到存储桶中的所有对象的版本 ID 均为空。

##### `s3:GetBucketVersioning` 获取桶的版本控制状态

##### `s3:DeleteObjectVersion` AWS 没找到

##### `s3:DeleteObjectVersionTagging` AWS 没找到

##### `s3:GetObjectVersion` AWS 没找到

##### `s3:BypassGovernanceRetention` 绕过监管模式

可对在监管模式下锁定的对象版本执行操作。

##### `s3:PutObjectRetention`

##### `s3:GetObjectRetention`

##### `s3:GetObjectLegalHold`

##### `s3:PutObjectLegalHold`

##### `s3:GetBucketObjectLockConfiguration`

##### `s3:PutBucketObjectLockConfiguration`

#### （5）桶通知 API

#### （6）对象生命周期管理 API

#### （7）对象加密 API

#### （8）桶复制 API



### 3、S3 策略 Condition

> 格式

```json
"Condition" : { "{condition-operator}" : { "{condition-key}" : "{condition-value}" }}
```



### 4、Admin 策略 Action



### 5、Admin 策略 Condition



## 四、MinIO 客户端

### 1、下载

```shell
curl https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
mv mc /usr/local/bin

# 为我的 minio （与 S3 兼容的）服务设置别名
# 以后就可以直接用【minio】访问服务，不需每次都带上秘钥校验
mc alias set minio http://172.16.56.101 dsognniq8Yl0X0GZ TXCMAREL2nvQj6rpXInPkxAduR7miBlp

# 测试连接
# 注意：指定 ACCESS_KEY，SECRET_KEY 的用户必须有权对服务执行操作
> mc admin info minio
●  minio1.com:9000
   Uptime: 1 day 
   Version: 2023-04-13T03:08:07Z
   Network: 2/2 OK 
   Drives: 4/4 OK 
   Pool: 1

●  minio2.com:9000
   Uptime: 1 day 
   Version: 2023-04-13T03:08:07Z
   Network: 2/2 OK 
   Drives: 4/4 OK 
   Pool: 1

Pools:
   1st, Erasure sets: 1, Drives per erasure set: 8

39 MiB Used, 2 Buckets, 2 Objects, 3 Versions
8 drives online, 0 drives offline
```

### 2、[`mc`](https://min.io/docs/minio/linux/reference/minio-mc.html)

#### （1）别名

```shell
# 语法：
mc alias set ALIAS HOSTNAME ACCESS_KEY SECRET_KEY

# 为我的 minio 服务设置一个别名
mc alias set minio http://172.16.56.101 dsognniq8Yl0X0GZ TXCMAREL2nvQj6rpXInPkxAduR7miBlp

# 列出所有别名
mc alias list

# 删除别名
mc alias remove <ALIAS>

# alias 配置文件： ~/.mc/config.json
{
    "version": "10",
    "aliases": {
        "gcs": {
            "url": "https://storage.googleapis.com",
            "accessKey": "YOUR-ACCESS-KEY-HERE",
            "secretKey": "YOUR-SECRET-KEY-HERE",
            "api": "S3v2",
            "path": "dns"
        },  
        "minio": {
            "url": "http://172.16.56.101:9000",
            "accessKey": "ucs",
            "secretKey": "UbV1bUM5ZUfYkMvqNWJ12ASdLWAj3xiw",
            "api": "s3v4",
            "path": "auto"
        },  
        "play": {
            "url": "https://play.min.io",
            "accessKey": "Q3AM3UQ867SPQQA43P2F",
            "secretKey": "zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG",
            "api": "S3v4",
            "path": "auto"
        },  
        "s3": {
            "url": "https://s3.amazonaws.com",
            "accessKey": "YOUR-ACCESS-KEY-HERE",
            "secretKey": "YOUR-SECRET-KEY-HERE",
            "api": "S3v4",
            "path": "dns"
        }   
    }   
}
```

#### （2）桶匿名策略

可以对单个桶及其内容设置匿名策略。
具有匿名策略的存储桶允许公共访问，客户端可以在其中执行策略授权的任何操作而无需身份验证。

```
mc anonymous [FLAGS] set-json FILE TARGET
mc anonymous [FLAGS] get TARGET
mc anonymous [FLAGS] get-json TARGET
mc anonymous [FLAGS] list TARGET
```

### 3、[`mc admin`](https://min.io/docs/minio/linux/reference/minio-mc-admin.html)

#### （1）用户管理

用户账号与用户 AccessKey 的区别：

- 用户账号可以登录后台管理系统，并且可以当作 AccessKey、SecurityKey 使用。
- Access 只能用在操作鉴权。

```shell
# 用户管理
NAME:
  mc admin user - manage users

USAGE:
  mc admin user COMMAND [COMMAND FLAGS | -h] [ARGUMENTS...]

COMMANDS:
  add      add a new user
  disable  disable user
  enable   enable user
  remove   remove user
  list     list all users
  info     display info of a user
  policy   export user policies in JSON format
  svcacct  manage service accounts
  sts      manage STS accounts 

# 用户 AccessKeys 管理
# 当前登录用户下创建的 AccessKey 归属于当前用户，权限也继承当前用户
NAME:
  mc admin user svcacct - manage service accounts

USAGE:
  mc admin user svcacct COMMAND [COMMAND FLAGS | -h] [ARGUMENTS...]

COMMANDS:
  add         add a new service account
  ls, list    list services accounts
  rm, remove  remove a service account
  info        display service account info
  edit, set   edit an existing service account
  enable      enable a service account
  disable     disable a service account   
  
# 用户的临时 Account 管理（暂不知道用在哪）
NAME:
  mc admin user sts info - display temporary account info

USAGE:
  mc admin user sts info ALIAS STS-ACCOUNT  
```

#### （2）组管理

```shell
NAME:
  mc admin group - manage groups

USAGE:
  mc admin group COMMAND [COMMAND FLAGS | -h] [ARGUMENTS...]

COMMANDS:
  add      add users to a new or existing group
  remove   remove group or members from a group
  info     display group info
  list     display list of groups
  enable   enable a group
  disable  disable a group  
```

#### （3）策略管理

```shell
NAME:
  mc admin policy - manage policies defined in the MinIO server

USAGE:
  mc admin policy COMMAND [COMMAND FLAGS | -h] [ARGUMENTS...]

COMMANDS:
  create    create a new IAM policy
  remove    remove an IAM policy
  list      list all IAM policies
  info      show info on an IAM policy
  attach    attach an IAM policy to a user or group
  detach    detach an IAM policy from a user or group
  entities  list policy association entities # 列出策略关联的实体      
```

#### （4）修复损坏

```
USAGE:
  mc admin heal [FLAGS] TARGET 
```

#### （5）查看服务器日志

```
USAGE:
  mc admin logs [FLAGS] TARGET [NODENAME]
```

#### （6）服务管理

```shell
NAME:
  mc admin service - restart, stop and unfreeze a MinIO cluster

USAGE:
  mc admin service COMMAND [COMMAND FLAGS | -h] [ARGUMENTS...]

COMMANDS:
  restart   restart a MinIO cluster
  stop      stop a MinIO cluster
  unfreeze  unfreeze S3 API calls on MinIO cluster 
```

