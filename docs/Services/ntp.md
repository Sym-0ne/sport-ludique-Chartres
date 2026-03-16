
# Déploiement d’un serveur NTP interne et automatisation avec Ansible

## 1. Mise en place du serveur NTP

Dans l’infrastructure, une VM Debian avec l’adresse IP 10.10.120.12 est
utilisée comme serveur NTP interne.

**Installation de Chrony**

Sur la VM 10.10.120.12 :

```
sudo apt update sudo apt install chrony -y
```

Chrony est utilisé car il est plus moderne et plus performant que ntpd
pour la synchronisation du temps.

**Configuration du serveur NTP**

Le fichier de configuration est situé ici :

```
/etc/chrony/chrony.conf
```

Les paramètres principaux ajoutés :

```
server 10.10.120.12 iburst driftfile /var/lib/chrony/chrony.drift
rtcsync makestep 1 3
```

Le serveur NTP est également configuré pour autoriser le réseau de
management à se synchroniser.

Après modification de la configuration :

```
sudo systemctl restart chrony sudo systemctl enable chrony
```

Le serveur 10.10.120.12 sert désormais de référence temporelle pour
toute l’infrastructure.

---

## 2. Automatisation avec Ansible

L’infrastructure comporte un grand nombre de VM Debian. Afin d’éviter
une configuration manuelle sur chaque machine, Ansible est utilisé pour
automatiser l’installation et la configuration du client NTP.

Une VM dédiée sert de serveur Ansible.

Les hôtes sont déclarés dans le fichier :

```
/etc/ansible/hosts
```

Exemple de groupe utilisé pour les machines Debian :

```
[all_linux] vm1 vm2 vm3 vm4
```

---

## 3. Création du playbook Ansible

Un playbook a été créé dans :

```/etc/ansible/playbooks/ntp.yml
```

Ce playbook permet de : 

- Installer le paquet chrony 
- Configurer le client NTP pour utiliser 10.10.120.12 
- Redémarrer le service chrony

**Contenu du playbook :**

```
    -   name: Configure NTP clients hosts: all_linux become: yes

    tasks:

    -   name: Installer chrony apt: name: chrony state: present

    -   name: Configurer chrony copy: dest: /etc/chrony/chrony.conf
        content: | server 10.10.120.12 iburst driftfile
        /var/lib/chrony/chrony.drift rtcsync makestep 1 3

    -   name: Redémarrer chrony service: name: chrony state: restarted
        enabled: yes
```

---

## 4. Exécution du playbook

Depuis le serveur Ansible :

```
ansible-playbook /etc/ansible/playbooks/ntp.yml```

Le playbook se connecte en SSH à toutes les machines du groupe all_linux
et applique automatiquement la configuration NTP.

---

## 5. Vérification

Pour vérifier que toutes les machines utilisent bien le serveur NTP
interne :


Crée un playbook rapide verif-ntp.yml :

```
- name: Vérifier la synchro NTP sur toutes les VM
  hosts: all_linux
  become: yes

  tasks:
    - name: Vérifier les sources NTP
      command: chronyc sources
      register: chrony_status

    - name: Afficher le statut NTP
      debug:
        msg: "{{ inventory_hostname }} -> {{ chrony_status.stdout }}"```

Puis lance-le :

```
ansible-playbook /etc/ansible/playbooks/verif-ntp.yml```

Résultat obtenu : pour chaque VM quelle source NTP elle utilise et si elle est synchronisée avec 10.10.120.12

---