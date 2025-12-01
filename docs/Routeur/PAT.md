# Configuration PAT sur R1 et R2 

---

## 1. Configurer le PAT 

Dans notre cas, nous avons besoin du PAT pour la redirection vers le reverse-proxy et le DNS autoritatif pour les requêtes provenant de l’extérieur.
<div class="annotate" markdown>
(1)
</div>
1. Les requêtes venant du réseau interne sont directement résolues par le DNS autoritatif.

---

### 1.1 Configuration R1 

```
ip nat inside source static tcp 172.28.62.1 53 183.44.28.1 53
ip nat inside source static udp 172.28.62.1 53 183.44.28.1 53
ip nat inside source static tcp 172.28.62.5 80 183.44.28.1 80
ip nat inside source static tcp 172.28.62.5 443 183.44.28.1 443
```

---

### 1.2 Configuration R2

```
ip nat inside source static tcp 172.28.62.11 53 221.87.128.2 53
ip nat inside source static udp 172.28.62.11 53 221.87.128.2 53
ip nat inside source static tcp 172.28.62.5 80 221.87.128.2 80
ip nat inside source static tcp 172.28.62.5 443 221.87.128.2 443
```

---

## 2. Vérification

Vérifier que le NAT est correctement configuré :
```
show ip nat translations
```
Il devrais en ressortir ceci pour R2 par exemple: 
```
Pro Inside global
tcp 221.87.128.2:80     172.28.62.11:53
tcp 221.87.128.2:443    172.28.62.5:80
tcp 221.87.128.2:53     172.28.62.5:80
udp 221.87.128.2:53     172.28.62.11:53
```

---