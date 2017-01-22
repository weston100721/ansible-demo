# ansible-demo
ansible learning

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
