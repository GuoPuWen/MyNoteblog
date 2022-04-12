本系列是docker学习连载篇

[docker--docker是什么？docker安装](https://blog.csdn.net/weixin_44706647/article/details/118443991)

[docker--docker容器命令、镜像命令](https://blog.csdn.net/weixin_44706647/article/details/118515325)

[docker--docker镜像原理](https://blog.csdn.net/weixin_44706647/article/details/118519771)

[docker--容器数据卷技术](https://blog.csdn.net/weixin_44706647/article/details/118541834)

[docker--docker网络模式](https://blog.csdn.net/weixin_44706647/article/details/118570457)

# Docker Compose作用

在前面，我们已经知道了，对于我们编写的应用，只需要编写一个dockerfile文件，便可快捷部署，编写dockerfile - build -运行，这样也就满足了我们的一些日常需求，但是随着网站流量的增加，可能需要做微服务、分布式等等，也就说不能再将全部的东西一股脑的放在一个容器上，例如我一个应用需要mysql环境、redis环境，nginx等等....如果这些应用都build成一个容器，每次运行的时候对每一个容器都要run一下，所以说这样还是很麻烦



docker-compose便是为了解决上面的问题，官方定义为定义和运行多个容器的应用，也就是实现一键build和run多个容器

官网地址为https://docs.docker.com/compose/

# 体验官方实例

安装Compose，可以查看官网文档如何安装，我是mac，所以在安装docker的时候已经安装好了这些实例，linux可以采取如下安装：

```shell
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

下载非常的慢，可以使用国内的安装源

```shell
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

//授执行权限
sudo chmod +x /usr/local/bin/docker-compose
```



在官方网站上提供一个使用python编写的例子，运行这个例子，体验docker compose带来的好处 https://docs.docker.com/compose/gettingstarted/

1. 创建composetest文件夹

```shell
[root@tdsql_1 ~]# mkdir composetest
[root@tdsql_1 ~]# cd composetest/
```

2. 创建app.py文件

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

3. 创建requirements.txt文件

```shell
flask
redis
```

4. 创建Dockerfile文件

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

文档上也做了非常详细的解释：

- 从Python 3.7映像开始构建一个映像。

- 将工作目录设置为/code。

- 设置flask命令使用的环境变量。

- 安装gcc和其他依赖项

- 复制requirements.txt并安装Python依赖项。

- 向映像添加元数据，以描述容器正在端口5000上侦听

- 复制当前目录。在项目到工作目录。的形象。

- 将容器的默认命令设置为flask run

5. 编写docker-compose.yml文件

```yaml
version: "3.3"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

6. 编译运行

```shell
[root@tdsql_1 composetest]# docker-compose up
```

7. 查看结果

```shell
[root@tdsql_1 ~]# curl localhost:5000
Hello World! I have been seen 1 times.
[root@tdsql_1 ~]# curl localhost:5000
Hello World! I have been seen 2 times.
[root@tdsql_1 ~]# curl localhost:5000
Hello World! I have been seen 3 times.
[root@tdsql_1 ~]# 
```

对上面例子的解释，可以稍微看一下python的代码，这就是一个计时器的应用，每当访问一次则将redis的hits节点加1，我们重点需要关注的细节是

![image-20210709115421869](http://cdn.noteblogs.cn/image-20210709115421869.png)

我们一连接的时候都是localhost:6379，可以发现在这个应用中并没有写ip地址，而是写一个hostname，可以推测出当使用docker-compose创建容器的时候，会自动创建一个网卡

![image-20210709115630349](http://cdn.noteblogs.cn/image-20210709115630349.png)

使用docker network inspect查看这张docker网卡的信息

```shell
[root@tdsql_1 ~]# docker network inspect composetest_default
```

![image-20210709115921171](http://cdn.noteblogs.cn/image-20210709115921171.png)

与我们创建的服务名刚好一样，也就是可以通过服务名访问到这些容器

# yaml编写规范

可以从上面发现，对于docker-compose应用最核心的地方在于docker-compose.yml的编写，而这块一般是分为三个层次:

- version：版本号，是向下兼容的
- services：服务
- 其他的东西，卷挂载等等

其实可以参照官方文档，已经写的很详细这块：https://docs.docker.com/compose/compose-file/

# 实战：搭建wordpress

wordpress是一款博客应用，相信很多人都搭建过，按照以往的是需要下载源码-改数据库-配置信息-运行，而有了docker-compose一切将会变得非常简单

官网地址：https://docs.docker.com/samples/wordpress/

1. 创建一个wordpress文件夹

```shell
[root@tdsql_1 composetest]# mkdir my_wordpress
[root@tdsql_1 composetest]# cd my_wordpress/	
```

2. 编写docker-compose.yml

```yml
version: "3.3"
    
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```

3. 运行

```shell
[root@tdsql_1 my_wordpress]# docker-compose up
```

![image-20210709141725871](http://cdn.noteblogs.cn/image-20210709141725871.png)

这样，搭建一个docker将会变得更简单

# 实战：用Java自己实现一个计数器

前面官网上用python给我们实现了一个计数器，这次我们使用Java SpringBoot项目创建一个微服务，模拟这个功能，并且自己编写docker-compose.yml

1. 使用idea创建SpringBoot项目，注意勾选web和redis
2. 编写controller

```java
@RestController
public class IndexController {

    @Autowired
    StringRedisTemplate redisTemplate;

    @GetMapping("/index")
    public String index(){
        Long view = redisTemplate.opsForValue().increment("view");
        return "view is" + view;
    }
}
```

3. 编写配置文件

```yaml
server.port=8080
spring.redis.host=redis
```

4. 编写Dockerfile

```dockerfile
FROM java:8

COPY *.jar /app.jar

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]
```

5. 编写docker-compose.yml

```yaml
version: '3.3'
services:
  webapp:
    build: .
    image: webappimage
    depends_on:
      - redis
    ports:
    - "8080:8080"
  redis:
    image: "library/redis:alpine"
```

6. 将jar包 Dockerfile docker-compose一起打包到服务器上
7. 运行

```shell
[root@tdsql_1 myspringboot]# docker-compose up
```

测试

![image-20210709145448488](http://cdn.noteblogs.cn/image-20210709145448488.png)

