## Exercice 4 

Placez-vous dans le répertoire du troisième atelier pratique :

```bash
cd ~/formation-ansible/atelier-03
```

Je démarre les VMs et me connecte au control:
```bash
vagrant up
vagrant ssh control
```

Je vérifie que Ansible est bien installé:
```bash
# type ansible
ansible is /usr/bin/ansible
```

J'édite le fichier hosts du Control Host :
```markdown
# vim /etc/hosts
127.0.0.1      localhost.localdomain      localhost
192.168.56.10  control.sandbox.lan        control
192.168.56.20  target01.sandbox.lan       target01
192.168.56.30  target02.sandbox.lan       target02
192.168.56.40  target03.sandbox.lan       target03
```
Je teste la connectivité de base des VM :
```bash
for Host in target01 target02 target03; do ping -c 1 -q $HOST; done
```
```bash
PING target01.sandbox.lan (192.168.56.20) 56(84) bytes of data.

--- target01.sandbox.lan ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.804/1.804/1.804/0.000 ms
PING target02.sandbox.lan (192.168.56.30) 56(84) bytes of data.

--- target02.sandbox.lan ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.893/1.893/1.893/0.000 ms
PING target03.sandbox.lan (192.168.56.40) 56(84) bytes of data.

--- target03.sandbox.lan ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.587/1.587/1.587/0.000 ms
```

Je collecte les clés SSH publiques des Target Hosts :
```bash
ssh-keyscan -t rsa target01 target02 target03 >> ~/.ssh/known_hosts
```
```bash
# target02:22 SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.10
# target01:22 SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.10
# target03:22 SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.10
```
Je vérifie la connectivité : 
```bash
ssh target01
ssh target02
ssh target03
```
```bash
vagrant@target01's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-116-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Feb 11 10:57:41 AM UTC 2025

  System load:           0.08
  Usage of /:            12.7% of 30.34GB
  Memory usage:          22%
  Swap usage:            0%
  Processes:             139
  Users logged in:       0
  IPv4 address for eth0: 10.0.2.15
  IPv6 address for eth0: fd00::a00:27ff:fec8:9864


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento

Use of this system is acceptance of the OS vendor EULA and License Agreements.
vagrant@target01:~$ exit
logout
Connection to target01 closed.
vagrant@target02's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-116-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Feb 11 10:57:53 AM UTC 2025

  System load:           0.04
  Usage of /:            12.6% of 30.34GB
  Memory usage:          24%
  Swap usage:            0%
  Processes:             139
  Users logged in:       0
  IPv4 address for eth0: 10.0.2.15
  IPv6 address for eth0: fd00::a00:27ff:fec8:9864


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento

Use of this system is acceptance of the OS vendor EULA and License Agreements.
vagrant@target02:~$ exit
logout
Connection to target02 closed.
vagrant@target03's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-116-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Feb 11 10:58:02 AM UTC 2025

  System load:           0.08
  Usage of /:            12.6% of 30.34GB
  Memory usage:          23%
  Swap usage:            0%
  Processes:             141
  Users logged in:       0
  IPv4 address for eth0: 10.0.2.15
  IPv6 address for eth0: fd00::a00:27ff:fec8:9864


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento

Use of this system is acceptance of the OS vendor EULA and License Agreements.
vagrant@target03:~$ 
```
Je génère le ssh-keygen :
```bash
ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/vagrant/.ssh/id_rsa
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:eFnipSf3wtJieTBXsCoZ8rNm4pm3rCfWgMmk+02wYNM vagrant@control
The key's randomart image is:
+---[RSA 3072]----+
|          .      |
|           o     |
|    . . . + .    |
|  o  o = B .     |
|.*.E  * S +      |
|o.=o.  = @ .     |
| .. oo+ = = .    |
|.  +oB+. + .     |
| ...*=o.         |
+----[SHA256]-----+

```
Je fais le copy-id: 
```bash
ssh-copy-id vagrant@target01
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@target01's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@target01'"
and check to make sure that only the key(s) you wanted were added.

vagrant@control:~$ ssh-copy-id vagrant@target02
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@target02's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@target02'"
and check to make sure that only the key(s) you wanted were added.

vagrant@control:~$ ssh-copy-id vagrant@target03
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/vagrant/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
vagrant@target03's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'vagrant@target03'"
and check to make sure that only the key(s) you wanted were added.
```
J'effectue le test du ping Ansible : 
```bash
ansible all -i target01,target02,target03 -m ping
```
```bash
target03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
target01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
target02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```