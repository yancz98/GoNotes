## 一、安装 & 部署

> CentOS 7

### 1、安装和配置必须的依赖项

在 CentOS 7上，下面的命令也会在系统防火墙中打开 HTTP、HTTPS 和 SSH 访问。这是一个可选步骤，如果您打算仅从本地网络访问极狐GitLab，则可以跳过它。

```
sudo yum install -y curl policycoreutils-python openssh-server perl
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld
```

（可选）下一步，安装 Postfix 以发送电子邮件通知。如果您想使用其他解决方案发送电子邮件，请跳过此步骤并在安装极狐GitLab 后[配置外部 SMTP 服务器](https://docs.gitlab.cn/omnibus/settings/smtp.html)。

```
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

在安装 Postfix 的过程中可能会出现一个配置界面，在该界面中选择“Internet Site”并按下回车。把“mail name”设置为您服务器的外部 DNS 域名并按下回车。如果还有其它配置界面出现，继续按下回车以接受默认配置。

### 2、下载/安装 GitLab

```
# 配置极狐GitLab 软件源镜像
curl -fsSL https://packages.gitlab.cn/repository/raw/scripts/setup.sh | /bin/bash

# 开始安装
sudo EXTERNAL_URL="https://gitlab.example.com" yum install -y gitlab-jh
```

### 3、访问极狐GitLab 实例并登录



### 4、卸载 Linux 软件包（Omnibus）

```
# （可选）在删除极狐GitLab 软件包（使用 apt 或 yum）之前，删除由 Omnibus GitLab 创建的所有用户和群组：
sudo gitlab-ctl stop && sudo gitlab-ctl remove-accounts

# 如果您在删除帐户或组时遇到问题，请手动运行 userdel 或 groupdel 来删除它们。您可能还想从 /home/ 中手动删除剩余的用户主目录。

# 要保留您的数据（代码库、数据库、配置），请停止极狐GitLab 并删除其 supervision 进程：
sudo systemctl stop gitlab-runsvdir
sudo systemctl disable gitlab-runsvdir
sudo rm /usr/lib/systemd/system/gitlab-runsvdir.service
sudo systemctl daemon-reload
sudo gitlab-ctl uninstall

# 要删除所有数据：
sudo gitlab-ctl cleanse && sudo rm -r /opt/gitlab

# 卸载软件包
sudo yum remove gitlab-jh
```

