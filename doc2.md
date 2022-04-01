01.04.2022.

# RAID polja i file sistemi

Dodali smo nekoliko diskova u VM, velićine 1GB.

```shell
korisnik@ODJ-O365-118:~$ ssh fidit@192.168.122.152
fidit@192.168.122.152's password: 
Last login: Fri Apr  1 12:18:14 2022
[fidit@archlinux ~]$ ls /dev/vd*
/dev/vda   /dev/vda2  /dev/vdc  /dev/vde
/dev/vda1  /dev/vdb   /dev/vdd  /dev/vdf
```

'vda' 'vdb'... su diskovi

'sudo fdisk -l' za detaljniji pregled diskova.

```shell
[fidit@archlinux ~]$ sudo fdisk -l
Disk /dev/vda: 12 GiB, 12884901888 bytes, 25165824 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 902FD2FE-154A-4635-9BF8-3FAB93BA67FC

Device     Start      End  Sectors Size Type
/dev/vda1   2048     4095     2048   1M BIOS boot
/dev/vda2   4096 25165790 25161695  12G Linux filesystem


Disk /dev/vdb: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
(...)
```

`sudo pacman -Syu` za full nadogradnju VM-a. Onda rebootamo da dobijemo novu verziju jezgre (linux 5.17) i systemd-a.

#### ZFS

Sun Microsystemsov file system i upravitelj volumena, često se koristi na Linux serverima. Nije dio jezgre Linuxa.

Za instalaciju paketa potrebno ih je samostalno kompajlirati (nalaze se na AUR - Arch User Repository). 

Hardened linux predviđa sigurnosne probleme koji bi mogli nastat i ograničava ponašanje aplikacija i OSa, Zen Linux kernel je optimiziran za igre i multimediju. 

Verziju kernela pronalazimo sa `uname -a`.

```shell
[fidit@archlinux ~]$ uname -a
Linux archlinux 5.17.1-arch1-1 #1 SMP PREEMPT Mon, 28 Mar 2022 20:55:33 +0000 x86_64 GNU/Linux
```

https://aur.archlinux.org/packages/zfs-dkms

`provjeri sto tocno je dkms` To je upravitelj dodataka za linux. 

Kliknemo na view PKGBUILD. To je skripta koju skinemo i pokrenemo.

Stvaramo datoteku 'PKGBUILD' i skriptu zaljepimo u to. 

```shell
[fidit@archlinux ~]$ makepkg
==> ERROR: Cannot find the fakeroot binary.
```

Trebamo instalirat `fakeroot`.

```shell
[fidit@archlinux ~]$ sudo pacman -S fakeroot
resolving dependencies...
looking for conflicting packages...

Packages (1) fakeroot-1.28-1

Total Download Size:   0.07 MiB
```

```shell
==> ERROR: 0001-only-build-the-module-in-dkms.conf.patch was not found in the build directory and is not a URL.
```

Fali nam taj file. U sources nađemo taj file na web stranici (iznad) i skinemo ga. Kliknemo na `plain` da dobijemo plain file. Link iskoristimo da ga skinemo:

```shell
[fidit@archlinux ~]$ curl -o 0001-only-build-the-module-in-dkms.conf.patch "https://aur.archlinux.org/cgit/aur.git/plain/0001-only-build-the-module-in-dkms.conf.patch?h=zfs-dkms"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1112  100  1112    0     0   5082      0 --:--:-- --:--:-- --:--:--  5100
```

```shell
(...)
==> Verifying source file signatures with gpg...
    zfs-2.1.4.tar.gz ... FAILED (unknown public key 6AD860EED4598027)
==> ERROR: One or more PGP signatures could not be verified!
```

Fale nam PGP signaturesi.

```shell
[fidit@archlinux ~]$ pacman-key --list-sigs | grep 6AD860EED4598027
gpg: Note: trustdb not writable
```

Možemo dodati svoje ključeve, ali moramo bit root korisnik.

```shell
[fidit@archlinux ~]$ sudo pacman-key --recv-keys 6AD860EED4598027
gpg: key 6AD860EED4598027: public key "Tony Hutter (GPG key for signing ZFS releases) <hutter2@llnl.gov>" imported
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   6  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: depth: 1  valid:   6  signed:  94  trust: 0-, 0q, 0n, 6m, 0f, 0u
gpg: depth: 2  valid:  88  signed:  32  trust: 88-, 0q, 0n, 0m, 0f, 0u
gpg: next trustdb check due at 2022-05-06
gpg: Total number processed: 1
gpg:               imported: 1
```

To jos uvjek ne radi pa je bolje sa `gpg` samo za svog korisnika dodat key i onda ce radit. 

```shell
[fidit@archlinux ~]$ gpg --recv-keys 6AD860EED4598027
gpg: /home/fidit/.gnupg/trustdb.gpg: trustdb created
gpg: key 6AD860EED4598027: public key "Tony Hutter (GPG key for signing ZFS releases) <hutter2@llnl.gov>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

Moramo instalirati i `patch`.

```shell
[fidit@archlinux ~]$ sudo pacman -S patch
```

Moramo instalirati i `autoconf` i `automake` (to nam fali jer install jos uvijek nece delat). Isto moramo i `dkms`.

Onda pokrenemo `makepkg` da buildamo. 

```shell
[fidit@archlinux ~]$ makepkg
==> Making package: zfs-dkms 2.1.4-1 (Fri 01 Apr 2022 01:04:12 PM UTC)
==> Checking runtime dependencies...
==> Checking buildtime dependencies...
==> Retrieving sources...
  -> Found zfs-2.1.4.tar.gz
  -> Found zfs-2.1.4.tar.gz.asc
  -> Found 0001-only-build-the-module-in-dkms.conf.patch
