# Playbook Ansible - Déploiement SSH Sécurisé

## Contexte

Ce playbook automatise le durcissement de l'authentification SSH sur l'ensemble des serveurs Linux de l'infrastructure. Il désactive l'authentification par mot de passe et impose l'utilisation de clés publiques, pour les utilisateurs réels (simon, wassim, david) ainsi que pour l'utilisateur de liaison `ansible`.

Les clés publiques des utilisateurs sont stockées sur le serveur `bdd` dans le répertoire `/home/bdd/clefssh`. Le playbook les récupère depuis cette source centrale avant de les déployer sur chaque hôte.

---

## Prérequis

Avant d'exécuter ce playbook, les conditions suivantes doivent être réunies :

- Le fichier d'inventaire `/etc/ansible/hosts` est à jour avec tous les serveurs cibles
- L'utilisateur `ansible` dispose des droits `sudo` sans mot de passe sur tous les hôtes (via `/etc/sudoers.d/ansible`)
- Les clés publiques des utilisateurs sont déposées sur `bdd` dans `/home/bdd/clefssh/`
- La clé publique d'Ansible (`ansible.pub`) est également présente dans ce répertoire

---

## Fichiers de configuration

### Inventaire `/etc/ansible/hosts`

L'inventaire recense tous les serveurs cibles. Chaque hôte déclare son IP, l'utilisateur Ansible de connexion (`ansible_user=ansible`) et l'utilisateur réel du système (`user_reel`).

```ini
[local]
127.0.0.1 ansible_connection=local

[serveurs]
bdd ansible_host=10.10.120.7 user_reel=bdd ansible_user=ansible
reverse-proxy ansible_host=10.10.120.80 ansible_user=ansible user_reel=user
reverse-proxy-sec ansible_host=10.10.120.90 ansible_user=ansible user_reel=user
web ansible_host=10.10.120.11 ansible_user=ansible user_reel=user
dns-autorite ansible_host=10.10.120.8 ansible_user=ansible user_reel=user
dns-autorite-sec ansible_host=10.10.120.18 ansible_user=ansible user_reel=user
dns-resolver ansible_host=10.10.120.9 user_reel=user ansible_user=ansible
dns-resolver-sec ansible_host=10.10.120.19 user_reel=user ansible_user=ansible
ca-autorite ansible_host=10.10.120.12 user_reel=certificat ansible_user=ansible
docker ansible_host=10.10.120.15 ansible_user=ansible user_reel=user
graylog ansible_host=10.10.120.16 ansible_user=ansible user_reel=user
datanode ansible_host=10.10.120.17 ansible_user=ansible user_reel=user
proxmox ansible_host=10.10.120.50 ansible_user=root
shop ansible_host=10.10.120.21 ansible_user=ansible user_reel=user
radius ansible_host=10.10.120.22 ansible_user=ansible user_reel=user
slam2 ansible_host=10.10.120.100 ansible_user=ansible user_reel=user

[all_linux:children]
serveurs

[all_linux:vars]
ansible_python_interpreter=/usr/bin/python3
```

### Sudoers `/etc/sudoers.d/ansible`

L'utilisateur `ansible` doit pouvoir exécuter des commandes en tant que root sans mot de passe sur chaque machine cible. Ce fichier doit être présent sur **chaque serveur de l'infrastructure** avant le premier lancement du playbook.

```
ansible ALL=(ALL) NOPASSWD:ALL
```

La création de ce fichier se fait manuellement lors de l'ajout d'un nouveau serveur, **avant** toute exécution de playbook sur cette machine. Utiliser `visudo` pour éviter les erreurs de syntaxe :

```bash
sudo visudo -f /etc/sudoers.d/ansible
```

Coller la ligne suivante :

```
ansible ALL=(ALL) NOPASSWD:ALL
```

Vérifier que le fichier est bien pris en compte :

```bash
sudo -l -U ansible
```

La sortie doit contenir `(ALL) NOPASSWD: ALL`.

> **Attention** : ne jamais éditer ce fichier avec `nano` ou `vim` directement. Une erreur de syntaxe dans un fichier sudoers peut bloquer tous les accès sudo sur la machine. Toujours passer par `visudo`.

### Configuration SSH résultante `/etc/ssh/sshd_config.d/01-ssh-secure.conf`

C'est le fichier déployé par le playbook sur chaque serveur. Il désactive l'authentification par mot de passe et active uniquement l'authentification par clé :

