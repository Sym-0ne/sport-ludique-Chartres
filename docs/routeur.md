# Configuration du routeur Cisco 1921 #
## Configuration initial
1. Première étape : Réinitialiser le routeur a sa configuration d'origine sans le mot de passe admin. 

```
#Démarer le routeur et presser CTR+BREAK pour être en mode "rommon"
confreg 0x2142
reset
#Le routeur reboot sans configuration
enable
configure terminal
config-register 0x2102 #Permet de remettre le switch sur le bon registre
end
write
```
2. Changer le Nom du routeur 
```
enable 
config
hostname CHA-R1
```
## Configuration des interfaces 
1. Configurer la première interface Gigabit0/0
```
interface GigabitEthernet0/0
 ip address 183.44.28.1 255.255.255.252
 ip nat outside
 no shutdown
```
2. Configurer la deuxième interface Gigabit0/1
```
interface GigabitEthernet0/1
 no shutdown
```
<div class="annotate" markdown>
> Info(1)
</div>
1. A noter que le port du switch coeur de réseau relié au routeur doit être configurer en mode trunk 

## Ajout des vlans
1. Ajout du Vlan de management
```
interface GigabitEthernet0/1.120
 encapsulation dot1Q 120
 ip address 10.10.120.20 255.255.255.0
 ip access-group 100 out
```
2. Ajout du Vlan d'interconnection
```
interface GigabitEthernet0/1.223
 encapsulation dot1Q 223
 ip address 172.28.63.2 255.255.255.0
 ip nat inside
```
## Activation du SSH 
1. Génération des clef RSA 
```
crypto key generate rsa
2048
```
2. Création d'un compte local
```
username admin privilege 15 secret MonMotDePasse
```
3. Activation du SSH sur les lignes VTY 
```
line vty 0 4
login local 
transport input ssh 
```
4. Selection de la bonne version SSH 
```
ip ssh version 2
```
## Configuration du routage

1. Activation du NAT 
```
ip nat inside source list 1 interface GigabitEthernet0/0 overload 
```
2. Création des routes de retour
```
ip route 172.28.32.0 255.255.255.0 172.28.63.1
ip route 172.28.33.0 255.255.255.0 172.28.63.1
ip route 172.28.35.0 255.255.255.0 172.28.63.1
ip route 172.28.63.128 255.255.255.128 172.28.63.1 #LAN2DMZ
ip route 172.28.62.0 255.255.255.0 172.28.63.1 #DMZ
```
3. Création de la route par défaut
```
ip route 0.0.0.0 0.0.0.0 183.44.28.2
```

## Mise en place des ACL
1. Creation de l'acces list contenant les réseaux qui auront accès a internet
```
access-list 1 permit 172.28.32.0 0.0.31.255 
```
<div class="annotate" markdown>
> Info (1)
</div>
1. A noter que le masque inversé renvoie vers le réseau 172.28.32.0/19 ce qui correspond a la plage de VLAN qui nous est atribué. 

2. Creation de l'access list qui vas bloquer l'accès a internet au réseau de management 
```
access-list 100 deny ip 10.10.120.0 0.0.0.255 any
access-list 100 permit ip any any
```
## Redirection DNS

La redirection DNS permettra aux machine n'ayant pas de serveur DNS dans leur configuration à pouvoir utiliser le DNS interne de Chartres

```
ip dns server 
ip name-server 172.28.33.2
ip domain-lookup
```
## Protocole HSRP
Le HSRP est un protocole qui permet de crée une adresse VIP (Virtual IP) qui permet la haute disponibilité ente 2 routeurs. 

Dans notre cas nous allon mettre en place le HSRP entre nos deux routeurs R1 et R2, chaque routeur est connecter a une FAI différente