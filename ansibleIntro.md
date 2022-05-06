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

Ansible dokumentacija: https://docs.ansible.com/, https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html

Napravili smo ansible playbook gdje instaliramo apache i osiguramo da servis radi:

```shell
[root@archlinux ~]# touch ansible-playbook.yml
[root@archlinux ~]# nano ansible-playbook.yml 
```

```shell
---
- name: Update web servers
  hosts: lokalna
  become: yes
  remote_user: root

  tasks:
  - name: Ensure apache is at the latest version
    ansible.builtin.pacman:
      name: apache
      state: latest
  - name: Ensure apache is started
    ansible.builtin.service:
      name: httpd
      state: started
```

Pokrenuli smo playbook:

```shell
[root@archlinux ~]# ansible-playbook ansible-playbook.yml 
 ___________________________ 
< PLAY [Update web servers] >
 --------------------------- 
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

 ________________________ 
< TASK [Gathering Facts] >
 ------------------------ 
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

[WARNING]: Platform linux on host localhost is using the discovered Python
interpreter at /usr/bin/python3.10, but future installation of another Python
interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-
core/2.12/reference_appendices/interpreter_discovery.html for more information.
ok: [localhost]
 _______________________________________________ 
< TASK [Ensure apache is at the latest version] >
 ----------------------------------------------- 
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

changed: [localhost]
 _________________________________ 
< TASK [Ensure apache is started] >
 --------------------------------- 
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

changed: [localhost]
 ____________ 
< PLAY RECAP >
 ------------ 
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

localhost                  : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

*Važno:* Ako ne pokrećemo playbook kao root, moramo dodati u playbook `become: yes`.

Možemo se uvjeriti da apache radi:

```shell
[root@archlinux ~]# ps aux | grep httpd
root         737  0.0  0.6   8588  6392 ?        Ss   14:24   0:00 /usr/bin/httpd -k start -DFOREGROUND
http         742  0.0  0.4 754168  4028 ?        Sl   14:24   0:00 /usr/bin/httpd -k start -DFOREGROUND
http         743  0.0  0.4 754168  4028 ?        Sl   14:24   0:00 /usr/bin/httpd -k start -DFOREGROUND
http         744  0.0  0.4 754168  4028 ?        Sl   14:24   0:00 /usr/bin/httpd -k start -DFOREGROUND
root         833  0.0  0.2   6672  2680 pts/0    S+   14:28   0:00 grep httpd
```



