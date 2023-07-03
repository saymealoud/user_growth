# dockerfile 命令说明
参考官网文档：
https://docs.docker.com/engine/reference/builder/
## FROM
设置当前镜像的基础镜像。

这个基础镜像，一般是在基础的操作系统之上，安装了一些需要的基础库和组件等。

下面是FROM命令的3种用法。
````
FROM [--platform=<platform>] <image> [AS <name>]
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
````
最常用也是最简单的用法，如下：
````
FROM alpine:3.16
````
如果没有指定:tag或者@digest，那么默认是最新版本:latest

参数
--platform=linux/amd64, linux/arm64, or windows/amd64

指定镜像的运行平台，MacBook M1芯片的电脑默认是 linux/arm64 平台，这类镜像在linux/amd64的服务器上是无法正常运行的。

windows上同理。

所以在制作镜像的时候，也需要注意这个参数。

公司内还是直接在linux服务器上做一个专门的镜像制作环境吧，和正式服务器运行环境保持一样，减少出错。

AS 指令可以给当前制作的镜像起一个别名。

在同文件中，下面另外一个镜像中可以被引用到。

````
FROM alpine:3.16 AS SourceCoder
RUN echo "abcd" > /root/abcd.txt
FROM alpine:3.16
RUN mkdir -p /data/code
COPY --from=SourceCoder /root/abcd.txt /data/code/abcd.txt
````
这种用法虽然不多，但是在多阶段源码编译的时候，需要在多个环境中才可以最终完成镜像制作，这时候就有用了。
## RUN
制作镜像时执行的shell命令。
可以是下面这几种写法。
````
RUN /bin/bash -c 'source $HOME/.bashrc; \
echo $HOME'
RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
RUN ["/bin/bash", "-c", "echo hello"]
RUN source $HOME/.bashrc && echo "hello"
RUN echo "hello"
````
如果是非常多、非常复杂的命令，建议把所有的命令写到一个shell脚本里面。

这样就只需要执行这个脚本就好了，不需要处理复杂的格式。
## CMD
镜像运行时，启动容器的第一个命令，也是1号进程，需要是一个服务类进程。

这个1号进程终止，整个容器也就终止了。

一个文件中只有最后一个CMD有效。
````
CMD ["executable","param1","param2"] (exec形式，首选)
CMD ["param1","param2"] (作为ENTRYPOINT命令的参数)
CMD command param1 param2 (shell形式)
````
类似RUN，多且复杂的命令写入到shell脚本里面，执行一个脚本更简单。
## EXPOSE
暴露容器内的端口，比如，容器内有web服务监听了80端口，就可以用它。
````
EXPOSE <port> [<port>/<protocol>...]
EXPOSE 80
EXPOSE 80/tcp
EXPOSE 80/udp
````
可以写多行，暴露多个端口。
也可以在容器启动时覆盖或者指定暴露的端口，如下：
````
docker run -p 80:80/tcp -p 80:80/udp ...
````
## ENV
设置环境变量，可以在镜像构建时使用，也可以在容器运行时使用。
````
ENV <key>=<value> ...

ENV MY_NAME="John Doe"
ENV MY_DOG=Rex\ The\ Dog
ENV MY_CAT=fluffy

ENV MY_NAME="John Doe" MY_DOG=Rex\ The\ Dog \
    MY_CAT=fluffy
    
ENV MY_VAE my-value
````
命令相当于linux里面的 
````
expose MY_NAME="John Doe"
````
## ADD
从 src 复制新文件、目录或远程文件URL，并将它们添加到路径 dest 的镜像文件系统中。
````
ADD [--chown=<user>:<group>] [--checksum=<checksum>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]

ADD --chown=55:mygroup files* /somedir/
ADD --chown=bin files* /somedir/
ADD --chown=1 files* /somedir/
ADD --chown=10:11 files* /somedir/
````
如果 src 是一个URL，会下载到 dest(COPY不支持)。

如果 src 是本地的一个压缩包文件，会自动解压到 dest 目录(COPY不支持)。

还支持下载git仓库(COPY不支持)。
````
# syntax=docker/dockerfile-upstream:master-labs
FROM alpine
ADD --keep-git-dir=true https://github.com/moby/buildkit.git#v0.10.1 /buildkit
ADD git@git.example.com:foo/bar.git /bar
````
## COPY
和前面的ADD命令很像，都是复制文件或者目录到镜像内。
````
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]

COPY --chown=55:mygroup files* /somedir/
COPY --chown=bin files* /somedir/
COPY --chown=1 files* /somedir/
COPY --chown=10:11 files* /somedir/
````
COPY命令支持一个选项 --from=<name> ，参考上面的FROM命令(ADD不支持)。

建议使用更简单的COPY命令。

ADD支持更复杂的功能，比如：远程文件下载，还是用curl/wget来支持吧。
解压缩直接用tar -x来完成也就好了。
## ENTRYPOINT
类似上面的CMD命令，定义容器启动时的执行命令。
````
ENTRYPOINT ["executable", "param1", "param2"] (exec形式)
ENTRYPOINT command param1 param2 (shell形式)

docker run --entrypoint (可以覆盖镜像的ENTRYPOINT指令)
````
Dockerfile应至少指定CMD或ENTRYPOINT命令之一。

将容器用作可执行文件时，应定义ENTRYPOINT。

CMD应该用作为ENTRYPOINT命令定义默认参数的方法。

当使用可选参数运行容器时，CMD将被重写。

官方文档中还有一个表格，说明了CMD/ENTRYPOINT两个命令交互关系与结果，需要用到的时候，可以再具体看下。
## VOLUME
创建一个挂载点。
````
VOLUME ["/data"]

VOLUME /myvol
````
## USER
设置用户和用户组
````
USER <user>[:<group>]
USER <UID>[:<GID>]

USER patrick
````
## WORKDIR
设置镜像、容器的当前工作目录。
````
WORKDIR /path/to/workdir

WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
当前的工作目录是 /a/b/c 目录
````
## ARG
设置镜像制作时的参数。
````
ARG <name>[=<default value>]

FROM ubuntu
ARG CONT_IMG_VER
ENV CONT_IMG_VER=v1.0.0
RUN echo $CONT_IMG_VER
$CONT_IMG_VER 读取到 ENV CONT_IMG_VER 的值 v1.0.0
````
更多命令和更详细的解释，请参考官网文档：
https://docs.docker.com/engine/reference/builder/