```
- name: net-tools module
  hosts: web1
  become: yes
  tasks:

    - name: test get_url
      get_url:
         url: https://www.python.org/ftp/python/3.9.0/python390.chm
         dest: /etc/test/
         checksum: md5:9ea6fc676f0fa3b95af3c5b3400120d6
    - name: 进行解压
      unarchive:
       src: /home/test/python390.chm
       dest: /home/test
       # 这个命令默认是使用本地的压缩文件，但是我们使用的是远程的压缩文件，
       # 所以，我们就需要设置remote_src这个参数
       remote_src: yes
```

