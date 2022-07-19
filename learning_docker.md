# 安装
## 使用国内源
```
sudo yum-config-manager \
--add-repo \
https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
```
## 卸载
```
sudo yum remove docker \
docker-common \
docker-selinux \
docker-engine
```
## 安装依赖包
```
sudo yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
```
## 使用yum安装 Docker CE
```
sudo yum-config-manager --enable docker-ce-edge
sudo yum makecache fast
sudo yum install docker-ce

```
## 全自动安装 Docker CE
```
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```
## 查看版本信息
```
sudo docker version
```
## 启动 Docker CE
```
$ sudo systemctl enable docker
$ sudo systemctl start docker
```
## 加入docker用户组
```
# 添加docker用户组，一般已存在，不需要执行
sudo groupadd docker     
# 将当前用户添加到docker用户组
sudo gpasswd -a $(whoami) docker或sudo gpasswd -a <用户名> docker
重启docker服务
sudo service docker restart
sudo systemctl restart docker.service
# 测试
docker version 

还不行就得重启centos系统后生效
```

## 添加内核参数
可选
```
$ sudo tee -a /etc/sysctl.conf <<-EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

$ sudo sysctl -p
```
## 配置加速器地址
进入阿里云->容器镜像服务->镜像工具->镜像加速器进行配置即可。可以执行：cat /etc/docker/daemon.json 查看一下内容。还可以使用docker info命令查看一下，镜像仓库是不是阿里云镜像就可以了




