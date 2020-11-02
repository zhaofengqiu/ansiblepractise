## 设置执行策略

默认的执⾏策略是按批并⾏处理的，假如总共 15 台主机，每次并 发 5 个线程执⾏的策略如下：

```
h1/h2/h3/h4h5
     |
     |
     |
h6/h7/h8/h9/h10 
     |
     |
     |
h11/h12/h13/h14/h15 
全部执⾏完后，进⼊下⼀批 依次类推
```



从 asible2.0 开始，可以通过在 playbook 中设置 strategy 的值改 变这策略，

也可以在 ansible.cfg 配置⽂件中设置⼀个默认的策略:

```init
[defaults]
strategy = free
```

或者在play-book中进行设置

```yaml
- hosts: webservers
  strategy: free
  tasks:
   - name: ping hosts
     ping:
```



改变后的策略，可以前赴后继的对主机进⾏执⾏ task，执⾏模式 如下：

```
h1/h2/h3/h4/h5 ------> h1/h2/h3/h6/h5 ------->h1/h2/h3/h6/h7 -----> 
```



## 设置多个执行队列

默认 Ansible 的执⾏队列有⼀个，就是并⾏执⾏，假如控制节 点的机器有多个 CPU，并且性能较好，可以打开多个执⾏队列，就 是并发。

⽅式⼀: 在 ansible.cfg 中设置

```ini
[defaults]
forks = 30
```

⽅式⼆： 在命令⾏⾥使⽤

```
ansible-playbook -f 3 my_playbook.yml
```

