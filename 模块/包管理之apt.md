## 安装

```
# 安装软件
- name: Install apache httpd  (state=present is optional)
  apt:
    name: apache2
    state: present
# 更新软件
- name: Update repositories cache and install "foo" package
  apt:
    name: foo
    update_cache: yes
# 删除ruan0j
- name: Remove "foo" package
  apt:
    name: foo
    state: absent

# 安装列表
- name: Install a list of packages
  apt:
    pkg:
    - foo
    - foo-tools
# 指定版本
- name: Install the version '1.00' of package "foo"
  apt:
    name: foo=1.00

- name: Update the repository cache and update package "nginx" to latest version using default release squeeze-backport
  apt:
    name: nginx
    state: latest
    default_release: squeeze-backports
    update_cache: yes

- name: Install latest version of "openjdk-6-jdk" ignoring "install-recommends"
  apt:
    name: openjdk-6-jdk
    state: latest
    install_recommends: no

- name: Update all packages to their latest version
  apt:
    name: "*"
    state: latest

- name: Upgrade the OS (apt-get dist-upgrade)
  apt:
    upgrade: dist

- name: Run the equivalent of "apt-get update" as a separate step
  apt:
    update_cache: yes

- name: Only run "update_cache=yes" if the last one is more than 3600 seconds ago
  apt:
    update_cache: yes
    cache_valid_time: 3600

- name: Pass options to dpkg on run
  apt:
    upgrade: dist
    update_cache: yes
    dpkg_options: 'force-confold,force-confdef'

- name: Install a .deb package
  apt:
    deb: /tmp/mypackage.deb

- name: Install the build dependencies for package "foo"
  apt:
    pkg: foo
    state: build-dep

- name: Install a .deb package from the internet
  apt:
    deb: https://example.com/python-ppq_0.1-1_all.deb

- name: Remove useless packages from the cache
  apt:
    autoclean: yes

- name: Remove dependencies that are no longer required
  apt:
    autoremove: yes
```





## 如何进行过滤

由于系统有ubuntu系统与centos系统，所以我们可以通过when去进行过滤

```
- name: Install apache Ubuntu
  apt:
    name: apache2
    state: present
   when: ansible_facts['distribution']=='Ubuntu'
 - name: Install apache httpd   CentOS
  yum:
    name: apache2
    state: present
   when: ansible_facts['distribution']=='CentOS'
```

