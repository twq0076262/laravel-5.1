# Laravel Homestead

# 1、简介
Laravel 致力于将整个 PHP 开发体验变得让人愉悦，包括本地开发环境。

Vagrant 提供了一个便捷的方式来管理和设置虚拟机。Laravel Homestead 是一个官方的、预安装好的 Vagrant 盒子，该盒子提供了一流的开发环境，有了它，我们不再需要在本地环境安装 PHP、HHVM、web 服务器以及其它服务器软件，我们也完全不用再担心误操作搞乱操作系统——因为 Vagrant 盒子是一次性的，如果出现错误，可以在数分钟内销毁并重新创建该 Vagrant 盒子！

Homestead 可以运行在 Windows、Mac 以及 Linux 系统上，其中已经安装好了 Nginx、PHP5.6、MySQL、Postgres、Redis、Memcached、Node 以及很多其它开发牛逼 Laravel 应用所需要的东西。

注意：如果你使用的是 Windows，需要开启系统的硬件虚拟化（VT-x），这通常可以通过 BIOS 来开启。
Homestead 目前基于 Vagrant 1.7 进行构建和测试。

## 1.1 包含软件

- Ubuntu 14.04
- PHP 5.6 / PHP 7.0
- HHVM
- Nginx
- MySQL
- Postgres
- Node (With PM2, Bower, Grunt, and Gulp)
- Redis
- Memcached（只支持PHP 5.x）
- Beanstalkd
- Laravel Envoy
- Blackfire Profiler
# 2、安装 & 设置
## 2.1 首次安装

在启用 Homestead 环境之前，需要先安装 Virtual Box 或者 VMWare 和 Vagrant，所有这些软件包都会常用操作系统提供了一个便于使用的可视化安装器。

### 2.1.1 安装 Homestead Vagrant 盒子

VirtualBox/VMWare 和 Vagrant 安装好了之后，在终端中使用能如下命令将 laravel/homesterad 添加到 vagrant 安装中。这将会花费一些时间下载该盒子，时间长短主要取决于你的网络连接速度：

```
vagrant box add laravel/homestead
```

如果上述命令执行失败，可以使用 vagrant 老版本的方式，这需要输入完整的 URL：

```
vagrant box add laravel/homestead https://atlas.hashicorp.com/laravel/boxes/homestead
```

### 2.1.2 克隆已有的 Homestead 仓库

你还可以通过简单克隆仓库代码来实现 Homestead 安装，考虑到克隆仓库到 home 目录下的 Homestead 文件夹，Homestead 盒子将会作为所有其他 Laravel 项目的主机：

```
git clone https://github.com/laravel/homestead.git Homestead
```

克隆完成后，在 Homestead 目录下运行 bash init.sh 命令来创建 Homestead.yaml 配置文件，Homestead.yaml 配置文件文件位于 ~/.homestead 目录：

```
bash init.sh
```

### 2.1.3 升级到 PHP 7.0

如果你已经在使用 PHP 5.x 版的 Homestead 盒子，可以轻松升级安装到 PHP 7.0。首先，克隆 laravel/homestead 的 php-7 分支到一个新的文件夹：

```
git clone -b php-7 https://github.com/laravel/homestead.git Homestead
```

不需要运行 init.sh 脚本来覆盖整个 Homestead.yaml 文件，你只需要简单添加该盒子到已存在的 Homestead.yaml 文件顶部即可：

```
box: laravel/homestead-7
```

接下来，从包含克隆 laravel/homestead 仓库的目录运行 vagrant up 命令即可。

## 2.2 配置 Homestead

### 2.2.1 设置 Provider

Homestead.yaml 文件中的 provider 键表示哪个 Vagrant 提供者将会被使用：virtualbox、vmware_fushion，还是 vmware_workstation，你可以将其设置为自己喜欢的提供者：

```
provider: virtualbox
```

### 2.2.2 设置 SSH key

在 Homestead.yaml 文件中还要配置公共 SSH key 的路径，如果没有 SSH key，那么在 Mac 或 Linux 上，可以通过如下命令来生成：

```
ssh-keygen -t rsa -C "you@homestead"
```

在 Windows 上，可以安装 Git 然后使用 Git 自带的“Git Bash”来执行上述命令。除此之外，你还可以使用 PUTTY 和 PUTTYgen 工具来生成 SSH key。

### 2.2.3 配置共享文件夹

Homestead.yaml 文件中的 folders 选项列出了所有你想要和 Homestead 环境进行共享的目录，一旦这些目录中的文件有了修改，将会在本地和 Homestead 环境中保持同步，如果有需要的话，你可以配置尽可能多的共享目录：

```
folders:
    - map: ~/Code
      to: /home/vagrant/Code
```

如果想要开启 NFS，只需简单添加一个标识到同步文件夹配置：

```
folders:
    - map: ~/Code
      to: /home/vagrant/Code
      type: "nfs"
```

### 2.2.4 配置 Nginx 站点

对 Nginx 不熟？没问题，sites 选项允许你方便的匹配“域名”到 Homestead 环境的某个目录，一个示例站点配置已经包含到 Homestead.yaml 文件。跟共享文件夹一样，你可以想配置多少个站点就配置多少个。Homestead 可以为你的每个 Laravel 项目充当方便的、虚拟化的开发环境：

