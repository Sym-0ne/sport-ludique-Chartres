# Documentation PAT pour DNS avec une seule IP publique

**Objectif :** Permettre l’accès depuis Internet aux deux serveurs DNS via une seule IP publique, en utilisant des ports externes différents pour le résolveur.

## 🔧 1. Configurer le PAT 

Le DNS utilise le port standard 53 pour TCP et UDP.
Le trafic provenant d’Internet sur le port 53 sera redirigé vers le serveur.

### Configuration R1 

```
ip nat inside source static tcp 172.28.62.1 53 183.44.28.1 53
ip nat inside source static udp 172.28.62.1 53 183.44.28.1 53
```

### Configuration R2

```
ip nat inside source static tcp 172.28.62.1 53 221.87.128.2 53
ip nat inside source static udp 172.28.62.1 53 221.87.128.2 53
```

**Explication :**

Tcp et udp : on ouvre les deux protocoles utilisés par DNS.<br>

- 172.28.62.1 53 : IP et port interne du DNS autoritaire.
- 183.44.28.1 53 : IP publique et port externe visible depuis Internet sur le R1
- 221.87.128.1 53 : IP publique et port externe visible depuis Internet sur le R2

## ✅ 2. Vérification

Vérifier que NAT est bien configuré :
```
show ip nat translations
```

