# Lab 10
## workathome config
- `sudo nmcli connection modify 'Wired connection 1' ipv4.addresses 192.168.100.104/24`
- `sudo nmcli connection modify 'Wired connection 1' ipv4.gateway "192.168.100.253"`
- `sudo nmcli connection down "Wired connection 1" && sudo nmcli connection up "Wired
connection 1"`
## Server software installation
```
[vagrant@companyrouter ~]$ sudo dnf install --assumeyes openvpn easy-rsa
```
- `openvpn --version`
- `sudo /usr/share/easy-rsa/3/easyrsa --version`
## Set up the PKI
- `cd`
- `sudo /usr/share/easy-rsa/3.1.6/easyrsa init-pki`

## Set up the CA
- `cd`
- `cd /usr/share/easy-rsa/3/`
- `sudo /usr/share/easy-rsa/3.1.6/easyrsa build-ca`

passphrase CA private key:root

name CA:CA_CYB
```bash
[vagrant@companyrouter ~]$ sudo ls /home/vagrant/pki
ca.crt           index.txt       inline  openssl-easyrsa.cnf  reqs     serial  vars.example
certs_by_serial  index.txt.attr  issued  private              revoked  vars
```

## Generate the server keys and certificate
- `sudo /usr/share/easy-rsa/3.1.6/easyrsa gen-req server nopass`

name server key: SERVERKEY (probeer `server`)
```bash
Notice
------
Private-Key and Public-Certificate-Request files created.
Your files are:
* req: /home/vagrant/pki/reqs/server.req
* key: /home/vagrant/pki/private/server.key
```
- `sudo /usr/share/easy-rsa/3.1.6/easyrsa sign-req server server`
```bash
Notice
------
Certificate created at:
* /home/vagrant/pki/issued/server.crt
```
Check server certificate using openssl:
```bash
[vagrant@companyrouter ~]$ sudo openssl verify -CAfile /home/vagrant/pki/ca.crt  /home/vagrant/pki/issued/server.crt
/home/vagrant/pki/issued/server.crt: OK
```
## Generate the client keys and certificate
- `sudo /usr/share/easy-rsa/3.1.6/easyrsa gen-req client nopass`
```bash
Notice
------
Private-Key and Public-Certificate-Request files created.
Your files are:
* req: /home/vagrant/pki/reqs/client.req
* key: /home/vagrant/pki/private/client.key
```
- `sudo /usr/share/easy-rsa/3.1.6/easyrsa sign-req client client`
```bash
Notice
------
Certificate created at:
* /home/vagrant/pki/issued/client.crt
```
check client certificate:
```bash
[vagrant@companyrouter ~]$ sudo openssl verify -CAfile /home/vagrant/pki/ca.crt  /home/vagrant/pki/issued/client.crt
/home/vagrant/pki/issued/client.crt: OK
```
## Generate the Diffie-Hellman parameters
- `sudo /usr/share/easy-rsa/3.1.6/easyrsa gen-dh`
```bash
DH parameters appear to be ok.

Notice
------

DH parameters of size 2048 created at:
* /home/vagrant/pki/dh.pem
```
check all files:
```bash
[vagrant@companyrouter ~]$ sudo ls -a -l /home/vagrant/pki/ca.crt
-rw------- 1 root root 1184 Aug 20 00:34 /home/vagrant/pki/ca.crt
[vagrant@companyrouter ~]$ sudo ls -a -l /home/vagrant/pki/dh.pem
-rw------- 1 root root 424 Aug 20 01:08 /home/vagrant/pki/dh.pem
[vagrant@companyrouter ~]$ sudo ls -a -l /home/vagrant/pki/issued
total 20
drwx------ 2 root root   42 Aug 20 01:01 .
drwx------ 8 root root 4096 Aug 20 01:08 ..
-rw------- 1 root root 4471 Aug 20 01:01 client.crt
-rw------- 1 root root 4601 Aug 20 00:54 server.crt
[vagrant@companyrouter ~]$ sudo ls -a -l /home/vagrant/pki/private
total 20
drwx------ 2 root root   79 Aug 20 01:00 .
drwx------ 8 root root 4096 Aug 20 01:08 ..
-rw------- 1 root root 1874 Aug 20 00:34 ca.key
-rw------- 1 root root 1704 Aug 20 01:00 client.key
-rw------- 1 root root 1704 Aug 20 00:52 server.key
-rw------- 1 root root 1704 Aug 20 00:49 server_name.key
[vagrant@companyrouter ~]$ sudo ls -a -l /home/vagrant/pki/reqs
total 16
drwx------ 2 root root   65 Aug 20 01:00 .
drwx------ 8 root root 4096 Aug 20 01:08 ..
-rw------- 1 root root  887 Aug 20 01:00 client.req
-rw------- 1 root root  891 Aug 20 00:52 server.req
-rw------- 1 root root  887 Aug 20 00:50 server_name.req
```
```
/home/vagrant/pki/ca.crt
Type: Certificate
Description: This is the Certificate Authority (CA) certificate. It is used to sign server and client certificates and to verify their authenticity. All clients and servers need this to establish trust.
```
```
/home/vagrant/pki/dh.pem
Type: Diffie-Hellman Parameters
Description: This file contains the Diffie-Hellman (DH) parameters used to establish a shared secret between the server and clients. It is essential for the secure key exchange process in the OpenVPN connection.
```
```
/home/vagrant/pki/issued/
Description: Directory containing issued certificates.
Files:
client.crt: Client certificate. This certificate is used by the client to authenticate itself to the server.
server.crt: Server certificate. This certificate is used by the server to authenticate itself to clients.
```
```
/home/vagrant/pki/private/
Description: Directory containing private keys.
Files:
ca.key: Private key of the CA. It is used to sign certificates and should be kept secure.
client.key: Private key corresponding to the client certificate. It is used by the client to authenticate itself securely.
server.key: Private key corresponding to the server certificate. It is used by the server to authenticate itself securely.
```
```
/home/vagrant/pki/reqs/
Description: Directory containing certificate signing requests (CSRs).
Files:
client.req: Certificate signing request for the client certificate. This file is generated by the client and sent to the CA for signing.
server.req: Certificate signing request for the server certificate. This file is generated by the server and sent to the CA for signing.
```

