## Exercice 6

Je démarre les VM et me connect au Control Host:
 ```bash
 vagrant up
 vagrant ssh ansible
 ```
J'installe successivement les paquets `tree`, `git` et `nmap` sur toutes les cibles :
 ```bash
 ansible all -m package -a "name=tree state=present"
 ansible all -m package -a "name=git state=present"
 ansible all -m package -a "name=nmap state=present"
 ```
 1ère fois exemple : 
 ```bash
suse | CHANGED => {
    "changed": true,
    "cmd": [
        "/usr/bin/zypper",
        "--quiet",
        "--non-interactive",
        "--xmlout",
        "install",
        "--type",
        "package",
        "--auto-agree-with-licenses",
        "--no-recommends",
        "--",
        "+nmap"
    ],
    "name": [
        "nmap"
    ],
    "rc": 0,
    "state": "present",
    "stderr": "",
    "stderr_lines": [],
        "</stream>"
    "update_cache": false
}
 ```
 2ème fois 
 ```bash
debian | SUCCESS => {
    "cache_update_time": 1704860215,
    "cache_updated": false,
    "changed": false
}
suse | SUCCESS => {
    "changed": false,
    "name": [
        "git"
    ],
    "rc": 0,
    "state": "present",
    "update_cache": false
}
rocky | SUCCESS => {
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
 ```
Je désinstalle successivement ces trois paquets en utilisant le paramètre supplémentaire `state=absent` :
 ```bash
 ansible all -m package -a "name=tree state=absent"
 ansible all -m package -a "name=git state=absent"
 ansible all -m package -a "name=nmap state=absent"
 ```
1ère fois
```bash
rocky | CHANGED => {
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
        "Removed: nmap-3:7.92-3.el9.x86_64"
    ]
}
```
2ème fois 
```bash
debian | SUCCESS => {
    "changed": false
}
suse | SUCCESS => {
    "changed": false,
    "name": [
        "nmap"
    ],
    "rc": 0,
    "state": "absent",
    "update_cache": false
}
rocky | SUCCESS => {
    "changed": false,
    "msg": "Nothing to do",
    "rc": 0,
    "results": []
}
```

Je copie le fichier **fstab** du Control vers les Hosts :
 ```bash
 ansible all -m copy -a "src=/etc/fstab dest=/tmp/test3.txt"
 ```
1ère fois
```bash
debian | CHANGED => {
    "changed": true,
    "checksum": "d39263691e31170df199aae3d93f7c556fbb3446",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "md5sum": "b6f1fe0d790a8d2f9b74b95df1c889dc",
    "mode": "0644",
    "owner": "root",
    "size": 743,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1739358564.4484885-6280-236377902215289/source",
    "state": "file",
    "uid": 0
}
rocky | CHANGED => {
    "changed": true,
    "checksum": "d39263691e31170df199aae3d93f7c556fbb3446",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "md5sum": "b6f1fe0d790a8d2f9b74b95df1c889dc",
    "mode": "0644",
    "owner": "root",
    "secontext": "unconfined_u:object_r:user_home_t:s0",
    "size": 743,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1739358564.614805-6279-60126614248317/source",
    "state": "file",
    "uid": 0
}
suse | CHANGED => {
    "changed": true,
    "checksum": "d39263691e31170df199aae3d93f7c556fbb3446",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "md5sum": "b6f1fe0d790a8d2f9b74b95df1c889dc",
    "mode": "0644",
    "owner": "root",
    "size": 743,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1739358564.4916928-6281-235620488395485/source",
    "state": "file",
    "uid": 0
}
```
2ème fois 
```bash
debian | SUCCESS => {
    "changed": false,
    "checksum": "d39263691e31170df199aae3d93f7c556fbb3446",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "path": "/tmp/test3.txt",
    "size": 743,
    "state": "file",
    "uid": 0
}
rocky | SUCCESS => {
    "changed": false,
    "checksum": "d39263691e31170df199aae3d93f7c556fbb3446",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "path": "/tmp/test3.txt",
    "secontext": "unconfined_u:object_r:user_home_t:s0",
    "size": 743,
    "state": "file",
    "uid": 0
}
suse | SUCCESS => {
    "changed": false,
    "checksum": "d39263691e31170df199aae3d93f7c556fbb3446",
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "path": "/tmp/test3.txt",
    "size": 743,
    "state": "file",
    "uid": 0
}
```

Je le supprime sur les Hosts :
 ```bash
 ansible all -m file -a "path=/tmp/test3.txt state=absent"
 ```
1ère fois
```bash
debian | CHANGED => {
    "changed": true,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
rocky | CHANGED => {
    "changed": true,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
suse | CHANGED => {
    "changed": true,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
```
2ème fois 
```bash
debian | SUCCESS => {
    "changed": false,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
rocky | SUCCESS => {
    "changed": false,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
suse | SUCCESS => {
    "changed": false,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
```

J'affiche l’espace utilisé par la partition principale sur tous les Target Hosts :
 ```bash
 ansible all -m command -a "df -h /"
 ```
1ère fois
```bash
debian | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3       124G  2.3G  115G   2% /
rocky | CHANGED | rc=0 >>
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/rl_rocky9-root   70G  2.4G   68G   4% /
suse | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3       124G  2.8G  118G   3% /
```
2ème fois 
La commande affiche le même résultat car on apporte pas de modification, on récupère juste l'état actuel des machines. Le résultat ne changerai que si on utilisait plus d'espace entre l'exécutions des 2 commandes. 

