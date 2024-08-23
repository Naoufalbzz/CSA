# Lab 3

### SSH
- ik heb een config file gemaakt op de company router om ssh te doen op alle servers in de internal company. Dat ziet er zo uit
```console
[vagrant@companyrouter ~]$ cat .ssh/config
Host web
        Hostname 172.30.20.10
        User vagrant
Host database
        Hostname 172.30.0.15
        User vagrant
Host win10
        Hostname 172.30.100.100
        User vagrant
Host dc
        Hostname 172.30.0.4
        User vagrant
```

- ik heb van companyrouter een bastion host gemaakt en heb op mijn eigen windows host een ssh config file gemaakt waardoor ik direct met alle servers in de internal company kan connecten via de companyrouter en door een 'proxy jump' uit te voeren. Dit is het config file
```console
Host companyrouter
        Hostname 192.168.100.253
        User vagrant
Host isprouter
        Hostname 192.168.100.254
        User vagrant
Host red
        Hostname 192.168.100.102
        User osboxes
Host web
        Hostname 172.30.20.10
        User vagrant
	ProxyJump companyrouter
Host dc
        Hostname 172.30.0.4
        User vagrant
	ProxyJump companyrouter
Host win10
        Hostname 172.30.100.100
        User vagrant
	ProxyJump companyrouter
Host database
        Hostname 172.30.0.15
        User vagrant
	ProxyJump companyrouter
```

- nadien heb ik een public/private key pair gemaakt op mijn windows host en de public key gekopieerd naar alle machines en de inhoud van de public key in de `autorized_keys` file gezet van elke machine zodat ik geen wachtwoorden meer hoef in te vullen wanneer ik ssh van mijn windows host met commando `ssh-copy-id vagrant@172.30.0.15`
- `ssh-keygen`
- `scp .\.ssh\id_rsa.pub win10:C:\Users\vagrant.insecure\key1_rsa.pub` OF `scp .\.ssh\id_rsa.pub web:~/key1_rsa.pub`
## Voorlopig netwerktabel
|                   | **IP**                                                                                    | **DNS**       | **Default gateway/routes**                                                                                                                                                                                                                                                                                                                                                           |
|-------------------|-------------------------------------------------------------------------------------------|---------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Companyrouter** | eth1:172.30.0.254/24      172.30.20.254/24      172.30.100.254/24 eth0:192.168.100.253/24 | 172.30.0.4    | default via 192.168.100.254 dev eth0 proto static metric 100 172.30.0.0/24 dev eth1 proto kernel scope link src 172.30.0.254 metric 101 172.30.20.0/24 dev eth1 proto kernel scope link src 172.30.20.254 metric 101 172.30.100.0/24 dev eth1 proto kernel scope link src 172.30.100.254 metric 101 192.168.100.0/24 dev eth0 proto kernel scope link src 192.168.100.253 metric 100 |
| **Isprouter**     | eth0:10.0.2.15/24 eth1:192.168.100.254/24                                                 | 10.0.2.3      | default via 10.0.2.2 dev eth0  metric 202 10.0.2.0/24 dev eth0 scope link  src 10.0.2.15 172.30.0.0/16 via 192.168.100.253 dev eth1 192.168.100.0/24 dev eth1 scope link  src 192.168.100.254                                                                                                                                                                                        |
| **red**           | enp0s3:192.168.100.102/24                                                                 | 8.8.8.8       | default via 192.168.100.254 dev enp0s3 onlink 169.254.0.0/16 dev enp0s3 scope link metric 1000 192.168.100.0/24 dev enp0s3 proto kernel scope link src 192.168.100.102                                                                                                                                                                                                               |
| **database**      | eth0:172.30.0.15/24                                                                       | 172.30.0.4    | default via 172.30.0.254 dev eth0 proto static metric 100 172.30.0.0/24 dev eth0 proto kernel scope link src 172.30.0.15 metric 100                                                                                                                                                                                                                                                  |
| **web**           | eth0:172.30.20.10/24                                                                      | 172.30.0.4    | default via 172.30.20.254 dev eth0 proto static metric 100 172.30.20.0/24 dev eth0 proto kernel scope link src 172.30.20.10 metric 100                                                                                                                                                                                                                                               |
| **dc**            | Ethernet:172.30.0.4/24                                                                    | ::1,127.0.0.1 | 172.30.0.254                                                                                                                                                                                                                                                                                                                                                                         |
| **win10**        | Ethernet:172.30.100.100                                                                   | 172.30.0.4    | 172.30.100.254                                                                                                                                                                                                                                                                                                                                                                       |














