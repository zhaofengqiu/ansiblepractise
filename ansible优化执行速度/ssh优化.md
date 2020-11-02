## 设置 SSH 为⻓连接



openssh5.6 版本后⽀持 Multiplexing

检查控制机器的 ssh 版本

```bash
[root@ansible-control command_module]# ssh -V
OpenSSH_7.4p1, OpenSSL 1.0.2k-fips  26 Jan 2017
```

设置 ansible 配置⽂件——`ansible.cfg`

```bash
ssh_args = -C -o ControlMaster=auto -o ControlPersist=10d
# ControlPersist=10d 表示保持⻓连接 10 天。
# 60s 是 60 秒
```

 建⽴⻓连接并测试,设置好后，重新连接⼀次被控主机，即可让控制主机和被控主机之 间建⽴⻓连接

验证⻓连接

```
[root@ansible-control command_module]#  ss -nta |grep ESTAB
ESTAB      0      0      192.168.243.139:22                 192.168.243.1:3224
ESTAB      0      0      192.168.243.139:22                 192.168.243.1:3221
ESTAB      0      0      192.168.243.139:22                 192.168.243.1:3225
```

输出中 有 ESTAB 状态的就代表是⻓连接，上文中有三台设备，目前处于长连接状态。

## 使用pipeline传输py文件

默认情况下 Ansible 执⾏过程中会把⽣成好的本地 python 脚本⽂件 PUT 到 远端机器。如果我们开启了 ssh 的 pipelining 特性，这个过程就会在 SSH 的会话中进⾏。

第一步，在 ansible.cfg 配置⽂件中设置 pipelining 为 True

```
pipelining = True
```

第二步，在远程机子上禁用`requiretty`

```bash
# Disable "ssh hostname sudo <cmd>", because it will
show the password in clear text.
# You have to run "ssh -t hostname sudo <cmd>".
#
# Defaults requiretty
```



