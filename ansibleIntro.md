### Ansible intro

Instaliramo ansible:

```shell
[root@archlinux ~]# sudo pacman -S ansible
```

Da radimo sa jednom (lokalnom mašinom):

```shell
[root@archlinux ~]# cat /etc/ansible/hosts 
[lokalna]
localhost ansible_connection=local
```

Testiramo ansible:

```shell
[root@archlinux ~]# ansible all -m ping
[WARNING]: Platform linux on host localhost is using the discovered Python
interpreter at /usr/bin/python3.10, but future installation of another Python
interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-
core/2.12/reference_appendices/interpreter_discovery.html for more information.
localhost | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false,
    "ping": "pong"
}
```

Ansible dokumentacija: https://docs.ansible.com/

