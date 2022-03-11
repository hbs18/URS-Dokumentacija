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
Dodatak: naredba `stat` služi za ispis informacijskog čvora datoteke
```
