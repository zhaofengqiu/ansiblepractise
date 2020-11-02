通过block进行task的异常捕获

```
---

- name: test block
  hosts: all

  tasks:
    - block:
      - name: debug before task failed
        debug:
          msg: "I execute normally"
      
      - name: failure
        command: /bin/echo
      rescue:
        - name: test rescue
          debug:
            msg: "I never execute, as above task is failing..."
      always:
        - name: always debug
          debug:
            msg: "this always executes"
      when: ansible_facts['distribution'] == "CentOS"

```

等价于

```
---

- name: test block
  hosts: all

  tasks:
    rey:
      - name: debug before task failed
        debug:
          msg: "I execute normally"
      
      - name: failure
        command: /bin/echo
      expect:
        - name: test rescue
          debug:
            msg: "I never execute, as above task is failing..."
      finally:
        - name: always debug
          debug:
            msg: "this always executes"
      when: ansible_facts['distribution'] == "CentOS"

```

