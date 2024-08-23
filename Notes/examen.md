1. `ansible --version`
2. `ansible -i inventory -m "ping" web`
3. `ansible -i inventory -m "win_ping" dc`
4. 
```yml
---
- name: tijd linux
  hosts: web
  tasks:
    - name: get time
      command: date
      register: date_result

    - name: Print current date and time
      debug:
        msg: "{{ inventory_hostname }}: {{ date_result.stdout }}"
- name: tijd windows
  hosts: dc,windowsclients
  tasks:
    - name: get time
      win_command: powershell.exe Get-Date
      register: date_result

    - name: Print current date and time
      debug:
        msg: "{{ inventory_hostname }}: {{ date_result.stdout }}"
```
5. 