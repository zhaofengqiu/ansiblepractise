# 原因

Ansible可以在两个方面简化Docker的工作。

其一就是Docker容器的编排。当你部署一 个“Docker化”的软件应用时，通常会创建多个Docker容器，并且容器内运行着不同的服 务。这些服务需要互相进行通信，因此你需要让容器按需互联，并且确保它们按照正确的 顺序启动。起初，Docker项目并没有提供编排工具，因此出现了一些第三方工具来填补这 个缺口。Ansible就是为编排而编写的，因此它天然适合部署基于Docker的应用。

另一个方面是创建Docker镜像,当开始创建更复杂镜像的时候，你马上就会想念Ansible提供的强大功能了。幸运 的是，你是可以使用Ansible的。



# 如何联系

+ 在本地主机上创建Docker镜像。
+ 将Docker镜像从本地主机推送到镜像仓库。
+ 将Docker镜像从镜像仓库中下载到远程主机
+ 在远程主机上启动Docker容器，在容器启动的时候将所需的配置信息传给它。



在Ansible 2.1发布时，Ansible项目发布了一个叫Ansible Container的工具，用于简化 Docker镜像和容器的相关工作。Ansible Container从Conductor容器中挂载含有Python运行时和相关库的目录到正在配 置的容器中。具体操作是，挂载Conductor容器中的/usr到正在配置容器的/_usr目录，配置 Ansible使用/_usr下面的Python解释器。

# 创建Docker镜像

下面我们使用Ansible Container创建Nginx镜像，进行相关操作

## 初始化命令

```
ansible-container init 
```

这个命令会在当前目录下面创建这些文件

```
|---ansible-requirements.txt
|---ansible.cfg
|---container.yml
|---requirementes.yml
```

## 创建role

我们需要一个配置容器的role。我们的role叫作ghost-nginx，因为它配置了一 个放在Ghost前面的Nginx镜像.

task/main.yml 文件内容

```
---
- name: remove default config
  file:
    path: /etc/nginx/conf.d/default.conf
    state: absent
- name: add ghost config
  copy:
    src: ghost.conf
    dest: /etc/nginx/conf.d/ghost.conf
```

## 配置 container.yml

这个文件也就是我们的playbook

类似于Docker Compose文件，不同的是它带有一些Ansible特有的字段，支持Jinja2风格的 变量，用于替换和过滤。

```
version: "2" #Ansible Container支持Docker Compose 版本2协议。
settings:
  conductor_base: debain:jessie # 使用debian:jessie作为Conductor容器的基础镜像
services: #service字段是一个映射，key是我们将要创建的容器的名字。中仅有一个容器。
  ac-nginx: # 我们为将要创建的容器起名为ac-nginx
    from: nginx # 指定nginx作为基础镜像。
    command: [nginx, -g,daemon off;]# 指定容器启动时需要运行的命令。
    roles: # 指定用于配置镜像的role。本例仅有一个role,ghost-nginx。
      - ghost-nginx
registries:{}# registries字段用于指定其他外部registry。我们没有配置它，留空。	
```

Ansible Container 不会将基础镜像自动拉取到本地，你必须在构建容器之前手动拉取基础镜像

```
docker pull nginx
```



## 构建镜像

```
ansible-container build
```

# 本地运行

```
version: "2" 
settings:
  conductor_base: debain:jessie  
services:  
  ac-nginx:  
    from: nginx  
    command: [nginx, -g,daemon off;]
    roles:  role,ghost-nginx。
      - ghost-nginx
    ports:
      - "8443:443"
      - "8000:80"
    dev_overrides:# 在该区段中包含本地运行的内容
      volumns:
        - $PWD/certs:/certs
      links:
        - ghost
  ghost:
    from: ghost
    dev_overrides:
     volumns:
       - $PWD/ghostdata:/var/lib/ghost
registries:{} 
```

```
可以使用如下命令启动本地机器上的容器：
$ ansible-container
可以使用如下命令停止它们：
$ ansible-container stop
如果想停止所有的容器并删除你创建的所有镜像，可使用如下命令：
$ ansible-container destroy
```



# 发布镜像到registry



需要配置container.yml中的registries区段指定一个registry。

![image-20201102021307546](../img/image-20201102021307546.png)

第一次推送镜像的时候，需要指定用户名作为命令行参数：

```
ansible-container push--username $YOUR_USERNAME
```



# 将容器部署到生产环境

TODO



