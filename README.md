
# 简单的基于`Docker`的本地`PHP开发环境`

### 包含了这些内容
> **使用了这些镜像：**
`mysql:8.0.25` 
`nginx:1.21.0-alpine` 
`phpdev:8.0.7-swoole4.6.7-alpine3.13（基于 php:8.0.7-fpm-alpine3.13 构建）` 
`redis:6.2.4-alpine3.13`

### PHP扩展列表
> `bcmath` `Core` `ctype` `curl` `date` `dom` `fileinfo` `filter` `ftp` `gd` `hash` `iconv` `json` `libxml` `mbstring` `mysqli` `mysqlnd` `openssl` `pcntl` `pcre` `PDO` `pdo_mysql` `pdo_sqlite` `Phar` `posix` `readline` `redis` `Reflection` `session` `SimpleXML` `sockets` `sodium` `SPL` `sqlite3` `standard` `swoole` `sysvmsg` `sysvshm` `tokenizer` `xml` `xmlreader` `xmlwriter` `Zend OPcache` `zip` `zlib`

### 目录和文件结构
> 相关挂载的目录可以自行在`docker-compose.yml`文件中进行调整，事实上完全可以直接使用默认配置
- `mysql` 挂载MySQL的目录 `/var/lib/mysql`
- `mysqlconfig` 挂载MySQL的配置目录 `/etc/mysql/conf.d`，目录下的 *.cnf 文件会自动被包含进 `/etc/mysql/my.cnf` 配置中，默认配置可以参考目录下的 `my.cnf` 文件
- `nginx/conf.d` 挂载Nginx的目录 `/etc/nginx/conf.d`，虚拟机文件可以放在此目录中，如 `./nginx/conf.d/default.conf`
- `nginx/html` 挂载Nginx的目录 `/usr/share/nginx/html`，项目目录，这个目录同时保证也挂载到 PHP 的 `/var/www/html` 项目目录
- `php` 挂载PHP的 `/usr/local/etc/php` 目录，其中目录 `conf.d` 必须，用于后续安装扩展配置所在目录，根目录下可以放置 PHP 配置文件 `php.ini`
- `redis` 根目录挂载在 `/usr/local/etc/redis` 目录，用于放置redis配置文件 `redis.conf`
- `redis/data` 目录挂载在 `/data` 目录，redis的持久化目录

### 使用方法（此处已经略过Docker的安装，请确保电脑上安装了Docker）
1. 下载或克隆本仓库
2. 如有需要对 `docker-compose.yml` 文件进行自定义修改，比如修改本地项目挂载目录，分别是`第31行`和`第48行`，同时注意，`两个地方应挂载同一个本地的目录`
   
    ```
    1.  - ./nginx/html:/usr/share/nginx/html
    ...
    1.  - ./nginx/html:/var/www/html
    ```
    将其中的 `./nginx/html` 换成本地的项目目录，这个目录里可能有你很多个项目

    ```
    1.  - /path/to/your/wwwroot:/usr/share/nginx/html
    ...
    1.  - /path/to/your/wwwroot:/var/www/html
    ```
3. 进入命令行模式下，进入此项目根目录，运行以下命令即可
    ```
    docker-compose up -d
    ```

### 注意事项
1. Nginx虚拟机配置时，转发到 PHP-FPM 时注意使用docker-compose服务的名称如 `php` 而非 `127.0.0.1`
    ```
    location ~ \.php$ {
        ...
        fastcgi_pass   php:9000; # 正确的
        #fastcgi_pass   127.0.0.1:9000; # 错误的
        ...
    }
    ```
2. Nginx虚拟机配置时，项目目录两处配置需要注意，`root` 配置应指向 `/usr/share/nginx/html/path/to/your/approot`，转发 PHP-FPM 的 `SCRIPT_FILENAME` 应指向 `/var/www/html/path/to/your/approo$fastcgi_script_name`
   
    ```
    ...
    root   /usr/share/nginx/html/path/to/your/approo;
    index  index.html index.htm index.php;
    ...
    location ~ \.php$ {
        ...
        fastcgi_param  SCRIPT_FILENAME  /var/www/html/path/to/your/approo$fastcgi_script_name;
        ...
    }
    ```

3. 项目中使用数据库连接是 `host` 设置为docker-compose中MySQL的服务名称如 `mysql` 而非 `127.0.0.1`
4. 修改配置文件记得重启服务/容器
   ```
   docker-compose restart nginx # 按需
   docker-compose restart php # 按需
   ```

### 安装新的PHP扩展
1. 进入 PHP 容器
```
docker exec -it php /bin/sh
```
2. 使用 `pecl` 或 `docker-php-ext-install` 安装扩展（或者其他源码编译安装）
```
pecl install swoole-4.6.7
docker-php-ext-install pdo_mysql
```
3. 通常第二步安装后就会启用扩展了，如果没有使用 `docker-php-ext-enable` 启用
```
docker-php-ext-enable swoole pdo_mysql
```