```
apt install dropbear-initramfs
```


# setup netatalk 
/etc/netatalk/afp.conf
```
[$f]
path = /storage/$u/data

[Time Machine]
path = /storage/$u/time_machine
time machine = yes
```

# EFI secure boot
http://kroah.com/log/blog/2013/09/02/booting-a-self-signed-linux-kernel/