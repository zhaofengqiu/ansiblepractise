分为动态inventory变量与静态inventory变量

##  inventory静态变量

### 主机变量

在inventory文件中跟在主机名后

```ini
[webserver]
172.18.0.3 user=lilei port=3309
172.18.0.4 user=lilei port=3309
example.com user=lilei port=3309
```

其中user与port就是静态变量



### 组变量

我们可以在inventory文件中统一设置变量，这样子就不用将变量进行单一设置，上面的静态变量可以写成下面的组变量

```ini
[webserver]
172.18.0.3 
172.18.0.4  
example.com 

[webserver:var]
user=lilei 
port=3309
```



### 主机组变量与主机变量冲突

那么优先以主机变量为准。原理就和编程中的局部变量与全局变量类似，函数中的局部变量优先级更高。

### 变量的继承

```ini
# cat hosts_v3
[webservers]
172.18.0.3
[dbservers]
172.18.0.4
[allservers]
[allservers:children]
dbservers
webservers
[allservers:vars]
user=lilei
```

 在资产继承的同时，对应的变量也发⽣了继承

```bash
# ansible allservers -i hosts_v3 -m debug -a
"var=user"
172.18.0.4 | SUCCESS => {
 "user": "lilei"
}
172.18.0.3 | SUCCESS => {
 "user": "lilei"
}
# ansible dbservers -i hosts_v3 -m debug -a
"var=user"
172.18.0.4 | SUCCESS => {
 "user": "lilei"
}
# ansible webservers -i hosts_v3 -m debug -a
"var=user"
172.18.0.3 | SUCCESS => {
 "user": "lilei"
}
```



### 内置变量

```
ansible_ssh_host  将要连接的远程主机名与你想要设定的主机的别名不同的话,可通过此变量设置.
ansible_ssh_port  ssh端⼝号.如果不是默认的端⼝号,通过此变量设置.
ansible_ssh_user  默认的 ssh ⽤户名
ansible_ssh_pass  ssh 密码(这种⽅式并不安全,官⽅强烈建议使⽤ --askpass 或 SSH 密钥)
ansible_sudo_pass sudo 密码(这种⽅式并不安全,官⽅强烈建议使⽤ --asksudo-pass)
ansible_sudo_exe (new in version 1.8) sudo 命令路径(适⽤于1.8及以上版本)
ansible_ssh_private_key_file ssh 使⽤的私钥⽂件.适⽤于有多个密钥,⽽你不想使⽤ SSH代理的情况.
ansible_python_interpreter ⽬标主机的 python 路径.适⽤于的情况: 系统中有多个Python, 或者命令路径不是"/usr/bin/python",⽐如/usr/local/bin/python3
```

