remoteclient `ip r`:
```
osboxes@osboxes:~$ ip r
default via 172.123.0.254 dev enp0s3 proto static metric 20100
169.254.0.0/16 dev enp0s3 scope link metric 1000
172.123.0.0/24 dev enp0s3 proto kernel scope link src 172.123.0.1 metric 100
```

remoterouter `ip r`:
```
osboxes@osboxes:~$ ip r
default via 192.168.100.253 dev enp0s3 onlink
169.254.0.0/16 dev enp0s8 scope link metric 1000
172.123.0.0/24 dev enp0s8 proto kernel scope link src 172.123.0.254
192.168.100.0/24 dev enp0s3 proto kernel scope link src 192.168.100.103
```

companyrouter `ip r`:
```
[vagrant@companyrouter system-connections]$ ip r
default via 192.168.100.254 dev eth0 proto static metric 100
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
172.30.0.0/24 dev eth1 scope link
172.30.20.0/24 dev eth1 scope link
172.30.100.0/24 dev eth1 scope link
172.123.0.0/24 via 192.168.100.103 dev eth0 proto static metric 100
192.168.100.0/24 dev eth0 proto kernel scope link src 192.168.100.253 metric 100
```

isprouter `ip r`:
```
isprouter:~$ ip r
default via 10.0.2.2 dev eth0  metric 202
10.0.2.0/24 dev eth0 scope link  src 10.0.2.15
172.30.0.0/16 via 192.168.100.253 dev eth1
172.123.0.0/24 via 192.168.100.103 dev eth1
192.168.100.0/24 dev eth1 scope link  src 192.168.100.254
```