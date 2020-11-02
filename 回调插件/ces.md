+ ⽤Python编写 引发错误，就是遇到问题后，主动抛出异常 

+ 返回以unicode编码的字符串，主要是兼容 Jinja2

  ```python
  from ansible.module_utils._text import to_text
  result_string = to_text(result_string)
  ```

  

+ 符合Ansible的配置和⽂档标准，就是可以通过 ansible.cfg 进⾏配置
  Ansible的在线帮助⽂档是根据每个模块的源代码中的
DOCUMENTATION模块⽣成的。该DOCUMENTATION块必须是有效的
YAML。

DOCUMENTATION模块说明：

```yaml
DOCUMENTATION = '''
  callback: log_plays
  type: notification
  short_description: write playbook output to log file
  version_added: historical
  description:
    - 此插件的详细描述信息。
    - 使⽤多条⽬，不要使⽤⼀个较⻓的语句。
    - 不应该提及模块名称。
 requirements:
    - 必须要求清单
    - 包括最低版本的限制
 # 下面的option是这个插件需要进行的配置选项，
 # env是指环境变量中需要定义的
 # ini这个是ansible.cfg中进行配置的选项section,指的是配置文件中的部分
 # key指的是其中的配置，比如下面这个例子
 # ini:
 #   - section: callback_log_plays
 #     key: log_folder
 # 在配置文件中就是这么表示
 # [callback_log_plays]
 # log_folder=/tmp/ansible/hosts
 # 其中section就是 [callback_log_plays]
 # key就是 这个log_folder关键字
 options:
   log_folder:
     version_added: '2.9' 此插件添加到 Ansible 时候的当时 Ansible 的版本。
     default: 选项的默认值，如果 required 是 False,则 default 可以设置
     description: 此选项的作⽤的详细说明。应该⽤完整的句⼦写成。
     env:
       - name: 环境变量的名字
     ini:
       - section: 在 asible.cfg 中的配置块名称key: log_folder在对应配置块下⾯的变量名称
         key: name
     required: True/False 必需时为 True,如果不设置，就认为不是必须的。
     type: int/str/list 不是必须的
```

要访问插件中的配置设置，请使 ⽤`self.get_option("log_folder")`。 如果需要显式个配置选项设置值，请使⽤`self.set_options()`

代码编写：