# 容器
## 启动ubuntu的bash（交互式，非守护）
运行了几个简单的命令，安装了vi，注意在容器里exit后安装的更新就都没了...  
启动时候也可以指定版本：docker run -t -i --name new_container ubuntu:12.04 /bin/bash 
* -t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上
* -i 则让容器的标准输入保持打开。
```
docker run -i -t ubuntu /bin/bash
root@063a7e6e1a9d:/# hostname
063a7e6e1a9d
root@063a7e6e1a9d:/# cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      063a7e6e1a9d
root@063a7e6e1a9d:/# ifconfig
bash: ifconfig: command not found
root@063a7e6e1a9d:/# ip
bash: ip: command not found
root@063a7e6e1a9d:/# ps -aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          1  0.0  0.0   4100  2172 pts/0    Ss   08:54   0:00 /bin/bash
root        619  0.0  0.0   5888  1412 pts/0    R+   09:02   0:00 ps -aux
root@063a7e6e1a9d:/# apt-get update && apt-get install vim
Hit:1 http://archive.ubuntu.com/ubuntu focal InRelease
Get:2 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:3 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Fetched 336 kB in 3s (128 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree
Reading state information... Done
vim is already the newest version (2:8.1.2269-1ubuntu5.7).
0 upgraded, 0 newly installed, 0 to remove and 33 not upgraded.
root@283fccc81a23:/# exit
```
## 容器命名
docker ps -a就可以看到定义的名字了，否则就是随机名字
```
[huawei@n148 ~]$ docker run --name bob_the_container -i -t ubuntu /bin/bash
root@1d3867b764e6:/# exit
exit
```
## 查看所有容器
```
[huawei@n148 ~]$ docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                       PORTS     NAMES
283fccc81a23   ubuntu         "/bin/bash"              2 minutes ago    Exited (130) 7 seconds ago             sharp_chatelet
063a7e6e1a9d   ubuntu         "/bin/bash"              10 minutes ago   Exited (0) 2 minutes ago               affectionate_ptolemy
0649fc44e8e7   nginx:latest   "/docker-entrypoint.…"   6 days ago       Exited (0) 6 days ago                  nginx-demo
4ae4617b7f67   55f4b40fe486   "/bin/sh -c 'apt-get…"   10 days ago      Exited (100) 10 days ago               upbeat_easley
fe635e905d30   hello-world    "/hello"                 2 years ago      Exited (0) 2 years ago                 pedantic_chandrasekhar
```
## 获得更多的容器信息
获取更全面的信息，且可以通过format加入筛选逻辑
```
docker inspect daemon_dave
docker inspect --format='{{ .State.Running }}' daemon_dave
docker inspect --format '{{ .NetworkSettings.IPAddress }}' daemon_dave
docker inspect --format '{{.Name}} {{.State.Running}}' daemon_dave bob_the_container
```
## run、start、restart、stop
除了使用名字还可以使用id作为参
```
[huawei@10 mynginx]$ docker run --name webserver -d -p 80:80 nginx
此时可以localhost查看默认页面，使用ps查看哪些或者的容器，然后可以关闭之
[huawei@10 mynginx]$ docker ps
[huawei@10 mynginx]$ docker stop webserver


[huawei@n148 ~]$ docker start bob_the_container
[huawei@n148 ~]$ docker restart bob_the_container
[huawei@n148 ~]$ docker stop bob_the_container

```
## attach与detach
如果容器正在运行则可以attach上去，参数也可以使用id
```
[huawei@n148 ~]$ docker attach bob_the_container

在容器里ctrl+d可以脱离
```
## 创建容器（守护式）
其实就是执行个dead loop，可以在外面stop停止之
```
[huawei@n148 ~]$ docker run --name daemon_dave -d ubuntu /bin/sh -c "while
> true; do echo hello world; sleep 1; done"
d7e2c3124fc0ed19d4793db768fbd2a1c7c5b90b214e8de2464a1d03054a14cd
[huawei@n148 ~]$ docker ps -a
CONTAINER ID   IMAGE          COMMAND                   CREATED          STATUS                        PORTS     NAMES
d7e2c3124fc0   ubuntu         "/bin/sh -c 'while\nt…"   4 seconds ago    Up 3 seconds                            daemon_dave
1d3867b764e6   ubuntu         "/bin/bash"               11 minutes ago   Exited (0) 4 minutes ago                bob_the_container
283fccc81a23   ubuntu         "/bin/bash"               15 minutes ago   Exited (130) 13 minutes ago             sharp_chatelet
063a7e6e1a9d   ubuntu         "/bin/bash"               23 minutes ago   Exited (0) 15 minutes ago               affectionate_ptolemy
0649fc44e8e7   nginx:latest   "/docker-entrypoint.…"    6 days ago       Exited (0) 6 days ago                   nginx-demo
4ae4617b7f67   55f4b40fe486   "/bin/sh -c 'apt-get…"    10 days ago      Exited (100) 10 days ago                upbeat_easley
fe635e905d30   hello-world    "/hello"                  2 years ago      Exited (0) 2 years ago                  pedantic_chandrasekhar


```
## 查看日志
```
[huawei@n148 ~]$ docker logs daemon_dave
hello world
hello world
hello world
hello world


这样相当于tail -f效果，需要手动ctrl+c才可退出
[huawei@n148 ~]$ docker logs -f daemon_dave

```
## 查看进程信息
```
查看top
[huawei@n148 ~]$ docker top daemon_dave


查看统计
[huawei@n148 ~]$ docker stats daemon_dave
```
## 在外部启动容器内的进程
在外部运行exec执行容器内的命令，然后进入容器查看结果，也可以直接进入交互
```
docker exec -d daemon_dave touch /etc/new_config_file
docker exec -t -i daemon_dave /bin/bash
仅有-i参数则是执行完毕就返回，-t则会有伪tty用于交互
使用ctrl+d可以detach
```

## 自动重启
运行run时刻加上 --restart=always 或 --restart=on-failure:5，会根据退出的返回值判断是否要重启，具体暂未测试