## Configure the server
Op `companyrouter`:
- `sudo cp /usr/share/doc/openvpn/sample/sample-config-files/server.conf /etc/openvpn/server/`
- `sudo nano /etc/openvpn/server/server.conf`
```bash
#################################################
# Sample OpenVPN 2.0 config file for            #
# multi-client server.                          #
#                                               #
# This file is for the server side              #
# of a many-clients <-> one-server              #
# OpenVPN configuration.                        #
#                                               #
# OpenVPN also supports                         #
# single-machine <-> single-machine             #
# configurations (See the Examples page         #
# on the web site for more info).               #
#                                               #
# This config should work on Windows            #
# or Linux/BSD systems.  Remember on            #
# Windows to quote pathnames and use            #
# double backslashes, e.g.:                     #
# "C:\\Program Files\\OpenVPN\\config\\foo.key" #
#                                               #
# Comments are preceded with '#' or ';'         #
#################################################

# Which local IP address should OpenVPN
# listen on? (optional)
local 192.168.100.253

# Which TCP/UDP port should OpenVPN listen on?
# If you want to run multiple OpenVPN instances
# on the same machine, use a different port
# number for each one.  You will need to
# open up this port on your firewall.
port 1194

# TCP or UDP server?
;proto tcp
proto udp

# "dev tun" will create a routed IP tunnel,
# "dev tap" will create an ethernet tunnel.
# Use "dev tap0" if you are ethernet bridging
# and have precreated a tap0 virtual interface
# and bridged it with your ethernet interface.
# If you want to control access policies
# over the VPN, you must create firewall
# rules for the the TUN/TAP interface.
# On non-Windows systems, you can give
# an explicit unit number, such as tun0.
# On Windows, use "dev-node" for this.
# On most systems, the VPN will not function
# unless you partially or fully disable
# the firewall for the TUN/TAP interface.
;dev tap
dev tun

# Windows needs the TAP-Win32 adapter name
# from the Network Connections panel if you
# have more than one.  On XP SP2 or higher,
# you may need to selectively disable the
# Windows firewall for the TAP adapter.
# Non-Windows systems usually don't need this.
;dev-node MyTap

# SSL/TLS root certificate (ca), certificate
# (cert), and private key (key).  Each client
# and the server must have their own cert and
# key file.  The server and all clients will
# use the same ca file.
#
# See the "easy-rsa" directory for a series
# of scripts for generating RSA certificates
# and private keys.  Remember to use
# a unique Common Name for the server
# and each of the client certificates.
#
# Any X509 key management system can be used.
# OpenVPN can also use a PKCS #12 formatted key file
# (see "pkcs12" directive in man page).
ca /home/vagrant/pki/ca.crt
cert /home/vagrant/pki/issued/server.crt
key /home/vagrant/pki/private/server.key  # This file should be kept secret

# Diffie hellman parameters.
# Generate your own with:
#   openssl dhparam -out dh2048.pem 2048
dh /home/vagrant/pki/dh.pem

# Network topology
# Should be subnet (addressing via IP)
# unless Windows clients v2.0.9 and lower have to
# be supported (then net30, i.e. a /30 per client)
# Defaults to net30 (not recommended)
;topology subnet

# Configure server mode and supply a VPN subnet
# for OpenVPN to draw client addresses from.
# The server will take 10.8.0.1 for itself,
# the rest will be made available to clients.
# Each client will be able to reach the server
# on 10.8.0.1. Comment this line out if you are
# ethernet bridging. See the man page for more info.
server 10.8.0.0 255.255.255.0

# Maintain a record of client <-> virtual IP address
# associations in this file.  If OpenVPN goes down or
# is restarted, reconnecting clients can be assigned
# the same virtual IP address from the pool that was
# previously assigned.
ifconfig-pool-persist ipp.txt

# Configure server mode for ethernet bridging.
# You must first use your OS's bridging capability
# to bridge the TAP interface with the ethernet
# NIC interface.  Then you must manually set the
# IP/netmask on the bridge interface, here we
# assume 10.8.0.4/255.255.255.0.  Finally we
# must set aside an IP range in this subnet
# (start=10.8.0.50 end=10.8.0.100) to allocate
# to connecting clients.  Leave this line commented
# out unless you are ethernet bridging.
;server-bridge 10.8.0.4 255.255.255.0 10.8.0.50 10.8.0.100

# Configure server mode for ethernet bridging
# using a DHCP-proxy, where clients talk
# to the OpenVPN server-side DHCP server
# to receive their IP address allocation
# and DNS server addresses.  You must first use
# your OS's bridging capability to bridge the TAP
# interface with the ethernet NIC interface.
# Note: this mode only works on clients (such as
# Windows), where the client-side TAP adapter is
# bound to a DHCP client.
;server-bridge

# Push routes to the client to allow it
# to reach other private subnets behind
# the server.  Remember that these
# private subnets will also need
# to know to route the OpenVPN client
# address pool (10.8.0.0/255.255.255.0)
# back to the OpenVPN server.
push "route 172.30.0.0 255.255.0.0"
;push "route 192.168.20.0 255.255.255.0"

# To assign specific IP addresses to specific
# clients or if a connecting client has a private
# subnet behind it that should also have VPN access,
# use the subdirectory "ccd" for client-specific
# configuration files (see man page for more info).

# EXAMPLE: Suppose the client
# having the certificate common name "Thelonious"
# also has a small subnet behind his connecting
# machine, such as 192.168.40.128/255.255.255.248.
# First, uncomment out these lines:
;client-config-dir ccd
;route 192.168.40.128 255.255.255.248
# Then create a file ccd/Thelonious with this line:
#   iroute 192.168.40.128 255.255.255.248
# This will allow Thelonious' private subnet to
# access the VPN.  This example will only work
# if you are routing, not bridging, i.e. you are
# using "dev tun" and "server" directives.

# EXAMPLE: Suppose you want to give
# Thelonious a fixed VPN IP address of 10.9.0.1.
# First uncomment out these lines:
;client-config-dir ccd
;route 10.9.0.0 255.255.255.252
# Then add this line to ccd/Thelonious:
#   ifconfig-push 10.9.0.1 10.9.0.2

# Suppose that you want to enable different
# firewall access policies for different groups
# of clients.  There are two methods:
# (1) Run multiple OpenVPN daemons, one for each
#     group, and firewall the TUN/TAP interface
#     for each group/daemon appropriately.
# (2) (Advanced) Create a script to dynamically
#     modify the firewall in response to access
#     from different clients.  See man
#     page for more info on learn-address script.
;learn-address ./script

# If enabled, this directive will configure
# all clients to redirect their default
# network gateway through the VPN, causing
# all IP traffic such as web browsing and
# and DNS lookups to go through the VPN
# (The OpenVPN server machine may need to NAT
# or bridge the TUN/TAP interface to the internet
# in order for this to work properly).
;push "redirect-gateway def1 bypass-dhcp"

# Certain Windows-specific network settings
# can be pushed to clients, such as DNS
# or WINS server addresses.  CAVEAT:
# http://openvpn.net/faq.html#dhcpcaveats
# The addresses below refer to the public
# DNS servers provided by opendns.com.
;push "dhcp-option DNS 208.67.222.222"
;push "dhcp-option DNS 208.67.220.220"

# Uncomment this directive to allow different
# clients to be able to "see" each other.
# By default, clients will only see the server.
# To force clients to only see the server, you
# will also need to appropriately firewall the
# server's TUN/TAP interface.
;client-to-client

# Uncomment this directive if multiple clients
# might connect with the same certificate/key
# files or common names.  This is recommended
# only for testing purposes.  For production use,
# each client should have its own certificate/key
# pair.
#
# IF YOU HAVE NOT GENERATED INDIVIDUAL
# CERTIFICATE/KEY PAIRS FOR EACH CLIENT,
# EACH HAVING ITS OWN UNIQUE "COMMON NAME",
# UNCOMMENT THIS LINE OUT.
;duplicate-cn

# The keepalive directive causes ping-like
# messages to be sent back and forth over
# the link so that each side knows when
# the other side has gone down.
# Ping every 10 seconds, assume that remote
# peer is down if no ping received during
# a 120 second time period.
keepalive 10 120

# For extra security beyond that provided
# by SSL/TLS, create an "HMAC firewall"
# to help block DoS attacks and UDP port flooding.
#
# Generate with:
#   openvpn --genkey tls-auth ta.key
#
# The server and each client must have
# a copy of this key.
# The second parameter should be '0'
# on the server and '1' on the clients.
;tls-auth ta.key 0 # This file is secret

# Select a cryptographic cipher.
# This config item must be copied to
# the client config file as well.
# Note that v2.4 client/server will automatically
# negotiate AES-256-GCM in TLS mode.
# See also the ncp-cipher option in the manpage
cipher AES-256-CBC

# Enable compression on the VPN link and push the
# option to the client (v2.4+ only, for earlier
# versions see below)
;compress lz4-v2
;push "compress lz4-v2"

# For compression compatible with older clients use comp-lzo
# If you enable it here, you must also
# enable it in the client config file.
;comp-lzo

# The maximum number of concurrently connected
# clients we want to allow.
;max-clients 100

# It's a good idea to reduce the OpenVPN
# daemon's privileges after initialization.
#
# You can uncomment this out on
# non-Windows systems.
;user nobody
;group nobody

# The persist options will try to avoid
# accessing certain resources on restart
# that may no longer be accessible because
# of the privilege downgrade.
persist-key
persist-tun

# Output a short status file showing
# current connections, truncated
# and rewritten every minute.
status openvpn-status.log

# By default, log messages will go to the syslog (or
# on Windows, if running as a service, they will go to
# the "\Program Files\OpenVPN\log" directory).
# Use log or log-append to override this default.
# "log" will truncate the log file on OpenVPN startup,
# while "log-append" will append to it.  Use one
# or the other (but not both).
;log         openvpn.log
;log-append  openvpn.log

# Set the appropriate level of log
# file verbosity.
#
# 0 is silent, except for fatal errors
# 4 is reasonable for general usage
# 5 and 6 can help to debug connection problems
# 9 is extremely verbose
verb 3

# Silence repeating messages.  At most 20
# sequential messages of the same message
# category will be output to the log.
;mute 20

# Notify the client that when the server restarts so it
# can automatically reconnect.
explicit-exit-notify 1
```
- `sudo systemctl start openvpn
## Configure the client
On `workathome`:
- `sudo apt update -y`
- `sudo apt install openssh-server`
- `sudo apt install openvpn`
- `sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/client/`
- `sudo nano /etc/openvpn/client/client.conf`
```bash
##############################################
# Sample client-side OpenVPN 2.0 config file #
# for connecting to multi-client server.     #
#                                            #
# This configuration can be used by multiple #
# clients, however each client should have   #
# its own cert and key files.                #
#                                            #
# On Windows, you might want to rename this  #
# file so it has a .ovpn extension           #
##############################################

