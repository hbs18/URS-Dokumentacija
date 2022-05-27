```
ansible-vault encrypt_string --vault-id vault_pass.txt 'f1d1t-s3cr3t' --name zaporka
```

vault_pass.txt je master password, on je u plaintextu.

'f1d1t-s3cr3t' je ono sto se kriptira, --name zaporka je pod kojom imenom varijable se kriptira

```yaml
- name: Update web servers
  hosts: lokalna
  become: yes
  vars:
    ime_korisnika: fidit
    ime_baze: fidit
    #zaporka: f1d1t-s3cr3t
    zaporka: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          35383463633634666432333538616663396163626235353265653434373736396635613337313933
          3134303964623363643666306265623830613632363365380a383935653463313339373961393938
          61656366363936646638383737336337363437363232636131616132323666653832356537363663
          6338333239373537350a336666363230613137366566353664333930313330333065393461363131
          6232
 (...)
```

U zaporka se stavlja kriptirana zaporka napraviljena komandom iznad.

izvodimo playbook sa lozinkom: `ansible-playbook --vault-password-file vault_pass.txt playbook2.yml `

