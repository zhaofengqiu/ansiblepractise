我们执行一个playbook的时候，只想要执行其中的某些task，那么我们只要给这个playbook中的task增加标签即可。

```yaml

- hosts: test70
  remote_user: root
  tasks:
  - name: task1
    file:
      path: /testdir/t1
      state: touch
    tags: t1
  - name: task2
    file: path=/testdir/t2
          state=touch
    tags: t2
  - name: task3
    file: path=/testdir/t3
          state=touch
    tags: t3
```

执行

```shell
ansible-playbook --tags=t2 testtag.yml
```

我们可以使用 --skip-tags选项指定"不执行的任务"，执行上述命令后，task1和task3会执行，task2不会执行，因为我们已经在命令中指定了'跳过'标签t2所对应的任务，相当于使用了'排除法'，t2对应的任务被排除了，其他任务都会执行。

之前描述的三种语法都可以指定标签，不过上例中，每个任务只有一个标签，其实，我们可以为每个任务添加多个标签，三种语法添加多个标签的示例如下

```
语法一：
tags:
 - testtag
 - t1
 
语法二：
tags: tag1,t1
 
语法三：
tags: ['tagtest','t2']
```

上述示例的语法一使用了YAML块序列的语法格式指定多个标签，语法二与语法三都是在原来语法的基础上，使用'逗号'隔开多个标签。



参考文章

http://www.zsythink.net/archives/2641

