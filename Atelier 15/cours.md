Atelier pratique

Placez-vous dans le répertoire du quinzième atelier pratique :

$ cd ~/formation-ansible/atelier-15

Voici les quatre machines virtuelles de cet atelier :
Machine virtuelle 	Adresse IP
ansible 	192.168.56.10
rocky 	192.168.56.20
debian 	192.168.56.30
suse 	192.168.56.40

Démarrez les VM :

$ vagrant up

Connectez-vous au Control Host :

$ vagrant ssh ansible

L’environnement de cet atelier est préconfiguré et prêt à l’emploi :

    Ansible est installé sur le Control Host.
    Le fichier /etc/hosts du Control Host est correctement renseigné.
    L’authentification par clé SSH est établie sur les trois Target Hosts.
    Le répertoire du projet existe et contient une configuration de base et un inventaire.
    Direnv est installé et activé pour le projet.
    Le validateur de syntaxe yamllint est également disponible.

Rendez-vous dans le répertoire des playbooks :

$ cd ansible/projets/ema/playbooks/
direnv: loading ~/ansible/projets/ema/.envrc
direnv: export +ANSIBLE_CONFIG

Les variables enregistrées

Au début de cette formation, je vous ai présenté les modules command et shell. Si vous découvrez Ansible, vous serez souvent tentés d’invoquer des commandes par le biais de ces deux modules. Pour vous rendre compte après coup qu’un module spécifique aurait été plus adapté à votre cas de figure :

    le module file au lieu de mkdir
    le module user au lieu de useradd
    le module synchronize au lieu de rsync
    etc.

Même si vous allez découvrir tout ça petit à petit, vous tomberez tôt ou tard sur une situation pour laquelle il n’y a pas de module. Ce n’est donc pas une mauvaise idée de voir de plus près le comportement de command et shell dans le contexte d’un playbook.

Prenons un premier exemple :

---  # command1.yml

- hosts: all
  gather_facts: false

  tasks:

    - name: Report root partition space usage
      command: df -h /

...

Le résultat de l’exécution de ce playbook vous surprendra probablement :

$ ansible-playbook command1.yml 

PLAY [all] ***************************************************************************

TASK [Report root partition space usage] *********************************************
changed: [rocky]
changed: [debian]
changed: [suse]

PLAY RECAP ***************************************************************************
debian     : ok=1  changed=1  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   
rocky      : ok=1  changed=1  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   
suse       : ok=1  changed=1  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0

    Contrairement à une invocation ad hoc, aucune information n’est affichée.
    Ansible rapporte un changement, même si nous sommes sûrs que la commande df ne modifie strictement rien sur le système cible.

Ansible ne procède pas à une analyse des commandes Linux invoquées et ne « sait » pas ce que nous faisons ici. C’est la raison pour laquelle il part du principe qu’il y a eu une modification. Nous pouvons paramétrer ce comportement grâce au paramètre changed_when. Dans ce cas précis, nous pouvons partir du principe que df n’entraînera jamais la moindre modification :

---  # command2.yml

- hosts: all
  gather_facts: false

  tasks:

    - name: Report root partition space usage
      command: df -h /
      changed_when: false

...

Effectivement, nous ne voyons plus aucun changement. En revanche, l’info recherchée ne s’affiche toujours pas :

$ ansible-playbook command2.yml 

PLAY [all] ***************************************************************************

TASK [Report root partition space usage] *********************************************
ok: [debian]
ok: [suse]
ok: [rocky]

PLAY RECAP ***************************************************************************
debian     : ok=1  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   
rocky      : ok=1  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   
suse       : ok=1  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0

On va quand-même pouvoir récupérer l’info grâce à une petite combine. Ansible permet d’enregistrer toutes les infos autour du déroulement d’une tâche dans une variable grâce à la directive register. Ces infos pourront être traitées par la suite dans une ou plusieurs tâches subséquentes :

---  # command3.yml

- hosts: all
  gather_facts: false

  tasks:

    - name: Report root partition space usage
      command: df -h /
      changed_when: false
      register: df_cmd

    - debug:
        msg: "{{df_cmd.stdout_lines}}"

...

Et voilà le travail :

$ ansible-playbook command3.yml 

PLAY [all] ***************************************************************************

TASK [Report root partition space usage] *********************************************
ok: [debian]
ok: [suse]
ok: [rocky]

TASK [debug] *************************************************************************
ok: [rocky] => {
    "msg": [
        "Filesystem                  Size  Used Avail Use% Mounted on",
        "/dev/mapper/rl_rocky9-root   70G  2.1G   68G   3% /"
    ]
}
ok: [debian] => {
    "msg": [
        "Filesystem      Size  Used Avail Use% Mounted on",
        "/dev/vda3       124G  1.8G  116G   2% /"
    ]
}
ok: [suse] => {
    "msg": [
        "Filesystem      Size  Used Avail Use% Mounted on",
        "/dev/sda3       124G  2.2G  118G   2% /"
    ]
}

PLAY RECAP ***************************************************************************
debian     : ok=2  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   
rocky      : ok=2  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   
suse       : ok=2  changed=0  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0

Techniquement parlant, une variable enregistrée (registered variable) équivaut à une variable set_fact. Elle est spécifique à l’hôte, c’est-à-dire visible uniquement pour l’hôte qui vient d’exécuter la tâche.

Dans l’exemple ci-dessous, on va afficher tout le contenu de la variable enregistrée df_cmd en nous limitant à l’hôte debian :

---  # command4.yml

- hosts: debian
  gather_facts: false

  tasks:

    - name: Report root partition space usage
      command: df -h /
      changed_when: false
      register: df_cmd

    - debug:
        msg: "{{df_cmd}}"

...

Selon la tâche que vous exécutez, une variable enregistrée est constituée de toute une série de champs. Dans notre exemple nous avons extrait les infos du champ stdout_lines qui contient le résultat de la commande ligne par ligne :

"stdout_lines": [
  "Filesystem      Size  Used Avail Use% Mounted on",
  "/dev/vda3       124G  1.8G  116G   2% /"
]

Voici grosso modo la signification de tous les champs de la variable df_cmd :

    changed : est-ce que la tâche a modifié le système ?
    failed : est-ce que la tâche a échoué ?
    cmd : la commande avec tous ses paramètres
    rc : le code retour UNIX de la commande
    start : heure de lancement
    end : heure de fin
    delta : temps d’exécution
    stdout : résultat complet avec des retours chariot \n
    stdout_lines : résultat complet ligne par ligne
    stderr : sortie erreur
    stderr_lines : sortie erreur ligne par ligne

Voici une variante plus simple pour afficher le contenu d’une variable avec le module debug :

---  # command5.yml

- hosts: all
  gather_facts: false

  tasks:

    - name: Report root partition space usage
      command: df -h /
      changed_when: false
      register: df_cmd

    - debug:
        var: df_cmd.stdout_lines

...

Exercice 10

    Écrivez un playbook kernel.yml qui affiche les infos détaillées du noyau sur tous vos Target Hosts. Utilisez la commande uname -a et le module debug avec le paramètre msg.
    Essayez d’obtenir le même résultat en utilisant le paramètre var du module debug.
    Écrivez un playbook packages.yml qui affiche le nombre total de paquets RPM installés sur les hôtes rocky et suse (rpm -qa | wc -l).

Quittez le Control Host :

$ exit

Supprimez toutes les VM :

$ vagrant destroy -f