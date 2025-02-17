## Exercice 13

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

Je commence par créer un répertoire templates à côté de mon playbook et j'ajoute le fichier chrony.conf.j2 :
```bash
mkdir -p templates
vim templates/chrony.conf.j2
```
```bash
{# {{ chrony_conf_path }} #}
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```

J'écris le playbook chrony.yml :
```bash
vim chrony.yml
```
```bash
---  # chrony.yml

- hosts: all
  become: true

  vars:
    chrony_conf_path: ""
    chrony_service_name: ""

  tasks:
    - name: Set chrony configuration path and service name for Debian/Ubuntu
      set_fact:
        chrony_conf_path: /etc/chrony/chrony.conf
        chrony_service_name: chrony
      when: ansible_os_family == "Debian"

    - name: Set chrony configuration path and service name for Rocky Linux
      set_fact:
        chrony_conf_path: /etc/chrony.conf
        chrony_service_name: chronyd
      when: ansible_distribution == "Rocky"

    - name: Set chrony configuration path and service name for SUSE Linux
      set_fact:
        chrony_conf_path: /etc/chrony/chrony.conf
        chrony_service_name: chronyd
      when: ansible_distribution == "openSUSE Leap"

    - name: Ensure chrony configuration directory exists
      file:
        path: "{{ chrony_conf_path | dirname }}"
        state: directory

    - name: Install custom chrony configuration
      template:
        src: templates/chrony.conf.j2
        dest: "{{ chrony_conf_path }}"
      notify: Restart chronyd

  handlers:
    - name: Restart chronyd
      service:
        name: "{{ chrony_service_name }}"
        state: restarted
```

Je vérifie la syntaxe des playbooks: 
Playbook chrony.yml :
```bash
yamllint chrony.yml
```

J'exécute le playbook chrony.yml :
```bash
ansible-playbook chrony.yml
```
```bash
PLAY [all] **********************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************
ok: [debian]
ok: [rocky]
ok: [ubuntu]
ok: [suse]

TASK [Set chrony configuration path and service name for Debian/Ubuntu] *********************************************************************************
skipping: [rocky]
ok: [debian]
skipping: [suse]
ok: [ubuntu]

TASK [Set chrony configuration path and service name for Rocky Linux] ***********************************************************************************
ok: [rocky]
skipping: [debian]
skipping: [suse]
skipping: [ubuntu]

TASK [Set chrony configuration path and service name for SUSE Linux] ************************************************************************************
skipping: [rocky]
skipping: [debian]
ok: [suse]
skipping: [ubuntu]

TASK [Ensure chrony configuration directory exists] *****************************************************************************************************
ok: [ubuntu]
ok: [debian]
ok: [rocky]
ok: [suse]

TASK [Install custom chrony configuration] **************************************************************************************************************
ok: [debian]
ok: [ubuntu]
ok: [rocky]
ok: [suse]

PLAY RECAP **********************************************************************************************************************************************
debian                     : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
rocky                      : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
suse                       : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0   
ubuntu                     : ok=4    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0  
```

Je quitte le Control Host et je supprime toutes les VM :
```bash
exit
vagrant destroy -f
```