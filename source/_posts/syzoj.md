---
title: 搭建 SYZOJ
date: 2022-06-27 09:43:22
tags: Ubuntu, SYZOJ, 指南
---

最近，想要在自己的服务器上搭建一个 OJ。在 Github 上搜索一下，各种各样的 OJ 到处都是。最终，我选择了 SYZOJ。上过 [LOJ](https://loj.ac/) 的应该都知道，SYZOJ 的 UI 特别好看。如果有时间的话，的确可以自己挑别的 OJ 系统二次开发一个 Semantic UI（SYZOJ 的 UI 框架）的界面，不过直接装 SYZOJ 肯定更让人舒服。另外，虽然 HUSTOJ 最近推出了 SYZOJ 的主题，但是我认为还是有少些地方不太像。

SYZOJ 的部署指南曾经十分详细，但是大概一年前突然又删掉了很多。那么为了方便大家部署，我就在这片博客里写一点指南。

我会尽可能将文章写得易懂，傻瓜也能部署的级别。

## 部署

### 你需要准备

~~这一步实际上对于大部分人来说是最困难的~~

- 一台云服务器（境内的云服务器在下载文件的时候可能会比较慢，且需要备案），请安装 Ubuntu 20.04 系统；
- 一个域名（这个可以在 [Freenom](https://www.freenom.com/zh/index.html?lang=zh) 上免费拿一个）

SYZOJ 的 docker 容器不知道为什么，我的服务器安装出现了错误。我认为自己配置一步一步安装更加保守，且易于开发。

### 安装依赖

让我们开始吧。以下所有命令都需要使用 `root` 权限执行。你可以在每条命令前面加上 `sudo`，也可以使用 `sudo su` 来切换到 `root` 账户。

首先，你需要运行下面的命令来安装 Node.js 16、Yarn 1、MariaDB 10.3 与 Redis 5 等工具。这些可以在文档中找到。

```sh
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
curl -fsSL https://deb.nodesource.com/setup_current.x | bash -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8
apt install -y software-properties-common
add-apt-repository 'deb [arch=amd64,arm64,ppc64el] http://mirrors.tuna.tsinghua.edu.cn/mariadb/repo/10.3/ubuntu focal main'
add-apt-repository ppa:chris-lea/redis-server # Ubuntu 20.04 不需要执行这条
apt update
apt install -y git mariadb-server redis-server nodejs yarn p7zip-full clang-format
```

**注意：安装时会出现让你填写数据库初始密码的界面，请直接按回车跳过。**

接下来，下载文件。境内的服务器可能会略微有点慢，不过速度总体还行。

```sh
rm -rf /opt/syzoj /etc/systemd/system/syzoj*
mkdir -p /opt/syzoj
cd /opt/syzoj
git clone https://github.com/syzoj/syzoj web
cd web
yarn
```

### 配置 OJ

接下来，我们要创建配置。

```sh
mkdir -p /opt/syzoj/config
cp /opt/syzoj/web/config-example.json /opt/syzoj/config/web.json
ln -s ../config/web.json /opt/syzoj/web/config.json
```

接下来，你需要编辑一下配置。首先，让我们打开这个文件。

```sh
vim /opt/syzoj/web/config.json
```

打开后，你需要修改的有：

- `title`：这个我认为大部分人都自己会改(^-^)；
- `session_secret`：这个请填一个随机字符串；
- `judge_token`：这个也填一个随机字符串，但是以后部署评测机时候要用，所以请记下来；
- `db`：在其中 `password` 的地方填一个随机字符串，这个马上安装数据库的时候也要用，请记录下来。

另外，生成随机字符串的快捷指令是 `echo $(dd if=/dev/urandom | base64 -w0 | dd bs=1 count=20 2>/dev/null)`，如果遇到了困难可以请服务器随机生成。

接下来，要创建一些存储临时文件（如 session）和数据（如上传的文件）的文件夹。

```sh
mv /opt/syzoj/web/uploads /opt/syzoj/data
ln -s ../data /opt/syzoj/web/uploads
mkdir /opt/syzoj/sessions
ln -s ../sessions /opt/syzoj/web/sessions
```

### 创建 syzoj 账户

因为以 `root` 账户运行网页端特别不安全，所以我们需要创建 syzoj 账户。

```sh
adduser --disabled-password --gecos "" syzoj # 以用户名 syzoj 为例
```

但是此时的网页端还无法读取到我们刚才创建的几个目录，我们要加上权限。

```sh
chown -R syzoj:syzoj /opt/syzoj/data /opt/syzoj/sessions /opt/syzoj/config/web.json
```

### 创建数据库

从这里开始，SYZOJ 的官方 Wiki 就开始省略了……不过没关系。

首先，让我们进入数据库。请运行 `mysql`。如果你第一步时使用了密码，那么请自己修改命令连接数据库。

然后创建数据库。

```sql
CREATE DATABASE `syzoj` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

接下来，我们要赋予权限。请把 `<password>` 替换成你在配置文件步骤中填在 `db` 的随机字符串。

```sql
GRANT ALL PRIVILEGES ON `syzoj`.* TO "syzoj"@"localhost" IDENTIFIED BY "<password>";
FLUSH PRIVILEGES;
```

### 跑起来！

现在，我们已经完成了配置，让我们启动服务！这一段的官方文档写得及其简陋，也是卡掉了大部分安装 SYZOJ 的玩家。

首先，让我们创建并编辑服务文件。请使用你喜欢的编辑器。我在这里使用 Vim。

```sh
vim /etc/systemd/system/syzoj-web.service
```

如果是 Vim 的话，按下 `i` 键进入插入模式，然后粘贴这些内容。

```ini
[Unit]
Description=SYZOJ web service
After=network.target mysql.service rc-local.service
Requires=mysql.service rc-local.service

[Service]
Type=simple
WorkingDirectory=/opt/syzoj/web
User=syzoj
Group=syzoj
ExecStart=/usr/bin/env NODE_ENV=production /usr/bin/node /opt/syzoj/web/app.js -c /opt/syzoj/config/web.json
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

按下 `ESC`，输入 `:wq`，按下 `Enter`，退出，启动它。

```sh
systemctl start syzoj-web.service
```

记得设为开机自启。

```sh
systemctl enable syzoj-web.service
```

此时，访问网站是没有作用的。我们需要 Nginx 反向代理。

在配置 Nginx 之前，请保证你的域名通过 A 解析到了你的服务器。

### Nginx 配置

不要急！先安装 Nginx。

```sh
apt install -y nginx
```

然后创建并编辑配置文件。

```sh
vim /etc/nginx/sites-enabled/syzoj
```

请输入以下内容。

```sh
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
    listen 80;
    listen [::]:80;
    
    server_name oimasterakioi.com;

    location / {
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Host $host;
        proxy_set_header Connection $connection_upgrade;
        proxy_pass http://127.0.0.1:5283;
    }
}
```

请将 `oimasterakioi.com` 替换成你自己的域名。

接下来，启动服务。

```sh
systemctl reload nginx
```

### 赋予管理员权限

**等待一小会儿**，如果之前的步骤全都正确的话，现在应该已经能看到界面了！请立即注册一个账号。

接下来，如果不出意外的话，你应该能看到你的账号 ID 为 1。我们要给它管理员权限。

全站管理员的权限是最高的，为了安全，不能在网页端进行设置。让我们进入数据库。

```sh
mysql
```

然后，赋予其全站管理员权限。

```mysql
UPDATE `syzoj`.`user` SET `is_admin` = 1 WHERE `id` = 1;
```

因为 SYZOJ 运行时有缓存（cache），所以修改后的结果并不能直接显现出来。我们需要重启网页端。

```sh
systemctl restart syzoj-web.service
```

接下来，您应该可以进入后台了！现在一切功能都是可以用的，除了——评测机。

### 准备沙箱

跟大部分的 OJ 一样，我们需要打开一些 Linux 内核中不默认开启的特性。首先，请打开 Ubuntu 的默认引导器——Grub 的配置文件。

```sh
vim /etc/default/grub
```

找到 `GRUB_CMDLINE_LINUX_DEFAULT` 一行，在原有的字符串后面添加 `cgroup_enable=memory swapaccount=1 systemd.unified_cgroup_hierarchy=0 syscall.x32=y`。例如，修改完后的结果可能是：

```sh
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash cgroup_enable=memory swapaccount=1 systemd.unified_cgroup_hierarchy=0 syscall.x32=y"
```

然后重启修改配置。部分云服务器可能修改完后 VNC 连接很卡，但是 ssh 功能应该是正常的。

```sh
update-grub && reboot
```

### 下载沙箱

这一步境内的服务器很慢，因为包含 SYZOJ 网站端所有的语言支持和 Ubuntu 18.04。境外的服务器大约需要 1-2 分钟左右，国内的服务器可能消耗时间较长。

```sh
wget -O /sandbox-rootfs.tar.xz https://github.com/syzoj/sandbox-rootfs/releases/download/210521/sandbox-rootfs-210521.tar.xz
```

如果服务器在国内，又不愿意卡在这一步，不如使用 `screen` 或者别的方法进行后台下载。

下载完成后，解压文件。

```sh
mkdir -p /opt/syzoj/sandbox/rootfs
cd /opt/syzoj/sandbox/
tar xvf /sandbox-rootfs.tar.xz
```

然后建立沙箱所需文件夹。

```sh
mkdir -p /opt/syzoj/sandbox/{bin,tmp1}
```

### 安装依赖

这一段也比较困难，官方文档中描述较少。

先安装几个软件。

```sh
apt install build-essential libboost-all-dev
apt install rabbitmq-server redis-server
```

装 Node.js 10。

```sh
mkdir -p /opt/syzoj
wget -O /tmp/node-v10.24.1-linux-x64.tar.xz https://nodejs.org/dist/latest-v10.x/node-v10.24.1-linux-x64.tar.xz
tar xf /tmp/node-v10.24.1-linux-x64.tar.xz -C /tmp
mv /tmp/node-v10.24.1-linux-x64 /opt/syzoj/node10

# 如果没有部署过网站端，则安装 yarn，但是我们刚才部署过了，所以可以跳
PATH=/opt/syzoj/node10/bin:$PATH /opt/syzoj/node10/bin/npm install yarn -g
```

另外，如果你的其他服务器也需要部署 Node.js 环境，不如看看我的另一篇文章：[在 Ubuntu 上安装 Node.js](https://blog.oimaster.ml/2022/06/21/install-nodejs/)。

### 下载评测端程序

没什么好说的。

```sh
cd /opt/syzoj
git clone https://github.com/syzoj/judge-v3
mv judge-v3 judge
cd judge
PATH=/opt/syzoj/node10/bin:$PATH yarn
PATH=/opt/syzoj/node10/bin:$PATH yarn build
```

### 配置评测机

```sh
cd /opt/syzoj
cp judge/daemon-config-example.json config/daemon.json
cp judge/runner-shared-config-example.json config/runner-shared.json
cp judge/runner-instance-config-example.json config/runner-instance.json
```

然后这里配置文件需要编辑一下。

```sh
vim config/daemon.json
```

其中，`ServerToken` 请改成在配置网页端时的 `judge_token`。

### 注册服务

```sh
vim /etc/systemd/system/syzoj-judge-daemon.service
```

填入以下内容。

```ini
[Unit]
Description=SYZOJ judge daemon service
After=network.target rabbitmq-server.service redis-server.service
Requires=rabbitmq-server.service redis-server.service

[Service]
Type=simple
WorkingDirectory=/opt/syzoj/judge
User=syzoj
Group=syzoj
ExecStart=/opt/syzoj/node10/bin/node /opt/syzoj/judge/lib/daemon/index.js -c /opt/syzoj/config/daemon.json

[Install]
WantedBy=multi-user.target
```

退出，注册另一个服务。

```sh
vim /etc/systemd/system/syzoj-judge-runner.service
```

```ini
[Unit]
Description=SYZOJ judge runner service
After=network.target rabbitmq-server.service redis-server.service
Requires=rabbitmq-server.service redis-server.service

[Service]
Type=simple
WorkingDirectory=/opt/syzoj/judge
User=root
Group=root
ExecStart=/opt/syzoj/node10/bin/node /opt/syzoj/judge/lib/runner/index.js -s /opt/syzoj/config/runner-shared.json -i /opt/syzoj/config/runner-instance.json

[Install]
WantedBy=multi-user.target
```

启动它们，并且设置为开机自启。

```sh
systemctl start syzoj-judge-runner.service
systemctl enable syzoj-judge-runner.service
systemctl start syzoj-judge-daemon.service
systemctl enable syzoj-judge-daemon.service
```

恭喜，完成！

## 其他

从这里开始，就是一些奇奇怪怪、可有可无的功能了。如果想要你的 OJ 更好，不如看看。注意，下面几项难度逐渐增加。

同时因为这是可选功能，所以叙述将会更加简略。

### 邮箱注册

当你公开后，没过多久，就总能看到有心智不成熟的人注册一大堆垃圾账号。

解决办法：邮箱验证。

请自行去您的邮箱平台上打开 SMTP 的功能，并且获取 SMTP 密码。对于部分国外邮箱，这些功能都是默认开启的，且可以直接拿邮箱登录密码进行操作。

以网易邮箱为例子：

![](https://s3.bmp.ovh/imgs/2022/06/27/b1979c30054f7017.png)

进入后，点击新增授权密码，进行验证。其他邮箱同理。

完成后，记录下你的密码，进入网站的 后台管理 - 配置文件，找到 `email`，改成


```yml
  "email": {
    "method": "smtp",
    "options": {
        "host": "smtp.163.com",
        "port": 465,
        "username": "xxx@163.com",
        "password": "xxx",
        "allowUnauthorizedTls": false
    }
  }
```

然后将上方 `register_mail` 改为 `true` 即可！

### 配置 Logo

历时 3 年，花费 200 万人民币的 Logo 制作完成！现在如何把它摆上去？

简单，让我们上传到服务器，然后进行配置即可。

```sh
cd /opt/syzoj/web/static
```

这里是静态文件目录。将自己的 Logo 上传到这里。为了叙述方便，不妨假设该 Logo 为 `xxx.png`

接下来同样进入 后台管理 - 配置文件，找到 `logo`，改成

```yml
  "logo": {
    "url": "/xxx.png",
    "width": xxxxx,
    "height": 36
  }
```

为了显示正常，请把 `height` 改成 36，同时 `width` 可以等比例缩小。

### 今日运势

今日运势的运势类型实在是太少了！我们需要自己添加。

```sh
vim /opt/syzoj/web/divine.json
```

然后用你的 JSON 知识进行添加。如果你不会 JSON，也可以仿照上面的格式。

保险起见，可以重启 SYZOJ web 服务。

### 二次开发 web

~~界面我觉得不用二次开发了~~

进入 `/opt/syzoj/web/views/`，找到对应的文件，进行修改。最后记得重启 web 服务端。

### 数据库玩砸了

玩砸技巧：新建题目，将题目 ID 设为 10000，然后删除它。接下来再创建一个新题，就会意外地发现新题的 ID 为 10001。

如果了解数据库，就知道显然是 `AUTO_INCREMENT` 的问题。

假设你现在有效的题目 ID 为 1~n，且你那些 10000 后面的题目不需要。

步骤：

1. 使用 `delete` 命令批量删除，或者是手动在网页端一个一个删
2. 在数据库中 `alter table problem AUTO_INCREMENT=xxx`，将 `xxx` 替换成 n+1。

如果 10000 后面的题目有用，只是想要消除空档，那么请查看 [这个网站](https://www.baidu.com).

不过，最好的解决方法是，**不要删除题目**。

---

如果还有问题，例如如果分布式评测、配置静态资源 CDN 等，请上 [官方 Wiki](https://github.com/syzoj/syzoj/wiki)。相信这样的人应该可以读懂了。

感谢大家的捧场。