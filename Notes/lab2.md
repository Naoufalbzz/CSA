# Lab 2

## Make sure to improve your documentation for all "insecure" stuff you notice.
- Perform a default nmap scan on all machines
  * `nmap -sP 172.30.0.4/16` voor ping sweep op netwerk
  * `nmap -Pn 172.30.0.10` op host ookal blockt host pings
* ```bash
        osboxes@osboxes:~$ nmap -sP 172.30.0.0/16
        Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-02 09:57 EDT
        Nmap scan report for 172.30.0.10
        Host is up (0.00096s latency).
        Nmap scan report for 172.30.0.15
        Host is up (0.0013s latency).
        Nmap scan report for 172.30.10.100
        Host is up (0.0016s latency).
        Nmap scan report for 172.30.10.101
        Host is up (0.0013s latency).
        Stats: 0:04:52 elapsed; 8192 hosts completed (4 up), 4096 undergoing Ping Scan
        Ping Scan Timing: About 56.09% done; ETC: 10:03 (0:01:13 remaining) 
     ```
* ```bash
        osboxes@osboxes:~$ nmap -sP 172.30.0.4
        Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-02 10:15 EDT
        Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
        Nmap done: 1 IP address (0 hosts up) scanned in 3.00 seconds
        osboxes@osboxes:~$ nmap -Pn 172.30.0.4
        Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-02 10:16 EDT
        Nmap scan report for 172.30.0.4
        Host is up (0.00085s latency).
        Not shown: 987 filtered ports
        PORT     STATE SERVICE
        22/tcp   open  ssh
        53/tcp   open  domain
        88/tcp   open  kerberos-sec
        135/tcp  open  msrpc
        139/tcp  open  netbios-ssn
        389/tcp  open  ldap
        445/tcp  open  microsoft-ds
        464/tcp  open  kpasswd5
        593/tcp  open  http-rpc-epmap
        636/tcp  open  ldapssl
        3268/tcp open  globalcatLDAP
        3269/tcp open  globalcatLDAPssl
        3389/tcp open  ms-wbt-server

        Nmap done: 1 IP address (1 host up) scanned in 4.34 seconds
        ```
* ```bash
        osboxes@osboxes:~$ nmap -Pn 172.30.0.10
        Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-02 10:20 EDT
        Nmap scan report for 172.30.0.10
        Host is up (0.64s latency).
        Not shown: 997 filtered ports
        PORT     STATE  SERVICE
        22/tcp   open   ssh
        80/tcp   open   http
        9090/tcp closed zeus-admin

        Nmap done: 1 IP address (1 host up) scanned in 70.89 seconds
  ```
* ```bash
        osboxes@osboxes:~$ nmap -Pn 172.30.0.15
        Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-02 10:24 EDT
        Nmap scan report for 172.30.0.15
        Host is up (0.00037s latency).
        Not shown: 997 closed ports
        PORT     STATE SERVICE
        22/tcp   open  ssh
        111/tcp  open  rpcbind
        3306/tcp open  mysql

        Nmap done: 1 IP address (1 host up) scanned in 0.06 seconds
  ```
* ```bash
        osboxes@osboxes:~$ nmap -Pn 172.30.255.254
        Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-02 10:30 EDT
        Nmap scan report for 172.30.255.254
        Host is up (0.00033s latency).
        Not shown: 998 closed ports
        PORT    STATE SERVICE
        22/tcp  open  ssh
        111/tcp open  rpcbind

        Nmap done: 1 IP address (1 host up) scanned in 0.06 seconds  
  ```

* ```bash
        osboxes@osboxes:~$ nmap -Pn 192.168.100.254
        Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-02 10:31 EDT
        Nmap scan report for 192.168.100.254
        Host is up (0.00024s latency).
        Not shown: 999 closed ports
        PORT   STATE SERVICE
        22/tcp open  ssh

        Nmap done: 1 IP address (1 host up) scanned in 0.06 seconds 
  ```
