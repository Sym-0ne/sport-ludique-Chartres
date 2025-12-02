# Installation et configuration de Ansible 

## Contexte 

Ansible est un outil d’automatisation qui permet de gérer et configurer plusieurs machines à distance sans installer d’agent supplémentaire. Dans notre cas, il sera utilisé pour déployer automatiquement l’agent GLPI sur toutes tes VM Linux, ce qui permettra de centraliser facilement l’inventaire matériel et logiciel dans [GLPI](https://sym-0ne.github.io/sport-ludique-Chartres/Services/GLPI/Glpi3/), sans avoir à intervenir manuellement sur chaque machine.

---

## 1. Installation

Depuis une machine Linux / Debian, mettre à jour le système :

```
sudo apt update
sudo apt upgrade -y
```

Installer Ansible. 

<div class="annotate" markdown>

Ubuntu propose maintenant Ansible via le dépôt officiel ou via apt directement.<br>
Pour la version stable on utilisera alors la version ```apt```

</div>

```
sudo apt install ansible -y
```

Vérifier l’installation

```
ansible --version
```

Tu devrais voir la version installée, par exemple ansible 2.14.

---

## 2. Configuration de base pour test 

Créer le fichier d’inventaire :

```
sudo mkdir -p /etc/ansible
sudo nano /etc/ansible/hosts
```

Exemple simple de contenu pour tester sur la machine locale :

```
[local]
127.0.0.1 ansible_connection=local
```

Tester la connexion :

```
ansible all -m ping
```

Si tout est OK, tu verras pong.

---

## 3. Configuration finale

Se rendre dans le fichier suivant : 

```
sudo nano /etc/ansible/hosts
```

Voici la configuration nécessaire pour notre infrastructure : 

<div class="annotate" markdown>

À chaque fois qu'on mettra en place une nouvelle VM Serveur, il faudra l'ajouter dans ce fichier de configuration afin que l'inventaire soit à jour par la suite dans le GLPI.

</div>

```
[local]
127.0.0.1 ansible_connection=local

[serveurs]
bdd ansible_host=10.10.120.7 ansible_user=bdd     
reverse-proxy ansible_host=10.10.120.80 ansible_user=user
reverse-proxy-sec ansible_host=10.10.120.90 ansible_user=user
web ansible_host=10.10.120.11 ansible_user=user
dns-autorite ansible_host=10.10.120.8 ansible_user=user
dns-autorite-sec ansible_host=10.10.120.18 ansible_user=user
dns-resolver ansible_host=10.10.120.9 ansible_user=user
dns-resolver-sec ansible_host=10.10.120.19 ansible_user=user
ca-autorite ansible_host=10.10.120.12 ansible_user=certificat
docker ansible_host=10.10.120.15 ansible_user=user
ansible ansible_host=10.10.120.14 ansible_user=ansible

[all_linux:children]
serveurs
```

---

## 4. Generation de la clé publique

Il n'est pas possible d'utiliser une connexion SSH via mot de passe car le serveur Ansible ne les connaît pas,
il faut donc mettre en place une authentification via clé publique, donc on va générer une clé publique pour le serveur Ansible :

```
ssh-keygen -t ed25519
```

On ne met pas en place de PassPhrase, car le serveur ne les connaîtra pas non plus donc ça revient au même que le mot de passe !

---

## 5. Copie de la clé publique sur les VM Serveurs de l'infrastructure 

On va copier la clé publique sur toutes les VM que l'on veut répertorier dans l'inventaire à l'aide d'une commande (scp auto) :

```
ssh-copy-id *utilisateur*@10.10.120.?
```

On y précisera le MÊME utilisateur utilisé dans ```ansible_user``` dans la configuration de l'inventaire via ```sudo nano /etc/ansible/hosts``` <br>
Faire cela pour toutes les VM présentes dans la configuration, une par une, même s'il y en a un grand nombre.

---

## 6. Test de connexion

Une fois les étapes précédentes correctement mises en place, nous ferons un test via la commande suivante :

```
ansible all -m ping 
```

Le résultat retourné doit être le suivant (ceci est un exemple):

```
[WARNING]: Host 'dns-resolver-sec' is using the discovered Python interpreter at '/usr/bin/python3.13', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.19/reference_appendices/interpreter_discovery.html for more information.
dns-resolver-sec | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.13"
    },
    "changed": false,
    "ping": "pong"
}
```

---

## 7. Mise en place Agent GLPI

Nous allons pousser un Agent GLPI sur toutes les machines Linux dans la documentation de la Mise en place de l'Inventaire [GLPI]https://sym-0ne.github.io/sport-ludique-Chartres/Services/GLPI/Glpi3/

---