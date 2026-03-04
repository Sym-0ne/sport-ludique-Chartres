# Prévol Login

Lorsque vous vous connectez à Graylog pour la première fois, vous devez utiliser les informations d'identification initiales pour l'interface Web Graylog, qui peuvent être trouvées dans le fichier journal après avoir démarré le service Graylog. 

Pour afficher votre mot de passe initial et les instructions incluses dans le fichier journal, entrez la commande suivante :

```
tail /var/log/graylog-server/server.log
```

# Accéder à l'interface Web

Après avoir terminé votre connexion prévol, vous pouvez accéder à l'interface Web avec vos identifiants habituels :

- Ouvrez un navigateur compatible et accédez à l'URL https://xxx.xxx.xxx.xxx:9000. Remplacez l'adresse IP de votre serveur Graylog.

- Connectez-vous en tant qu'administrateur et utilisez le secret de mot de passe que vous avez créé lorsque vous avez installé Graylog.

![alt text](graylog.png)
