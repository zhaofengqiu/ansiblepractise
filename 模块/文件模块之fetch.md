## fetch模块

fetch模块就是从远端拉取文件到本地

```
- name: fetch a file
  fetch:
   src: 远端的文件
   dst: 本地的文件
```

如果本地没有这个目录。那么fetch模块就会自动创建出来，如果下拉的是文件夹，那么就会把整个文件夹拉下来。