# Specify that we are a client and that we
# will be pulling certain config file directives
# from the server.
client

# Use the same setting as you are using on
# the server.
# On most systems, the VPN will not function
# unless you partially or fully disable
# the firewall for the TUN/TAP interface.
;dev tap
dev tun

# Windows needs the TAP-Win32 adapter name
# from the Network Connections panel
# if you have more than one.  On XP SP2,
# you may need to disable the firewall
# for the TAP adapter.
;dev-node MyTap

# Are we connecting to a TCP or
# UDP server?  Use the same setting as
# on the server.
;proto tcp
proto udp

# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote 192.168.100.253 1194
;remote my-server-2 1194

# Choose a random host from the remote
# list for load-balancing.  Otherwise
# try hosts in the order specified.
;remote-random

# Keep trying indefinitely to resolve the
# host name of the OpenVPN server.  Very useful
# on machines which are not permanently connected
# to the internet such as laptops.
resolv-retry infinite

# Most clients don't need to bind to
# a specific local port number.
nobind

# Downgrade privileges after initialization (non-Windows only)
;user nobody
;group nobody

# Try to preserve some state across restarts.
persist-key
persist-tun

# If you are connecting through an
# HTTP proxy to reach the actual OpenVPN
# server, put the proxy server/IP and
# port number here.  See the man page
# if your proxy server requires
# authentication.
;http-proxy-retry # retry on connection failures
;http-proxy [proxy server] [proxy port #]

# Wireless networks often produce a lot
# of duplicate packets.  Set this flag
# to silence duplicate packet warnings.
;mute-replay-warnings

# SSL/TLS parms.
# See the server config file for more
# description.  It's best to use
# a separate .crt/.key file pair
# for each client.  A single ca
# file can be used for all clients.
ca /home/osboxes/ca.crt
cert /home/osboxes/client.crt
key /home/osboxes/client.key

# Verify server certificate by checking that the
# certificate has the correct key usage set.
# This is an important precaution to protect against
# a potential attack discussed here:
#  http://openvpn.net/howto.html#mitm
#
# To use this feature, you will need to generate
# your server certificates with the keyUsage set to
#   digitalSignature, keyEncipherment
# and the extendedKeyUsage to
#   serverAuth
# EasyRSA can do this for you.
remote-cert-tls server

# If a tls-auth key is used on the server
# then every client must also have the key.
;tls-auth ta.key 1

# Select a cryptographic cipher.
# If the cipher option is used on the server
# then you must also specify it here.
# Note that v2.4 client/server will automatically
# negotiate AES-256-GCM in TLS mode.
# See also the data-ciphers option in the manpage
cipher AES-256-CBC

# Enable compression on the VPN link.
# Don't enable this unless it is also
# enabled in the server config file.
#comp-lzo

# Set log file verbosity.
verb 3

# Silence repeating messages
;mute 20

# allow username password when connecting to VPN
auth-user-pass
```
On `companyrouter`:
```bash
[vagrant@companyrouter ~]$ sudo openvpn /etc/openvpn/server/server.conf
```
On `workathome` (vagrant:vagrant):
```bash
osboxes@osboxes:~$ sudo openvpn /etc/openvpn/client/client.conf
```
- `curl http://172.30.20.8`
```bash
osboxes@osboxes:~$ curl http://172.30.20.8
<!DOCTYPE html>
<html>
<head>
    <title>Insecure Cyb</title>
</head>
<body>

<h1>Welcome to Insecure.cyb!</h1>
<img src="https://www.eff.org/files/banner_library/http_warning.png">

<h2>We learn what it means to be not secure :) </h2>

    <br />
<b>Warning</b>:  mysqli::__construct(): (HY000/2002): No route to host in <b>/var/www/html/index.php</b> on line <b>26</b><br />
<p>Error: Could not connect to the database</p>
    <!-- HTML Form -->
    <form method="POST" action="">
        <label for="login">Login:</label>
        <input type="text" name="login" id="login" />
        <input type="submit" value="Search" />
    </form>
</body>
</html>
```
On `kali`:
- `sudo ettercap -Tq -i eth0 -M arp:remote /192.168.100.104// /192.168.100.253//`
- Open `Wireshark`:
![openvpn](/img/openvpn.png)

`auth-user-pass` in client.conf so that we can use a user from `companyrouter` (vagrant:vagrant)

`PAM` (Pluggable Authentication Modules): A framework used in Linux systems to manage authentication. It allows system administrators to define how different applications and services authenticate users by stacking modules in a specific order. Each module performs a task like checking passwords, managing login attempts