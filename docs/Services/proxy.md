Comment faire une CA interne sur un SN210 (SNS v4.3.29)

Connecte-toi à l’interface Web du SN210 en admin.

Accéder à la configuration SSL / PKI :

Va dans Configuration → Protection applicative → Protocoles.

Sélectionne le protocole SSL, puis clique sur Configuration globale (ou “Go to global configuration”) pour accéder à la partie proxy SSL. 
Stormshield Documentation
+1

Définir l’autorité de signature (signing CA) pour le proxy SSL :

Dans l’onglet Proxy, tu vas voir une section “Génération des certificats pour émuler le serveur SSL” (Generate certificates to emulate the SSL‑server). 
Stormshield Documentation

Là, tu peux choisir une CA signataire, définir un mot de passe, et paramétrer la durée de vie des certificats émis par cette CA. 
Stormshield Documentation
+1

Si tu ne définis pas de CA personnalisée, Stormshield utilise sa “default authority” par défaut.

Gérer les autorités de confiance :

Toujours dans la configuration SSL, va dans l’onglet Autorités de certification personnalisées (Customized certificate authorities) : tu peux y ajouter des CA privées que tu souhaites que le SN210 “croie”. 
Stormshield Documentation

Ensuite, dans l’onglet Autorités de certification publiques, tu peux activer / désactiver les CA publiques que tu veux juger “de confiance” pour le proxy SSL. 
Stormshield Documentation

Et dans l’onglet Certificats de confiance (Trusted certificates), ajoute les certificats (serveurs) que tu veux explicitement faire confiance. 
Stormshield Documentation

Appliquer la CA signataire au filtrage SSL :

Une fois ta CA interne / “signing authority” configurée, il faut que le filtrage SSL l’utilise. Assure-toi qu’elle est bien sélectionnée dans la configuration SSL du proxy.

Clique sur Appliquer (“Apply”) pour enregistrer la config. 
Stormshield Documentation

Configurer la politique de filtrage SSL :

Va dans Configuration → Politique de sécurité → Filtrage SSL. 
Stormshield Documentation

Crée ou modifie une politique SSL : tu vas définir des règles selon des catégories d’URL / de certificats (CN). 
Stormshield Documentation

Action “Déchiffrer” pour les catégories où tu veux intercepter / inspecter. 
Stormshield Documentation

Action “Passer sans déchiffrer” ou “Bloquer sans déchiffrer” selon les cas (ex : certains sites sensibles à laisser passer sans décryptage) 
Stormshield Documentation

Assure-toi de mettre les règles dans le bon ordre, car le SN210 les évalue séquentiellement. 
Stormshield Documentation

Exporter la CA publique :

Une fois la CA signataire configurée, exporte son certificat racine depuis le SN210 (certificat public de la CA) pour l’installer sur les postes clients.

L’interface Web devrait avoir une option “Exporter certificat CA” (ou équivalent) dans la page des certificats / PKI.

Installer la CA sur les postes clients :

Sur chaque machine (Windows, Linux, mac, etc.), installe le certificat racine exporté comme autorité de certification racine de confiance.

Cela permet aux navigateurs / aux systèmes d’accepter les certificats “faux” / “interceptés” générés par le proxy SSL du SN210.

Tester le déchiffrement SSL :

Depuis un poste client, accède à des sites HTTPS.

Vérifie :

Si le site charge correctement.

Dans le navigateur, regarde le certificat : il doit être signé par ta CA interne (ex : SN210-CA).

Sur le SN210, consulte les logs de filtrage SSL pour voir les connexions inspectées.