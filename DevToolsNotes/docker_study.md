# Docker 入门

基本重要概念：

> 仓库：可以理解为远程存储镜像的服务器；
>
> 镜像：本地仓库所拥有的可运行镜像，可以理解为面向对象编程中的类；
>
> 容器：镜像运行的具体实例，可以理解为面向对象编程中的对象。

## 常用命令

```shell
# 1. 下载镜像
docker pull [OPTIONS] NAME:[TAG]
# 2. 查看本地镜像
docker images [OPTIONS] [REPOSITORY:[TAG]]
# 3. 运行镜像
docker run [OPTIONS] IMAGE [COMMAND][ARG...]
# 4. 查看 docker 容器运行状态
docker ps
# 5. 进入到 docker 容器内部
docker exec -it <container_id or container_name> /bin/bash
docker exec -it <container_id or container_name> /bin/sh
# 5.1 容器中没用 bash or sh
docker exec -it <container_id> ash

```

### Docker 网络模式

> Bridge：每一个容器拥有自己的虚拟网卡（绝大部分都使用桥接模式）；
>
> Host：每一个容器与主机共享一张网卡；
>
> None：不为容器配置虚拟网卡。
>
> 桥接模式使用较多：一般地，配合端口映射使用：
>
> ```shell
> docker run -p local_port:container_port IMAGE
> ```

实操：通过 Docker 启动 `NGINX` 服务器，端口映射访问。

## Dockerfile

功能类似于 makefile，用于制作 docker 容器，下例是编写一个 Dockerfile 文件，将 go 项目中生成的可执行文件，复制到容器的 `/bin` 目录下，并且在容器中执行该可执行文件。

```dockerfile
FROM alpine:latest		# 引用的基本镜像，可以理解为继承
MAINTAINER hqing		# 容器制作的 author
COPY ./main /bin/my_main    # 将当前目录下的可执行文件复制到容器的 /bin 目录下
CMD /bin/my_main		# 执行容器中的可执行文件
```

