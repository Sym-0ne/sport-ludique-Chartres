# Documentation PAT pour DNS avec une seule IP publique

## Contexte.

| Serveur DNS                  | IP interne      | Port interne | IP publique      | Port externe | Protocole |
|-------------------------------|----------------|-------------|-----------------|--------------|-----------|
| DNS autoritaire               | 172.28.62.1    | 53          | 183.44.28.1     | 53           | TCP/UDP   |
| Résolveur DNS                 | 172.28.33.4    | 53          | 183.44.28.1     | 53         | TCP/UDP   |


Objectif : Permettre l’accès depuis Internet aux deux serveurs DNS via une seule IP publique, en utilisant des ports externes différents pour le résolveur.

## Étape 2 : Configurer le PAT pour le DNS autoritaire

Le DNS autoritaire interne utilise le port standard 53 pour TCP et UDP.
Le trafic provenant d’Internet sur le port 53 sera redirigé vers le serveur interne.

```
ip nat inside source static tcp 172.28.62.1 53 183.44.28.1 53
ip nat inside source static udp 172.28.62.1 53 183.44.28.1 53
```

**Explication :**

Tcp et udp : on ouvre les deux protocoles utilisés par DNS.

172.28.62.1 53 : IP et port interne du DNS autoritaire.
172.28.33.4 53 : IP et port interne du DNS Resolveur.
183.44.28.1 53 : IP publique et port externe visible depuis Internet.

## Étape 3 : Vérification

Vérifier que NAT est bien configuré :
```
show ip nat translations
```

### Schema conceptuel : 

               Internet
                   |
           IP publique : 183.44.28.1
           -------------------------
          |                         |
   Port 53 (TCP/UDP)          Port 1053 (TCP/UDP)
          |                         |
  DNS autoritaire             Résolveur DNS
  172.28.62.1                 172.28.33.4
