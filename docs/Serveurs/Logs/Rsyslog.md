# Configuration des VM pour envoyer les logs vers Graylog en UDP

## Objectif :
----------

L’objectif est de configurer toutes les VM Linux/Debian de l'infrastructure afin qu’elles envoient leurs logs vers le serveur Graylog via UDP.<br>
Cela permet de centraliser la collecte et la visualisation des logs, d’identifier rapidement les problèmes et de suivre les activités de toutes les VM.<br>
Nous utilisons Ansible pour automatiser l’installation et la configuration, garantissant que toutes les machines sont configurées de manière cohérente.<br>

---

## 1 : Installer rsyslog sur la VM Ansible

```
sudo apt install rsyslog
```

---

## 2 : Créer le playbook

Éditer le fichier :

```
sudo nano /etc/ansible/playbooks/rsyslog_graylog.yml
```                                   

Mettre le contenu suivant :

```
- name: Configurer toutes les VM pour envoyer les logs vers Graylog en UDP
  hosts: all
  become: true
  vars:
    graylog_server_ip: "10.10.120.16"
    graylog_udp_port: 514

  tasks:
    - name: Forcer apt à utiliser IPv4
      lineinfile:
        path: /etc/apt/apt.conf.d/99force-ipv4
        line: 'Acquire::ForceIPv4 "true";'

    - name: Mettre à jour le cache apt
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Installer rsyslog 
      apt:
        name:
          - rsyslog
        state: present

    - name: Créer le dossier rsyslog.d si nécessaire
      file:
        path: /etc/rsyslog.d
        state: directory
        mode: '0755'

    - name: Déployer le fichier de configuration rsyslog pour Graylog (UDP) avec hostname
    copy:
        dest: /etc/rsyslog.d/90-graylog.conf
        content: |
        *.* @{{ graylog_server_ip }}:{{ graylog_udp_port }};RSYSLOG_ForwardFormat
        owner: root
        group: root
        mode: '0644'

    - name: Redémarrer rsyslog pour appliquer la nouvelle config
    service:
        name: rsyslog
        state: restarted
        enabled: yes

    - name: Envoyer un log de test pour apparaître dans Graylog
      shell: logger "Test log initial depuis {{ inventory_hostname }}"
```

---

## 3 : Exécuter le playbook

```
ansible-playbook -i /etc/ansible/hosts /etc/ansible/playbooks/rsyslog_graylog.yml
```

--- 

## 4 : Vérifier la configuration

```
ansible all_linux -m shell -a 'logger "Test Graylog UDP depuis {{ inventory_hostname }}"'
```

Après avoir lancé cette commande sur Ansible, vérifiez sur le Graylog si les logs apparaissent.

---