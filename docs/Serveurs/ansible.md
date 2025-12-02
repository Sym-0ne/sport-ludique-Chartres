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

<div class="annotate" markdown>

```
sudo apt install ansible -y
```

Vérifier l’installation

```
ansible --version
```

Tu devrais voir la version installée, par exemple ansible 2.14.

---

## 2. Configuration de base

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
