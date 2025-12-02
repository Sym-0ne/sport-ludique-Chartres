# Deploiement de l'agent GLPI sur toutes les VM Linux via Ansible

## 1. Prérequis

Nous avons besoin de :

✔ L’URL du serveur GLPI (ex : http://10.10.120.15:2000)
✔ Le lien de téléchargement de l’agent GLPI Linux 
✔ Le port utilisé par l’agent (par défaut 62354) 

---

## 2. Créer un dossier dédié aux playbooks

Sur ton serveur Ansible :

```
sudo mkdir -p /etc/ansible/playbooks
cd /etc/ansible/playbooks
```

---

## 3. Créer le playbook d’installation GLPI

Crée le fichier :

```
sudo nano /etc/ansible/playbooks/install_glpi_agent.yml
```

Colle ce playbook prêt à l’emploi :

📌 install_glpi_agent.yml (copie EXACTE à mettre dans le fichier)

```
---
- name: Installer l'agent GLPI sur toutes les VM Linux
  hosts: all_linux
  become: yes

  vars:
    glpi_agent_version: "1.7"
    glpi_agent_url: "https://github.com/glpi-project/glpi-agent/releases/download/${glpi_agent_version}/glpi-agent_${glpi_agent_version}_all.deb"

  tasks:

    - name: Mettre à jour les dépôts APT
      apt:
        update_cache: yes

    - name: Télécharger le package GLPI Agent
      get_url:
        url: "{{ glpi_agent_url }}"
        dest: /tmp/glpi-agent.deb
        mode: '0644'

    - name: Installer le package GLPI Agent
      apt:
        deb: /tmp/glpi-agent.deb
        state: present

    - name: Activer le service GLPI Agent
      systemd:
        name: glpi-agent
        enabled: yes
        state: restarted

```

---

## 4. Lancer l’installation sur TOUTES tes VM

Depuis le serveur Ansible :

```
ansible-playbook /etc/ansible/playbooks/install_glpi_agent.yml
```

Si tout est OK, tu verras des lignes changed=true un peu partout.

---

## 5. Vérifier que l’agent fonctionne

Exécute :

```
ansible all_linux -a "systemctl status glpi-agent" -b
```

Tu dois voir quelque chose comme :

```
Active: active (running)
```

---