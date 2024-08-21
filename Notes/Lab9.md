# Lab 9
## Wazuh VM
### VM config
- `sudo nano /etc/netplan/01-network-manager-all.yaml`
```yaml
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s3:
     addresses:
       - 172.30.0.6/24
     nameservers:
        addresses:
          - 172.30.0.4
```
- `sudo netplan apply`
- DNS aanpassen
- `sudo systemctl stop systemd-resolved`
- `sudo systemctl disable systemd-resolved`
- `sudo rm /etc/resolv.conf`
- `sudo nano /etc/resolv.conf` 
```yaml
nameserver 172.30.0.4
``` 
- `sudo systemctl restart NetworkManager`
### Wazuh installation
- `curl -sO https://packages.wazuh.com/4.8/wazuh-install.sh && sudo bash ./wazuh-install.sh -a`
```   
User: admin
Password: c?VYhhC0b6w6*l.lk9KF62NSqClSF+BU
```
- https://172.30.0.6:443


### Wazuh agents (web,database,companyrouter)

```bash
rpm --import https://packages.wazuh.com/key/GPG-KEY-WAZUH
```

```bash
cat > /etc/yum.repos.d/wazuh.repo << EOF
[wazuh]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=EL-\$releasever - Wazuh
baseurl=https://packages.wazuh.com/4.x/yum/
protect=1
EOF
```

```bash
WAZUH_MANAGER="172.30.0.6" yum install wazuh-agent
```

```bash
systemctl daemon-reload
systemctl enable wazuh-agent
systemctl start wazuh-agent
```

```bash
sed -i "s/^enabled=1/enabled=0/" /etc/yum.repos.d/wazuh.repo
```
- What is File Integrity Monitoring? Try to monitor the home directory of a user on a specific machine.
  - File Integrity Monitoring checks for changes to files and directories, such as modifications, deletions, or additions. It helps in detecting unauthorized changes and potential breaches
- Demo: 
  - `<directories realtime="yes">/root</directories>` toevoegen aan `/var/ossec/etc/ossec.conf` bij section `File integrity monitoring` onder `dir to check`
  - `sudo systemctl restart wazuh-agent`
  - `sudo touch /root/test`
  - Op wazuh server naar `https://172.30.0.6:443`->klik op web agent->klik op File integrity monitoring-> we zien dat er een file is aangemaakt
- What is meant with Regulatory Compliance? Give 2 frameworks that can be explored.
  - Ensures that systems and processes adhere to legal and regulatory requirements.
  - `GDPR` (General Data Protection Regulation): Europese wetgeving die gebaseerd op is het beschermen van de privacy een data van een individu.
  - `HIPAA` (Health Insurance Portability and Accountability Act): wetgeving die de gevoelige medische data van patiënten beschermt. 
  - `PCI-DSS` (Payment Card Industry Data Security Standard)
