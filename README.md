# dockerfile-first

## simple sample
创建一个文件Dockerfile-1
``` dockerfile
FROM alpine

ENTRYPOINT [ "echo"]
CMD ["first"]
```

``` bash
$ docker build -f .\Dockerfile-1 -t first .

$ docker run --rm first
first
$ docker run --rm first "sss"
sss
```

## hello sample

1. 创建文件
- 创建文件夹: D:\1.source\pythonpath, 并cd D:\1.source\pythonpath, 当然, 你也可以随意创建文件夹
- 创建文件夹: src,并在src下面创建两个空文件夹 inputs, outputs
- 创建文件README.md, 内容可以是空的
- 创建文件Dockerfile-hello, 内容如下:

``` dockerfile

FROM alpine as alpine-builder
# 参数: docker build时,有效
ARG a1=/src
# 参数: docker run时, 有效
ENV e1=1111

# 拷贝本地的REAME.md文件到docker的/src文件夹下
COPY ./README.md ${a1}/

# 创建两个文件夹,并且里面存放一个1.md的文件
RUN mkdir -p ${a1}/inputs \
    && mkdir -p ${a1}/outputs \
    && touch ${a1}/inputs/1.md \
    && touch ${a1}/outputs/1.md

# 容器启动是, 执行命令,会在该目录下执行
WORKDIR /src

ENTRYPOINT [ "sh","-c" ]
CMD [ "echo a1:${a1},e1:${e1} && ls -lR" ]

# CMD [ "sh","-c","echo a1:${a1},e1:${e1} && ls -lR" ]

```
2. 构建镜像
``` bash
$ docker build --no-cache -f .\Dockerfile-hello -t hello-dockerfile .

```
3. 运行镜像
``` bash
$ docker run --rm hello-dockerfile

a1:,e1:1111
.:
total 12
-rwxr-xr-x    1 root     root          2638 Sep 18 17:26 README.md
drwxr-xr-x    2 root     root          4096 Sep 18 17:27 inputs
drwxr-xr-x    2 root     root          4096 Sep 18 17:27 outputs

./inputs:
total 0
-rw-r--r--    1 root     root             0 Sep 18 17:27 1.md

./outputs:
total 0
-rw-r--r--    1 root     root             0 Sep 18 17:27 1.md
```
4. 运行镜像,带着`-v`参数

如果是windows: 把D: 改成/d,例如:

D:\1.source\pythonpath =>/d/1.source/pythonpath

docker run -v参数, 启动的时候, 永远是宿主机覆盖docker内的文件, 如果你发现宿主机的文件夹有被写入内容了, 那是在启动之后, 程序写入的内容

### 完全覆盖
容器内的inputs文件夹, 和 outputs文件夹,被宿主机的空文件夹覆盖了​
``` bash
$ docker run -v /d/1.source/pythonpath/dockerfile-first/src:/src --rm hello-dockerfile

a1:,e1:1111
.:
total 0
drwxrwxrwx    1 root     root          4096 Sep 18 16:07 inputs
drwxrwxrwx    1 root     root          4096 Sep 18 16:07 outputs

./inputs:
total 0

./outputs:
total 0
```

### 程序写入
容器内的inputs文件夹, 和 outputs文件夹,被宿主机的空文件夹覆盖了
​然后, 程序想outputs中写入了文件1111.md
``` bash

$ docker run -v /d/1.source/pythonpath/dockerfile-first/src:/src --rm hello-dockerfile "touch /src/outputs/1111.md && ls -lR"
.:
total 0
drwxrwxrwx    1 root     root          4096 Sep 18 16:07 inputs 
drwxrwxrwx    1 root     root          4096 Sep 18 17:34 outputs

./inputs:
total 0

./outputs:
total 0
-rw-r--r--    1 root     root             0 Sep 18 17:34 1111.md

```


## 说明

`docker build` 是把所有的文件, 打包到镜像里面
`docker run`的时候, 就是执行  `CMD` 或 `ENTRYPOINT`


`CMD`和`ENTRYPOINT`
- 共同点: 都是在`docker run`的时候执行
- 区别: 命令1 等价于 命令2

**命令1**
``` bash
# 作为指令
ENTRYPOINT [ "sh","-c" ]
# 所有内容都作为参数
CMD [ "echo 1111" ]
```

**命令2**
``` bash
# 第一个参数作为指令, 后面的都是参数
CMD [ "sh","-c","echo 1111" ]
```

`RUN`: 在`docker build`的时候执行

