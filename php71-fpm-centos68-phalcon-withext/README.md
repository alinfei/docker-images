
# php71-fpm-centos68-phalcon-withext

基于自行编译的 php71-fpm-centos68 镜像，父镜像是 centos:6.8。

与php71-fpm-centos68-phalcon功能一样，只是PHP扩展部分采用直接将php71-fpm-centos68-phalcon镜像编译好的so文件复制到镜像内，而省去了编译的过程，且节约了空间（php71-fpm-centos68 800M，php71-fpm-centos68-phalcon 1.4G，php71-fpm-centos68-phalcon-withext 831M）。

php7.1-phalcon的开发环境。包含：

- CentOS 6.8
- PHP 7.1.12
- php-fpm
- Nginx 1.12.2
- Redis 3.2.6

包含的PHP附加扩展：

- swoole 1.10.1
- phpredis 3.0.0
- yar 2.0.2
- phpalcon 3.3.1
- xhprof_tideways 4.1.5
- msgpack
- protobuf
- yaconf
- mongodb
- seaslog
- gearman 1.1.18

相关目录：

- php根目录：/usr/local/php/
- php可执行程序目录：/usr/local/php/bin/、/usr/local/php/sbin/
- php配置目录：/usr/local/php/etc/
- nginx根目录：/usr/local/nginx/
- nginx可执行程序目录：/usr/local/nginx/sbin/
- nginx配置目录：/usr/local/nginx/conf/

其中gearman依赖libgearman.so.8库，需要把该库文件复制到`/usr/local/lib/`。

## 如何使用

1、首次使用需要先编译成镜像：

``` bash
docker build -t php71-fpm-centos68-phalcon-withext .
```
或者执行bulid.sh。

编译需要30分钟左右。实际视机器性能而定。

>注：因为父镜像是[`php71-fpm-centos68`](https://github.com/52fhy/docker-images/tree/master/php71-fpm-centos68)故需要先编译生成该镜像。已存在则忽略。

也可以下载打包好的镜像文件 php71-fpm-centos68-phalcon-withext.tar.gz (239M，https://pan.baidu.com/s/1bpYKnCF，密码: nged)，然后：
```
docker load --input php71-fpm-centos68-phalcon-withext.tar.gz
```

或者：
```
docker pull hub.c.163.com/jiancaigege/php71-fpm-centos68-phalcon-withext:latest
```

2、编译完成后可以创建容器了。默认会自动启动Nginx、php-fpm服务：

``` bash
docker run -d --name yphp -p 80:80 -p 9000:9000 -v /work/:/work/ php71-fpm-centos68-phalcon-withext
docker ps
```

其中`-v /work/:/work/`指定将本机的`/work`目录挂载到容器内的`/work`目录。建议挂载。本机的`/work`目录必须存在。

`docker ps`看到的内容：

```
CONTAINER ID 	IMAGE    		COMMAND 	CREATED 		STATUS 			PORTS 										NAMES
db13127cb76b php71-fpm-centos68-phalcon-withext  "/run.sh"   1 second ago  Up 1 second   0.0.0.0:80->80/tcp, 0.0.0.0:9000->9000/tcp   yphp
```
表名容器正则运行。如果`STATUS`是Exit(1)则说明容器异常退出，如果Exit(0)说明容器只执行了命令就立即退出了。
可以使用`docker logs db13127cb76b` 查看容器具体退出情况。db13127cb76b是某个容器ID（CONTAINER ID）。
如果是Exit(0)可能是没有维持住后台运行，建议在命令加入`tail -f`这种可以一直运行的命令。

3、如果想自行配置php.ini、nginx.conf，可以将配置目录映射到宿主机：
``` bash 
docker run -d --name yphp -p 80:80 -p 9000:9000 \
	-v /work/:/work/  \
	-v "/work/yphp/php/etc/":/usr/local/php/etc/  \
	-v "/work/yphp/nginx/conf/":/usr/local/nginx/conf/  \
	-v "/work/yphp/nginx/logs/":/usr/local/nginx/logs/  \
	php71-fpm-centos68-phalcon-withext
```

其中`/work/yphp`是本地宿主机的一个目录，里面包含php和nginx的配置，目录结构：
```
/work/yphp/
|--nginx
	|--conf
		|--nginx.conf
		|--vhost
|--php
	|--etc
		|--php.ini
		|--php-fpm.d
			|--www.conf
```
如果不想对每个配置都进行映射，也可以指定需要映射的配置文件：
```bash
docker run -d --name yphp -p 80:80 -p 9000:9000 \
	-v /work/:/work/  \
	-v "/work/yphp/php/etc/php.ini":/usr/local/php/etc/php.ini  \
	-v "/work/yphp/nginx/conf/nginx.conf":/usr/local/nginx/conf/nginx.conf  \
	-v "/work/yphp/nginx/conf/vhost/":/usr/local/nginx/conf/vhost/  \
	-v "/work/yphp/nginx/logs/":/usr/local/nginx/logs/  \
	php71-fpm-centos68-phalcon-withext
```

>注意：由于这里覆盖了php.ini，所以想要开启第三方扩展的话，想要提前在php.ini里追加：
```
extension=redis.so
extension=swoole.so
extension=yar.so
extension=yaconf.so
extension=phalcon.so
extension=seaslog.so
extension=gearman.so
extension=mongodb.so
extension=tideways.so
extension=protobuf.so
extension=msgpack.so
```

或者启动后追加然后重启容器里的php-fpm：
``` bash
docker exec 容器ID killall php-fpm
docker exec 容器ID php-fpm
```

4、如果不需要一开始就运行服务，想进入容器自行启动，可以直接进入容器：
``` bash
docker run -it --name yphp -v /work/:/work/ php71-fpm-centos68-phalcon-withext /bin/bash
```

退出容器使用exit。

5、想对当前容器打包为镜像，方便后续使用：（需要先停止容器）
``` bash
# 从容器生成镜像
# -m 加一些改动信息，-a 指定作者相关信息，db13127cb76b这一串为容器id，再后面为新镜像的名字
docker commit -m "create images" -a "52fhy"  db13127cb76b  php71-fpm-centos68:yphp_v1 

# 导出镜像
docker save -o yphp.tar php71-fpm-centos68-phalcon-withext:yphp_v1

# 导入镜像
docker load --input yphp.tar
```

导出的镜像可以压缩以节约空间。

6、增加PHP扩展  

有下面几种方法，只要编译出了.so文件就简单了。以演示xdebug为例：

1) 如果已经编译好了.so文件，直接复制到挂载目录(例如`/work`)。使用`docker exec -it 容器ID /bin/bash`进入容器，通过`php-config`可以看到扩展所在目录，复制到该目录，修改php.ini，追加`zend_extension=xdebug.so`，重启php-fpm即可。

>注：一般扩展都是`extension=xxx.so`，xdebug特殊。

2) 需要先进入容器，使用`pecl install xdebug`自动编译生成.so文件。会自动复制到扩展目录。修改php.ini重启php-fpm生效。