```
PasswordAuthentication no
PubkeyAuthentication yes
```

---

## Playbook `ssh_secure.yml`

### Variables

| Variable | Valeur | Description |
|---|---|---|
| `clefs_users` | `[simon.pub, wassim.pub, david.pub]` | Clés publiques des utilisateurs réels |
| `clef_ansible` | `ansible.pub` | Clé publique de l'utilisateur ansible |
| `clef_dir` | `/home/bdd/clefssh` | Répertoire source des clés sur bdd |
| `sshd_conf_file` | `/etc/ssh/sshd_config.d/01-keyonly.conf` | Fichier de configuration SSH déployé |

### Déroulement des tâches

Le playbook s'exécute sur le groupe `serveurs` avec `remote_user: ansible` et `gather_facts: no`.

#### 1. Récupération des clés depuis BDD

Les deux premières tâches utilisent `ansible.builtin.slurp` avec `delegate_to: bdd` et `run_once: true` pour lire les fichiers de clés sur le serveur BDD sans les copier localement.

```yaml
- name: Récupérer les clés SSH des utilisateurs depuis la BDD
  ansible.builtin.slurp:
    src: "{{ clef_dir }}/{{ item }}"
  register: ssh_keys_users
  loop: "{{ clefs_users }}"
  delegate_to: bdd
  become: yes
  run_once: true

- name: Récupérer la clé SSH ansible depuis la BDD
  ansible.builtin.slurp:
    src: "{{ clef_dir }}/{{ clef_ansible }}"
  register: ssh_key_ansible
  delegate_to: bdd
  become: yes
  run_once: true
```

#### 2. Configuration de l'utilisateur ansible

Création du répertoire `~/.ssh` pour l'utilisateur `ansible` avec les permissions appropriées, puis ajout de sa clé publique dans `authorized_keys` :

```yaml
- name: Créer ~/.ssh pour l'utilisateur ansible
  ansible.builtin.file:
    path: "/home/ansible/.ssh"
    state: directory
    mode: "0700"
    owner: ansible
    group: ansible

- name: Ajouter uniquement la clé ansible à l'utilisateur ansible
  ansible.posix.authorized_key:
    user: ansible
    key: "{{ ssh_key_ansible.content | b64decode }}"
    state: present
```

#### 3. Configuration de l'utilisateur réel

Même logique pour l'utilisateur réel (défini par `user_reel` dans l'inventaire). Les trois clés (simon, wassim, david) sont ajoutées en boucle :

```yaml
- name: Créer ~/.ssh pour l'utilisateur réel
  ansible.builtin.file:
    path: "/home/{{ user_reel }}/.ssh"
    state: directory
    mode: "0700"
    owner: "{{ user_reel }}"
    group: "{{ user_reel }}"
  become: yes

- name: Ajouter les clés de simon, wassim, david à l'utilisateur réel
  ansible.posix.authorized_key:
    user: "{{ user_reel }}"
    key: "{{ item.content | b64decode }}"
    state: present
  loop: "{{ ssh_keys_users.results }}"
  become: yes
```

#### 4. Durcissement SSH

Création du répertoire `sshd_config.d` si absent, puis dépôt du fichier de configuration qui désactive l'authentification par mot de passe :

```yaml
- name: Créer le répertoire de configuration SSH si nécessaire
  ansible.builtin.file:
    path: /etc/ssh/sshd_config.d
    state: directory
    owner: root
    group: root
    mode: "0755"
  become: yes

- name: Forcer authentification par clé
  ansible.builtin.copy:
    content: |
      PasswordAuthentication no
      PubkeyAuthentication yes
    dest: "{{ sshd_conf_file }}"
    owner: root
    group: root
    mode: "0644"
  become: yes
```

#### 5. Redémarrage SSH

Redémarrage du service SSH pour appliquer la nouvelle configuration :

```yaml
- name: Redémarrer SSH
  ansible.builtin.service:
    name: ssh
    state: restarted
  become: yes
```

---

## Exécution

Depuis le serveur Ansible :

```bash
ansible-playbook /etc/ansible/playbooks/ssh_secure.yml
```

Pour cibler un seul hôte :

```bash
ansible-playbook /etc/ansible/playbooks/ssh_secure.yml --limit dns-autorite
```

Pour effectuer un dry-run avant d'appliquer :

```bash
ansible-playbook /etc/ansible/playbooks/ssh_secure.yml --check
```

---