## 删除容器
需要先关掉再删，注意这里是仅仅删除容器，镜像不收影响
```
docker rm fe635e905d30
docker rm bob_the_container
docker rm `sudo docker ps -a -q`	删除所有，暂未测试

也可以清理所有已终止的容器
$ docker container prune

```
## 导入与导出
未测试
```
$ docker export 7691a814370e > ubuntu.tar
$ cat ubuntu.tar | docker import - test/ubuntu:v1.0
```
# 镜像

## 镜像的pull、push
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```
$ docker pull ubuntu:16.04
未提供镜像仓库地址从Docker Hub获取镜像，未提供仓库则取官方镜像library/ubuntu 仓库中标签为16.04的镜像。


也可以进行push，未测试
$ docker push username/ubuntu:17.10
```
## 列出当前拥有的
```
[huawei@n148 ~]$ docker images 或 docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
nginx         <none>    55f4b40fe486   2 weeks ago     142MB
nginx         latest    605c77e624dd   6 months ago    141MB
python        3.8       ce9555db0df5   6 months ago    909MB
ubuntu        latest    ba6acccedd29   8 months ago    72.8MB
python        3.5       3687eb5ea744   22 months ago   871MB
hello-world   latest    bf756fb1ae65   2 years ago     13.3kB
ubuntu        12.04     5b117edd0b76   5 years ago     104MB
```
## 删除本地镜像
可以指定id号，也可以匹配名字
```
[huawei@n148 mynginx]$ docker image rm 3687eb5ea744
[huawei@n148 mynginx]$ docker image rm $(docker image ls -q python)


```
## 查找镜像

```
[huawei@n148 ~]$ docker search gitbook
NAME                          DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
fellah/gitbook                GitBook                                         48                   [OK]
billryan/gitbook              Docker for GitBook with Unicode support         16                   [OK]
gitbook/nuts                  Releases/downloads server with auto-updater …   6
shuoshuo/gitbook-builder      A Docker Image for building PDF ebook with G…   2                    [OK]
khs1994/gitbook               Build GitBook In Docker                         1


获取后运行之
[huawei@n148 ~]$ docker pull fellah/gitbook
[huawei@n148 ~]$ docker run -i -t fellah/gitbook /bin/bash
```


