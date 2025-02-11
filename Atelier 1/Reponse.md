## Exercice 1

Démarrez la VM ubuntu depuis le répertoire atelier-01.

```vagrant up ubuntu```

Connectez-vous à cette VM.

```vagrant ssh ubuntu```

Rafraîchissez les informations sur les paquets.

```sudo apt update```

Recherchez le paquet ansible avec les options qui vont bien.

```apt search --names-only ansible ```

Installez le paquet officiel fourni par la distribution.

```sudo apt install ansible ```

Vérifiez si l’installation s’est bien déroulée.

```ansible --version```

Notez la version d’Ansible.

```ansible 2.10.8```

Déconnectez-vous et supprimez la VM.

```vagrant destroy -f ubuntu```

## Exercice 2

Répétez l’exercice précédent en configurant un dépôt PPA (Personal Package Archive) pour Ansible :

```
$ sudo apt-add-repository ppa:ansible/ansible
```

Notez la version fournie par ce dépôt tiers et comparez avec la version officielle de l’exercice précédent.

## Exercice 3

Lancez une VM Rocky Linux et installez Ansible en utilisant PIP et Virtualenv.

Important /!\ Notez bien que contrairement à Debian, le paquet python3-venv n’est pas nécessaire ici, étant donné que Virtualenv fait partie des modules standard de Python dans cette distribution.