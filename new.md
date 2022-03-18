# Virt-manager, ovlasti

#### 18.03.2022.

Preuzeli smo arch linux qcow2 cloud sliku (https://gitlab.archlinux.org/archlinux/arch-boxes/-/jobs/50645/artifacts/file/output/Arch-Linux-x86_64-cloudimg-20220318.50645.qcow2). Generirali smo user-data sa lozinkom.

```
#cloud-config
users:
  - name: fidit
    passwd: asdf1234
    chpasswd: { expire: False }
```

Sa xorrisom generiramo cloud-init.iso file. 

U virt manageru kreiramo novi VM sa tipom import existing disk image (skroz dolje opcija na prvom ekranu u create VM wizardu).

U virt manageru dodajemo novi storage tipa CDROM i odaberemo generirani iso file. 

Imali smo problem sa ulogiranjem lozinkom, pa smo instalirali alat `mkpasswd`.

Hashiranu zaporku smo napravili:

```shell
korisnik@ODJ-O365-118:~/archNovi$ mkpasswd --method=SHA-512 --rounds=4096
Password: 
$6$rounds=4096$Mv/9pkk4GT$M0OKnHD9x5pOxUSDt1Besdl36oxr2wwcwBuvb2ahaw7jMJA9SI2RLT.e.MCS2393SSsp7AsMm4PnwEdVM70Dj.
```

U `Password: ` prompt utipkali smo lozinku koju hoćemo da VM ima pri log inu. Ovdje je `Mv/9pkk4GT` sol, između dva $ znaka.


Običnu lozinku u `user-data` zamjenili smo hashiranom lozinkom:

```
#cloud-config
users:
  - name: fidit
    passwd: $6$rounds=4096$Mv9pkk4GT$M0OKnHD9x5pOxUSDt1Besdl36oxr2wwcwBuvb2ahaw7jMJA9SI2RLT.e.MCS2393SSsp7AsMm4PnwEdVM70Dj
```

Obrisali smo virtualnu mašinu i ponovili postupak uspostave VMa.

Ni to nije radilo pa smo postavili user-data na:


```
#cloud-config
users:
  - default
system_info:
  default_user:
    name: fidit
    plain_text_passwd: 'asdf1234'
    gecos: arch Cloud User
    groups: [wheel, adm]
    sudo: ["ALL=(ALL) NOPASSWD:ALL"]
    shell: /bin/bash

```

`sudo: ["ALL=(ALL) NOPASSWD:ALL"]` omogućuje da korisnik postane korijenski korisnik bez lozinke (?)

Ni to nije radilo, trebalo je dodat liniju;

```
#cloud-config
users:
  - default
system_info:
  default_user:
    name: fidit
    lock_passwd: False
    plain_text_passwd: 'asdf1234'
    gecos: arch Cloud User
    groups: [wheel, adm]
    sudo: ["ALL=(ALL) NOPASSWD:ALL"]
    shell: /bin/bash
```

Kad radimo lokalni login, `lock_passwd: False` je potreban. Sada radi login.

Kada je cloud-init uspješan i gotov sa svojim radom, dobit ćemo ovakav output (nakon par minuta):

<img src="Slika zaslona s 2022-03-18 14-38-46.png"></img>

Tada možemo login napravit (lozinka će proraditi).

U `ps aux` kada imamo user `systemd+`, + znaci da cjelo ime usera nije stalo.

## Userdbctl

Popis korisnika.

Postoje korisnici za dbus, http, ftp, mail... kako bi se moglo svakome od njih dati što manje ovlasti, to jest da ne bi zloupotrebljavali svoje ovlasti.

## Prava

```shell
[fidit@archlinux ~]$ sudo chown http datoteka
```

Datoteka `datoteka` je dobila vlasnika korisnika `http`.

Folderu ftp i fileovima unutar njega od group i o mičemo prava read i execute:

```shell
[fidit@archlinux ~]$ sudo chmod -R go-rx ftp/
```

## Grupe

Pristup grafičkoj kartici imaju user `root` i grupa `video`:

```shell
[fidit@archlinux ~]$ ls -la /dev/dri/card0
crw-rw----+ 1 root video 226, 0 Mar 18 13:58 /dev/dri/card0
```

Ako dodamo korisnika `fidit` u grupu `video`, on će imati pristup grafičkoj kartici:

```shell
[fidit@archlinux ~]$ sudo usermod -aG video fidit
[fidit@archlinux ~]$ grep video /etc/group
video:x:985:fidit
```

Dodatno: pristup disku i pristup datotečnom sustavu nije isto. Pristup datot. sustavu imamo stalno, a za pristup disku moramo se dodati u grupu `disk`.

Naredbom `useradd` dodajemo korisnika:

```shell
[fidit@archlinux ~]$ sudo useradd korisnik -c "Obicni korisnik" -s /bin/bash -m
```

Lozinku dodajemo alatom `passwd`:

```shell
[fidit@archlinux ~]$ sudo passwd korisnik
New password: 
Retype new password: 
passwd: password updated successfully
```

Korisnik je sada dodan:

```shell
[fidit@archlinux ~]$ sudo grep korisnik /etc/shadow
korisnik:$6$Lx25pABYyUX8pY1s$hyduQkAuWV0nSWxtTrhlpLGRXprblCVswg79RXb98TFj.WKIkKyHYhbm2UNfvuxJcy3jp6Ihe6I4cJWY8iVJj.:19069:0:99999:7:::
```

Naredbom `usermod` uređujemo postoječeg korisnika. 

```shell
[fidit@archlinux ~]$ sudo usermod -s /bin/false korisnik
```

Postavkom ljuske na `/bin/false` za nekog korisnika onemogučujemo mu pristup shellu.

## Red Hat Ansible

Alat za automatizaciju infrastrukture. 

Instalacija (na archu):

```shell
[fidit@archlinux ~]$ sudo pacman -S ansible ansible-lint
resolving dependencies...
looking for conflicting packages...
(...)
```

U VS Codeu pišemo Ansible skripte. Instaliramo ekstenziju Remote Development. 

Na Ubuntu hostu, u VS Codeu se spajamo na Arch VM 1, na kojem je pokrenut Ansible i pomoću Ansiblea na VM 1 upravljamo VMom 2.

Podižemo još jedan Arch VM (na Ubuntu hostu, u Virt-manageru). Cloud init .iso file možemo kopirati, i on se ne mjenja (read-only je). qcow2 fileovi su read write, znaći mjenjaju se.

U VS Codeu kliknemu na zeleni remote gumb u donjem ljevom kutu i spojimo se na ssh od VM 1 (npr. `fidit@192.168.122.157`).

U novom VS Codeu (gdje u remote kutu piše adresa na koju smo spojeni) koji se otvorio napravimo novi file i odaberemo tip YAML. Nazovemo ga `playbook.yml`.

Sadržaj datoteke playbook.yml:

```

```

Instaliramo nano:
```shell
[fidit@archlinux ~]$ sudo pacman -S nano
```

Kako bi ansible vidio ostale hostove, kreiramo datoteku:

```shell
[fidit@archlinux ~]$ sudo touch /etc/ansible/hosts
```

U datoteku ubacimo:

```
[ja]
192.168.122.152

[ostali]
192.168.122.227
```

```
Važno: ansible podržava samo ssh preko ključa, ne preko lozinki!
```

Na VM1 generiramo ssh key:

```shell
[fidit@archlinux ~]$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/fidit/.ssh/id_rsa): 
(...)
```

Nakon što se izgeneriralo, ovo:

```shell
[fidit@archlinux ~]$ cat .ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC9mHUts+D5nXdvR7P1kXiEr4O4q4c66vFpGd3Lr9gsy9tZzKczq8sz3eKOdd/s/MKDmUE7ZLAVoA2s5menNXhtryfgnMtWn1rlqlC6oCFTqsSiC425rrMMdNnY8ghx7xH8+NlyZOFS/qRZpqdC7MC4N4nl/yvS4NFEiVCgS7ffbLcCdsDIXnni/eiVmVHP1GZkmxMOhvDHNrjt0ud0/B/EdIpmiSw01yb1w3uEjYpVKXAXEA5LBTXZidpkuaxdodCTSZIXN7kM1+SJQC8G7YwLS2eA04mGsOzHl3aMIWuL7Nd/czwbjRpWyOl/O59edBBhxvXPiryP28oIMHQE5nY0CzxllXoacUl+RHeeor4M0noeE5Qbd4GJIcizMwO7DBaNtxD8R0ewPXgIZrbe/O1diJzbmZJBl5/1dOL/9PWhzNNSjkZGj66WTvhdVS6KG/+GhIyo59zoVz2ChT6caxZFiGrar9DPLwgUDwkslg61AEnNGKBBHRkgPdmL5bCvkas= fidit@archlinux
```

zaljepimo u `.ssh/authorized_keys` na oba VMa.

Sada će ansible ping raditi:

```shell
[fidit@archlinux ~]$ ansible all -m ping
The authenticity of host '192.168.122.227 (192.168.122.227)' can't be established.
ED25519 key fingerprint is SHA256:7QjN4cVwduJUV06OI6mbbsnrjH7/Bun5FPXvImvXB94.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? [WARNING]: Platform linux on host 192.168.122.152 is using the discovered Python interpreter at
/usr/bin/python3.10, but future installation of another Python interpreter could change the meaning of
that path. See https://docs.ansible.com/ansible-core/2.12/reference_appendices/interpreter_discovery.html
for more information.
192.168.122.152 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false,
    "ping": "pong"
}
yes
[WARNING]: Platform linux on host 192.168.122.227 is using the discovered Python interpreter at
/usr/bin/python3.10, but future installation of another Python interpreter could change the meaning of
that path. See https://docs.ansible.com/ansible-core/2.12/reference_appendices/interpreter_discovery.html
for more information.
192.168.122.227 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false,
    "ping": "pong"
}
```

Važno: treba dva put mu dat `yes`.






