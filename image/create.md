##创建我们自己的images
别人的镜像虽然好，但不一定适合我们。
我们可以对这些镜像做一些修改，有2个方法：利用现成的镜像进行修改或者利用dockerfile创建。

###使用docker commit 来扩展一个image
先使用下载的image启动容器。
```
$ sudo docker run -t -i training/sinatra /bin/bash
root@0b2616b0e5a8:/#
```
注意：记住容器的ID ，稍后还会用到。

在容器中添加json和gem两个应用。
```
root@0b2616b0e5a8:/# gem install json
```
当结束后，我们使用exit来退出，现在我们的容器已经被我们改变了，使用docker commint命令来提交更新后的副本。
```
$ sudo docker commit -m "Added json gem" -a "Docker Newbee" 0b2616b0e5a8 ouruser/sinatra:v2
4f177bd27a9ff0f6dc2a830403925b5360bfe0b93d476f7fc3231110e7f71b1c
```
其中，-m 来指定提交的说明信息，跟我们使用的版本控制工具一样；-a 可以指定更新的用户信息；之后是用来创建镜像的容器的ID；最后指定目标image的仓库名和tag信息。创建成功后会返回这个image的id信息。


使用docker images来查看新创建的image。
```
$ sudo docker images
REPOSITORY          TAG     IMAGE ID       CREATED       VIRTUAL SIZE
training/sinatra    latest  5bc342fa0b91   10 hours ago  446.7 MB
ouruser/sinatra     v2      3c59e02ddd1a   10 hours ago  446.7 MB
ouruser/sinatra     latest  5db5f8471261   10 hours ago  446.7 MB
```
之后，可以使用新的image来启动容器
```
$ sudo docker run -t -i ouruser/sinatra:v2 /bin/bash
root@78e82f680994:/#
```

###利用dockerfile 来创建 image
使用`docker commit` 来扩展一个image比较简单，但它不容易在一个团队中分享它。我们可以使用`docker build` 来创建一个新的image。为此，首先需要创建一个dockerfile，包含一些如何创建image的指令。

新建一个目录和一个dockerfile
```
$ mkdir sinatra
$ cd sinatra
$ touch Dockerfile
```
Dockerfile中每一条指令都创建image的一层，例如：
```
# This is a comment
FROM ubuntu:14.04
MAINTAINER Docker Newbee <newbee@docker.com>
RUN apt-get -qq update
RUN apt-get -qqy install ruby ruby-dev
RUN gem install sinatra
```
Dockerfile基本的语法是
* 使用#来注释
* FROM指令告诉docker 使用哪个image作为源image
* 接着是维护者的信息
* RUN开头的指令会在创建中运行，比如安装一个软件包，在这里使用apt 来安装了一些软件

编写完成Dockerfile后可以使用docker build来生成image。

```
$ sudo docker build -t="ouruser/sinatra:v2" .
Uploading context  2.56 kB
Uploading context
Step 0 : FROM ubuntu:14.04
 ---> 99ec81b80c55
Step 1 : MAINTAINER Kate Smith <ksmith@example.com>
 ---> Running in 7c5664a8a0c1
 ---> 2fa8ca4e2a13
Removing intermediate container 7c5664a8a0c1
Step 2 : RUN apt-get -qq update
 ---> Running in b07cc3fb4256
 ---> 50d21070ec0c
Removing intermediate container b07cc3fb4256
Step 3 : RUN apt-get -qqy install ruby ruby-dev
 ---> Running in a5b038dd127e
Selecting previously unselected package libasan0:amd64.
(Reading database ... 11518 files and directories currently installed.)
Preparing to unpack .../libasan0_4.8.2-19ubuntu1_amd64.deb ...
Setting up ruby (1:1.9.3.4) ...
Setting up ruby1.9.1 (1.9.3.484-2ubuntu1) ...
Processing triggers for libc-bin (2.19-0ubuntu6) ...
 ---> 2acb20f17878
Removing intermediate container a5b038dd127e
Step 4 : RUN gem install sinatra
 ---> Running in 5e9d0065c1f7
. . .
Successfully installed rack-protection-1.5.3
Successfully installed sinatra-1.4.5
4 gems installed
 ---> 324104cde6ad
Removing intermediate container 5e9d0065c1f7
Successfully built 324104cde6ad
```
其中-t标记来添加tag，指定新的image的用户信息。
.是Dockerfile所在的路径（当前目录），也可以替换为一个具体的Dockerfile的路径。

我们可以看到build进程在执行操作。它要做的第一件事情就是上传这个Dockerfile内容，因为所有的操作都要依据Dockerfile来进行。
然后，dockfile中的指令被一条一条的执行。每一步都创建了一个新的容器，在容器中执行指令并提交修改（就跟之前介绍过的`docker commit`一样）。当所有的指令都执行完毕之后，返回了最终的image id。所有的中间步骤所产生的容器都被删除和清理了。

*注意一个image不能超过127层

现在可以利用新创建的images来启动一个容器。
```
$ sudo docker run -t -i ouruser/sinatra:v2 /bin/bash
root@8196968dac35:/#

```
还可以用tag命令来修改images的tag。
```
$ sudo docker tag 5db5f8471261 ouruser/sinatra:devel
$ sudo docker images ouruser/sinatra
REPOSITORY          TAG     IMAGE ID      CREATED        VIRTUAL SIZE
ouruser/sinatra     latest  5db5f8471261  11 hours ago   446.7 MB
ouruser/sinatra     devel   5db5f8471261  11 hours ago   446.7 MB
ouruser/sinatra     v2      5db5f8471261  11 hours ago   446.7 MB
```