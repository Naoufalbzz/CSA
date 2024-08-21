# Lab 12
## Ansible installation
- `sudo dnf update -y`
- `sudo dnf install -y python3-pip`
- `sudo pip3 install ansible`
```bash
[vagrant@companyrouter ~]$ ansible --version
ansible [core 2.15.12]
  config file = None
  configured module search path = ['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.9/site-packages/ansible
  ansible collection location = /home/vagrant/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.9.18 (main, Jul  3 2024, 00:00:00) [GCC 11.4.1 20231218 (Red Hat 11.4.1-3)] (/usr/bin/python3)
  jinja version = 3.1.4
  libyaml = True
```
- `ssh-keygen -t rsa -b 2048 -f ~/.ssh/ansible_key -N ""` (-N voor lege passphrase)
Op Linux hosts:
- `sudo adduser ansible`
- `sudo passwd ansible`
- ansible:root -> creds nieuwe ansible user
- `sudo usermod -aG wheel ansible` (toevoegen aan sudo groep)
Op Windows host:
- `New-LocalUser -Name "ansible" -Password (ConvertTo-SecureString -AsPlainText "root" -Force) -FullName "ansible"`
- `Add-LocalGroupMember -Group "Administrators" -Member "ansible"`
```bash
winrm quickconfig
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
```
IP veranderen en voor ELKE host doen voor ansible:
- `ssh-copy-id -i ~/.ssh/ansible_key.pub ansible@172.30.0.6`
Op `companyrouter` om met windows host te werken:
- `pip install "pywinrm>=0.2.2"`
Ansible folder:
- `mkdir ansible`
- `sudo nano inventory`
```bash
[companyrouter]
localhost ansible_connection=local

[dc]
172.30.0.4:5985

[dc:vars]
ansible_user=vagrant
ansible_password=vagrant
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore

[web]
172.30.20.8

[database]
172.30.0.15

[linux]
localhost ansible_connection=local
172.30.20.8
172.30.0.15

[windowsclients]
172.30.100.100:5985

[windowsclients:vars]
ansible_user=vagrant
ansible_password=vagrant
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

db en web (linux groep zonder company want loopback) manueel ssh pub key in authorized_keys zetten
```bash
[vagrant@companyrouter ansible]$ ansible -i inventory -m "win_ping" dc
172.30.0.4 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
[vagrant@companyrouter ansible]$ ansible -i inventory -m "win_shell" -a "hostname" dc
172.30.0.4 | CHANGED | rc=0 >>
dc

[vagrant@companyrouter ansible]$ ansible -i inventory -m "ping" linux
localhost | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
172.30.20.8 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
172.30.0.15 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

1. Run an ad-hoc ansible command to check if the date of all machines are configured the same. Are you able to use the same Windows module for Linux machines and vice versa?
```bash
[vagrant@companyrouter ansible]$ ansible -i inventory web -m command -a "date"
172.30.20.8 | CHANGED | rc=0 >>
Wed Aug 21 01:00:33 UTC 2024
```
```bash
[vagrant@companyrouter ansible]$ ansible -i inventory dc -m win_command -a "powershell.exe Get-Date"
172.30.0.4 | CHANGED | rc=0 >>

Wednesday, August 21, 2024 3:05:09 AM
```
2. Create a playbook (or ad-hoc command) that pulls all "/etc/passwd" files from all Linux machines locally to the ansible controller node for every machine seperately.
```bash
[vagrant@companyrouter ansible]$ ansible -i inventory web -m fetch -a "src=/etc/passwd dest=/home/vagra
nt/ flat=yes"
172.30.20.8 | SUCCESS => {
    "changed": false,
    "checksum": "35c28d5c23cc529ce2c5a99cf1d0569c51dd7c8c",
    "dest": "/home/vagrant/passwd",
    "file": "/etc/passwd",
    "md5sum": "ebc409b62e1ee73131289b1fdb4d0b52"
}
```
3. Create a playbook (or ad-hoc command) that creates the user "walt" with password "Friday13th!" on all Linux machines.
```bash
[vagrant@companyrouter ansible]$ ansible-playbook -i inventory drie.yml

PLAY [Create User Walt] *******************************************************************************

TASK [Gathering Facts] ********************************************************************************
ok: [172.30.20.8]

TASK [Create user walt] *******************************************************************************
[DEPRECATION WARNING]: Encryption using the Python crypt module is deprecated. The Python crypt module
 is deprecated and will be removed from Python 3.13. Install the passlib library for continued
encryption functionality. This feature will be removed in version 2.17. Deprecation warnings can be
disabled by setting deprecation_warnings=False in ansible.cfg.
changed: [172.30.20.8]

PLAY RECAP ********************************************************************************************
172.30.20.8                : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
playbook `drie.yml`:
```yml
---
- name: Create User Walt
  hosts: web
  become: yes
  tasks:
    - name: Create user walt
      user:
        name: walt
        password: "{{ 'Friday13th!' | password_hash('sha512') }}"
        state: present
```
4. Create a playbook (or ad-hoc command) that pulls all users that are allowed to log in on all Linux machines.
```bash
[vagrant@companyrouter ansible]$ ansible-playbook -i inventory vier.yml

PLAY [Retrieve all users that can login on all linux machines] *********************************************************

TASK [Gathering Facts] *************************************************************************************************
ok: [172.30.20.8]

TASK [Get allowed login users] *****************************************************************************************
ok: [172.30.20.8]

TASK [Display allowed login users] *************************************************************************************
ok: [172.30.20.8] => {
    "allowed_users.stdout_lines": [
        "root",
        "vagrant",
        "ansible",
        "walt"
    ]
}

