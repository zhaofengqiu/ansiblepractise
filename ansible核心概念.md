## investory文件

管理节点的清单列表

可以按照组进行划分，将机子划分到不同的组中，比如web组和database组。这样子，在命令行中就可以直接通过管理组对这些组节点分块管理。

```
[web1]
...

[web2]
..

[database]
```



## play book

这个就是通过play book对不同的分组机器进行不同的task管理。

```
Role(web server)
	task(git)
	task(apache)
Role(database)
	task(mysql)
这个playbook的作用就是对web组执行git和apache两个命令,而对database组执行mysql指令。
```

其中的task就是一个action，一个Role中可以有多个action



## model

ansible通过源代码实现了很多的模块，而play book中就可以去进行调用这些model。

ansible官方手册

```
https://docs.ansible.com/
```





## set_fact 关键字

task中的单位是单个主机。






## 逐台主机运行

Ansible并行地在所有涉及的主机上执行每一个task。有时候你希望逐台 主机来执行task。最典型的例子就是对负载均衡后面的应用服务器进行升级。通常来说， 你会将应用服务器从负载均衡上摘除，更新它，然后再添加回去。但是你肯定不希望将所 有应用服务器都从负载均衡上摘除，这会导致服务不可用。

使用serial语句对Ansible并行执行play的主机数量进行限制。

使用max_fail_percentage语句来指定Ansible在将整个play置为失败 之前的最大失败主机比例。

如果在负载均衡后面有四台主机，并且有一台主机执行task失败，那么Ansible还将继 续执行play，因为没有超过25%的阈值。然而，如果有第二台主机执行task失败，Ansible 将会让整个play失败。

```
- name:  a host to run
  hosts: myhosts
  serial:1
  tasks:
  ...
```

如果要，一次运行一批主机，那么只要将serial改成其他数字或者传入百分比也可以。

## 只执行一次

例如，也许有多台 应用服务器运行在负载均衡后面，你希望执行一个数据库迁移，但是你只需在一台应用服 务器上执行这个迁移。

```yaml
- name: run the database migrations
  command: /opt/run_migrations
  run_once: true

```



对于每一个任务，Ansible都是在所有主机之间并行执行的。 

在开始下一个任务之前，Ansible会等待所有主机都完成上一个任务。

所以，一个任务会被全部inventory主机执行完毕之后再去执行。在自由策略下

一个playbook中的三个任务，会被分成三步，每一步都要等全部主机执行完之后才能到下一步。并且执行每一步的时候是前仆后继的补充主机。



ansible比shell文件更好的点：

+ Ansible对于模板有完美的支持，而且可以在各种位置中定义变 量。

## playbook文件

每个play必须包含如下两项。

name 描述这个play的一段文字注释。Ansible将会在play开始运行的时候将这段文字打印出 来。



become 如果为真，Ansible会在运行每个任务的时候都切换为（默认）root用户。在管理 Ubuntu服务器的时候，这个配置会非常有用，因为Ubuntu默认不允许使用root用户进行 SSH登录。但是这个也有一个前提，那么就是这个用户具有sudo权限，否则就无法切换成功。



## task执行

Ansible模块首先会在 采取任何行动之前检查主机的状态是否需要改变。如果主机的状态与模块的参数相匹配， 那么Ansible不会在这台主机上做任何操作并直接响应ok。



## 变量需要括号



```
- name
  host
  tasks:
    -
```

## 关于handler的几件需要记住的事情

handler只会在所有task执行完后执行。哪怕被通知了多次，它也只执行一次。

当play 中定义了多个handler时，handler按照play中定义的顺序执行，而不是通知的顺序。

handler唯一常见的用途是重启服务和重启机器。就我个人而 言，我仅用它重启服务。



## 提权

提权的前提，是用户在sudo组中·，如果不在就没法提权。

除了root用户，也可以切换到其他用户

```
become: True
become_user: postgres
```

切换到postgres用户

