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