```
sites:
    - map: homestead.app
      to: /home/vagrant/Code/Laravel/public
```

你可以通过设置 hhvm 为 true 让所有的 Homestead 站点使用 HHVM：

```
sites:
    - map: homestead.app
      to: /home/vagrant/Code/Laravel/public
      hhvm: true
```

默认情况下，每个站点都可以通过 HTTP（端口号：8000）和 HTTPS（端口号：44300）进行访问。

### 2.2.5 Hosts 文件

不要忘记把 Nginx 站点中的域名添加到本地机器上的 hosts 文件，该文件会将对本地域名的请求重定向到 Homestead 环境，在 Mac 或 Linux 上，该文件位于 /etc/hosts，在 Windows 上，位于 C:\Windows\System32\drivers\etc\hosts，添加方式如下：

```
192.168.10.10  homestead.app
```

确保 IP 地址和你的 Homestead.yaml 文件中列出的一致，一旦你将域名放置到 hosts 文件，就可以在浏览器中通过该域名访问站点了！

```
http://homestead.app
```

## 2.3 启动 Vagrant Box

编辑好 Homestead.yaml 文件后，在 Homestead 目录下运行 vagrant up 命令，vagrant 将会启动虚拟机并自动配置共享文件夹以及 Nginx 站点。

销毁该机器，可以使用 vagrant destroy --force

## 2.4 为指定项目安装 Homestead

全局安装 Homestead 将会使每个项目共享同一个 Homestead 盒子，你还可以为每个项目单独安装 Homestead，这样就会在该项目下创建 Vagrantfile，允许其他人在该项目中执行 vagrant up 命令，在指定项目根目录下使用 Composer 执行安装命令如下：

```
composer require laravel/homestead --dev
```

这样就在项目中安装了 Homestead。Homestead 安装完成后，使用 make 命令生成 Vagrantfile 和Homestead.yaml 文件，make 命令将会自动配置 Homestead.yaml 中的 sites 和 folders 属性。

**Mac/Linux：**

```
php vendor/bin/homestead make
```

**Windows:**

```
vendor\bin\homestead make
```

接下来，在终端中运行 vagrant up 命令然后在浏览器中通过 http://homestead.app 访问站点。不要忘记在 /etc/hosts（Linux）文件中添加域名 homestead.app。

# 3、日常使用
## 3.1 通过 SSH 连接

你可以在 Homestead 目录下通过终端命令 vagrant ssh 以 SSH 方式连接到虚拟机，但是如果你需要以更平滑的方式连接到 Homestead，可以为主机添加一个别名来快速连接到 Homestead 盒子。创建完别名后，可以简单使用 vm 命令来从任何地方以 SSH 方式连接到 Homestead 机器：

```
alias vm="ssh vagrant@127.0.0.1 -p 2222"
```

## 3.2 连接到数据库

默认已经在 homestead 中为 MySQL 和 Postgres 数据库做好了配置，更加方便的是，Laravel 的 local 数据库配置已经为使用数据库做好了设置。

想要通过本地的 Navicat 或 Sequel Pro 连接上 MySQL 或 Postgres 数据库，可以通过新建连接来实现，主机 IP 都是127.0.0.1，对于 MySQL 而言，端口号是 33060，对 Postgres 而言，端口号是54320，用户名/密码是 homestead/secret。

注意：只有从本地连接 Homestead 的数据库时才能使用这些非标准的端口，在 Homestead 环境中还是应该使用默认的3306和5432端口进行数据库连接配置。
## 3.3 添加更多站点

Homestead 环境在运行时，你可能想要添加额外 Laravel 应用到 Nginx 站点，你可以在单个Homestead环境中运行多个 Laravel 应用，添加站点很简单，只需将站点添加到 Homestead.yaml 文件，然后在 Homestead 目录中运行 vagrant provision 命令即可。

注意：该处理是不可逆的，运行 provision 命令时，已经存在的数据库会被销毁并重建。
## 3.4 端口转发配置

默认情况下，Homestead 端口转发配置如下：

- **SSH:** 2222 → Forwards To 22
- **HTTP:** 8000 → Forwards To 80
- **HTTPS:** 44300 → Forwards To 443
- **MySQL:** 33060 → Forwards To 3306
- **Postgres:** 54320 → Forwards To 5432
### 3.4.1 转发更多端口

如果你想要在 Vagrant 盒子添加更多端口转发，做如下转发协议设置即可：

```
ports:
    - send: 93000
      to: 9300
    - send: 7777
      to: 777
      protocol: udp
```

# 4、使用 Blackfire Profiler 进行性能分析
SensioLabs 的 Blackfire Profiler 能自动收集代码执行数据，比如内存、CPU 时间、硬盘 I/O 等，Homestead 使得在应用中使用该 profiler 变得轻而易举。

所有需要的软件包已经安装到 Homestead 盒子，你只需要在 Homestead.yaml 文件中设置 Blackfire Server ID 和 token：

```
blackfire:
    - id: your-server-id
      token: your-server-token
      client-id: your-client-id
      client-token: your-client-token
```

配置好 Blackfire 的凭证之后，在 Homestead 目录下使用 vagrant provision 重新指配盒子。在此之前，确保你已经查看过 Blackfire 文档了解了如何为你的浏览器安装相应应的 Blackfire 扩展。