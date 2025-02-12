## Exercice 5

Je démarre les VM et me connecte au Control Host :
```bash
vagrant up
vagrant ssh control
```

J'édite le fichier hosts:
```bash
sudo vim /etc/hosts
```
```bash
192.168.56.10  control.sandbox.lan    control
192.168.56.20  target01.sandbox.lan   target01
192.168.56.30  target02.sandbox.lan   target02
192.168.56.40  target03.sandbox.lan   target03
```


Je configure l’authentification par clé SSH :
Je génère une paire de clés SSH :
```bash
ssh-keygen
```

Je distribue la clé publique sur les Target Hosts :
```bash
ssh-copy-id vagrant@target01
ssh-copy-id vagrant@target02
ssh-copy-id vagrant@target03
```
```bash
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
The authenticity of host 'target01 (192.168.56.20)' can't be established.
ED25519 key fingerprint is SHA256:SzJYNFxdtinL9LdXn7lPx+7qpWZz38t2oKjOTAJ8I2w.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@target01's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@target01'"
and check to make sure that only the key(s) you wanted were added.

/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
The authenticity of host 'target02 (192.168.56.30)' can't be established.
ED25519 key fingerprint is SHA256:SzJYNFxdtinL9LdXn7lPx+7qpWZz38t2oKjOTAJ8I2w.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@target02's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@target02'"
and check to make sure that only the key(s) you wanted were added.

/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
The authenticity of host 'target03 (192.168.56.40)' can't be established.
ED25519 key fingerprint is SHA256:SzJYNFxdtinL9LdXn7lPx+7qpWZz38t2oKjOTAJ8I2w.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: [hashed name]
    ~/.ssh/known_hosts:4: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@target03's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@target03'"
and check to make sure that only the key(s) you wanted were added.
```

J'installe Ansible et vérifie la version:
```bash
sudo apt update
sudo apt-add-repository ppa:ansible/ansible
sudo apt install ansible
ansible --version
```
```bash
ansible [core 2.17.8]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/vagrant/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/vagrant/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Mar 22 2024, 16:50:05) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.0.3
  libyaml = True
```

J'envoie un premier ping Ansible sans configuration :
```bash
ansible all -i target01,target02,target03 -m ping
```
```bash
target01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false,
    "ping": "pong"
}
[WARNING]: Platform linux on host target02 is using the discovered Python
interpreter at /usr/bin/python3.10, but future installation of another Python
interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
target02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false,
    "ping": "pong"
}
[WARNING]: Platform linux on host target03 is using the discovered Python
interpreter at /usr/bin/python3.10, but future installation of another Python
interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
target03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false,
    "ping": "pong"
}
```

Je crée un répertoire projet :
```bash
mkdir -p ~/monprojet
cd ~/monprojet
```

Je crée un fichier ansible.cfg et vérifie s'il est pris en compte par ansible :
```bash
touch ansible.cfg
ansible --version | head -n 2
```
```bash
ansible [core 2.17.8]
  config file = /home/vagrant/monprojet/ansible.cfg
```

Je créer un inventaire nommé hosts et le rajoute au fichier ansible.cfg :
```bash
touch hosts
vim ansible.cfg
```
```bash
[defaults]
inventory = ./hosts
```
Je crée un répertoire pour la journalisation et la rajoute à ansible :
```bash
mkdir -p ~/journal
vim ansible.cfg
```
```bash
[defaults]
inventory = ./hosts
log_path = ~/journal/ansible.log
```
Je test la configuration : 
```bash
ansible all -i target01,target02,target03 -m ping
cat ~/journal/ansible.log | head -n 8
```
```bash
2025-02-12 10:04:46,839 p=4587 u=vagrant n=ansible | [WARNING]: Platform linux on host target02 is using the discovered Python
interpreter at /usr/bin/python3.10, but future installation of another Python
interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.

```


J'ajoute le groupe **testlab** au fichier ``hosts`` :
```bash
vim hosts
```
```bash
[testlab]
target01
target02
target03

[testlab:vars]
ansible_user=vagrant
```

J'envoie un ping Ansible vers le groupe de machines [all] :
```bash
ansible all -m ping
```
```bash
target01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false,
    "ping": "pong"
}
```

Je définis l’élévation des droits :

J'ajoute la ligne suivante au fichier hosts:
```bash
vim hosts
```
```bash
[testlab]
target01
target02
target03

[testlab:vars]
ansible_user=vagrant
ansible_become=yes
```

J'affiche la première ligne du fichier shadow sur tous les Target Hosts :
```bash
ansible all -a "head -n 1 /etc/shadow"
```
```bash
target02 | CHANGED | rc=0 >>
root:*:19769:0:99999:7:::

target01 | CHANGED | rc=0 >>
root:*:19769:0:99999:7:::

target03 | CHANGED | rc=0 >>
root:*:19769:0:99999:7:::
```

Je quitte le Control Host et je supprime toutes les VM de l’atelier :
```bash
exit
vagrant destroy -f
```

