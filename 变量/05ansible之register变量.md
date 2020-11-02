往往⽤于保存⼀个task任务的执⾏结果, 以便于debug时使⽤。 或者将此次task任务的结果作为条件，去判断是否去执⾏其他task 任务。 注册变量在PlayBook中通过register关键字去实现。

```yaml
-  name: install a package and print the result
   hosts: webservers
   remote_user: root
   tasks:
     -  name: install nginx package
        yum: name=nginx state=present
        register: install_result
```

执行完这个task后，会将数据注册到install_result这个变量中。后面如果要使用变量，那么直接用这个install_result变量即可。

