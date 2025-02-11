## Exercice 1

Démarrez la VM ubuntu depuis le répertoire atelier-01 : 

```vagrant up ubuntu```

Connectez-vous à cette VM :

```vagrant ssh ubuntu```

Rafraîchissez les informations sur les paquets :

```sudo apt update```

Recherchez le paquet ansible avec les options qui vont bien :

```apt search --names-only ansible ```

Installez le paquet officiel fourni par la distribution :

```sudo apt install ansible ```

Vérifiez si l’installation s’est bien déroulée :

```ansible --version```

Notez la version d’Ansible :

```ansible 2.10.8```

Déconnectez-vous et supprimez la VM :

```vagrant destroy -f ubuntu```

## Exercice 2

Je répète les étapes précédantes : 
```bash
vagrant up ubuntu
vagrant ssh ubuntu
sudo apt update
```

Je rajoute la configuration d'un dépôt PPA pour Ansible :
```bash
sudo apt-add-repository ppa:ansible/ansible
```
J'installe ansible à partir du PPA: 

```bash
sudo apt install ansible 
```
La version fournie par ce dépôt est : 
``` bash
ansible --version
    ansible [core 2.17.8]
``` 
**la version est plus récente que le dépôt officiel**

## Exercice 3

Lancez une VM Rocky Linux et installez Ansible en utilisant PIP et Virtualenv.

Création et connexion : 
```bash
vagrant up rocky
vagrant ssh rocky
```
Installation de ansible sur un venv python : 
```bash
sudo dnf install python3 python3-pip
pip3 install virtualenv
virtualenv ansible-env
source ansible-env/bin/activate
pip install ansible
```
**ansible --version** : 
```bash
ansible [core 2.15.13]
  config file = None
  configured module search path = ['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/vagrant/ansible-env/lib/python3.9/site-packages/ansible
  ansible collection location = /home/vagrant/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/vagrant/ansible-env/bin/ansible
  python version = 3.9.21 (main, Dec  5 2024, 00:00:00) [GCC 11.5.0 20240719 (Red Hat 11.5.0-2)] (/home/vagrant/ansible-env/bin/python)
  jinja version = 3.1.5
  libyaml = True
```
Destruction de la VM : 
```bash
deactivate
exit
vagrant destroy
```