## Firewall
- dropt alle input/output/forward packets als ze niet voldoen aan een andere firewall rule(komt niet voor de regels, de drop gebeurt op einde)
```
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT DROP
```

```
[vagrant@companyrouter ~]$ sudo iptables -A INPUT -s 172.30.100.0/24 -j ACCEPT
[vagrant@companyrouter ~]$ sudo iptables -A FORWARD -s 172.30.100.0/24 -j ACCEPT
[vagrant@companyrouter ~]$ sudo iptables -A OUTPUT -s 172.30.100.0/24 -j ACCEPT
```
```
[vagrant@companyrouter ~]$ sudo iptables -L INPUT -v -n && sudo iptables -L FORWARD -v -n && sudo iptables -L OUTPUT -v -n
Chain INPUT (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     0    --  *      *       172.123.0.0/24       0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       192.168.100.103      0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       172.123.0.0/24       0.0.0.0/0
 4372  272K ACCEPT     0    --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
    0     0 ACCEPT     0    --  *      *       192.168.100.254      0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       172.30.0.0/16        192.168.100.254
    0     0 ACCEPT     0    --  *      *       172.30.0.0/16        192.168.100.103
    0     0 ACCEPT     0    --  *      *       192.168.100.0/24     172.30.20.8
    0     0 ACCEPT     0    --  *      *       192.168.100.102      172.30.20.8
    0     0 ACCEPT     0    --  *      *       192.168.100.102      172.30.20.8
   33  3692 ACCEPT     0    --  *      *       172.30.0.0/16        172.30.0.0/16
    0     0 ACCEPT     0    --  *      *       172.30.20.0/24       172.30.0.0/24
    0     0 ACCEPT     0    --  *      *       172.30.20.8          172.30.0.4
    0     0 ACCEPT     0    --  *      *       172.30.0.0/16        0.0.0.0/0
   42  3024 ACCEPT     0    --  *      *       192.168.100.1        0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       172.30.20.8          192.168.100.104
    0     0 ACCEPT     0    --  *      *       192.168.100.104      172.30.20.8
    0     0 ACCEPT     0    --  *      *       192.168.100.254      0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       192.168.100.253      0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       0.0.0.0/0            192.168.100.254
Chain FORWARD (policy DROP 134 packets, 9000 bytes)
 pkts bytes target     prot opt in     out     source               destination
 4645 3712K DOCKER-USER  0    --  *      *       0.0.0.0/0            0.0.0.0/0
 4645 3712K DOCKER-ISOLATION-STAGE-1  0    --  *      *       0.0.0.0/0            0.0.0.0/0

    0     0 ACCEPT     0    --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 DOCKER     0    --  *      docker0  0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     0    --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     0    --  docker0 docker0  0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       192.168.100.254      172.30.0.0/24
    0     0 ACCEPT     0    --  *      *       192.168.100.254      172.30.20.0/24
    0     0 ACCEPT     0    --  *      *       192.168.100.254      172.30.100.0/24
    0     0 ACCEPT     0    --  *      *       172.30.100.0/24      192.168.100.254
    0     0 ACCEPT     0    --  *      *       172.30.20.0/24       192.168.100.254
    0     0 ACCEPT     0    --  *      *       172.30.0.0/24        192.168.100.254
    0     0 ACCEPT     0    --  *      *       192.168.100.103      0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       172.123.0.0/24       0.0.0.0/0
 3992 3671K ACCEPT     0    --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
    0     0 ACCEPT     0    --  *      *       192.168.100.254      0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       172.30.0.0/16        192.168.100.254
    0     0 ACCEPT     0    --  *      *       172.30.0.0/16        192.168.100.103
    0     0 ACCEPT     0    --  *      *       172.30.0.0/16        192.168.100.104
    0     0 ACCEPT     0    --  *      *       192.168.100.104      172.30.0.0/16
    0     0 ACCEPT     0    --  *      *       172.30.0.0/16        172.123.0.0/24
    0     0 ACCEPT     0    --  *      *       192.168.100.0/24     172.30.20.0/24
    0     0 ACCEPT     0    --  *      *       192.168.100.0/24     172.30.20.8
    0     0 ACCEPT     0    --  *      *       192.168.100.102      172.30.20.8
  360 21600 ACCEPT     0    --  *      *       172.30.0.0/16        172.30.0.0/16
    0     0 ACCEPT     0    --  *      *       172.30.0.0/16        192.168.100.0/24
   47  3835 ACCEPT     0    --  *      *       0.0.0.0/0            10.0.2.0/24
    0     0 ACCEPT     0    --  *      *       10.0.2.0/24          0.0.0.0/0
   49  3316 ACCEPT     0    --  *      *       172.30.0.0/16        0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       192.168.100.1        0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       172.30.20.8          192.168.100.104
    0     0 ACCEPT     0    --  *      *       192.168.100.104      172.30.20.8
    0     0 ACCEPT     0    --  *      *       192.168.100.254      0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       192.168.100.253      0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       0.0.0.0/0            192.168.100.253
    0     0 ACCEPT     0    --  *      *       192.168.100.104      192.168.100.254
   23  1500 ACCEPT     0    --  *      *       10.8.0.6             0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       0.0.0.0/0            10.8.0.6
Chain OUTPUT (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     0    --  *      *       172.123.0.0/24       0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       192.168.100.103      0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       172.123.0.0/24       0.0.0.0/0
 4699  534K ACCEPT     0    --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
    0     0 ACCEPT     0    --  *      *       192.168.100.254      0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       172.30.0.0/16        192.168.100.254
    0     0 ACCEPT     0    --  *      *       172.30.0.0/16        192.168.100.103
    0     0 ACCEPT     0    --  *      *       192.168.100.0/24     172.30.20.8
    0     0 ACCEPT     0    --  *      *       192.168.100.102      172.30.20.8
  244 14686 ACCEPT     0    --  *      *       172.30.0.0/16        0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       192.168.100.1        0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       172.30.20.8          192.168.100.104
    0     0 ACCEPT     0    --  *      *       192.168.100.104      172.30.20.8
    0     0 ACCEPT     0    --  *      *       192.168.100.254      0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       0.0.0.0/0            192.168.100.254
   52  3992 ACCEPT     0    --  *      *       192.168.100.253      0.0.0.0/0
    0     0 ACCEPT     0    --  *      *       0.0.0.0/0            192.168.100.253
```
- `sudo apt install iptables-persistent`
- `sudo systemctl start iptables`
- `sudo systemctl enable iptables`
- `sudo iptables-save > /etc/sysconfig/iptables`
- `sudo iptables -L -v -n --line-numbers` (list all)
- `sudo iptables -L INPUT -v -n && sudo iptables -L FORWARD -v -n && sudo iptables -L OUTPUT -v -n`
- `sudo iptables -D INPUT 3` (delete)
- `sudo iptables -A INPUT -s <IP_ADDRESS> -j ACCEPT` (add)