3) pecl如果提示找不到该扩展，则使用源码编译。例如：
``` bash
git clone https://github.com/xdebug/xdebug.git
cd xdebug/
phpize 
./configure 
make
make install
```

一般可以在pecl.php.net找到源码。例如：http://pecl.php.net/package/xdebug 。编译方法差不多：
``` bash
wget http://pecl.php.net/get/xdebug-2.5.5.tgz \
    && tar xzf xdebug-2.5.5.tgz && cd xdebug-2.5.5/ \
	&& phpize \
    && ./configure \
    && make && make install
```

7、compose一键启动

docker-compose.yml
```
# development.yml
version: '2'
services:
  php-fpm:
    image: php71-fpm-centos68-phalcon-withext
    restart: always
    volumes:
      - /work/:/work/
      -"/work/yphp/php/etc/":/usr/local/php/etc/  
      -"/work/yphp/nginx/conf/":/usr/local/nginx/conf/  
      -"/work/yphp/nginx/logs/":/usr/local/nginx/logs/  
    ports:
      - 90:9000
      - 80-90:80-90
```

然后：
```
compose up -d 
```

或者不使用镜像里的nginx：
```
version: '2'
services:  
  nginx:
    image: daocloud.io/library/nginx:1.12.2-alpine
    restart: always
	links: 
        - php-fpm:yphp71
    volumes:
      - /work/:/work/
      -"/work/yphp/nginx/conf/":/usr/local/nginx/conf/  
      -"/work/yphp/nginx/logs/":/usr/local/nginx/logs/  
    ports:
      - 80-90:80-90
	  
  php-fpm:
    image: php71-fpm-centos68-phalcon-withext
    restart: always
    volumes:
      - /work/:/work/
      -"/work/yphp/php/etc/":/usr/local/php/etc/
```

注意：nginx容器由于使用了links，那么nginx配置文件里fastcgi_pass需要改成 yphp71:9000；php-fpm容器的php-fpm.d/www.conf里listen需要监听地址 0.0.0.0，而不是127.0.0.1。




