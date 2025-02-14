## Exercice 10

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

J'écris le playbook kernel.yml :
```bash
vim kernel.yml
```
```bash
---  # kernel.yml

- hosts: all
  gather_facts: false

  tasks:
    - name: Get kernel information
      command: uname -a
      register: kernel_info

    - name: Display kernel information using msg
      debug:
        msg: "{{ kernel_info.stdout }}"

    - name: Display kernel information using var
      debug:
        var: kernel_info.stdout
```

Je vérifie la syntaxe correcte de mon playbook kernel.yml :
```bash
yamllint kernel.yml
```

Pas d'erreur n'a été relevée donc j'imagine que la syntaxe est bonne !

Je l'exécute pour vérifier qu'il fonctionne bien :
```bash
ansible-playbook kernel.yml
```
```bash
PLAY [all] *********************************************************************************************************************

TASK [Get kernel information] *********************************************************************************************************************
changed: [debian]
changed: [rocky]
changed: [suse]

TASK [Display kernel information using msg] *************************************************************************************************************
ok: [rocky] => {
    "msg": "Linux rocky 5.14.0-362.13.1.el9_3.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Dec 13 14:07:45 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux"
}
ok: [debian] => {
    "msg": "Linux debian 6.1.0-17-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.69-1 (2023-12-30) x86_64 GNU/Linux"
}
ok: [suse] => {
    "msg": "Linux suse 5.14.21-150500.55.39-default #1 SMP PREEMPT_DYNAMIC Tue Dec 5 10:06:35 UTC 2023 (2e4092e) x86_64 x86_64 x86_64 GNU/Linux"
}

TASK [Display kernel information using var] *************************************************************************************************************
ok: [rocky] => {
    "kernel_info.stdout": "Linux rocky 5.14.0-362.13.1.el9_3.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Dec 13 14:07:45 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux"
}
ok: [debian] => {
    "kernel_info.stdout": "Linux debian 6.1.0-17-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.69-1 (2023-12-30) x86_64 GNU/Linux"
}
ok: [suse] => {
    "kernel_info.stdout": "Linux suse 5.14.21-150500.55.39-default #1 SMP PREEMPT_DYNAMIC Tue Dec 5 10:06:35 UTC 2023 (2e4092e) x86_64 x86_64 x86_64 GNU/Linux"
}

PLAY RECAP *********************************************************************************************************************
debian                     : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
rocky                      : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

J'écris le playbook packages.yml :
```bash
vim packages.yml
```
```bash
---  # packages.yml

- hosts: rocky,suse
  gather_facts: false

  tasks:
    - name: Get the number of installed RPM packages
      shell: rpm -qa | wc -l
      register: rpm_count
      changed_when: false  # La commande ne modifie rien

    - name: Display the number of installed RPM packages
      debug:
        msg: "Number of installed RPM packages: {{ rpm_count.stdout_lines[0] }}"
```

Je vérifie la syntaxe correcte de mon playbook packages.yml :
```bash
yamllint packages.yml
```

Pas d'erreur n'a été relevée donc j'imagine que la syntaxe est bonne !

Je l'exécute pour vérifier qu'il fonctionne bien :
```bash
ansible-playbook packages.yml
```
```bash
PLAY [rocky,suse] **************************************************************************************************
TASK [Get the number of installed RPM packages] ********************************************************************
ok: [rocky]
ok: [suse]

TASK [Display the number of installed RPM packages] ****************************************************************
ok: [rocky] => {
    "msg": "Number of installed RPM packages: 671"
}
ok: [suse] => {
    "msg": "Number of installed RPM packages: 917"
}

PLAY RECAP *********************************************************************************************************
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Je quitte le Control Host et je supprime toutes les VM :
```bash
exit
vagrant destroy -f
```
