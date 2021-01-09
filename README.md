DNMP（Docker + Nginx + MySQL + PHP7/5 + Redis）是一款全功能的**LNMP一键安装程序**。

> 使用前最好提前阅读一遍[目录](#目录)，以便快速上手，遇到问题也能及时排除。交流QQ群：**572041090**。

**[[ENGLISH]](README-en.md)** -
[**[GitHub地址]**](https://github.com/yeszao/dnmp) -
[**[Gitee地址]**](https://gitee.com/yeszao/dnmp)

DNMP项目特点：
1. `100%`开源
2. `100%`遵循Docker标准
3. 支持**多版本PHP**共存，可任意切换（PHP5.4、PHP5.6、PHP7.1、PHP7.2、PHP7.3)
4. 支持绑定**任意多个域名**
5. 支持**HTTPS和HTTP/2**
6. **PHP源代码、MySQL数据、配置文件、日志文件**都可在Host中直接修改查看
7. 内置**完整PHP扩展安装**命令
8. 默认支持`pdo_mysql`、`mysqli`、`mbstring`、`gd`、`curl`、`opcache`等常用热门扩展，根据环境灵活配置
9. 可一键选配常用服务：
    - 多PHP版本：PHP5.4、PHP5.6、PHP7.1-7.3
    - Web服务：Nginx、Openresty
    - 数据库：MySQL5、MySQL8、Redis、memcached、MongoDB、ElasticSearch
    - 消息队列：RabbitMQ
    - 辅助工具：Kibana、Logstash、phpMyAdmin、phpRedisAdmin、AdminMongo
10. 实际项目中应用，确保`100%`可用
11. 所有镜像源于[Docker官方仓库](https://hub.docker.com)，安全可靠
11. 一次配置，**Windows、Linux、MacOs**皆可用

# 目录
- [1.目录结构](#1目录结构)
- [2.快速使用](#2快速使用)
- [3.PHP和扩展](#3PHP和扩展)
    - [3.1 切换Nginx使用的PHP版本](#31-切换Nginx使用的PHP版本)
    - [3.2 安装PHP扩展](#32-安装PHP扩展)
    - [3.3 Host中使用php命令行（php-cli）](#33-host中使用php命令行php-cli)
    - [3.4 使用composer](#34-使用composer)
- [4.添加快捷命令](#4添加快捷命令)
- [5.使用Log](#5使用log)
    - [5.1 Nginx日志](#51-nginx日志)
    - [5.2 PHP-FPM日志](#52-php-fpm日志)
    - [5.3 MySQL日志](#53-mysql日志)
- [6.数据库管理](#6数据库管理)
    - [6.1 phpMyAdmin](#61-phpmyadmin)
    - [6.2 phpRedisAdmin](#62-phpredisadmin)
- [7.在正式环境中安全使用](#7在正式环境中安全使用)
- [8.常见问题](#8常见问题)
    - [8.1 如何在PHP代码中使用curl？](#81-如何在php代码中使用curl)
    - [8.2 Docker使用cron定时任务](#82-Docker使用cron定时任务)
    - [8.3 Docker容器时间](#83-Docker容器时间)
    - [8.4 如何连接MySQL和Redis服务器](#84-如何连接MySQL和Redis服务器)


## 1.目录结构

```
/
├── data                        数据库数据目录
│   ├── esdata                  ElasticSearch 数据目录
│   ├── mongo                   MongoDB 数据目录
│   ├── mysql                   MySQL8 数据目录
│   └── mysql5                  MySQL5 数据目录
├── services                    服务构建文件和配置文件目录
│   ├── elasticsearch           ElasticSearch 配置文件目录
│   ├── mysql                   MySQL8 配置文件目录
│   ├── mysql5                  MySQL5 配置文件目录
│   ├── nginx                   Nginx 配置文件目录
│   ├── php                     PHP5.6 - PHP7.3 配置目录
│   ├── php54                   PHP5.4 配置目录
│   └── redis                   Redis 配置目录
├── logs                        日志目录
├── docker-compose-simple.yml   简单版本的 Docker 服务配置示例文件
├── docker-compose-full.yml     完整版本的 Docker 服务配置示例文件
├── env.smaple                  环境配置示例文件
└── www                         PHP 代码目录
```

## 2.快速使用
1. 本地安装
    - `git`
    - `Docker`(系统需为Linux，Windows 10 Build 15063+，或MacOS 10.12+，且必须要`64`位）
    - `docker-compose 1.7.0+`
2. `clone`项目：
    ```
    $ git clone https://github.com/yeszao/dnmp.git
    ```
3. 如果不是`root`用户，还需将当前用户加入`docker`用户组：
    ```
    $ sudo gpasswd -a ${USER} docker
    ```
4. 拷贝并命名配置文件（Windows系统请用copy命令），启动：
    ```
    $ cd dnmp
    $ cp env.sample .env
    $ cp docker-compose-simple.yml docker-compose.yml
    $ docker-compose up
    ```
    > 这里我们使用 docker-compose-simple.yml 文件内的服务，是简单版本，只包含Nginx、PHP7.2和MySQL8 `3`个服务。如需更多服务，比如Redis、PHP5.4、MongoDB，ElasticSearch等，请参考 docker-compose-full.yml 文件内的服务列表，把需要的拷贝到 docker-compose.yml 文件再`up`即可。

    > 注意：Windows安装360安全卫士的同学，请先将其退出，不然安装过程中可能Docker创建账号过程可能被拦截，导致启动时文件共享失败。
5. 在浏览器中访问：`http://localhost`或`https://localhost`(自签名HTTPS演示)就能看到效果。
    > 演示PHP代码在文件`./www/localhost/index.php`，里面包含了连接mysql服务器和redis服务器的代码，实际使用时可参考此代码。
6. 如需管理服务，请在命令后面加上服务器名称，dnmp支持的服务名有：`nginx`、`php`、`php54`、`mysql`、`mongo`、`redis`、`phpmyadmin`、`phpredisadmin`、`elasticsearch`、`adminmongo`、`rabbitmq`、`kibana`
```bash
$ docker-compose up                         # 创建并且启动所有容器
$ docker-compose up 服务1 服务2 ...         # 创建并且启动指定的多个容器
$ docker-compose up -d 服务1 服务2 ...      # 创建并且已后台运行的方式启动多个容器


$ docker-compose start 服务1 服务2 ...      # 启动服务
$ docker-compose stop 服务1 服务2 ...       # 停止服务
$ docker-compose restart 服务1 服务2 ...    # 重启服务
$ docker-compose build 服务1 服务2 ...      # 构建或者重新构建服务


$ docker-compose rm 服务1 服务2 ...         # 删除并且停止容器
$ docker-compose down 服务1 服务2 ...       # 停止并删除容器，网络，图像和挂载卷
```


## 3.PHP和扩展
### 3.1 切换Nginx使用的PHP版本
在使用 `docker-compose-simple.yml` 的情况下，我们只构建建 **PHP7** 版本的容器，

要使用其他版本，请参考`docker-compose-full.yml`添加服务，如**PHP5.4**，构建完成后修改Nginx 配置的`fastcgi_pass`选项。

例如，示例的 [http://localhost](http://localhost) 用的是PHP7.2，Nginx 配置：
```
    fastcgi_pass   php:9000;
```
要改用PHP5.4，修改为：
```
    fastcgi_pass   php54:9000;
```
再 **重启 Nginx** 生效。
```bash
$ docker exec -it dnmp_nginx_1 nginx -s reload
```
### 3.2 安装PHP扩展
PHP的很多功能都是通过扩展实现，而安装扩展是一个略费时间的过程，
所以，除PHP内置扩展外，在`env.sample`文件中我们仅默认安装少量扩展，
如果要安装更多扩展，请打开你的`.env`文件修改如下的PHP配置，
增加需要的PHP扩展：
```bash
PHP_EXTENSIONS=pdo_mysql,opcache,redis       # PHP 要安装的扩展列表，英文逗号隔开
PHP54_EXTENSIONS=opcache,redis                 # PHP 5.4要安装的扩展列表，英文逗号隔开
```
然后重新build PHP镜像。
    ```bash
    docker-compose build php
    docker-compose up -d
    ```
可用的扩展请看同文件的`PHP extensions`注释块说明。

### 3.3 Host中使用php命令行（php-cli）
1. 打开主机的 `~/.bashrc` 或者 `~/.zshrc` 文件，加上：
```bash
php () {
    tty=
    tty -s && tty=--tty
    docker run \
        $tty \
        --interactive \
        --rm \
        --volume $PWD:/www:rw \
        --workdir /www \
        dnmp_php php "$@"
}
```
2. 让文件起效：
```
source ~/.bashrc
```
3. 然后就可以在主机中执行php命令了：
```bash
~ php -v
PHP 7.2.13 (cli) (built: Dec 21 2018 02:22:47) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.13, Copyright (c) 1999-2018, by Zend Technologies
    with Xdebug v2.6.1, Copyright (c) 2002-2018, by Derick Rethans
```
### 3.4 使用composer
**我们建议在主机HOST中使用composer，避免PHP容器变得庞大**。
1. 在主机创建一个目录，用以保存composer的配置和缓存文件：
    ```
    mkdir ~/dnmp/composer
    ```
2. 打开主机的 `~/.bashrc` 或者 `~/.zshrc` 文件，加上：
    ```
    composer () {
        tty=
        tty -s && tty=--tty
        docker run \
            $tty \
            --interactive \
            --rm \
            --user $(id -u):$(id -g) \
            --volume ~/dnmp/composer:/tmp \
            --volume /etc/passwd:/etc/passwd:ro \
            --volume /etc/group:/etc/group:ro \
            --volume $(pwd):/app \
            composer "$@"
    }

    ```
3. 让文件起效：
    ```
    source ~/.bashrc
    ```
4. 在主机的任何目录下就能用composer了：
    ```
    cd ~/dnmp/www/
    composer create-project yeszao/fastphp project --no-dev
    ```
5. （可选）如果提示需要依赖，用`--ignore-platform-reqs --no-scripts`关闭依赖检测。
6. （可选）第一次使用 composer 会在 ~/dnmp/composer 目录下生成一个config.json文件，可以在这个文件中指定国内仓库，例如：
    ```
    {
        "config": {},
        "repositories": {
            "packagist": {
                "type": "composer",
                "url": "https://packagist.laravel-china.org"
            }
        }
    }

    ```

## 4.添加快捷命令
在开发的时候，我们可能经常使用`docker exec -it`切换到容器中，把常用的做成命令别名是个省事的方法。

打开~/.bashrc，加上：
```bash
alias dnginx='docker exec -it dnmp_nginx_1 /bin/sh'
alias dphp72='docker exec -it dnmp_php_1 /bin/sh'
alias dphp54='docker exec -it dnmp_php54_1 /bin/sh'
alias dmysql='docker exec -it dnmp_mysql_1 /bin/bash'
alias dredis='docker exec -it dnmp_redis_1 /bin/sh'
```

## 5.使用Log

Log文件生成的位置依赖于conf下各log配置的值。

### 5.1 Nginx日志
Nginx日志是我们用得最多的日志，所以我们单独放在根目录`log`下。

`log`会目录映射Nginx容器的`/var/log/nginx`目录，所以在Nginx配置文件中，需要输出log的位置，我们需要配置到`/var/log/nginx`目录，如：
```
error_log  /var/log/nginx/nginx.localhost.error.log  warn;
```


### 5.2 PHP-FPM日志
大部分情况下，PHP-FPM的日志都会输出到Nginx的日志中，所以不需要额外配置。

另外，建议直接在PHP中打开错误日志：
```php
error_reporting(E_ALL);
ini_set('error_reporting', 'on');
ini_set('display_errors', 'on');
```

如果确实需要，可按一下步骤开启（在容器中）。

1. 进入容器，创建日志文件并修改权限：
    ```bash
    $ docker exec -it dnmp_php_1 /bin/bash
    $ mkdir /var/log/php
    $ cd /var/log/php
    $ touch php-fpm.error.log
    $ chmod a+w php-fpm.error.log
    ```
2. 主机上打开并修改PHP-FPM的配置文件`conf/php-fpm.conf`，找到如下一行，删除注释，并改值为：
    ```
    php_admin_value[error_log] = /var/log/php/php-fpm.error.log
    ```
3. 重启PHP-FPM容器。

### 5.3 MySQL日志
因为MySQL容器中的MySQL使用的是`mysql`用户启动，它无法自行在`/var/log`下的增加日志文件。所以，我们把MySQL的日志放在与data一样的目录，即项目的`mysql`目录下，对应容器中的`/var/lib/mysql/`目录。
```bash
slow-query-log-file     = /var/lib/mysql/mysql.slow.log
log-error               = /var/lib/mysql/mysql.error.log
```
以上是mysql.conf中的日志文件的配置。



## 6.数据库管理
本项目默认在`docker-compose.yml`中开启了用于MySQL在线管理的*phpMyAdmin*，以及用于redis在线管理的*phpRedisAdmin*，可以根据需要修改或删除。

### 6.1 phpMyAdmin
phpMyAdmin容器映射到主机的端口地址是：`8080`，所以主机上访问phpMyAdmin的地址是：
```
http://localhost:8080
```

MySQL连接信息：
- host：(本项目的MySQL容器网络)
- port：`3306`
- username：（手动在phpmyadmin界面输入）
- password：（手动在phpmyadmin界面输入）

### 6.2 phpRedisAdmin
phpRedisAdmin容器映射到主机的端口地址是：`8081`，所以主机上访问phpMyAdmin的地址是：
```
http://localhost:8081
```

Redis连接信息如下：
- host: (本项目的Redis容器网络)
- port: `6379`


## 7.在正式环境中安全使用
要在正式环境中使用，请：
1. 在php.ini中关闭XDebug调试
2. 增强MySQL数据库访问的安全策略
3. 增强redis访问的安全策略


## 8 常见问题
### 8.1 如何在PHP代码中使用curl？
参考这个issue：[https://github.com/yeszao/dnmp/issues/91](https://github.com/yeszao/dnmp/issues/91)

### 8.2 Docker使用cron定时任务 
[Docker使用cron定时任务](https://www.awaimai.com/2615.html)

### 8.3 Docker容器时间
容器时间在.env文件中配置`TZ`变量，所有支持的时区请看[时区列表·维基百科](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)或者[PHP所支持的时区列表·PHP官网](https://www.php.net/manual/zh/timezones.php)。

### 8.4 如何连接MySQL和Redis服务器
这要分两种情况，

第一种情况，在**PHP代码中**。
```php
// 连接MySQL
$dbh = new PDO('mysql:host=mysql;dbname=mysql', 'root', '123456');

// 连接Redis
$redis = new Redis();
$redis->connect('redis', 6379);
```
因为容器与容器是`expose`端口联通的，而且在同一个`networks`下，所以连接的`host`参数直接用容器名称，`port`参数就是容器内部的端口。更多请参考[《docker-compose ports和expose的区别》](https://www.awaimai.com/2138.html)。

第二种情况，**在主机中**通过**命令行**或者**Navicat**等工具连接。主机要连接mysql和redis的话，要求容器必须经过`ports`把端口映射到主机了。以 mysql 为例，`docker-compose.yml`文件中有这样的`ports`配置：`3306:3306`，就是主机的3306和容器的3306端口形成了映射，所以我们可以这样连接：
```bash
$ mysql -h127.0.0.1 -uroot -p123456 -P3306
$ redis-cli -h127.0.0.1
```
这里`host`参数不能用localhost是因为它默认是通过sock文件与mysql通信，而容器与主机文件系统已经隔离，所以需要通过TCP方式连接，所以需要指定IP。



## License
MIT