## 登录与登出dockerhub
可以使用login、logout进行登入登出，并且在本地会记录此信息到文件
```
[huawei@n148 ~]$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: choupiwww
Password:
WARNING! Your password will be stored unencrypted in /home/huawei/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[huawei@n148 ~]$ cat $HOME/.docker/config.json
{
        "auths": {
                "https://index.docker.io/v1/": {
                        "auth": "Y2hvdXBpd3d3OjFxYXoyd3N4Mw=="
                }
        }
}[huawei@n148 ~]$docker logout
Removing login credentials for https://index.docker.io/v1/
[huawei@n148 ~]$ cat $HOME/.docker/config.json
{
        "auths": {}
}[huawei@n148 ~]$

```
## 使用commit制作镜像
首先使用了debain作为基础，然后update后下载了一个程序，最终commit之。  
其实可以将commit理解为将被修改的copy on write数据再次存储为一层镜像
```
[huawei@10 docker]$ docker run -it --name cowsay --hostname cowsay debian bash
root@cowsay:/# apt-get update
root@cowsay:/# apt-get install -y cowsay fortune
root@cowsay:/# /usr/games/fortune | /usr/games/cowsay
 _______________________________________
/ You will give someone a piece of your \
\ mind, which you can ill afford.       /
 ---------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
root@cowsay:/# exit
exit


docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]

[huawei@10 docker]$ docker commit cowsay test/cowsayimage
sha256:180aaa495ec129edacfb27597e9761326481948b18b2d772f25811694a4376d9

这里commit参数：
1 cowsay是原始的容器id
2 test/cowsayimage是新的镜像仓库和镜像名
另外还可以加入更多的参数，--author 是指定修改的作者，--message 则是记录本次修改的内容。
commit提交的只是创建容器的镜像与容器的当前状态之间有差异的部分，这使得该更新非常轻量。




[huawei@10 docker]$ docker run test/cowsayimage /usr/games/cowsay "Moo"
 _____
< Moo >
 -----
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
[huawei@10 docker]$

```
## 查看镜像的改动
上面基于debian的commit后可以查看原始镜像的改动步骤
```
[huawei@10 docker]$ docker ps -a
CONTAINER ID   IMAGE              COMMAND                  CREATED          STATUS                      PORTS     NAMES
3a64f9abce16   test/cowsayimage   "/usr/games/cowsay M…"   12 minutes ago   Exited (0) 12 minutes ago             vigorous_mclean
187431042c53   debian             "bash"                   15 minutes ago   Exited (0) 13 minutes ago             cowsay
[huawei@10 docker]$ docker diff 187431042c53

```
## 使用Dockerfile定制镜像
接下来将上面commit的流程换为自动化的脚本方式
```
$ mkdir cowsay
$ cd cowsay
$ touch Dockerfile

[huawei@10 cowsay]$ vi Dockerfile
FROM debian
RUN apt-get update && apt-get install -y cowsay fortune


构建镜像
[huawei@10 cowsay]$ docker build -t test/cowsay-dockerfile .
还支持从压缩包、Git repo、标准输入、管道来进行构建，暂未做测试


查看新构建的镜像
[huawei@10 cowsay]$ docker images
REPOSITORY               TAG       IMAGE ID       CREATED             SIZE
test/cowsay-dockerfile   latest    65b46cec189e   3 minutes ago       191MB
test/cowsayimage         latest    180aaa495ec1   27 minutes ago      191MB

然后运行之
[huawei@10 cowsay]$ docker run test/cowsay-dockerfile /usr/games/cowsay "Moo"
 _____
< Moo >
 -----
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

```
## 更进一步给Dockerfile传参
首先使用ENTRYPOINT设置启动程序
```
向Dockerfile加入ENTRYPOINT行后保存，重新build并运行

[huawei@10 cowsay]$ cat Dockerfile
FROM debian
RUN apt-get update && apt-get install -y cowsay fortune
ENTRYPOINT ["/usr/games/cowsay"]


[huawei@10 cowsay]$ docker build -t test/cowsay-dockerfile .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM debian
 ---> d2780094a226
Step 2/3 : RUN apt-get update && apt-get install -y cowsay fortune
 ---> Using cache
 ---> 65b46cec189e
Step 3/3 : ENTRYPOINT ["/usr/games/cowsay"]
 ---> Running in d139c26c00ed
Removing intermediate container d139c26c00ed
 ---> 51d00fa4a8fe
Successfully built 51d00fa4a8fe
Successfully tagged test/cowsay-dockerfile:latest
[huawei@10 cowsay]$ docker run test/cowsay-dockerfile "Moo"
 _____
< Moo >
 -----
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

```

