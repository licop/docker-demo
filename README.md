## docker的实践学习

最近在搭建全找项目，用到mysql存储数据，发现使用docker安装部署mysql的有很多好处，可以方便迁移到其他环境，可以免除部署时因为环境和版本产生复杂的问题。而且可以安装多个docker实例，甚至不同版本，只要端口不同就不会项目影响。尤其当我们使用Docker Compose时，我们可以整个项目的安装程序都定义在一个程序中，这样就可以便于开源自己项目和便于其他人贡献代码。其他人只需克隆项目并通过**docker-compose**启动项目即可。可以做到像前端项目一样简单，没有比这跟简单的了。

### 关于docker

Docker的思想来自于集装箱，集装箱解决了什么问题？在一艘大船上，可以把货物规整的摆放起来。并且各种各样的货物被集装箱标准化了，集装箱和集装箱之间不会互相影响。那么我就不需要专门运送水果的船和专门运送化学品的船了。只要这些货物在集装箱里封装的好好的，那我就可以用一艘大船把他们都运走。

在实际应用中，不同的应用程序可能会有不同的应用环境，比如.net开发的网站和php开发的网站依赖的软件就不一样，如果把他们依赖的软件都安装在一个服务器上就要调试很久，而且很麻烦，还会造成一些冲突。比如IIS和Apache访问端口冲突。这个时候你就要隔离.net开发的网站和php开发的网站。常规来讲，我们可以在服务器上创建不同的虚拟机在不同的虚拟机上放置不同的应用，但是虚拟机开销比较高。docker可以实现虚拟机隔离应用环境的功能，并且开销比虚拟机小，小就意味着省钱了。

Docker技术的一些核心概念：

- **镜像（Image)** 是一个只读模板，用于指示创建容器。镜像分层(layers)构建的，而定义这些层次的文件叫Dockerfile, 
- **容器（Container）** 是镜像的可运行的实例。容器可通过API或CLI（命令行）进行操控。
- **仓库（Repository）** 负责对Docker镜像进行管理的服务，我们可以把自己的镜像发布上去，分享给其他人使用
- **Dockerfile**  Docker 可以依照Dockerfile 的内容，自动化地构建镜像。 Dockerfile 是包含着用户想要如何构建镜像的所有命令的文本。

### 常用命令解析

#### docker build -t test/myapp .

docker build 命令用于使用 Dockerfile 创建镜像，可以指定项目文件位置也可以使用github路径， -t给镜像命名; . 告诉应该在当前的目录下寻找Dockerfile文件。

#### docker run

  docker run 通过镜像创建一个新的容器并运行一个命令 
 
   - docker run  --name mynginx -d  -p 80:8080 nginx:latest    使用docker镜像nginx:latest以后台模式启动一个容器, 将容器的 8080 端口映射到主机的 80 端口，并将容器命名为mynginx。
   - docker run -d --restart=always --name my-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=licop mysql:5.7  通过mysql:5.7镜像部署一个 名为licop的mysql数据库

#### docker ps
   
   列出docker容器的信息
   
#### docker exec
   
   在容器里执行信息
   - docker exec -it <mysql-container-id> mysql -p 连接上指定容器的数据库
   - docker exec -it 9df70f9a0714 /bin/bash  通过 exec 命令对容器id为9df70f9a0714 执行 bash:

#### docker logs -f <container-id>
  
  可以查看容器运行的日志
    

### 使用 Docker Compose

通过命令建立多容器的项目

```
docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"

docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
```

对应完整的docker-compose.yml

```
  version: "3.7"

 services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

使用**docker-compose up -d** 和**docker-compose down**即可打开和关闭项目


### 更多文章学习

- [官方Getting Started文档](https://github.com/docker/getting-started)
- [知乎问题：如何通俗解释Docker是什么？](https://www.zhihu.com/question/28300645)
- [node官方文档：如何docker化node程序](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)
- [docker 命令大全](https://www.runoob.com/docker/docker-run-command.html)


