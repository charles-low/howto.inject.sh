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