## 安装

```
sudo pip install ansible
sudo yum install ansible
```

## 测试
```shell
ansible all -m ping
ansible host -i hosts -m ping
```
执行的hosts可以通过外部变量传递
```
ansible-playbook extraHost.yml --extra-vars "@hosts.yml"
ansible-playbook extraHost.yml --extra-vars "@hosts.json"
```

下载文件

```
ansible-playbook download.yml
```


# 1. ansible.cfg

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

## 4.5 变量优先级
1. ansible-playbook -i hostsforvar getvar.yml --extra-vars "test_vars=line"
2. roles/xxx/var/main.yml
3. playbook中的vars
4. host_vars/hostname.yml中的变量
5. group_vars/中的变量
6. roles/xxx/defaults/main.yml 中的变量。

## 4.6 extra-vars的外部变量的传递语法

额外的变量设置为`键=值`、`YAML` 或 `JSON`，例如`--extra-vars "@some_file.json"`
```
--extra-vars "test_vars=line"
--extra-vars '{"pacman":"mrs","ghosts":["inky","pinky","clyde","sue"]}'
--extra-vars "@some_file.json"
--extra-vars "@some_file.yml"
```

# 5. roles
## 5.1 role的优先级

- 与playbook并列的roles目录。
- `/etc/ansible/roles`。
- ansible.cfg中指定的位置，如下所示。

```
[defaults]
roles_path = ~/ansible_roles
```

可以通过`ANSIBLE_ROLES_PATH`环境变量来覆盖这个设置。

## 5.2 新建一个role

```
ansible-galaxy init --offline roles/{{ name }}
```

## 6 测试环境搭建

安装`vagrant_1.9.1_x86_64.deb`和`virtualbox-5.1_5.1.12-112440-Ubuntu-xenial_amd64.deb`
具体安装过程可以参考[vagrantup](https://www.vagrantup.com/docs/installation/) 和[virtualbox](https://www.virtualbox.org/wiki/Documentation)官网。


```shell
vagrant init ubuntu/trusty64
vagrant up
```

Vagrantfile文件配置：

```
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  # config.vm.box = "ubuntu/trusty64"
  config.ssh.insert_key = false
  config.vm.define "vagrant1" do |vagrant1|
    vagrant1.vm.box = "ubuntu/trusty64"
    vagrant1.vm.network "forwarded_port", guest: 80, host: 8080
    vagrant1.vm.network "forwarded_port", guest: 443, host: 8443
  end
  config.vm.define "vagrant2" do |vagrant2|
    vagrant2.vm.box = "ubuntu/trusty64"
    vagrant2.vm.network "forwarded_port", guest: 80, host: 8081
    vagrant2.vm.network "forwarded_port", guest: 443, host: 8444
  end
  config.vm.define "vagrant3" do |vagrant3|
    vagrant3.vm.box = "ubuntu/trusty64"
    vagrant3.vm.network "forwarded_port", guest: 80, host: 8082
    vagrant3.vm.network "forwarded_port", guest: 443, host: 8445
  end
end
```
