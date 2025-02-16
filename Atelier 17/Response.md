## Exercice 12

Je démarre les VM et me connecte au Control Host :

```bash
vagrant up
vagrant ssh ansible
```
Je me rends dans le répertoire des playbooks :
```bash
cd ansible/projets/ema/playbooks/
ls -l
```

J'écris les playbooks :  
Playbook chrony-01.yml :

```bash
vim chrony-01.yml
```
```bash
--- # chrony-01.yml

- hosts: all

  tasks:
    - name: Install chrony on Debian/Ubuntu
      apt:
        name: chrony
        state: present
      when: ansible_os_family == "Debian"

    - name: Install chrony on Rocky Linux
      dnf:
        name: chrony
        state: present
      when: ansible_distribution == "Rocky"

    - name: Install chrony on SUSE Linux
      zypper:
        name: chrony
        state: present
      when: ansible_distribution == "openSUSE Leap"

    - name: Start and enable chronyd service on Debian/Ubuntu
      service:
        name: chrony
        state: started
        enabled: true
      when: ansible_os_family == "Debian"

    - name: Start and enable chronyd service on Rocky Linux
      service:
        name: chronyd
        state: started
        enabled: true
      when: ansible_distribution == "Rocky"

    - name: Start and enable chronyd service on SUSE Linux
      service:
        name: chronyd
        state: started
        enabled: true
      when: ansible_distribution == "openSUSE Leap"

    - name: Ensure /etc/chrony directory exists
      file:
        path: /etc/chrony
        state: directory
        mode: '0755'

    - name: Check if chrony.conf exists
      stat:
        path: /etc/chrony/chrony.conf
      register: chrony_conf

    - name: Create empty chrony.conf if it does not exist
      copy:
        dest: /etc/chrony/chrony.conf
        content: ""
      when: not chrony_conf.stat.exists

    - name: Backup original chrony.conf
      command: mv /etc/chrony/chrony.conf /etc/chrony/chrony.conf.bak
      args:
        creates: /etc/chrony/chrony.conf.bak
      when: chrony_conf.stat.exists

    - name: Install custom chrony configuration
      copy:
        dest: /etc/chrony/chrony.conf
        content: |
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Restart chronyd

  handlers:
    - name: Restart chronyd
      service:
        name: chronyd
        state: restarted
```

Playbook chrony-02.yml :
```bash
vim chrony-02.yml
```
```bash
---
# chrony-02.yml

- hosts: all

  vars:
    chrony_package: ""
    chrony_service: ""
    chrony_confdir: ""

  tasks:
    - name: Set variables for Debian/Ubuntu
      set_fact:
        chrony_package: chrony
        chrony_service: chrony
        chrony_confdir: /etc/chrony
      when: ansible_os_family == "Debian"

    - name: Set variables for Rocky Linux
      set_fact:
        chrony_package: chrony
        chrony_service: chronyd
        chrony_confdir: /etc/chrony
      when: ansible_distribution == "Rocky"

    - name: Set variables for SUSE Linux
      set_fact:
        chrony_package: chrony
        chrony_service: chronyd
        chrony_confdir: /etc/chrony
      when: ansible_distribution == "openSUSE Leap"

    - name: Install chrony package
      package:
        name: "{{ chrony_package }}"
        state: present

    - name: Start and enable chronyd service
      service:
        name: "{{ chrony_service }}"
        state: started
        enabled: true

    - name: Ensure chrony configuration directory exists
      file:
        path: "{{ chrony_confdir }}"
        state: directory
        mode: '0755'

    - name: Check if chrony.conf exists
      stat:
        path: "{{ chrony_confdir }}/chrony.conf"
      register: chrony_conf

    - name: Create empty chrony.conf if it does not exist
      copy:
        dest: "{{ chrony_confdir }}/chrony.conf"
        content: ""
      when: not chrony_conf.stat.exists

    - name: Backup original chrony.conf
      command: mv "{{ chrony_confdir }}/chrony.conf" "{{ chrony_confdir }}/chrony.conf.bak"
      args:
        creates: "{{ chrony_confdir }}/chrony.conf.bak"
      when: chrony_conf.stat.exists

    - name: Install custom chrony configuration
      copy:
        dest: "{{ chrony_confdir }}/chrony.conf"
        content: |
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Restart chronyd

  handlers:
    - name: Restart chronyd
      service:
        name: "{{ chrony_service }}"
        state: restarted
```

