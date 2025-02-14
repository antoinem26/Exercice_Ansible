## Exercice 9

Je démarre les VM et me connecte au Control Host :

```bash
vagrant up
vagrant ssh control
```
Je me rends dans le répertoire du projet :
```bash
cd ansible/projets/ema/
ls -l
```

J'écris le playbook myvars1.yml :
```bash
vim myvars1.yml
```
```bash
- hosts: all
  gather_facts: false

  vars:
    mycar: Tesla
    mybike: Ducati

  tasks:
    - debug:
        msg: "Car: {{ mycar }}, Bike: {{ mybike }}"
```
Je valide la syntaxe :
```bash
yamllint myvars1.yml
```

J'exécute le playbook myvars1.yml avec les valeurs par défaut :
```bash
ansible-playbook myvars1.yml
```
```bash
PLAY [all] 
*******************************************************************************************************

TASK [debug] ******************************************************************************************************************
ok: [target01] => {
    "msg": "Car: Tesla, Bike: Ducati"
}
ok: [target03] => {
    "msg": "Car: Tesla, Bike: Ducati"
}
ok: [target02] => {
    "msg": "Car: Tesla, Bike: Ducati"
}

PLAY RECAP 
******************************************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

Je le ré-exécute en remplaçant les valeurs avec des extra vars :
Remplacement de la variable **mycar** : 
```bash
ansible-playbook myvars1.yml -e mycar=BMW
```
```bash
PLAY [all] *******************************************************************************************************************

TASK [debug] *******************************************************************************************************************
ok: [target02] => {
    "msg": "Car: BMW, Bike: Ducati"
}
ok: [target01] => {
    "msg": "Car: BMW, Bike: Ducati"
}
ok: [target03] => {
    "msg": "Car: BMW, Bike: Ducati"
}

PLAY RECAP *******************************************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
Changement de la variable **mybike**:
```bash
ansible-playbook myvars1.yml -e mybike=Harley
```
```bash
PLAY [all] *******************************************************************************************************************

TASK [debug] *******************************************************************************************************************
ok: [target01] => {
    "msg": "Car: Tesla, Bike: Harley"
}
ok: [target03] => {
    "msg": "Car: Tesla, Bike: Harley"
}
ok: [target02] => {
    "msg": "Car: Tesla, Bike: Harley"
}

PLAY RECAP *******************************************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Remplacement des 2 variables en même temps : 
```bash
ansible-playbook myvars1.yml -e mycar=BMW -e mybike=Harley
```
```bash
PLAY [all] *******************************************************************************************************************

TASK [debug] *******************************************************************************************************************
ok: [target02] => {
    "msg": "Car: BMW, Bike: Harley"
}
ok: [target03] => {
    "msg": "Car: BMW, Bike: Harley"
}
ok: [target01] => {
    "msg": "Car: BMW, Bike: Harley"
}

PLAY RECAP *******************************************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

J'écris le playbook myvars2.yml :
```bash
vim myvars2.yml
```
```bash

- hosts: all
  gather_facts: false

  tasks:
    - set_fact:
        mycar: Tesla
        mybike: Ducati

    - debug:
        msg: "Car: {{ mycar }}, Bike: {{ mybike }}"
```
Je valide la syntaxe :
```bash
yamllint myvars2.yml
```
J'exécute le playbook avec les valeurs par défaut :
```bash
ansible-playbook myvars2.yml
```
```bash
PLAY [all] ********************************************************************************************************************

TASK [set_fact] ********************************************************************************************************************
ok: [target01]
ok: [target02]
ok: [target03]

TASK [debug] ********************************************************************************************************************
ok: [target01] => {
    "msg": "Car: Tesla, Bike: Ducati"
}
ok: [target02] => {
    "msg": "Car: Tesla, Bike: Ducati"
}
ok: [target03] => {
    "msg": "Car: Tesla, Bike: Ducati"
}

PLAY RECAP ********************************************************************************************************************
target01                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

Je le ré-exécute en remplaçant les valeurs avec des extra vars :

```bash
ansible-playbook myvars2.yml -e mycar=BMW
ansible-playbook myvars2.yml -e mybike=Harley
ansible-playbook myvars2.yml -e mycar=BMW -e mybike=Harley
```
```bash
PLAY [all] ********************************************************************************************************************

