观察当前的 Playbook，不能发现，当我的配置⽂件没有发⽣变化 时，每次依然都会去触发TASK "reload nginx server"。 如何能做到只有配置⽂件发⽣变化的时候才去触发TASK "reload nginx server"，这样的处理才是最完美的实现。此时可以使⽤ handlers 属性。

```YAML
 - name: handlers playbook example
   hosts: webservers
   gather_facts: no
   tasks:
    - name: update nginx main config
      copy: src=nginx.conf dest=/etc/nginx/
      tags: updateconfig
      notify: reload nginx server
handlers:
  - name: reload nginx server
    service: name=nginx state=reloaded
    when:
      - nginxsyntax.rc == 0
      - nginxrunning.stat.exists == true
```

问题就是，程序怎么知道它并没有发生改变？