- delete REJECT rule om te pingen
- add REJECT rule voor red (voor beide subnets) -> can only ping web
- `sudo iptables -I INPUT -s 192.168.100.102 -d 172.30.0.0/24 -j DROP`
- `sudo iptables -I FORWARD -s 192.168.100.102 -d 172.30.0.0/24 -j DROP`
- allow all ports needed and default block all
```
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 9090 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 53 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 88 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 135 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 139 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 389 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 445 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 464 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 593 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 636 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 3268 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 3269 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 3389 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 111 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
```
```
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT DROP
```
- `sudo iptables -A FORWARD -s 172.30.0.0/24 -d 192.168.100.254 -j ACCEPT` voor internet access

## DHCP
- `/etc/dhcp/dhcpd.conf` aanpassen
```
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd
#

# option definitions common to all supported networks...
option domain-name "insecure.cyb";
option domain-name-servers 172.30.0.4;

default-lease-time 600;
max-lease-time 7200;

# Use this to enble / disable dynamic dns updates globally.
#ddns-update-style none;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;


subnet 172.30.100.0 netmask 255.255.255.0 {
  range 172.30.100.100 172.30.100.200;
  option routers 172.30.100.254;
}
```
```
sudo systemctl start dhcpd
sudo systemctl enable dhcpd
```

