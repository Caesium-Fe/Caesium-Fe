虚拟化的意义：便于管理物理硬件资源，提高硬件资源利用率。

```markdown
docker中有这样几个概念：
dockerfile
image
container

实际上你可以简单的把image理解为可执行程序，container就是运行起来的进程。
那么写程序需要源代码，那么“写”image就需要dockerfile，dockerfile就是image的源代码，docker就是"编译器"。
因此我们只需要在dockerfile中指定需要哪些程序、依赖什么样的配置，之后把dockerfile交给“编译器”docker进行“编译”，也就是docker build命令，生成的可执行程序就是image，之后就可以运行这个image了，这就是docker run命令，image运行起来后就是docker container。
```

## Centos下配置docker

```bash
-----用于centos6.x版本系统
#如果内核版本过低，则更新
yum update -y
#安装第三方扩展源(如果本机没有dockers源的话)epel-release：
yum install epel-release -y
#然后查看文件夹下是否有两个末尾带epel文件
ll /etc/yum.repos.d/ |grep epel
#安装docker软件包：
yum install docker-io lxc *cgroup* device-mapp* -y
#检测docker是否部署成功
yum list docker
rpm -qa|grep docker
#启动都docker引擎服务 
service docker start
#查看docker引擎服务进程
ps -ef|grep docker
#查看docker版本
docker version

-----用于centos7.x版本系统
#安装第三方扩展源(如果本机没有dockers源的话)epel-release：
yum install epel-release -y
#然后查看文件夹下是否有两个末尾带epel文件
ll /etc/yum.repos.d/ |grep epel
#安装docker软件包：
yum install docker -y
#检测docker是否部署成功
yum list docker
rpm -qa|grep docker
#启动都docker引擎服务
systemctl start docker.service
#查看docker引擎服务进程
ps -ef|grep docker
#查看docker版本
docker version
```

## 使用docker实现web容器

```bash
#搜索nginx镜像
docker search nginx
#从docker仓库中下载nginx镜像（下载对应名称）
docker pull docker.io/nginx
#查看已下载nginx镜像列表
docker images
docker images|grep -i nginx
ll /var/lib/docker/image/
#基于nginx镜像启动nginx应用容器
docker run -itd -p 80:80 nginx:latest
-p:publish发布端口，开启DNAT映射，将宿主机80(第一个)映射至容器80
注意这里宿主机的一个端口只能属于一个容器，但是容器可以都为同一个端口，但映射在宿主机上，宿主机的端口就必须不同。
#查看Nginx容器的运行状态和ip
docker ps
#完整信息 --->"IPAddress":"172.17.0.2"
docker inspect $(docker ps -aq)|grep -i ipaddr|tail -1
#只看到ip地址 --->172.17.0.2
docker inspect $(docker ps -aq)|grep -i ipaddr|tail -1|cut -d"\"" -f4
docker inspect $(docker ps -aq)|grep -i ipaddr|tail -1|awk -F\" '{print $4}'
docker inspect $(docker ps -aq)|grep -i ipaddr|tail -1|grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}"
docker inspect $(docker ps -aq)|grep -i ipaddr|tail -1|sed 's/"//g;s/ //g;s/,//g;s/://g;s/IPAddress//g'
#通过浏览器访问宿主机ip+80端口，Nginx web服务

#循环新建100个容器
for i in 'seq 0 99';do docker run -itd -p 80$i:8080 tomcat:latest;done

#查看容器运行状态
docker ps|grep tomacat
#移除多余容器
docker rm -f 需要移除的容器id
```

## 配置nginx负载均衡tomacat服务器

```bash
#首先进入到nginx容器中,id是容器进程id
docker exec -it id /bin/bash
#直接在当前容器目录里搜索配置文件
find / -name nginx.conf
#将搜索到的目录打开
cd /etc/nginx
#查看conf文件
cat nginx.conf
#注意nginx的虚拟主机在include中，所以直接复制打开目录就行
#查看default.conf,但是会有空行和带#的无用行第二个指令可以去除
cat default.conf
grep -vE "#|^$" default.conf
#需要修改时，由于docker容器里并没有安装编辑工具，所以使用sed命令编辑


```