Playbook reset-chrony.yml (me permet de retester la conf de zéro):
```bash
vim reset-chrony.yml
```
```bash
---
# reset-chrony.yml

- hosts: all
  become: yes

  tasks:
    - name: Stop and disable chronyd service on Debian/Ubuntu
      service:
        name: chrony
        state: stopped
        enabled: no
      when: ansible_os_family == "Debian"
      ignore_errors: yes

    - name: Stop and disable chronyd service on Rocky Linux
      service:
        name: chronyd
        state: stopped
        enabled: no
      when: ansible_distribution == "Rocky"
      ignore_errors: yes

    - name: Stop and disable chronyd service on SUSE Linux
      service:
        name: chronyd
        state: stopped
        enabled: no
      when: ansible_distribution == "openSUSE Leap"
      ignore_errors: yes

    - name: Uninstall chrony on Debian/Ubuntu
      apt:
        name: chrony
        state: absent
        purge: yes
      when: ansible_os_family == "Debian"
      ignore_errors: yes

    - name: Uninstall chrony on Rocky Linux
      dnf:
        name: chrony
        state: absent
      when: ansible_distribution == "Rocky"
      ignore_errors: yes

    - name: Uninstall chrony on SUSE Linux
      zypper:
        name: chrony
        state: absent
      when: ansible_distribution == "openSUSE Leap"
      ignore_errors: yes

    - name: Remove chrony configuration directory
      file:
        path: /etc/chrony
        state: absent
      ignore_errors: yes

    - name: Remove chrony drift file
      file:
        path: /var/lib/chrony/drift
        state: absent
      ignore_errors: yes

    - name: Remove chrony log directory
      file:
        path: /var/log/chrony
        state: absent
      ignore_errors: yes
```

Je vérifie la syntaxe des playbooks: 
Playbook chrony-01.yml :
```bash
yamllint chrony-01.yml
```

Playbook chrony-02.yml :
```bash
yamllint chrony-02.yml
```

Playbook chrony-02.yml :
```bash
yamllint reset-chrony.yml
```

J'exécute les playbooks :
J'exécute le playbook chrony-01.yml :
```bash
ansible-playbook chrony-01.yml
```
```bash
ansible-playbook chrony-01.yml

PLAY [all] ******************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************
ok: [debian]
ok: [rocky]
ok: [ubuntu]
ok: [suse]

TASK [Install chrony on Debian/Ubuntu] ******************************************************************************************************************
skipping: [rocky]
skipping: [suse]
changed: [debian]
changed: [ubuntu]

TASK [Install chrony on Rocky Linux] ********************************************************************************************************************
skipping: [debian]
skipping: [suse]
skipping: [ubuntu]
changed: [rocky]

TASK [Install chrony on SUSE Linux] *********************************************************************************************************************
skipping: [rocky]
skipping: [debian]
skipping: [ubuntu]
changed: [suse]

TASK [Start and enable chronyd service on Debian/Ubuntu] ************************************************************************************************
skipping: [rocky]
skipping: [suse]
ok: [debian]
ok: [ubuntu]

TASK [Start and enable chronyd service on Rocky Linux] **************************************************************************************************
skipping: [debian]
skipping: [suse]
skipping: [ubuntu]
changed: [rocky]

TASK [Start and enable chronyd service on SUSE Linux] ***************************************************************************************************
skipping: [rocky]
skipping: [debian]
skipping: [ubuntu]
changed: [suse]

TASK [Ensure /etc/chrony directory exists] **************************************************************************************************************
ok: [ubuntu]
ok: [debian]
changed: [rocky]
changed: [suse]

TASK [Check if chrony.conf exists] ********************************************************************************************************************
ok: [debian]
ok: [ubuntu]
ok: [rocky]
ok: [suse]

TASK [Create empty chrony.conf if it does not exist] ****************************************************************************************************
skipping: [debian]
skipping: [ubuntu]
changed: [rocky]
changed: [suse]

TASK [Backup original chrony.conf] ********************************************************************************************************************
skipping: [rocky]
skipping: [suse]
changed: [debian]
changed: [ubuntu]

TASK [Install custom chrony configuration] **************************************************************************************************************
changed: [debian]
changed: [ubuntu]
changed: [rocky]
changed: [suse]

RUNNING HANDLER [Restart chronyd] *********************************************************************************************************************
changed: [ubuntu]
changed: [debian]
changed: [rocky]
changed: [suse]

PLAY RECAP ******************************************************************************************************************************************
debian                     : ok=8    changed=4    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0   
rocky                      : ok=8    changed=6    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0   
suse                       : ok=8    changed=6    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0   
ubuntu                     : ok=8    changed=4    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0   
```