* The webserver also acts as a reverse proxy for another (java-application). The app can be viewed by browsing to www.insecure.cyb/cmd. The java application is configured with a systemd service.TODO: double check this configuration and how it is properly configured. 
   - 
 ```bash
[vagrant@web ~]$ cd /etc/systemd/system
[vagrant@web system]$ ls
basic.target.wants                          default.target          multi-user.target.wants      remote-fs.target.wants
ctrl-alt-del.target                         getty.target.wants      network-online.target.wants  sockets.target.wants
dbus-org.freedesktop.nm-dispatcher.service  graphical.target.wants  nginx.service.d              sysinit.target.wants
dbus.service                                insecurewebapp.service  php-fpm.service.d            timers.target.wants
[vagrant@web system]$ cat insecurewebapp.service
[Unit]
Description = start script for insecurewebapp

[Service]
SyslogIdentifier=insecurewebapp
Type=simple
ExecStart = /usr/bin/java -server -Xms128m -Xmx512m -jar /opt/insecurewebapp/app.jar
User=root

[Install]
WantedBy = multi-user.target
```
- The configuration of the webserver (the http://www.insecure.cyb website) and how it is connected to the database, what are the credentials, is this secure?
  - The http://www.insecure.cyb website is configured with a PHP script (index.php) that connects to a MySQL database using the following details
  - Database Host: `172.30.0.15`
  - Database Username: `sammy`
  - Database Password: `FLAG-741852`
  - Database Name: `users`  
  - The database credentials are hard-coded in the index.php file. This is highly insecure because anyone with access to the file can view the credentials, leading to potential unauthorized database access.
* De configuratie van de webserver als een reverse proxy naar http://www.insecure.cyb/cmd. Hoe is dit ingesteld, bekijk de reverse proxy configuratie en het systemd config bestand. Welke poort draait de Java-app? Waar bevindt de jar zich? Waar bevindt zich het systemd configuratiebestand? Hoe kun je deze applicatie neerhalen zonder http://www.insecure.cyb neer te halen?
   - The reverse proxy is set up to forward requests to a local service running on port 8000. This configuration forwards requests made to /cmd, /assets, and /exec to the internal service running on http://localhost:8000.
  - ```
        LoadModule proxy_module modules/mod_proxy.so
        LoadModule proxy_http_module modules/mod_proxy_http.so

        ProxyPass "/cmd" "http://localhost:8000/"
        ProxyPassReverse "/cmd" "http://localhost:8000/"
        ProxyPass "/assets" "http://localhost:8000/assets"
        ProxyPassReverse "/assets" "http://localhost:8000/assets"
        ProxyPass "/exec" "http://localhost:8000/exec"
        ProxyPassReverse "/exec" "http://localhost:8000/exec"
        ```
  - The Java application is running on port 8000, as indicated by the proxy settings in httpd.conf.
  - The jar file for the Java application is located at /opt/insecurewebapp/app.jar.
  - This indicates that the Java application is started using the ExecStart directive, which runs the command to start the Java application (/usr/bin/java -server -Xms128m -Xmx512m -jar /opt/insecurewebapp/app.jar).
- Stopping the Java Application Without Affecting the Main Website
  - ```sudo systemctl stop insecurewebapp```