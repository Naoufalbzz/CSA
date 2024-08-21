# Lab 4
## SSH Port Forwarding
- `ssh -L 8033:172.30.20.8:80 vagrant@companyrouter` -> http://localhost:8033/cmd
- `ssh -L 3309:172.30.0.15:3306 vagrant@companyrouter` -> mysql workbench
- combined: `ssh -L 8090:172.30.20.8:80 -L 3304:172.30.0.15:3306 vagrant@companyrouter`
- Why is this an interesting approach from a security point of view?
  -  It allows access to services behind firewalls and encrypts traffic, enhancing security.
-  When would you use local port forwarding?
   -  To securely access internal services from your local machine through an SSH server and if you want to access services that are in a private network protected by a firewall.
- When would you use remote port forwarding?
  - To make a local service accessible from a remote machine via an SSH server.
- Which of the two are more "common" in security?
  -  Local Port Forwarding: It is more commonly used to access internal services securely.
-  Some people call SSH port forwarding also a "poor man's VPN". Why?
   - It creates a secure tunnel for accessing services, similar to a VPN but simpler and typically involving fewer ports.  
## SSH Port Forwarding Options
- `ssh -L`: Forward a port on your local machine to a remote server
- `ssh -R`: Forward a port on a remote server to your local machine
- `ssh -J`: Connect to a final destination server through an intermediary SSH server (basically bastion host)

## IDS/IPS
- `sudo dnf install epel-release dnf-plugins-core`
- `sudo dnf copr enable @oisf/suricata-7.0`
- `sudo dnf install suricata`
- `/etc/suricata/suricata.yaml` aanpassen, `address-groups` en `af-packet` regel
- `sudo suricata -c /etc/suricata/suricata.yaml -i eth0`
- `tail -f /var/log/suricata/fast.log`
- in suricata.yaml rule-file regel aanpassen naar eigen file
- in rule file:`alert icmp 192.168.100.102 any -> 172.30.20.8 any (msg:"ICMP Ping detected from red to database"; itype:8; sid:1000001; rev:1;)` rule voor icmp van red naar web te capturen
- `alert icmp 192.168.100.102 any -> 172.30.0.5 any (msg:"ICMP Ping detected from red to database"; itype:8; sid:1000001; rev:1;)`

- Best Placement for IDS/IPS? 
  -  network router/gateway to monitor all incoming and outgoing traffic
- What traffic can be seen?
  - Traffic between all devices that pass through the IDS/IPS installation point
- What traffic (if any) will be missed and when?
  -  Traffic that does not pass through the monitoring point.
  -  If rules for specific attacks or traffic types are not set up, those threats may pass undetected
  -  Incorrectly written rules might either fail to trigger on actual threats or generate false positives
-  Verify that you see packets (in tcpdump) from red to the database. Try this by issuing a ping and by using the hydra mysql attack as seen previously. Are you able to see this traffic in tcpdump? What about a ping between the webserver and the database?
   -  `sudo tcpdump -i eth0 host 192.168.100.102 and host 172.30.0.15`
   -  yes, we can see the ping and hydra attack on tcpdump. 
   -  We can't see the pings from web to db
-  What is the difference between the fast.log and the eve.json files?
  - fast.log: A simple, line-by-line log of alerts, it’s quick to read and primarily used for speed and simplicity  
  - eve.json: A more detailed, structured log format in JSON, which can include various types of events beyond alerts, such as DNS requests, HTTP logs
- Create a rule that alerts as soon as a ping is performed between two machines (for example red and database)
  - `alert icmp 192.168.100.102 any -> 172.30.0.5 any (msg:"ICMP Ping detected from red to database"; itype:8; sid:1000001; rev:1;)`
- Test your out-of-the-box configuration and browse on your red machine to www.insecure.cyb/cmd and enter "id" as an evil command. Does it trigger an alert? If not are you able to make it trigger an alert?
  - `alert tcp 172.30.20.8 any -> 192.168.100.102 any (msg:"ET ATTACK_RESPONSE Output of id command from HTTP server"; flow:established; content:"uid="; pcre:"/^\d+[^\r\n\s]+/R"; content:" gid="; within:5; pcre:"/^\d+[^\r\n\s]+/R"; content:" groups="; within:8; classtype:bad-unknown; sid:2019284; rev:3; metadata:created_at 2014_09_26, updated_at 2019_07_26;)` 
- Create an alert that checks the mysql tcp port and rerun a hydra attack to check this rule. Can you visually see this bruteforce attack in the fast.log file? Tip: monitor the file live with an option of tail.
  - `alert tcp 172.30.0.15 3306 -> 192.168.100.102 any (msg:"MySQL "; flow:established,to_client; dsize:<251; byte_test:1,<,0xfb,0,little; content:"|ff 15 04 23 32 38 30 30 30|"; offset:4; threshold: type threshold, track by_src, count 5, seconds 120; classtype:attempted-recon; sid:2010494; rev:5; metadata:created_at 2010_07_30, updated_at 2024_03_06;)`
- Go have a look at the Suricata documentation. What is the default configuration of Suricata, is it an IPS or IDS?
  - By default, Suricata is set up in IDS mode
- What do you have to change to the setup to switch to the other (IPS or IDS)? You are free to experiment more and go all out with variables (for the networks) and rules. Make sure you can conceptually explain why certain rules would be useful and where (= from which subnet to which subnet) they should be applied?

  - To switch to IPS mode, you need to configure Suricata to operate in line mode, which involves editing the configuration file:
  
     ```yaml
      af-packet:
        - interface: eth0
          # Other settings...
          # Enable inline mode
          use-mmap: yes
          defrag: yes
          checksum-checks: yes
          ips-mode: true
      ```
  - Set the default-action for rules to drop or reject for IPS in `/etc/suricata/suricata.yaml`
- `sudo nano /var/log/suricata/suricata.log`
- `/etc/suricata/suricata.yaml`
- `tail -f /var/log/suricata/fast.log`
- `sudo nano /var/lib/suricata/rules/suricata2.rules`
- Traffic Patterns and Rule Placement:

  - Internal vs. External Traffic: Rules can be tailored based on whether traffic is internal (within the organization) or external (from outside). For example, blocking external IP ranges trying to access internal services.
  - Critical Assets: Apply stricter rules on subnets containing critical assets or sensitive data to enhance security where it’s most needed.
- Performance Implications:

  - IDS Mode: Involves less performance overhead as it only monitors and logs without intervening in traffic.
  - IPS Mode: Can introduce latency and requires careful tuning to avoid dropping legitimate traffic. Ensure the hardware and network setup can handle the traffic volume and processing requirements.
- What is the difference between an IPS and firewall?
  -  A firewall primarily acts as a barrier between a trusted internal network and untrusted external networks. It can block or allow traffic based on predefined security rules, such as IP addresses, ports, and protocols and won't inspect content of traffic deeply
  -  IPS goes beyond basic filtering by deeply inspecting the content of the traffic. It can detect and prevent specific patterns or behaviors that indicate an attack
-  Do you understand why Suricata can offer this protection whilst a firewall cannot?
   -   A firewall is more focused on filtering traffic based on IPs, ports, and protocols but does not deeply inspect the contents of the traffic or analyze the behavior of the traffic streams
-   On which layers of the OSI-model do they work?
    - Firewall: Operates mainly at Layer 3 (Network) and Layer 4 (Transport). It handles routing, IP filtering, and port/protocol-based rules.   
    - IPS (Suricata): Operates mainly at Layer 4 (Transport) and Layer 7 (Application). It inspects packet contents, reassembles sessions, and analyzes application-level data for malicious patterns.