* ```bash
        osboxes@osboxes:~$ nmap -Pn 172.30.10.100
        Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-02 10:32 EDT
        Nmap scan report for 172.30.10.100
        Host is up (0.00068s latency).
        Not shown: 995 closed ports
        PORT     STATE SERVICE
        22/tcp   open  ssh
        135/tcp  open  msrpc
        139/tcp  open  netbios-ssn
        445/tcp  open  microsoft-ds 
        3389/tcp open  ms-wbt-server

        Nmap done: 1 IP address (1 host up) scanned in 5.29 seconds
  ```

* Enumerate the most interesting ports (you found in the previous step) by issuing a service enumeration scan (banner grab scan).
  - `nmap -sV -Pn -p 22,80,9090 172.30.0.10`
  - ```bash
        osboxes@osboxes:~$ nmap -sV -Pn -p 22,53,88,135,139,389,445,464,593,636,3268,3269,3389 172.30.0.4
        Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-02 10:53 EDT
        Nmap scan report for 172.30.0.4
        Host is up (0.0012s latency).

        PORT     STATE SERVICE       VERSION
        22/tcp   open  ssh           OpenSSH for_Windows_8.0 (protocol 2.0)
        53/tcp   open  domain?
        88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-08-02 14:53:35Z)
        135/tcp  open  msrpc         Microsoft Windows RPC
        139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
        389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: insecure.cyb0., Site: Default-First-Site-Name)
        445/tcp  open  microsoft-ds?
        464/tcp  open  kpasswd5?
        593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
        636/tcp  open  tcpwrapped
        3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: insecure.cyb0., Site: Default-First-Site-Name)
        3269/tcp open  tcpwrapped
        3389/tcp open  ms-wbt-server Microsoft Terminal Services
        1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
        SF-Port53-TCP:V=7.80%I=7%D=8/2%Time=66ACF2F5%P=x86_64-pc-linux-gnu%r(DNSVe
        SF:rsionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\x
        SF:04bind\0\0\x10\0\x03");
        Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

        Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
        Nmap done: 1 IP address (1 host up) scanned in 142.40 seconds
    ```  
  - ```bash
        osboxes@osboxes:~$ nmap -sV -Pn -p 22,80,9090 172.30.0.10
        Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-02 10:56 EDT
        Nmap scan report for 172.30.0.10
        Host is up (0.00080s latency).

        PORT     STATE  SERVICE    VERSION
        22/tcp   open   ssh        OpenSSH 8.7 (protocol 2.0)
        80/tcp   open   http       Apache httpd 2.4.53 ((AlmaLinux))
        9090/tcp closed zeus-admin

        Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
        Nmap done: 1 IP address (1 host up) scanned in 6.29 seconds
    ```
  - ```bash

        osboxes@osboxes:~$ nmap -sV -p 22,111,3306 172.30.0.15
        Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-02 11:03 EDT
        Nmap scan report for 172.30.0.15
        Host is up (0.0012s latency).

        PORT     STATE SERVICE VERSION
        22/tcp   open  ssh     OpenSSH 8.7 (protocol 2.0)
        111/tcp  open  rpcbind 2-4 (RPC #100000)
        3306/tcp open  mysql?
        1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
        SF-Port3306-TCP:V=7.80%I=7%D=8/2%Time=66ACF556%P=x86_64-pc-linux-gnu%r(NUL
        SF:L,4E,"J\0\0\0\n8\.0\.32\0\)\0\0\x0077K`C\tu\*\0\xff\xff\xff\x02\0\xff\x
        SF:df\x15\0\0\0\0\0\0\0\0\0\0A4G>g\t\x07\x13/Fo!\0caching_sha2_password\0"
        SF:)%r(GenericLines,73,"J\0\0\0\n8\.0\.32\0\)\0\0\x0077K`C\tu\*\0\xff\xff\
        SF:xff\x02\0\xff\xdf\x15\0\0\0\0\0\0\0\0\0\0A4G>g\t\x07\x13/Fo!\0caching_s
        SF:ha2_password\0!\0\0\x01\xff\x84\x04#08S01Got\x20packets\x20out\x20of\x2
        SF:0order")%r(GetRequest,73,"J\0\0\0\n8\.0\.32\0\*\0\0\0YaJ5W\x01%\x15\0\x
        SF:ff\xff\xff\x02\0\xff\xdf\x15\0\0\0\0\0\0\0\0\0\0~Y\\sZ\x010/c\x02lE\0ca
        SF:ching_sha2_password\0!\0\0\x01\xff\x84\x04#08S01Got\x20packets\x20out\x
        SF:20of\x20order")%r(HTTPOptions,73,"J\0\0\0\n8\.0\.32\0\+\0\0\0W_3\x20T\x
        SF:0bH9\0\xff\xff\xff\x02\0\xff\xdf\x15\0\0\0\0\0\0\0\0\0\0\+j\x1c\x01`_Jg
        SF:I'\x03\x0e\0caching_sha2_password\0!\0\0\x01\xff\x84\x04#08S01Got\x20pa
        SF:ckets\x20out\x20of\x20order")%r(RTSPRequest,73,"J\0\0\0\n8\.0\.32\0,\0\
        SF:0\0\x0e\x7fAw\x10@\x08O\0\xff\xff\xff\x02\0\xff\xdf\x15\0\0\0\0\0\0\0\0
        SF:\0\0HE\x05\x0eZ%\x14\)d\r%y\0caching_sha2_password\0!\0\0\x01\xff\x84\x
        SF:04#08S01Got\x20packets\x20out\x20of\x20order")%r(RPCCheck,73,"J\0\0\0\n
        SF:8\.0\.32\0-\0\0\0`@\[Q\x14Z\x06\x19\0\xff\xff\xff\x02\0\xff\xdf\x15\0\0
        SF:\0\0\0\0\0\0\0\0\x19\x111\?/,\x0e\x01-M\x02\*\0caching_sha2_password\0!
        SF:\0\0\x01\xff\x84\x04#08S01Got\x20packets\x20out\x20of\x20order")%r(DNSV
        SF:ersionBindReqTCP,73,"J\0\0\0\n8\.0\.32\0\.\0\0\0<6\x02~\x7f\tjm\0\xff\x
        SF:ff\xff\x02\0\xff\xdf\x15\0\0\0\0\0\0\0\0\0\0b\x1eOm0\[\n\x1aUn\x1d\x04\
        SF:0caching_sha2_password\0!\0\0\x01\xff\x84\x04#08S01Got\x20packets\x20ou
        SF:t\x20of\x20order")%r(DNSStatusRequestTCP,73,"J\0\0\0\n8\.0\.32\0/\0\0\0
        SF:\]i\x0b\x15ubLQ\0\xff\xff\xff\x02\0\xff\xdf\x15\0\0\0\0\0\0\0\0\0\0P2<\
        SF:"\x7f8\x06x\x1ft\x19\x19\0caching_sha2_password\0!\0\0\x01\xff\x84\x04#
        SF:08S01Got\x20packets\x20out\x20of\x20order")%r(Help,73,"J\0\0\0\n8\.0\.3
        SF:2\x000\0\0\0TA6\x08M\?\x0fV\0\xff\xff\xff\x02\0\xff\xdf\x15\0\0\0\0\0\0
        SF:\0\0\0\x0008\x01\x7fb\+\x02~vj\x13\x1a\0caching_sha2_password\0!\0\0\x0
        SF:1\xff\x84\x04#08S01Got\x20packets\x20out\x20of\x20order")%r(SSLSessionR
        SF:eq,73,"J\0\0\0\n8\.0\.32\x001\0\0\0Bt@\x15Z\.m9\0\xff\xff\xff\x02\0\xff
        SF:\xdf\x15\0\0\0\0\0\0\0\0\0\0\x0e\x7f\x20\x0f\x0e\|C>Z%Zj\0caching_sha2_
        SF:password\0!\0\0\x01\xff\x84\x04#08S01Got\x20packets\x20out\x20of\x20ord
        SF:er");

        Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
        Nmap done: 1 IP address (1 host up) scanned in 16.29 seconds
    ```

- What database software is running on the database machine? What version?
  * mysql with version 8.0.32
- brute force database
  * `nmap --script mysql-brute -p 3306 172.30.0.15`-> no valid accounts found
  * `git clone https://github.com/danielmiessler/SecLists.git`
  * `hydra -l toor -P SecLists/Passwords/2020-200_most_used_passwords.txt 172.30.0.15 mysql`
  * toor:summer
- What webserver software is running on web?
  * Apache httpd 2.4.53 ((AlmaLinux))
- Does scanning the DC show you the name of the domain?
  * yes, Domain: insecure.cyb0., Site: Default-First-Site-Name
- Try the -sC option with nmap on the windows 10. What is this option?
  * `nmap -sC 172.30.10.100`
  * runs a set of default scripts against the target
  * ```bash
        osboxes@osboxes:~$ nmap -sC 172.30.10.100
        Starting Nmap 7.80 ( https://nmap.org ) at 2024-08-03 08:43 EDT
        Nmap scan report for 172.30.10.100
        Host is up (0.00025s latency).
        Not shown: 995 closed ports
        PORT     STATE SERVICE
        22/tcp   open  ssh
        | ssh-hostkey:
        |   3072 96:56:49:e7:1d:b0:1c:51:a2:3c:90:b2:f4:5d:6b:0a (RSA)
        |   256 a2:a6:04:d3:5d:bb:07:5f:58:bd:aa:7b:7b:db:7b:9b (ECDSA)
        |_  256 e1:21:5c:82:62:5e:cf:fd:b0:da:54:99:74:33:32:05 (ED25519)
        135/tcp  open  msrpc
        139/tcp  open  netbios-ssn
        445/tcp  open  microsoft-ds
        3389/tcp open  ms-wbt-server
        | rdp-ntlm-info:
        |   Target_Name: insecure
        |   NetBIOS_Domain_Name: insecure
        |   NetBIOS_Computer_Name: WIN10
        |   DNS_Domain_Name: insecure.cyb
        |   DNS_Computer_Name: win10.insecure.cyb
        |   Product_Version: 10.0.19041
        |_  System_Time: 2024-08-03T12:43:48+00:00
        | ssl-cert: Subject: commonName=win10.insecure.cyb
        | Not valid before: 2024-06-06T22:53:30
        |_Not valid after:  2024-12-06T22:53:30
        |_ssl-date: 2024-08-03T12:43:48+00:00; 0s from scanner time.

        Host script results:
        | smb2-security-mode:
        |   2.02:
        |_    Message signing enabled but not required
        | smb2-time:
        |   date: 2024-08-03T12:43:50
        |_  start_date: N/A

        Nmap done: 1 IP address (1 host up) scanned in 17.91 seconds
    ```

- Try to SSH (using vagrant/vagrant) from red to another machine. Is this possible?
  * yes

## Network Segmentation
- What is meant with the term 'attack vector'?
  * A way for attackers to enter a network or system
- Is there already network segmentation done on the company network?
  * no
- Remember what a DMZ is? What machines would be in the DMZ in this environment?
  * A DMZ is a subnet that is exposed to the internet but isolated from the internal network. It is used to host public-facing services (e.g., web servers) while minimizing exposure to the internal network.
  * Web and the companyrouter would be in the DMZ
- What could be a disadvantage of using network segmentation in this case? Tip: win10 <-> dc interaction.
  * Reduced Accessibility: For example, if win10 and dc are into different subnets, they might have difficulty interacting unless specific firewall rules or routing configurations are set up.

DNS record toevoegen:
- `sudo nano /etc/hosts`
```
172.30.20.8         www.insecure.cyb
172.30.20.8:8000    www.insecure.cyb/cmd
```
###### ip aanpassen alma en windows
- `sudo vi eth0.nmconnection`
- `sudo nano /etc/network/interfaces` (Wss deze)
- `sudo nmcli con mod eth0 ipv4.addresses 172.30.0.15/24`
- `sudo nmcli con modify eth1 ipv4.method manual`
- `sudo nmcli con down eth1`
- `sudo nmcli con up eth1`
- `New-NetIPAddress -InterfaceAlias Ethernet0 -IPAddress 172.16.1.2 -PrefixLength 24 -DefaultGateway 172.16.1.1`
###### DNS managen windows
- `dnscmd /enumrecords <ZoneName> @ /type A`
- `dnscmd /recorddelete <ZoneName> <NodeName> A`
- `dnscmd /recordadd <ZoneName> <NodeName> A <IPAddress>`
##### subnetten
- op companyrouter: firewall rule dat alles van isprouter naar subnets accepteert en dan rule dat alles van 'fake internet' netwerk blokkeert
- 
        sudo iptables -A FORWARD -s 192.168.100.254 -d 172.30.0.0/24 -j ACCEPT
        sudo iptables -A FORWARD -s 192.168.100.254 -d 172.30.20.0/24 -j ACCEPT
        sudo iptables -A FORWARD -s 192.168.100.254 -d 172.30.100.0/24 -j ACCEPT

- 
        sudo iptables -A FORWARD -s 192.168.100.0/24 -d 172.30.0.0/24 -j DROP
        sudo iptables -A FORWARD -s 192.168.100.0/24 -d 172.30.100.0/24 -j DROP 
- nieuwe A record in /etc/hosts om naar web te surfen van red
```
osboxes@osboxes:~$ sudo cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       osboxes
172.30.20.10    www.insecure.cyb
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
```
insecure\vagrant@DC C:\Users\vagrant>dnscmd /enumrecords insecure.cyb @ /type A

Returned records:
@                [Aging:3705469] 600 A  172.30.0.4
companyrouter            3600 A 172.30.0.254
database                 3600 A 172.30.0.15
db               3600 A 172.30.0.15
dc               3600 A 172.30.0.4
isprouter                3600 A 192.168.100.254
red              3600 A 192.168.100.102
web              3600 A 172.30.20.10
win10            [Aging:3713110] 1200 A 172.30.100.100
www              3600 A 172.30.20.10

Command completed successfully.
```
## DNS companyrouter
```
[vagrant@companyrouter ~]$ sudo nmcli con modify "Wired connection 1" ipv4.dns "172.30.0.4"
[vagrant@companyrouter ~]$ sudo nmcli con modify "Wired connection 1" ipv4.ignore-auto-dns yes
[vagrant@companyrouter ~]$ sudo nmcli con down 'Wired connection 1'
Connection 'Wired connection 1' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)
[vagrant@companyrouter ~]$ sudo nmcli con up 'Wired connection 1'
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
```
- dns server aanpassen en persistent maken door dns line aan te passen in `/etc/NetworkManager/system-connections/`
## Open, closed, filtered ports
- Open = actively accepting connections this indicates that a service or application is running and listening on that port
- Closed = reachable but not accepting connections,no service is listening on that port, but the port is not blocked or filtered
- filtered = nmap cannot determine if the port is open or closed, firewall or other network device is blocking the scan













## DNS
View Existing A Records:

Before making changes, you might want to view the existing DNS A records in the zone.
Use the following command to list all A records in a specific DNS zone:
cmd
```
dnscmd /enumrecords <ZoneName> @ /type A
```
Replace <ZoneName> with the name of your DNS zone (e.g., example.com).

Delete the Existing A Record:

If you need to change an existing A record, you'll first need to delete the old record:
cmd
```
dnscmd /recorddelete <ZoneName> <NodeName> A
```
Replace <ZoneName> with your DNS zone name and <NodeName> with the name of the host (e.g., www if the FQDN is www.example.com).

Add a New A Record:

After deleting the old record, you can add a new A record with the updated IP address:
cmd
```
dnscmd /recordadd <ZoneName> <NodeName> A <IPAddress>
```
```
dnscmd /zonerefresh insecure.cyb
dnscmd /zoneload insecure.cyb
dnscmd /zoneupdatefromds insecure.cyb
```