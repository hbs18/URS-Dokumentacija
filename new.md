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