==> Validating source files with sha256sums...
    zfs-2.1.4.tar.gz ... Passed
    zfs-2.1.4.tar.gz.asc ... Skipped
    0001-only-build-the-module-in-dkms.conf.patch ... Passed
==> Validating source files with b2sums...
    zfs-2.1.4.tar.gz ... Passed
    zfs-2.1.4.tar.gz.asc ... Skipped
    0001-only-build-the-module-in-dkms.conf.patch ... Passed
==> Verifying source file signatures with gpg...
    zfs-2.1.4.tar.gz ... Passed
==> Extracting sources...
  -> Extracting zfs-2.1.4.tar.gz with bsdtar
==> Starting prepare()...
patching file scripts/dkms.mkconf
configure.ac: warning: AM_GNU_GETTEXT is used, but not AM_GNU_GETTEXT_VERSION or AM_GNU_GETTEXT_REQUIRE_VERSION
libtoolize: putting auxiliary files in AC_CONFIG_AUX_DIR, 'config'.
libtoolize: copying file 'config/ltmain.sh'
libtoolize: putting macros in AC_CONFIG_MACRO_DIRS, 'config'.
libtoolize: copying file 'config/libtool.m4'
libtoolize: copying file 'config/ltoptions.m4'
libtoolize: copying file 'config/ltsugar.m4'
libtoolize: copying file 'config/ltversion.m4'
libtoolize: copying file 'config/lt~obsolete.m4'
configure.ac:48: installing 'config/compile'
configure.ac:42: installing 'config/missing'
==> Starting build()...
==> Entering fakeroot environment...
==> Starting package()...
==> Tidying install...
  -> Removing libtool files...
  -> Purging unwanted files...
  -> Removing static library files...
  -> Stripping unneeded symbols from binaries and libraries...
  -> Compressing man and info pages...
==> Checking for packaging issues...
==> Creating package "zfs-dkms"...
  -> Generating .PKGINFO file...
  -> Generating .BUILDINFO file...
  -> Generating .MTREE file...
  -> Compressing package...
==> Leaving fakeroot environment.
==> Finished making: zfs-dkms 2.1.4-1 (Fri 01 Apr 2022 01:04:32 PM UTC)
```

`zfs-dkms-2.1.4-1-any.pkg.tar.zst` je kernel modul kojeg smo izgradili u obliku paketa za pacman. 

```shell
[fidit@archlinux ~]$ sudo pacman -U zfs-dkms-2.1.4-1-any.pkg.tar.zst 
loading packages...
resolving dependencies...
warning: cannot resolve "zfs-utils=2.1.4", a dependency of "zfs-dkms"
:: The following package cannot be upgraded due to unresolvable dependencies:
      zfs-dkms

:: Do you want to skip the above package for this upgrade? [y/N] n
error: failed to prepare transaction (could not satisfy dependencies)
:: unable to satisfy dependency 'zfs-utils=2.1.4' required by zfs-dkms
```

Fale nam `zfs-utils`. To mozemo sami compileat. https://aur.archlinux.org/packages/zfs-utils

```


    https://github.com/zfsonlinux/zfs/releases/download/zfs-2.1.4/zfs-2.1.4.tar.gz
    https://github.com/zfsonlinux/zfs/releases/download/zfs-2.1.4/zfs-2.1.4.tar.gz.asc
    zfs.initcpio.hook
    zfs.initcpio.install

```

Prva dva vec imamo, druga dva moramo preuzeti. 

```
[fidit@archlinux ~]$ curl -o zfs.initcpio.hook "https://aur.archlinux.org/cgit/aur.git/plain/zfs.initcpio.hook?h=zfs-utils"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3620  100  3620    0     0  16028      0 --:--:-- --:--:-- --:--:-- 16088
[fidit@archlinux ~]$ curl -o zfs.initcpio.install "https://aur.archlinux.org/cgit/aur.git/plain/zfs.initcpio.install?h=zfs-utils"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2301  100  2301    0     0  11099      0 --:--:-- --:--:-- --:--:-- 11115
```

Skinemo i PKGBUILD za zfs utils i nazovemo ga drugacije (inace to ne radimo ali mozemo ovdje iznimno).

`pkgbuild -p (ili c, ne znam) PKGBUILD-utils`

Kad je kompajlanje gotovo:

```shell
[fidit@archlinux ~]$ sudo pacman -U zfs-dkms-2.1.4-1-any.pkg.tar.zst zfs-utils-2.1.4-1-x86_64.pkg.tar.zst 
loading packages...
resolving dependencies...
```

Za upravljanje ZFS volumenima su `zfs` i `zfs pool`. 

```shell
[fidit@archlinux ~]$ sudo modprobe zfs
modprobe: FATAL: Module zfs not found in directory /lib/modules/5.17.1-arch1-1
```

ZFS nam trenutno ne radi, moramo imat instalirane linux headers. (https://wiki.archlinux.org/title/Dynamic_Kernel_Module_Support)

```shell
[fidit@archlinux ~]$ sudo pacman -S linux-headers
```

Trebamo pricekat dulje jer ponovo kompajlira zfs da ispravi mane koje su nastale kad smo ga instalirali bez headera. 

