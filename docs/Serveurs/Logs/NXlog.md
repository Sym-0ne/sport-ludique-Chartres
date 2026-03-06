# Mise en place de NXLog pour l’envoi de log de l'AD et HMAIL vers Graylog

## Objectif

---

L’objectif est de configurer toutes les VM Windows Server de l'infrastructure afin qu’elles envoient leurs logs vers le serveur Graylog via UDP.<br>
Cela permet de centraliser la collecte et la visualisation des logs, d’identifier rapidement les problèmes et de suivre les activités de toutes les VM.<br>

---

## Input

Si l'input n'est pas encore créé vous pouvez allez voir [ici](https://sym-0ne.github.io/sport-ludique-Chartres/Serveurs/Logs/InterfaceWebLog/) à l'étape 3.

## Installer NXLog sur Windows Server

Vous pouvez télécharger gratuitement l'agent NXLog [ici](https://nxlog.co/downloads/nxlog-ce#nxlog-community-edition).

Puis, sélectionnez "Windows", cochez la case "Windowz x86-64" et lancez le téléchargement.

Sur votre machine Windows, lancez l'installation via le package "nxlog-ce-3.2.2329.msi". Suivez l'assistant et effectuez l'installation... La configuration s'effectuera par la suite.

## Configurer NXLog pour Graylog

NXLog étant installé sur la machine, nous pouvons éditer son fichier de configuration situé à l'emplacement suivant :

```
C:\Program Files\nxlog\conf\nxlog.conf
```

En complément de la configuration déjà présente dans le fichier "nxlog.conf", vous devez ajouter ces lignes à la fin :

```
# Récupérer les journaux de l'observateur d'événements
<Input in>
    Module      im_msvistalog
</Input>

# Déclarer le serveur Graylog (selon input)
<Extension gelf>
    Module        xm_gelf
</Extension>

<Output graylog_udp>
    Module        om_udp
    Host          192.168.10.220
    Port	  12201
    OutputType    GELF_UDP
</Output>

# Routage des flux in vers out
 <Route 1>
     Path        in => graylog_udp
 </Route>
```

Sauvegardez les changements et redémarrez le service NXLog à partir d'une console PowerShell ouverte en tant qu'administrateur :

```
Restart-Service nxlog
```

Enfin aller sur l'interface Web de Graylog pour vérifier si les logs sont bien envoyés.