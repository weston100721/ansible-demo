# ansible-demo
ansible learning

## 1. 安装

```shell
sudo pip install ansible

sudo yum install ansible
```

测试
```shell
ansible all -m ping
ansible host -i hosts -m ping
```

配置文件`ansible.cfg`的读取顺序：

1. ANSIBLE_CONFIG环境变量所指的位置
2. 当前目录的`ansible.cfg`
3. `~/.ansible.cfg`(用户目录下的`ansible.cfg`)
4. `/etc/ansible/ansible.cfg`

**建议：** 将`ansible.cfg`与playbooks一起放在当前目录。

# 2. 模块
1. apt
2. copy
3. file
4. service
5. template

可以通过`ansible-doc file`来了解file模块，其他类似。


# 3. inventory

名称|默认值|描述
:---:|:---:|:---:
ansible_ssh_host|主机名字|SSH目的的主机名或IP
ansible_ssh_port|22|SSH目的端口
ansible_ssh_user|root|
ansible_ssh_pass|none|SSH认证所使用的密码
ansible_connection|smart|
ansible_ssh_private_key_file|none|
ansible_shell_type|sh|
ansible_python_interpreter|/usr/bin/python|python的解释器
ansible\_\*\_interpreter|none|类似python的解释器一样，根据其语言而定

# 4. 变量

## 4.1 查看变量
```shell
ansible all -m setup
ansible all -m setup -a 'filter=ansible_eth*'
- debug: var=myvarname
```

## 4.2 变量注册

```
- name: capture output of whoami command
  command: whoami
  register: login
  ignore_errors: True  # 出错就会终止，但是有了这个配置就可以解决这个问题。
- debug: var="user is {{ login.stdout }}"
```

## 4.3 访问变量中字典的key

```
ansible_eth1['ipv4']['address']
ansible_eth1['ipv4'].address
ansible_eth1.ipv4['address']
ansible_eth1.ipv4.address
```

## 4.4 fact
### 4.4.1 查看fact子集
```
ansible all -m setup -a 'filter=ansible_eth*'
```

### 4.4.2 本地fact
将多个文件放置在`/etc/ansible/facts.d/`目录下，支持一下几种格式：

- .ini格式
- JSON格式

以这种方式加载的fact是key为ansible_local的特殊变量。
