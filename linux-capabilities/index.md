Five set of capabilities
- Effective capabilities (CapEff) - verified for each privilege action 
- Permitted capabilities (CapPrm) - can be effective when needed using syscalls
- Inherited capabilities (CapLnh) 
- Ambient capabilities (CapAmb)
- Bounding set (CapBnd) - superset of the capabilities, cannot exceed the set of privilege

# printing capabilities
```capsh --print```
```
grep Cap /proc/$BASHPID/status
capsh --decode=0000003fffffffff

sudo -l
sudo capsh --print

sudo bash
capsh --print
```

# Ping - capability aware

## trace the capget and getset calls
```sudo strace ping 10.0.2.2```

## set cap for ping
```sudo setcap 'cap_net_raw+p' /bin/ping``` (add to CapPrm)

# TCPDUMP - capability inaware

``` sudo set cap 'cap_net_raw+p' /usr/sbin/tcpdump``` does not work because it is not capability aware
in this case we need to set both effective and permitted set of tcpdump
``` sudo set cap 'cap_net_raw+ep' /usr/sbin/tcpdump```

# Ambient capability
Reference: https://lwn.net/Articles/636533/
https://s3hh.wordpress.com/2015/07/25/ambient-capabilities/

need to set via C binary
[source](https://gist.githubusercontent.com/infinity0/596845b5eea3e1a02a018009b2931a39/raw/db1c6fbb66825b7628ebf274d3ddebe4ba13795e/ambient.c)
[local](./ambient.c)

```
gcc -Wl,--no-as-needed -lcap-ng -o ambient ambient.c
sudo setcap cap_setpcap,cap_net_raw,cap_net_admin,cap_sys_nice+eip ambient
./ambient /bin/bash
```

# use of systemd config file
Non-root with CAP granted
```
[Unit]
Description=Capability Demo
[Service]
Type=forking ExecStart=/home/student/servicebinary User=student AmbientCapabilities=CAP_NET_BIND_SERVICE
```
Root with bounding capability
```
[Unit]
Description=Capability Demo
[Service]
Type=forking ExecStart=/home/student/servicebinary CapabilityBoundingSet=CAP_NET_BIND_SERVICE
```

# Reference
Linux capabilities in practice (​https://blog.container-solutions.com/linux-capabilities-in-practice​)
Linux Audit (​https://linux-audit.com/linux-capabilities-101/​)