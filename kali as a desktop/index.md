# Turn on AFP discovery
``` apt install netatalk
/etc/init.d/avahi-daemon start
systemctl enable avahi-daemon
```


# Build your own gvfs for Kali

the default binaries of kali doesn't include gvfs backends for afp. Therefore we try to build our own

1. Change /etc/apt/sources.list to include apt-src

``` sudo apt update && sudo apt install build-essentials ```

Install build dependencies

``` sudo apt-get build-dep gvfs ```
``` sudo apt-get source gvfs```

We realise it is using meson instead of typeical makefile
change debian/rules to include -Dafp as a test first


``` dpkg-buildpackage -us -uc     ```

# Enable printer support

```
apt install cups printer-drivers-all
```

# Chinese input
sudo apt install im-config ibus ibus-cangjie
sudo im-config -n ibus

# setup dictionary like mac
https://forums.linuxmint.com/viewtopic.php?t=200767

apt install xsel gnome-dictionary
apt install hunspell goldendict
set customer shortcut
### this not work as intended
bash -c "gnome-dictionary --look-up=$(xsel | tr '\n' ' ' | sed -r 's/^[^[:alpha:]]*([-[:alpha:]]*).*$/\1/')"
### we tested manually with this though
gnome-dictionary -D wn --look-up=test
gnome-dictionary --look-up=test


## offline
https://ubuntuforums.org/showthread.php?t=145949

apt install dict gcide dictd dict-wn dict

apt install dict-moby-thesaurus dict-devil dict-vera

## testing the selection
bash -c "zenity --info --text=$(xsel)"


# backup and restore
   https://ostechnix.com/backup-and-restore-linux-desktop-system-settings-with-dconf/


# setup netatalk 
/etc/netatalk/afp.conf
[$f]
path = /home/$u

# EFI and secure boot
https://wiki.debian.org/UEFI
``` efibootmgr
BootCurrent: 0001
Timeout: 1 seconds
BootOrder: 0001,0003,0002,0000
Boot0000* Windows Boot Manager
Boot0001* kali
Boot0002* LAN : IBA GE Slot 00C8 v1553
Boot0003* M.2 SATA :SAMSUNG MZNLN128HCGR-00007 : PART 0 : Boot Drive
                                                                        

efibootmgr -v 
BootCurrent: 0001
Timeout: 1 seconds
BootOrder: 0001,0003,0002,0000
Boot0000* Windows Boot Manager	VenHw(99e275e7-75a0-4b37-a2e6-c5385e6c00cb)WINDOWS.........x...B.C.D.O.B.J.E.C.T.=.{.9.d.e.a.8.6.2.c.-.5.c.d.d.-.4.e.7.0.-.a.c.c.1.-.f.3.2.b.3.4.4.d.4.7.9.5.}...d................
Boot0001* kali	HD(1,GPT,2c92e14c-45c2-4d43-8e82-73cade5a91c5,0x800,0x100000)/File(\EFI\kali\grubx64.efi)
Boot0002* LAN : IBA GE Slot 00C8 v1553	BBS(Network,,0x0)..BO
Boot0003* M.2 SATA :SAMSUNG MZNLN128HCGR-00007 : PART 0 : Boot Drive	BBS(HD,,0x0)..BO

```
https://wiki.debian.org/SecureBoot
shim


machine owner key (MOK)
mokutil

Generate MOK key
```
$ openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -days 36500 -subj "/CN=My Name/" -nodes
$ openssl x509 -inform der -in MOK.der -out MOK.pem
```

Enroll
```
$ sudo mokutil --import MOK.der # prompts for one-time password
$ sudo mokutil --list-new # recheck your key will be prompted on next boot

<rebooting machine then enters MOK manager EFI utility: enroll MOK, continue, confirm, enter password, reboot>

$ sudo dmesg | grep cert # verify your key is loaded
```

mokutil --sb-state     
SecureBoot disabled
Platform is in Setup Mode


sign kenerl and module
```
$ sbsign --key MOK.priv --cert MOK.pem /boot/vmlinuz-$ver --output vmlinuz-$ver
$ sudo mv vmlinux-$ver /boot/vmlinux-$ver

# cd /lib/modules/4.19.0-6-amd64/updates/dkms
# /usr/lib/linux-kbuild-4.19/scripts/sign-file sha256 /root/MOK.priv /root/MOK.der vboxdrv.ko
```

1. Only trusted MOK key
2. MOk key verfiy boot loader (grub)
3. linux kernel signed by user

https://askubuntu.com/questions/1247826/secure-boot-verification-of-initramfs