## Vérification post-déploiement

Sur un serveur cible, vérifier que le fichier a bien été créé :

```bash
sudo cat /etc/ssh/sshd_config.d/01-ssh-secure.conf
```

Résultat attendu :

```
PasswordAuthentication no
PubkeyAuthentication yes
```

Tester que la connexion par clé fonctionne depuis le serveur Ansible :

```bash
ansible serveurs -m ping
```

Tenter une connexion par mot de passe depuis un poste tiers doit être refusée.

---

## Points d'attention

- Le serveur `proxmox` utilise `ansible_user=root` et n'a pas de variable `user_reel`. Les tâches utilisant `user_reel` sont commentées avec un `when: inventory_hostname != 'bdd'` ou similaire pour les cas particuliers.
- Le serveur `bdd` est à la fois source des clés (via `delegate_to`) et hôte cible du playbook. Les tâches de déploiement sur `bdd` lui-même sont actives, les conditions `when` correspondantes ayant été désactivées.
- La valeur `run_once: true` sur les tâches `slurp` garantit que la lecture des clés sur BDD n'est effectuée qu'une seule fois, même si le playbook tourne sur de nombreux hôtes.

---

## Playbook complet `ssh_secure.yml`

Fichier prêt à copier-coller dans `/etc/ansible/playbooks/ssh_secure.yml` :

```yaml
---
- hosts: serveurs
  gather_facts: no
  remote_user: ansible
  vars:
    clefs_users:
      - simon.pub
      - wassim.pub
      - david.pub
    clef_ansible: ansible.pub
    clef_dir: /home/bdd/clefssh
    sshd_conf_file: /etc/ssh/sshd_config.d/01-keyonly.conf

  tasks:

    # -------------------------------------------------------
    # Récupération des clés depuis le serveur BDD
    # -------------------------------------------------------

    - name: Récupérer les clés SSH des utilisateurs depuis la BDD
      ansible.builtin.slurp:
        src: "{{ clef_dir }}/{{ item }}"
      register: ssh_keys_users
      loop: "{{ clefs_users }}"
      delegate_to: bdd
      become: yes
      run_once: true

    - name: Récupérer la clé SSH ansible depuis la BDD
      ansible.builtin.slurp:
        src: "{{ clef_dir }}/{{ clef_ansible }}"
      register: ssh_key_ansible
      delegate_to: bdd
      become: yes
      run_once: true

    # -------------------------------------------------------
    # Configuration pour l'utilisateur ansible
    # -------------------------------------------------------

    - name: Créer ~/.ssh pour l'utilisateur ansible
      ansible.builtin.file:
        path: "/home/ansible/.ssh"
        state: directory
        mode: "0700"
        owner: ansible
        group: ansible

    - name: Ajouter uniquement la clé ansible à l'utilisateur ansible
      ansible.posix.authorized_key:
        user: ansible
        key: "{{ ssh_key_ansible.content | b64decode }}"
        state: present

    # -------------------------------------------------------
    # Configuration pour l'utilisateur réel - via sudo depuis ansible
    # -------------------------------------------------------

    - name: Créer ~/.ssh pour l'utilisateur réel
      ansible.builtin.file:
        path: "/home/{{ user_reel }}/.ssh"
        state: directory
        mode: "0700"
        owner: "{{ user_reel }}"
        group: "{{ user_reel }}"
      become: yes

    - name: Ajouter les clés de simon, wassim, david à l'utilisateur réel
      ansible.posix.authorized_key:
        user: "{{ user_reel }}"
        key: "{{ item.content | b64decode }}"
        state: present
      loop: "{{ ssh_keys_users.results }}"
      become: yes

    # -------------------------------------------------------
    # Durcissement SSH
    # -------------------------------------------------------

    - name: Créer le répertoire de configuration SSH si nécessaire
      ansible.builtin.file:
        path: /etc/ssh/sshd_config.d
        state: directory
        owner: root
        group: root
        mode: "0755"
      become: yes

    - name: Forcer authentification par clé
      ansible.builtin.copy:
        content: |
          PasswordAuthentication no
          PubkeyAuthentication yes
        dest: "{{ sshd_conf_file }}"
        owner: root
        group: root
        mode: "0644"
      become: yes

    # -------------------------------------------------------
    # Redémarrage SSH
    # -------------------------------------------------------

    - name: Redémarrer SSH
      ansible.builtin.service:
        name: ssh
        state: restarted
      become: yes
```
