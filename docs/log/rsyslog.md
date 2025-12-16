---

```sudo apt install rsyslog
```

---

```
---
- name: Configurer rsyslog pour Graylog
  hosts: linux_servers
  become: true
  tasks:

    - name: Déployer le fichier de configuration rsyslog
      copy:
        dest: /etc/rsyslog.d/90-graylog.conf
        content: |
          *.* @10.10.120.16:514   # UDP
          # Pour TCP, remplacer par @@10.10.120.16:514
        owner: root
        group: root
        mode: '0644'

    - name: Redémarrer rsyslog
      systemd:
        name: rsyslog
        state: restarted
        enabled: yes
```

---

```
ansible-playbook -i hosts push_rsyslog_graylog.yml
```

--- 

test :

```
logger "Test log vers Graylog"
```

---