```python
from __future__ import (absolute_import, division, print_function)

__metaclass__ = type
DOCUMENTATION = '''
  callback: mysql_plays
  type: notification
  short_description: 将 playbook 的执⾏结果输出到MySQL 中。
  version_added: historical
  description:
    - 这个回调插件将会把输出存⼊ MySQL 服务器中。
  requirements:
    - 需要配置到 ansible.cfg 中 Whitelist
    - 可以被访问的 MySQL 服务器实例
    - Python 版本对应的 pymysql 或者 mysqlclient 模块
  options:
    mysql_host:
    version_added: '2.9'
    default: locallhost
    description: MySQL 服务器 IP或者主机名.
      env:
        - name: ANSIBLE_MYSQL_HOST
      ini:
        - section: callback_mysql_plays
          key: mysql_host
  mysql_port:
    version_added: '2.9'
    default: 3306
    description: MySQL 服务器监听端⼝.
      env:
        - name: ANSIBLE_MYSQL_PORT
      ini:
        - section: callback_mysql_plays
          key: mysql_port
      type: int
  mysql_user:
    version_added: '2.9'
    default: ansible
    description: MySQL 服务器登录⽤户.
    env:
      - name: ANSIBLE_MYSQL_USER
    ini:
      - section: callback_mysql_plays
        key: mysql_user
  mysql_password:
    version_added: '2.9'
    default: 'QFedu123!'
    description: MySQL 服务器登录⽤户.
    env:
      - name: ANSIBLE_MYSQL_PASSWORD
    ini:
      - section: callback_mysql_plays
        key: mysql_password
  mysql_db:
    version_added: '2.9'
    default: ansible
    description: 存放数据的库名称.
    env:
      - name: ANSIBLE_MYSQL_DB
    ini:
      - section: callback_mysql_plays
        key: db
  mysql_table:
    version_added: '2.9'
    default: playsresult
    description: 存放数据的表名称.
    env:
      - name: ANSIBLE_MYSQL_TABLE
    ini:
      - section: callback_mysql_plays
        key: mysql_table
'''

import json
import getpass
from ansible.module_utils.common._collections_compat
import MutableMapping
from ansible.parsing.ajson import AnsibleJSONEncoder
from ansible.plugins.callback import CallbackBase
from ansible.errors import AnsibleError
from ansible.module_utils._text import to_native

try:
    import pymysql as mysqldb

    pwd = "password"
    database = "db"
except ImportError:
    try:
        import MySQLdb as mysqldb

        pwd = "passwd"
        database = "database"
    except ImportError:
        raise AnsibleError("找不到 pymysql 或mysqlclient 模块。")


class CallbackModule(CallbackBase):
    """
   把 playbook 的结果保存到 MySQL 数据库中，默认的库.表是 ansible.playsresult
   """
    CALLBACK_VERSION = 2.0
    CALLBACK_TYPE = 'notification'
    CALLBACK_NAME = 'mysql_plays'
    CALLBACK_NEEDS_WHITELIST = True
    TIME_FORMAT = "%b %d %Y %H:%M:%S"
    MSG_FORMAT = "%(now)s - %(category)s - %(data)s\n\n"
    def __init__(self):
        super(CallbackModule, self).__init__()
    def set_options(self, task_keys=None,
                    var_options=None, direct=None):
        """
            ⽤于设置选项和获取选项， 选项包含了⾃定义的选项
            """
        super(CallbackModule,self).set_options(task_keys=task_keys,
                                               var_options=var_options, direct=direct)
        self.mysql_host =self.get_option("mysql_host")
        self.mysql_port =self.get_option("mysql_port")
        self.mysql_user=self.get_option("mysql_user")
        self.mysql_password =self.get_option("mysql_password")
        self.mysql_db =self.get_option("mysql_db")
        self.mysql_table =self.get_option("mysql_table")
        self.user = getpass.getuser()
    def _mysql(self):
        """
            连接数据库，返回数据库对象和游标对象
            """
        db_conn={"host": self.mysql_host,
                 "port": self.mysql_port,
                 "user": self.mysql_user,
                 pwd: self.mysql_password,
                 database: self.mysql_db}
        try:
            db = mysqldb.connect(**db_conn)
        except Exception as e:
            raise AnsibleError("%s" %to_native(e))
            cursor= db.cursor()
            return db, cursor
    def _execute_sql(self, host, category, data):
        if isinstance(data, MutableMapping):
            if '_ansible_verbose_override' in data:
                # avoid save extraneous data
                data = 'omitted'
            else:
                data = data.copy()
                invocation =data.pop('invocation', None)
                data = json.dumps(data,cls=AnsibleJSONEncoder)
                if invocation is not None:
                    data = json.dumps(invocation) + " => %s " % data
            sql = """
                insert into {}
                    (host,user,category,result)
                    values(%s,%s,%s,%s)
                    """.format(self.mysql_table)
            db, cursor = self._mysql()

            try:
            # 执⾏ sql，记录事件类型和事件结果
                cursor.execute(sql, (host, self.user,
                                     category, data))
                db.commit()
            except Exception as e:
                raise AnsibleError("%s" %
                               to_native(e))
            finally:
                cursor.close()
                db.close()

        def runner_on_failed(self, host, res,
                             ignore_errors=False):
            self._execute_sql(host, 'FAILED', res)

        def runner_on_ok(self, host, res):
            self._execute_sql(host, 'OK', res)

        def runner_on_skipped(self, host, item=None):
            self._execute_sql(host, 'SKIPPED', '...')

        def runner_on_unreachable(self, host, res):
            self._execute_sql(host, 'UNREACHABLE',
                              res)

            def runner_on_async_failed(self, host, res,
                                       jid):
                self._execute_sql(host, 'ASYNC_FAILED',
                                  res)

            def playbook_on_import_for_host(self, host,
                                            imported_file):
                self._execute_sql(host, 'IMPORTED',
                                  imported_file)

            def playbook_on_not_import_for_host(self,
                                                host, missing_file):
                self._execute_sql(host, 'NOTIMPORTED',
                                  missing_file)
```

保存插件到有效的⽬录下

把插件保存为 `mysql_plays.py` ⽂件，并存到ansible 控制节点的 如下⽬录下: `~/.ansible/plugins/callback/`

开启使⽤插件

在 `ansible.cfg` 中编辑如下配置

```ini
callback_whitelist = mysql_plays
```

默认此插件仅对 playbook ⽣效，假如希望在 ad-hoc (快捷命令) 中⽣效，继续打开如下配置，并设置 True

```ini
bin_ansible_callbacks = True
```

配置插件使⽤的选项

此插件已经有默认值，如果想修改需在 ansible.cfg ⽂件的最后添 加如下配置

```init
[callback_mysql_plays]
mysql_host = MySQL IP
mysql_port = MySQL 监听端⼝
mysql_user = MySQL ⽤户
mysql_password = MySQL 密码
mysql_db = MySQL 库名
mysql_table = MySQL 表名
```