Je reset la configuration avec le playbook reset-chrony.yml :
```bash
ansible-playbook reset-chrony.yml
```

```bash
PLAY [all] *************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************
ok: [debian]
ok: [rocky]
ok: [ubuntu]
ok: [suse]

TASK [Stop and disable chronyd service on Debian/Ubuntu] ************************************************************************************************
skipping: [rocky]
skipping: [suse]
changed: [debian]
changed: [ubuntu]

TASK [Stop and disable chronyd service on Rocky Linux] **************************************************************************************************
skipping: [debian]
skipping: [suse]
skipping: [ubuntu]
changed: [rocky]

TASK [Stop and disable chronyd service on SUSE Linux] ***************************************************************************************************
skipping: [rocky]
skipping: [debian]
skipping: [ubuntu]
changed: [suse]

TASK [Uninstall chrony on Debian/Ubuntu] ****************************************************************************************************
skipping: [rocky]
skipping: [suse]
changed: [ubuntu]
changed: [debian]

TASK [Uninstall chrony on Rocky Linux] ****************************************************************************************************
skipping: [debian]
skipping: [suse]
skipping: [ubuntu]
changed: [rocky]

TASK [Uninstall chrony on SUSE Linux] ****************************************************************************************************
skipping: [rocky]
skipping: [debian]
skipping: [ubuntu]
changed: [suse]

TASK [Remove chrony configuration directory] ****************************************************************************************************
ok: [debian]
ok: [ubuntu]
changed: [rocky]
changed: [suse]

TASK [Remove chrony drift file] ****************************************************************************************************
ok: [debian]
ok: [ubuntu]
ok: [rocky]
ok: [suse]

TASK [Remove chrony log directory] ****************************************************************************************************
ok: [ubuntu]
ok: [debian]
ok: [rocky]
ok: [suse]

PLAY RECAP ********************************************************************************************************************
debian                     : ok=6    changed=2    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   
rocky                      : ok=6    changed=3    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   
suse                       : ok=6    changed=3    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   
ubuntu                     : ok=6    changed=2    unreachable=0    failed=0    skipped=4    rescued=0    ignored=0   
```



J'exécute le playbook chrony-02.yml :
```bash
ansible-playbook chrony-02.yml
```
```bash
PLAY [all] **************************************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************
ok: [debian]
ok: [rocky]
ok: [ubuntu]
ok: [suse]

TASK [Set variables for Debian/Ubuntu] **************************************************************************************************************
skipping: [rocky]
ok: [debian]
skipping: [suse]
ok: [ubuntu]

TASK [Set variables for Rocky Linux] **************************************************************************************************************
ok: [rocky]
skipping: [debian]
skipping: [suse]
skipping: [ubuntu]

TASK [Set variables for SUSE Linux] **************************************************************************************************************
skipping: [rocky]
skipping: [debian]
ok: [suse]
skipping: [ubuntu]

TASK [Install chrony package] **************************************************************************************************************
changed: [rocky]
changed: [debian]
changed: [ubuntu]
changed: [suse]

TASK [Start and enable chronyd service] **************************************************************************************************************
ok: [ubuntu]
ok: [debian]
changed: [rocky]
changed: [suse]

TASK [Ensure chrony configuration directory exists] *****************************************************************************************************
ok: [debian]
ok: [ubuntu]
changed: [rocky]
changed: [suse]

TASK [Check if chrony.conf exists] **************************************************************************************************************
ok: [ubuntu]
ok: [debian]
ok: [rocky]
ok: [suse]

TASK [Create empty chrony.conf if it does not exist] ****************************************************************************************************
skipping: [debian]
skipping: [ubuntu]
changed: [rocky]
changed: [suse]

TASK [Backup original chrony.conf] **************************************************************************************************************
skipping: [rocky]
skipping: [suse]
changed: [ubuntu]
changed: [debian]

TASK [Install custom chrony configuration] **************************************************************************************************************
changed: [debian]
changed: [ubuntu]
changed: [rocky]
changed: [suse]

RUNNING HANDLER [Restart chronyd] **************************************************************************************************************
changed: [ubuntu]
changed: [debian]
changed: [rocky]
changed: [suse]

PLAY RECAP **************************************************************************************************************
debian                     : ok=9    changed=4    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
rocky                      : ok=9    changed=6    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
suse                       : ok=9    changed=6    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
ubuntu                     : ok=9    changed=4    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
```

Je quitte le Control Host et je supprime toutes les VM :
```bash
exit
vagrant destroy -f
```