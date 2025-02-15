## Exercice 11

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
Playbook pkg-info.yml :
```bash
vim pkg-info.yml
```
```bash
---  # pkg-info.yml

- hosts: all

  tasks:
    - name: Display package manager
      debug:
        msg: >-
          Host [{{ inventory_hostname }}] uses
          {{ ansible_pkg_mgr }} as package manager.
```

Playbook python-info.yml :
```bash
vim python-info.yml
```
```bash
---  # python-info.yml

- hosts: all

  tasks:
    - name: Display Python version
      debug:
        msg: >-
          Host [{{ inventory_hostname }}] has Python version
          {{ ansible_python_version }} installed.
```

Playbook dns-info.yml :
```bash
vim dns-info.yml
```
```bash
---  # dns-info.yml

- hosts: all

  tasks:
    - name: Display DNS servers
      debug:
        msg: >-
          Host [{{ inventory_hostname }}] uses the following DNS servers:
          {{ ansible_dns.nameservers }}
```

Je vérifie la syntaxe des playbooks: 
Playbook pkg-info.yml :
```bash
yamllint pkg-info.yml
```

Playbook python-info.yml :
```bash
yamllint python-info.yml
```

Playbook dns-info.yml :
```bash
yamllint dns-info.yml
```
Pas d'erreur dont je pense que c'est bon !

J'exécute les playbooks :
J'exécute le playbook pkg-info.yml :

```bash
ansible-playbook pkg-info.yml
```
```bash
PLAY [all] *******************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [debian]
ok: [rocky]
ok: [suse]

TASK [Display package manager] *******************************************************************************************************************
ok: [rocky] => {
    "msg": "Host [rocky] uses dnf as package manager."
}
ok: [debian] => {
    "msg": "Host [debian] uses apt as package manager."
}
ok: [suse] => {
    "msg": "Host [suse] uses zypper as package manager."
}

PLAY RECAP *******************************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

J'exécute le playbook python-info.yml :

```bash
ansible-playbook python-info.yml
```
```bash
PLAY [all] ********************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************
ok: [debian]
ok: [rocky]
ok: [suse]

TASK [Display Python version] ********************************************************************************************************************
ok: [rocky] => {
    "msg": "Host [rocky] has Python version 3.9.18 installed."
}
ok: [debian] => {
    "msg": "Host [debian] has Python version 3.11.2 installed."
}
ok: [suse] => {
    "msg": "Host [suse] has Python version 3.6.15 installed."
}

PLAY RECAP *********************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

J'exécute le playbook dns-info.yml :
```bash
ansible-playbook dns-info.yml
```
```bash
PLAY [all] *******************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************
ok: [debian]
ok: [rocky]
ok: [suse]

TASK [Display DNS servers] *******************************************************************************************************************
ok: [rocky] => {
    "msg": "Host [rocky] uses the following DNS servers: ['10.0.2.3']"
}
ok: [debian] => {
    "msg": "Host [debian] uses the following DNS servers: ['4.2.2.1', '4.2.2.2', '208.67.220.220']"
}
ok: [suse] => {
    "msg": "Host [suse] uses the following DNS servers: ['10.0.2.3']"
}

PLAY RECAP ********************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Je quitte le Control Host et je supprime toutes les VM :
```bash
exit
vagrant destroy -f
```