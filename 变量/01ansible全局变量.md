在命令行中输入进去的,在命令行进行输入的的变量,使用`-e`参数

## key=value 的形式

```
 ansible all -i localhost, -m debug -a "msg='my key is {{ key }}'" -e "key=value"
```

## yaml格式

```yaml
# cat a.yml
---
name: qfedu
type: school
```

```shell
 ansible all -i localhost, -m debug -a "msg='name is {{ name }}, type is {{ type }}'" -e @a.yml
```

## json格式

```json
# cat a.json
{"name":"qfedu","type":"school"}

ansible all -i localhost, -m debug -a "msg='name is {{ name }}, type is {{ type }}'" -e @a.json

```

后面两种格式都要使用`@`符号连接文件





