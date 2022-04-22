### 22.4.2022

```shell
[fidit@archlinux ~]$ systemd-analyze
Startup finished in 1.491s (kernel) + 1min 5.947s (userspace) = 1min 7.439s 
graphical.target reached after 1min 5.569s in userspace
```

Za detaljniji ispis vremena:
```shell
[fidit@archlinux ~]$ systemd-analyze blame
39.703s reflector-init.service
33.972s pacman-init.service
10.113s cloud-init.service
 9.491s ldconfig.service
 9.193s systemd-machine-id-commit.service
 5.021s dev-vda2.device
 3.737s cloud-config.service
 3.503s systemd-udevd.service
 3.151s systemd-tmpfiles-setup.service
 3.053s systemd-networkd-wait-online.service
 1.197s cloud-init-local.service
 1.117s systemd-journal-catalog-update.service
  867ms systemd-sysusers.service
  674ms sshdgenkeys.service
  608ms systemd-journal-flush.service
  371ms cloud-final.service
  293ms systemd-random-seed.service
  144ms systemd-networkd.service
  (...)
```

Možemo viditi o čemu ovisi koji servis:
```shell
[fidit@archlinux ~]$ cat /etc/systemd/system/reflector-init.service 
[Unit]
Description=Initializes mirrors for the VM
After=network-online.target
Wants=network-online.target
Before=sshd.service cloud-final.service
ConditionFirstBoot=yes

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=reflector --latest 20 --protocol https --sort rate --save /etc/pacman.d/mirrorlist

[Install]
WantedBy=multi-user.target
[fidit@archlinux ~]$ cat /etc/systemd/system/pacman-init.service 
[Unit]
Description=Initializes Pacman keyring
Before=sshd.service cloud-final.service
ConditionFirstBoot=yes

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/pacman-key --init
ExecStart=/usr/bin/pacman-key --populate archlinux

[Install]
WantedBy=multi-user.target
[fidit@archlinux ~]$ cat /etc/systemd/system/cloud-final.service 
cat: /etc/systemd/system/cloud-final.service: No such file or directory
[fidit@archlinux ~]$ cat /usr/lib/systemd/system/cloud-final.service 
[Unit]
Description=Execute cloud user/final scripts
After=network-online.target cloud-config.service rc-local.service
Wants=network-online.target cloud-config.service


[Service]
Type=oneshot
ExecStart=/usr/bin/cloud-init modules --mode=final
RemainAfterExit=yes
TimeoutSec=0
KillMode=process
TasksMax=infinity

# Output needs to appear in instance console output
StandardOutput=journal+console

[Install]
WantedBy=cloud-init.target
```

Neki su u `/usr/lib/`, neki u `/etc/`

`ConditionFirstBoot=true|false` je bitno, primjerice cloud init to moze imat i moguce je da pokreće se svaki put kad se sistem boota. Ako smo promjenili neke postavke cloud init moze ih tako vratit nazad na kako su bile pa to mozemo iskljucit da to ne radi. 

Možemo viditi što `.target` fileovi pokreću:

```shell
[fidit@archlinux ~]$ systemctl list-dependencies cloud-init.target
cloud-init.target
● ├─cloud-config.service
● ├─cloud-final.service
● ├─cloud-init-local.service
● └─cloud-init.service
```

Mogu se pokretati i `init` i `final` servisi istovremeno jer `final` zna čekati za `init` bude gotov.

Za grafičku analizu procesa:

```shell
[fidit@archlinux ~]$ systemd-analyze dot
digraph systemd {
	"systemd-tmpfiles-setup-dev.service"->"systemd-journald.socket" [color="green"];
	"systemd-tmpfiles-setup-dev.service"->"system.slice" [color="green"];
	"systemd-tmpfiles-setup-dev.service"->"kmod-static-nodes.service" [color="green"];
(...)
```

Isto možemo napraviti:
```shell
[fidit@archlinux ~]$ systemd-analyze plot > time.svg
```

Dohvaćanje fileova sa VMa pomoću SFTP:

```shell
korisnik@ODJ-O365-118:~$ sftp fidit@192.168.122.46
Connected to 192.168.122.46.
sftp> get time.svg
Fetching /home/fidit/time.svg to time.svg
/home/fidit/time.svg                          100%  124KB  56.8MB/s   00:00    
sftp> 
```