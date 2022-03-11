# Arch linux i cloud init

#### 11.03.2022

Preuzmi Arch linux qcow2 image sa stranice https://gitlab.archlinux.org/archlinux/arch-boxes/-/jobs/50048/artifacts/file/output/Arch-Linux-x86_64-cloudimg-20220310.50048.qcow2 i napravi kopiju.

```shell
korisnik@ODJ-O365-118:~/Preuzimanja$ qemu-system-x86_64 -m 512M mojArch.qcow2
```

Ovime se boota VM.

Podsjetnik: image se resizea sa

```shell
korisnik@ODJ-O365-118:~/Preuzimanja$ qemu-img resize mojArch.qcow2 +10G
```

Kreiraj meta-data i user-data fileove u diru gdje je qcow2 slika. U user-data:

```
#cloud-config
users:
  - name: fidit
    ssh_authorized_keys:
      - ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBLsUoyamyIUdMhZ9iwuDfCpzoMOX6FlPXJhV>
```

`meta-data` ostaviti prazno. 

SSH ključevi generiraju se 

```shell
korisnik@ODJ-O365-118:~$ ssh-keygen -t ecdsa
```

U user-data ide linija koja se dobiva sa:

```shell
korisnik@ODJ-O365-118:~$ cat .ssh/id_ecdsa.pub 
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTIt(...)
```

Generiramo sliku sa podacima naredbom:

```shell
korisnik@ODJ-O365-118:~/Preuzimanja$ xorriso -as genisoimage -output cloud-init.iso -volid CIDATA -joliet -rock user-data meta-data
xorriso 1.5.2 : RockRidge filesystem manipulator, libburnia project.

Drive current: -outdev 'stdio:cloud-init.iso'
Media current: stdio file, overwriteable
Media status : is blank
Media summary: 0 sessions, 0 data blocks, 0 data,  319g free
Added to ISO image: file '/user-data'='/home/korisnik/Preuzimanja/user-data'
xorriso : UPDATE :       1 files added in 1 seconds
Added to ISO image: file '/meta-data'='/home/korisnik/Preuzimanja/meta-data'
xorriso : UPDATE :       2 files added in 1 seconds
ISO image produced: 184 sectors
Written to medium : 184 sectors at LBA 0
Writing to 'stdio:cloud-init.iso' completed successfully.
```

Pokrećemo VM sa networkingom i cdromom naredbom:

```shell
korisnik@ODJ-O365-118:~/Preuzimanja$ qemu-system-x86_64 -m 512M mojArch.qcow2 -cdrom cloud-init.iso -nic user,hostfwd=tcp::60022-:22
```

Treba pričekat da VM pokaže svoje SSH host keyeve (to može potrajat par minuta). Tek onda će se moć spojit na mašinu pomoću ssh.

Spajamo se pomoću SSH:

```shell
korisnik@ODJ-O365-118:~/Preuzimanja$ ssh -p 60022 fidit@localhost
```

Ako javi error:

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:DQ2Z1m/BLmgrOdmlHgM/z7OdGoejL9QzuGEe19h2b1Y.
Please contact your system administrator.
Add correct host key in /home/korisnik/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/korisnik/.ssh/known_hosts:2
  remove with:
  ssh-keygen -f "/home/korisnik/.ssh/known_hosts" -R "[localhost]:60022"
ECDSA host key for [localhost]:60022 has changed and you have requested strict checking.
Host key verification failed.
```

znaći da se mašina na istom portu promjenila od zadnji put kad smo se spajali na tu adresu i port i ssh misli da se mašina zamjenila, te da se izvodi potencijalni MITM napad.

U mom slučaju, to se dogodilo jer sam prije imao VM (kojeg sam izbrisao i pokrenuo novi) na istoj adresi i portu.

To možemo popravit tako da samo izvedemo naredbu koja piše u error poruci.