更进一步使用另一个脚本来控制启动逻辑
```
创建sh脚本并设置执行权限

[huawei@10 cowsay]$ cat entrypoint.sh
#!/bin/bash
if [ $# -eq 0 ]; then
 /usr/games/fortune | /usr/games/cowsay
 else
 /usr/games/cowsay "$@"
fi
[huawei@10 cowsay]$ chmod +x entrypoint.sh


修改Dockerfile，先copy脚本到容器，改为启动此文件
[huawei@10 cowsay]$ cat Dockerfile
FROM debian
RUN apt-get update && apt-get install -y cowsay fortune
COPY entrypoint.sh /
ENTRYPOINT ["/entrypoint.sh"]



再次构建后再次启动，则可以实现根据参数输出结果了
[huawei@10 cowsay]$ docker build -t test/cowsay-dockerfile .
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM debian
 ---> d2780094a226
Step 2/4 : RUN apt-get update && apt-get install -y cowsay fortune
 ---> Using cache
 ---> 65b46cec189e
Step 3/4 : COPY entrypoint.sh /
 ---> 6d41a85a885f
Step 4/4 : ENTRYPOINT ["/entrypoint.sh"]
 ---> Running in dcdff02066b9
Removing intermediate container dcdff02066b9
 ---> ba2b01572d39
Successfully built ba2b01572d39
Successfully tagged test/cowsay-dockerfile:latest
[huawei@10 cowsay]$ docker run test/cowsay-dockerfile
 ________________________________________
/ Try the Moo Shu Pork. It is especially \
\ good today.                            /
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
[huawei@10 cowsay]$ docker run test/cowsay-dockerfile Hello Moo
 ___________
< Hello Moo >
 -----------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

```
## push到docker hub
需要提前使用cli登录docker
```
push前添加作者信息到Dockerfile，如下

[huawei@10 cowsay]$ cat Dockerfile
FROM debian
MAINTAINER Wei Hua <381513159@qq.com>
RUN apt-get update && apt-get install -y cowsay fortune
COPY entrypoint.sh /
ENTRYPOINT ["/entrypoint.sh"]

获取id号
[huawei@10 cowsay]$ docker images
REPOSITORY                               TAG       IMAGE ID       CREATED             SIZE
<none>                                   <none>    2e4b18061536   36 minutes ago      124MB
chpiwww/cowsay                           latest    ba2b01572d39   About an hour ago   191MB
test/cowsay-dockerfile                   latest    ba2b01572d39   About an hour ago   191MB
<none>                                   <none>    51d00fa4a8fe   About an hour ago   191MB
test/cowsayimage222                      latest    07853326409f   2 hours ago         191MB
test/cowsayimage                         latest    180aaa495ec1   2 hours ago         191MB
nginx                                    v3        640ba0b3b9ee   3 hours ago         141MB
nginx                                    v2        3bbeed8654df   3 hours ago         141MB
<none>                                   <none>    98775ec51603   13 days ago         191MB
debian                                   latest    d2780094a226   2 weeks ago         124MB
nginx                                    latest    605c77e624dd   6 months ago        141MB
hello-world                              latest    feb5d9fea6a5   9 months ago        13.3kB
ubuntu                                   16.04     b6f507652425   10 months ago       135MB
ubuntu                                   14.04     13b66b487594   15 months ago       197MB
soulteary/docker-gitbook-pdf-generator   1.0.0     539b69f2f478   23 months ago       507MB
debian                                   wheezy    10fcec6d95c4   3 years ago         88.3MB

设置tag
[huawei@10 cowsay]$ docker tag ba2b01572d39 choupiwww/cowsay:statle

push
[huawei@10 cowsay]$ docker push choupiwww/cowsay:statle
The push refers to repository [docker.io/choupiwww/cowsay]
7a41cb7f7f8f: Pushed
e840f05aea32: Pushed
97d5fec864d8: Pushed
statle: digest: sha256:c498f57211f6ea12d6a30dc3cf2d195edec9a8b38f8ac09606c30f57c64acb90 size: 948

push成功后可以在docker hub自己的主页中看到信息了，接下来可以pull一下
[huawei@n148 playground]$ docker search cowsay:statle
NAME      DESCRIPTION   STARS     OFFICIAL   AUTOMATED
[huawei@n148 playground]$ docker pull choupiwww/cowsay:statle
statle: Pulling from choupiwww/cowsay
1339eaac5b67: Pull complete
b960e817cb2a: Pull complete
3cad8f5c5233: Pull complete
Digest: sha256:c498f57211f6ea12d6a30dc3cf2d195edec9a8b38f8ac09606c30f57c64acb90
Status: Downloaded newer image for choupiwww/cowsay:statle
docker.io/choupiwww/cowsay:statle


接下来运行一下
[huawei@n148 playground]$ docker run choupiwww/cowsay:statle hooooo
 ________
< hooooo >
 --------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

```