## Sysmon
- Download Sysmon op win10
- create `sysconfig.xml` in zelfde folder als Sysmon binaries:
```
<Sysmon schemaversion="4.10">
   <HashAlgorithms>md5</HashAlgorithms>
   <EventFiltering>
      <!--SYSMON EVENT ID 1 : PROCESS CREATION-->
      <ProcessCreate onmatch="include">
         <Image condition="contains">mimikatz.exe</Image>
      </ProcessCreate>
      <!--SYSMON EVENT ID 2 : FILE CREATION TIME RETROACTIVELY CHANGED IN THE FILESYSTEM-->
      <FileCreateTime onmatch="include" />
      <!--SYSMON EVENT ID 3 : NETWORK CONNECTION INITIATED-->
      <NetworkConnect onmatch="include" />
      <!--SYSMON EVENT ID 4 : RESERVED FOR SYSMON STATUS MESSAGES, THIS LINE IS INCLUDED FOR DOCUMENTATION PURPOSES ONLY-->
      <!--SYSMON EVENT ID 5 : PROCESS ENDED-->
      <ProcessTerminate onmatch="include" />
      <!--SYSMON EVENT ID 6 : DRIVER LOADED INTO KERNEL-->
      <DriverLoad onmatch="include" />
      <!--SYSMON EVENT ID 7 : DLL (IMAGE) LOADED BY PROCESS-->
      <ImageLoad onmatch="include" />
      <!--SYSMON EVENT ID 8 : REMOTE THREAD CREATED-->
      <CreateRemoteThread onmatch="include">
         <SourceImage condition="contains">mimikatz.exe</SourceImage>
      </CreateRemoteThread>
      <!--SYSMON EVENT ID 9 : RAW DISK ACCESS-->
      <RawAccessRead onmatch="include" />
      <!--SYSMON EVENT ID 10 : INTER-PROCESS ACCESS-->
      <ProcessAccess onmatch="include">
         <SourceImage condition="contains">mimikatz.exe</SourceImage>
      </ProcessAccess>
      <!--SYSMON EVENT ID 11 : FILE CREATED-->
      <FileCreate onmatch="include" />
      <!--SYSMON EVENT ID 12 & 13 & 14 : REGISTRY MODIFICATION-->
      <RegistryEvent onmatch="include" />
      <!--SYSMON EVENT ID 15 : ALTERNATE DATA STREAM CREATED-->
      <FileCreateStreamHash onmatch="include" />
      <PipeEvent onmatch="include" />
   </EventFiltering>
</Sysmon>
```
- met Admin rechten `Sysmon64.exe -accepteula -i sysconfig.xml`
- Download mimikatz op win10
- `cd C:\Users\vagrant\Downloads\mimikatz_trunk\x64`
- `.\mimikatz.exe`
- Open `Event viewer`->`Applications and Services Logs`-> `Microsoft`-> `Windows`->`Sysmon`->`Operational`
- ![image](../img/sysmon.png)
### Configure win10 als wazuh agent
```powershell
# Voorbeeld URL van de installer
$url = "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.8.0-1.msi"

# Bestemming voor het opslaan van de gedownloade installer
$output = "C:\Users\vagrant\Downloads\wazuh-agent-4.8.0-1.msi"

# Voer de download uit met Invoke-WebRequest
Invoke-WebRequest -Uri $url -OutFile $output
```
- `.\wazuh-agent-4.8.1-1.msi /q WAZUH_MANAGER="172.30.0.6"`
- `NET START Wazuh`
- In GUI -> `View` -> `View config` -> `<address>172.30.0.6</address>`
- `cd 'C:\Program Files (x86)\ossec-agent\'`
- `.\agent-auth.exe -m 172.30.0.6`
- Refresh op wazuh GUI
- `cd 'C:\Program Files (x86)\ossec-agent\'`
- `notepad .\ossec.conf`
```
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```
- `net stop wazuh`
- `net start wazuh`
- op wazuh manager:
- `sudo nano /var/ossec/etc/rules/local_rules.xml`
```
<group name="windows, sysmon, sysmon_process-anomalies,">
   <rule id="100000" level="12">
     <if_group>sysmon_event1</if_group>
     <field name="win.eventdata.image">mimikatz.exe</field>
     <description>Sysmon - Suspicious Process - mimikatz.exe</description>
   </rule>
   <rule id="100001" level="12">
     <if_group>sysmon_event8</if_group>
     <field name="win.eventdata.sourceImage">mimikatz.exe</field>
     <description>Sysmon - Suspicious Process mimikatz.exe created a remote thread</description>
   </rule>
   <rule id="100002" level="12">
     <if_group>sysmon_event_10</if_group>
     <field name="win.eventdata.sourceImage">mimikatz.exe</field>
     <description>Sysmon - Suspicious Process mimikatz.exe accessed $(win.eventdata.targetImage)</description>
   </rule>
</group>
```
- `sudo systemctl restart wazuh-manager`
```
C:\Users\Walt\Downloads\mimikatz_trunk\x64\
.\mimikatz.exe
```
![sysmon2](/img/sysmon2.png)
![sysmon3](/img/sysmon3.png)