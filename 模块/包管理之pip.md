## 使用虚拟环境

使用虚拟环境进行安装软件

```shell
- name: pip install flask
      pip:
        name: flask 	
        state: present
        virtualenv: /home/vagrant/vir/
        virtualenv_command: /usr/bin/python3 -m venv

```

首先创建一个虚拟环境，创建好之后再进行flask的安装

先执行`virtualenv`与`virtualenv_command`再执行安装flask。

pip需要注意的就是如何使用ansible中的pipi模块进行虚拟环境的安装。

