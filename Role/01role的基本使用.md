role的作用是编程语言中的模块。模块化编程可以降低代码的编写，我们可以把一些函数放到模块中，通过role可以使得playbook结构更加简单。

role其实就是将一些task划分出去，在外部引用关键字，从而实现task的复用。创建role就是在`roles`文件夹中创建相应role的文件夹。一个role文件的放置位置

```
roles/module_name/tasks/main.yml
```



在外部play-book中，只要在tasks下进行相应的role引入进来。

```powershell
tasks:
  -  incloud_role:
       name:install_python
  -  include_role:
       name:deploy-code
     
```

由于引入是写死的，所以我们设置role的目录结构也必须写死。

```
roles/module_name/tasks/main.yml
```

只有如上的role文件才能在tasks中引入。

我们可以按照不同的模块进行role的划分，比如安装python的task都放到`roles/install-python`文件夹中。将task划分出去





那么能够引入一个role中的一个模块的部分task嘛？



Ansible到哪里查找我的role

Ansible将会到与你的playbook并列的roles目录下寻找role。

也会在在/etc/ansible/roles中查找系统级的role。

```
# 文件组成:
# task文件
roles/module_name/tasks/main.yml
# 上传到主机的文件，这样子就不用考虑roles如果需要文件那应该放在哪里了
# 模板文件
roles/module_name/templates
# handler文件
roles/database/handlers/main.yml
# 不应该被覆盖的变量
roles/database/vars/main.yml
# 可以被覆盖的默认变量
roles/database/defaults/main.yml
# role的从属信息
roles/database/meta/main.yml
```

## 如何引用

在tasks下的列表中使用即可

第一种方式，当使用role的时候，需要在playbook中增加一个roles区段。roles区段需要列出一个所 使用role的列表。在我们的范例中，这个列表中包括两个role:database和mezzanine。

```yaml
- name: deplay mezzanine  on vagraant
  vars_files:
    - secrets.yml
  roles: 
    - role: database
      database_name: "{{ mezzanine_proj_name}}"
```

第二种方式，就是在外部play-book中，只要在tasks下进行相应的role引入进来。

```yaml
tasks:
  -  incloud_role:
       name:install_python
  -  include_role:
       name:deploy-code
     
```

对于第一种方式，我们需要考虑在调用你的role之前或者之后运行一些task。因此，使用pre-tasks和post-tasks部分

```
- name: deplay mezzanine  on vagraant
  vars_files:
    - secrets.yml
  pre_tasks:
    - name: update the apt cache
      apt: update_cache=yes
  roles: 
    - role: database
      database_name: "{{ mezzanine_proj_name}}"
  post_tasks:
    - name: notify slack that servers have been updated
      local_action:>
        slack
        domaon=acme.slack.com
        ...
```

从属roles

```
dependencies:
-{ role: ntp,ntp_server=ntp.ubuntu.com }
```

根据ansible的特性，当定义一个role的时候，可以指定他依赖一个或者多个其他role，ansible将会确保被指定为从属的role一定会被先执行。执行完之后，再执行本role。

