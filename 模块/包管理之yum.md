## 统一安装软件

参数：

name:代表包的名称

state: 

+ present 和installed 是直接安装软件
+ latest 确保安装的软件是最新的
+ remove 和absent是删除软件包

```shell
- name: file module
  hosts: web1
  tasks:
    - name: test the package
      yum: 
        name: git
        state: present
        
```

## 一次性安装多个软件

```shell
# 通过列表
- name: Install a list of packages (suitable replacement for 2.11 loop deprecation warning)
  yum:
    name:
      - nginx
      - postgresql
      - postgresql-server
    state: present
# 通过变量循环
- name: Install a list of packages with a list variable
  yum:
    name: "{{ packages }}"
  vars:
    packages:
    - httpd
    - httpd-tools
# 直接下载url进行安装
- name: Install the nginx rpm from a remote repo
  yum:
    name: http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
    state: present
- name: Install nginx rpm from a local file
  yum:
    name: /usr/local/src/nginx-release-centos-6-0.el6.ngx.noarch.rpm
    state: present
```



