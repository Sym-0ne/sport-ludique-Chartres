# Deploiement de l'agent GLPI sur toutes les VM Linux via Ansible

## 1. Prérequis

Nous avons besoin de :

✔ L’URL du serveur GLPI (ex : http://10.10.120.15:2000)<br>
✔ Le lien de téléchargement de l’agent GLPI Linux<br>
✔ Le port utilisé par l’agent (par défaut 62354)<br>

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
- name: Installer et configurer l'agent GLPI sur toutes les VM Linux
  hosts: all_linux
  become: yes

  vars:
    glpi_release_folder: "1.7"
    glpi_agent_filename: "glpi-agent_1.7-1_all.deb"
    glpi_agent_url: "https://github.com/glpi-project/glpi-agent/releases/download/{{ glpi_release_folder }}/{{ glpi_a>
    glpi_server_url: "http://10.10.120.15:2000/front/inventory.php"

  tasks:

    - name: 1. Mettre à jour le cache APT
      ansible.builtin.apt:
        update_cache: yes
      tags: update

    - name: 2. Télécharger le package GLPI Agent (DEB)
      ansible.builtin.get_url:
        url: "{{ glpi_agent_url }}"
        dest: "/tmp/glpi-agent.deb"
        mode: '0644'

    - name: 3. Installer le package GLPI Agent
      ansible.builtin.apt:
        deb: "/tmp/glpi-agent.deb"
        state: present
        update_cache: yes
      notify:
        - Démarrer et Activer GLPI Agent

    # --- TÂCHES DE NETTOYAGE ET DE CONFIGURATION (NO AUTH) ---

    - name: 4a. Configurer l'URL du serveur GLPI dans agent.cfg
      ansible.builtin.lineinfile:
        path: /etc/glpi-agent/agent.cfg
        regexp: '^server ='
        line: 'server = {{ glpi_server_url }}'
        state: present
      notify: Redémarrer GLPI Agent pour appliquer la config

    - name: 4b. Supprimer l'ancienne configuration de login (pour Aucune Authentification)
      ansible.builtin.lineinfile:
        path: /etc/glpi-agent/agent.cfg
        regexp: '^user ='
        state: absent
      notify: Redémarrer GLPI Agent pour appliquer la config


    - name: 4c. Supprimer l'ancienne configuration de mot de passe
      ansible.builtin.lineinfile:
        path: /etc/glpi-agent/agent.cfg
        regexp: '^password ='
        state: absent
      notify: Redémarrer GLPI Agent pour appliquer la config

  handlers:

    # 1. Handler pour démarrer l'agent après l'installation
    - name: Démarrer et Activer GLPI Agent
      ansible.builtin.systemd:
        name: glpi-agent
        enabled: yes
        state: started

    # 2. Handler pour redémarrer l'agent après une modification de config (celui qui manquait)
    - name: Redémarrer GLPI Agent pour appliquer la config
      ansible.builtin.systemd:
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

Forcer l'inventaire si après le lancement les machines n'apparaissent pas dans le GLPI.

```
ansible all_linux -b -m shell -a "sudo glpi-agent --server http://10.10.120.15:2000/front/inventory.php --force"
```

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