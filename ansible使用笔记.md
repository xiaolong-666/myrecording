# ansible使用笔记

## 测试是否能够访问远程
```bash
[~/WorkSpace/infraconf]$ ansible kvm -u devops -K -m ping                                       *[master]
BECOME password: 
[WARNING]: Platform linux on host 10.8.10.248 is using the discovered Python interpreter at
/usr/bin/python, but future installation of another Python interpreter could change this. See
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more
information.
10.8.10.248 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
[WARNING]: Platform linux on host 10.8.10.152 is using the discovered Python interpreter at
/usr/bin/python, but future installation of another Python interpreter could change this. See
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more
information.
10.8.10.152 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```
其中 kvm为host中的模块名，-u 指定用户，-K 密码验证，-m ping 指定模块



若不能访问，则需要手动添加key



## 指定yml文件中，某个task执行，跳过其它task
```bash
[~/WorkSpace/infraconf]$ ansible-playbook -l kvm -u devops site.yml                             *[master]

PLAY [Deploy ISO server] *********************************************************************************
skipping: no hosts matched

PLAY [Deploy KVM server] *********************************************************************************
...

Monday 15 March 2021  10:29:42 +0800 (0:00:00.434)       0:01:24.840 ********** 
=============================================================================== 
Install common packages (UOS) -------------------------------------------------------------------- 71.72s
ssh : Set authorized key -------------------------------------------------------------------------- 5.11s
Gathering Facts ----------------------------------------------------------------------------------- 2.62s
common : Disable system services ------------------------------------------------------------------ 1.88s
user : Ensure group exists ------------------------------------------------------------------------ 0.92s
Create device user -------------------------------------------------------------------------------- 0.67s
user : Set sudo ----------------------------------------------------------------------------------- 0.47s
common : Change repository ------------------------------------------------------------------------ 0.46s
hostname : Set host name -------------------------------------------------------------------------- 0.46s
hostname : Change hosts --------------------------------------------------------------------------- 0.43s
Install common packages (CentOS) ------------------------------------------------------------------ 0.05s
Playbook run took 0 days, 0 hours, 1 minutes, 24 seconds
```
其中 -l kvm 限制hosts为kvm的执行，-u devops 指定登录用户