PLAY RECAP *************************************************************************************************************
172.30.20.8                : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
```yml
---
- name: Retrieve all users that can login on all linux machines
  hosts: web
  become: yes
  tasks:
    - name: Get allowed login users
      shell: "grep '/bin/bash' /etc/passwd | cut -d: -f1"
      register: allowed_users
      changed_when: false

    - name: Display allowed login users
      debug:
        var: allowed_users.stdout_lines
```
5. Create a playbook (or ad-hoc command) that calculates the hash (md5sum for example) of a binary (for example the ss binary).
```bash
[vagrant@companyrouter ansible]$ ansible-playbook -i inventory vijf.yml

PLAY [Calculate Hash of Binary] ****************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [172.30.20.8]

TASK [Get md5sum of ss binary] *****************************************************************************
changed: [172.30.20.8]

TASK [Display binary hash] *********************************************************************************
ok: [172.30.20.8] => {
    "msg": "The md5sum of ss is 0e29cedd88229b39df3dd5b636344782  /usr/sbin/ss"
}

PLAY RECAP *************************************************************************************************
172.30.20.8                : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
```yml
---
- name: Calculate Hash of Binary
  hosts: web
  tasks:
    - name: Get md5sum of ss binary
      command: md5sum /usr/sbin/ss
      register: binary_hash

    - name: Display binary hash
      debug:
        msg: "The md5sum of ss is {{ binary_hash.stdout }}"
```
6. Create a playbook (or ad-hoc command) that shows if Windows Defender is enabled and if there are any folder exclusions configured on the Windows client. This might require a bit of searching on how to retrieve this information through a command/PowerShell.
```bash
[vagrant@companyrouter ansible]$ ansible-playbook -i inventory zes.yml

PLAY [Check Windows Defender] ******************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [172.30.0.4]

TASK [Check if Windows Defender is enabled] ****************************************************************
changed: [172.30.0.4]

TASK [Get folder exclusions] *******************************************************************************
changed: [172.30.0.4]

TASK [Display Windows Defender status and exclusions] ******************************************************
ok: [172.30.0.4] => {
    "msg": "Windows Defender Status:\n\r\nDisableRealtimeMonitoring\r\n-------------------------\r\n                    False\r\n\r\n\r\n\n\nFolder Exclusions:\n\r\nExclusionPath    \r\n-------------    \r\n{c:\\, C:\\Windows}\r\n\r\n\r\n"
}

PLAY RECAP *************************************************************************************************
172.30.0.4                 : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
```yml
---
- name: Check Windows Defender
  hosts: dc
  tasks:
    - name: Check if Windows Defender is enabled
      win_shell: |
        Get-MpPreference | Select-Object -Property DisableRealtimeMonitoring
      register: defender_status

    - name: Get folder exclusions
      win_shell: |
        Get-MpPreference | Select-Object -Property ExclusionPath
      register: folder_exclusions

    - name: Display Windows Defender status and exclusions
      debug:
        msg: |
          Windows Defender Status:
          {{ defender_status.stdout }}

          Folder Exclusions:
          {{ folder_exclusions.stdout }}
```
7. Create a playbook (or ad-hoc command) that copies a file (for example a txt file) from the ansible controller machine to all Linux machines.
```bash
[vagrant@web ~]$ ls
authorized_keys  backup.sh  borg-keyfile.txt  important-files  noges  test.txt  testtt  testwazuyh
[vagrant@web ~]$ cat test.txt
test123
```
```bash
[vagrant@companyrouter ansible]$ ansible-playbook -i inventory zeven.yml

PLAY [Copy File to Linux Machines] *************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [172.30.20.8]

TASK [Copy text file] **************************************************************************************
changed: [172.30.20.8]

PLAY RECAP *************************************************************************************************
172.30.20.8                : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
```yml
---
- name: Copy File to Linux Machines
  hosts: web
  become: yes
  tasks:
    - name: Copy text file
      copy:
        src: /home/vagrant/test.txt
        dest: /home/vagrant/test.txt
```
8. Create the same as 7 but for Windows machines.
```bash
PS C:\Users\vagrant> ls


    Directory: C:\Users\vagrant


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         9/20/2023   1:21 PM                .ssh
d-r---         9/20/2023   1:21 PM                3D Objects
d-r---         9/20/2023   1:21 PM                Contacts
d-r---         9/20/2023   1:21 PM                Desktop
d-r---         7/24/2024   3:52 PM                Documents
d-r---         9/20/2023   1:21 PM                Downloads
d-r---         9/20/2023   1:21 PM                Favorites
d-r---         9/20/2023   1:21 PM                Links
d-r---         9/20/2023   1:21 PM                Music
d-r---         9/20/2023   1:21 PM                Pictures
d-r---         9/20/2023   1:21 PM                Saved Games
d-r---         9/20/2023   1:21 PM                Searches
d-r---         9/20/2023   1:21 PM                Videos
-a----         8/21/2024   2:08 AM        2266755 get-pip.py
-a----          8/1/2024  10:48 PM            576 key1_rsa.pub
-a----         8/21/2024   2:12 AM       25932664 python-installer.exe
-a----         8/21/2024   3:47 AM              8 test.txt


PS C:\Users\vagrant> cat .\test.txt
test123
```
```bash
[vagrant@companyrouter ansible]$ ansible-playbook -i inventory acht.yml

PLAY [Copy File to Linux Machines] *************************************************************************

TASK [Gathering Facts] *************************************************************************************
ok: [172.30.0.4]

TASK [Copy text file] **************************************************************************************
changed: [172.30.0.4]

PLAY RECAP *************************************************************************************************
172.30.0.4                 : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
```yml
---
- name: Copy File to Linux Machines
  hosts: dc
#  become: yes
  tasks:
    - name: Copy text file
      copy:
        src: /home/vagrant/test.txt
        dest: C:\Users\vagrant\test.txt
```