TASK [set_fact] ********************************************************************************************************************
ok: [target02]
ok: [target01]
ok: [target03]

TASK [debug] ********************************************************************************************************************
ok: [target01] => {
    "msg": "Car: BMW, Bike: Ducati"
}
ok: [target02] => {
    "msg": "Car: BMW, Bike: Ducati"
}
ok: [target03] => {
    "msg": "Car: BMW, Bike: Ducati"
}

PLAY RECAP ********************************************************************************************************************
target01                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


PLAY [all] ********************************************************************************************************************

TASK [set_fact] ********************************************************************************************************************
ok: [target01]
ok: [target02]
ok: [target03]

TASK [debug] ********************************************************************************************************************
ok: [target01] => {
    "msg": "Car: Tesla, Bike: Harley"
}
ok: [target02] => {
    "msg": "Car: Tesla, Bike: Harley"
}
ok: [target03] => {
    "msg": "Car: Tesla, Bike: Harley"
}

PLAY RECAP ********************************************************************************************************************
target01                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   


PLAY [all] ********************************************************************************************************************

TASK [set_fact] ********************************************************************************************************************
ok: [target01]
ok: [target02]
ok: [target03]

TASK [debug] ********************************************************************************************************************
ok: [target01] => {
    "msg": "Car: BMW, Bike: Harley"
}
ok: [target02] => {
    "msg": "Car: BMW, Bike: Harley"
}
ok: [target03] => {
    "msg": "Car: BMW, Bike: Harley"
}

PLAY RECAP ********************************************************************************************************************
target01                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

J'écris le playbook myvars3.yml :
```bash
vim myvars3.yml
```
```bash
- hosts: all
  gather_facts: false

  tasks:
    - debug:
        msg: "Car: {{ mycar }}, Bike: {{ mybike }}"
```

Je valide la syntaxe :
```bash
yamllint myvars3.yml
```

Je définis les valeurs par défaut dans un fichier :
```bash
mkdir -p group_vars
vim group_vars/all.yml
```
```bash
mycar: VW
mybike: BMW
```

Je définis les valeurs pour la **target02** :
```bash
mkdir -p host_vars
vim host_vars/target02.yml
```
```bash
mycar: Mercedes
mybike: Honda
```

J'exécute le playbook myvars3.yml :
```bash
ansible-playbook myvars3.yml
```
```bash
PLAY [all] ********************************************************************************************************************

TASK [debug] ********************************************************************************************************************
ok: [target01] => {
    "msg": "Car: VW, Bike: BMW"
}
ok: [target03] => {
    "msg": "Car: VW, Bike: BMW"
}
ok: [target02] => {
    "msg": "Car: Mercedes, Bike: Honda"
}

PLAY RECAP ********************************************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

J'écris le playbook display_user.yml :
```bash
vim display_user.yml
```
```bash
- hosts: all
  gather_facts: false

  vars_prompt:
    - name: user
      prompt: "Please enter the username"
      default: "microlinux"
      private: false

    - name: password
      prompt: "Please enter the password"
      default: "yatahongaga"
      private: true

  tasks:
    - debug:
        msg: "User: {{ user }}, Password: {{ password }}"
```
Je vérifie la validation de la syntaxe :
```bash
yamllint display_user.yml
```

J'exécute le playbook display_user.yml :
```bash
ansible-playbook display_user.yml
```

1ère fois exemple :
```bash
Please enter the username [microlinux]: antoine
Please enter the password [yatahongaga]: 

PLAY [all] ********************************************************************************************************************

TASK [debug] ********************************************************************************************************************
ok: [target01] => {
    "msg": "User: antoine, Password: antoine"
}
ok: [target03] => {
    "msg": "User: antoine, Password: antoine"
}
ok: [target02] => {
    "msg": "User: antoine, Password: antoine"
}

PLAY RECAP ********************************************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Je quitte le Control et je supprime toutes les VM :
```bash
exit
vagrant destroy -f
```
