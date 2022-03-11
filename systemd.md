# systemd

#### 11.03.2022

Alati systemd:

```shell
[root@archlinux ~]# ls -lt /usr/bin/*ctl
-rwxr-xr-x 1 root root  30960 Feb 14 11:59 /usr/bin/wdctl
-rwxr-xr-x 1 root root  51568 Feb 14 11:59 /usr/bin/zramctl
-rwxr-xr-x 1 root root  84208 Feb 10 07:00 /usr/bin/bootctl
-rwxr-xr-x 1 root root  92384 Feb 10 07:00 /usr/bin/busctl
-rwxr-xr-x 1 root root  55544 Feb 10 07:00 /usr/bin/coredumpctl
-rwxr-xr-x 1 root root 145776 Feb 10 07:00 /usr/bin/homectl
-rwxr-xr-x 1 root root  30824 Feb 10 07:00 /usr/bin/hostnamectl
-rwxr-xr-x 1 root root  79992 Feb 10 07:00 /usr/bin/journalctl
-rwxr-xr-x 1 root root  26728 Feb 10 07:00 /usr/bin/localectl
-rwxr-xr-x 1 root root  59608 Feb 10 07:00 /usr/bin/loginctl
-rwxr-xr-x 1 root root 100592 Feb 10 07:00 /usr/bin/machinectl
-rwxr-xr-x 1 root root 112752 Feb 10 07:00 /usr/bin/networkctl
-rwxr-xr-x 1 root root  18528 Feb 10 07:00 /usr/bin/oomctl
-rwxr-xr-x 1 root root  51416 Feb 10 07:00 /usr/bin/portablectl
-rwxr-xr-x 1 root root 137472 Feb 10 07:00 /usr/bin/resolvectl
-rwxr-xr-x 1 root root 272768 Feb 10 07:00 /usr/bin/systemctl
-rwxr-xr-x 1 root root  43112 Feb 10 07:00 /usr/bin/timedatectl
-rwxr-xr-x 1 root root  43224 Feb 10 07:00 /usr/bin/userdbctl
-rwxr-xr-x 1 root root  43152 Jan 29 22:02 /usr/bin/auditctl
-rwxr-xr-x 1 root root  30776 Feb 13  2021 /usr/bin/sysctl
-rwxr-xr-x 1 root root  55320 Jul  7  2020 /usr/bin/keyctl
```

Parametar `t` sortira po datumu stvaranja. 

### bootctl

```shell
[root@archlinux ~]# bootctl
Couldn't find EFI system partition. It is recommended to mount it to /boot or /efi.
Alternatively, use --esp-path= to specify path to mount point.
System:
    Not booted with EFI
```
Bootctl je systemdov bootlaoder. Arch ga ne koristi, nego koristi svoj.

### busctl

systemdov kontroler sabirnica. U ovom smislu, sabirnica služi kako bi procesi koji su pokrenuti lokalno mogu komunicirati među sobom (ne radi se o hardverskim sabirnicama). 

Naredba `busctl` pokazuje koji procesi trenutno aktivno komuniciraju međusobno.

`org.freedesktom.DBus`

```
Dodatak: naredba stat (datoteka) služi za ispis informacijskog čvora datoteke
```

`busctl` koristimo kad provjeravamo rade li sabirnice kako trebaju, komuniciraju li međusobno... Koristi se uglavnom za debugging.

### coredumpctl

Služi za pregled core dumpova. Oni se generiraju kad se programi crashaju i u njima su informacije o crashu. Umjesto da pregledavamo stranice i stranice logova, pogledamo samo core dump ovim alatom.

To radi tako da je uvijek aktivan socket u kojeg programi mogu javiti kada su crashali.

###  hostnamectl

Pita `hostnamed` za informacije kao OS, HW vendor... i daje informacije o računalu.

Ispis statusa: 

```shell
[root@archlinux ~]# hostnamectl status
 Static hostname: archlinux
       Icon name: computer-vm
         Chassis: vm 🖴
(...)
```

Postavljanje vrijednosti:

```shell
[root@archlinux ~]# hostnamectl icon-name computer
```

Dohvaćanje vrijednost:

```shell
[root@archlinux ~]# hostnamectl icon-name
computer
```

Kada postavljamo vrijednosti kao što su `chassis`, možemo postaviti samo određene tipove. Vrstu tipa koja se može staviti možemo viditi u man pageu.

`deployment` je u kojoj je produkcijskoj okolini računalo. Po defaulti nije postavljeno. Primjerice, može biti `production`, `staging`, `development`...

`location` se može koristiti kako bi definirali fizičku lokaciju računala. 

### journalctl

Za sad ništa...

### localectl

Za upravljanje localeom na računalu.  

```shell
[root@archlinux ~]# localectl
   System Locale: LANG=en_US.UTF-8
       VC Keymap: us
      X11 Layout: n/a
```

Napomena: nano se na archu instalira `[root@archlinux ~]# pacman -S nano`

Da dodamo novi locale, u `locale.gen` dodajemo novi locale prvo.

```shell
[root@archlinux ~]# nano /etc/locale.gen
```

```
# Created by cloud-init v. 22.1 on Fri, 11 Mar 2022 14:24:47 +0000
en_US.UTF-8 UTF-8
hr_HR.UFT-8 UTF-8   <-- ovo smo dodali
```

Nakon toga pokrenemo naredbu `locale-gen` da generiramo localeove koje smo dodali.

```shell
[root@archlinux ~]# locale-gen
Generating locales...
  en_US.UTF-8... done
  hr_HR.UTF-8... done
Generation complete.
```

Novi locale postavljamo naredbom (i provjeravamo):

```shell
[root@archlinux ~]# localectl set-locale hr_HR.UTF-8
[root@archlinux ~]# localectl
   System Locale: LANG=hr_HR.UTF-8
       VC Keymap: us
      X11 Layout: n/a
```

Locale možemo također i podesiti alatom `cloud-init`.






