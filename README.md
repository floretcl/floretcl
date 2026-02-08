# ABOUT

- 👋 Hi, I’m Clément Floret. I'm from France🇫🇷.
- 📫 How to reach me clement.floret@protonmail.com

# NOTES

## Commande CISCO IOS Routeur et Switch

### 1. MODES ET ACCÈS GÉNÉRAUX
- enable : Accès mode privilégié (Prompt #).
- configure terminal (ou conf t) : Mode configuration globale.
- exit : Retour au niveau précédent.
- end (ou Ctrl+Z) : Retour direct au mode privilégié.
- no [X]  : Désactive une configuration.
- copy running-config startup-config (ou wr) : Sauvegarder en NVRAM.
  
### 2. CONFIGURATION DE BASE & SÉCURITÉ
#### Système et Identification
- hostname [NOM] : Nommer l'équipement.
- banner motd #[MESSAGE]# : Message d'avertissement légal.
- no ip domain-lookup : Désactive la recherche DNS (évite les blocage sur erreurs de frappe).
- service password-encryption : Chiffre les mots de passe dans la config.
#### Sécurisation des accès
- enable secret [MDP] : Mot de passe du mode privilégié (chiffré).
- security passwords min-length [N] : Longueur minimale des mots de passe.
- username [NOM] secret [MDP] : Créer un utilisateur local.
- login block-for [DUREE BLOCAGE SEC] attempts [NB ESSAIS] within [DUREE ESSAIS SEC] : Protection contre les attaques brute-force.
#### Lignes (Console et Distante)
- line console [0] : Configuration du port physique.
- line vty [0 4] : Configuration Telnet/SSH.
- password [MDP] puis login : Authentification simple.
- login local : Utilise les comptes username locaux.
- transport input [ssh | telnet | all | none] : Choix du protocole.
- exec-timeout [MIN] [SEC] : Temps d'inactivité avant déconnexion.

### 3. CONFIGURATION IP & INTERFACES
#### IPv4 & IPv6
- interface [TYPE/NUMERO] : Entrer dans la config du port (ex: int g0/0).
- interface range [DEBUT]-[FIN] : Configurer un groupe de ports (ex: int range f0/1-24).
- description [TEXTE] : Label de l'interface.
- ip address [IP] [MASQUE] : Assigner une IPv4.
- ipv6 address [IP/PREFIX] [LINK-LOCAL] : Assigner une IPv6 global ou local
- ipv6 unicast-routing : Requis sur routeur pour acheminer l'IPv6.
- no shutdown : Activer l'interface.
- duplex [auto | full | half] : Mode de transmission.
#### Spécifique Switch (Commutateur)
- vlan [ID]name [NOM] : Créer un VLAN.
- switchport mode access puisswitchport access vlan [ID] : Assigner un port à un VLAN.
- switchport mode trunk : Faire passer plusieurs VLAN sur un port (802.1Q).
- interface vlan 1 puisip address [IP] [MASQUE] : IP de management du Switch.
- ip default-gateway [IP PASSERELLE] : Passerelle par défaut du Switch.
  
### 4. ROUTAGE ET SERVICES
- ip route [IP_DEST] [MASQUE] [IP_PASSERELLE] : Route statique IPv4.
- no ip http server : Désactiver l'interface web (sécurité).
- no ip directed-broadcasts : Désactiver les diffusions dirigées (autorise 255.255.255.255 seulement).
- auto secure : Assistant de sécurisation global.
#### Pré-requis SSH
1. ip domain-name [NOM DE DOMAINE] : Défini un nom de domaine
2. crypto key generate rsa general-keys modulus [1024/2048] : Définin une clé de chiffrement SSH
3. line vty  puis login local puis transport input ssh.
   
### 5. VÉRIFICATION ET DIAGNOSTIC (SHOW)
#### États et Performances
- show [ip/ipv6] interface brief : État résumé des ports (UP/DOWN) et IPs.
- show [ip/ipv6] interfaces : État des ports (UP/DOWN) et IPs.
- show [ip/ipv6] interface [] : État d'un port (UP/DOWN) et IP.
- show interface [] : État de toutes les interfaces.
- show ip ports all : Liste des ports ouverts (anciennement : show control-panel host open-ports).
- show running-config : Configuration actuelle en RAM.
- show ip route : Table de routage.
- show arp : Cache ARP (correspondances IP/MAC).
- show vlan brief : (Switch uniquement) Liste des VLANs et ports.
- show protocols : Protocoles actifs.
- show version : Infos matériel, logiciel et registre.
#### Voisinage (CDP)
- show cdp neighbors [detail] : Voir les voisins (même sans ping réussi) (Cisco).
- no cdp run : Désactiver CDP globalement.
- no cdp enable : Désactiver CDP sur une interface seule.
#### Debugging
- debug [NOM_PROTOCOLE] : Surveillance temps réel (ex: debug ip icmp). 
- undebug all (ou no debug all) : Arrêter tous les debugs.
- terminal monitor : Affiche les logs/debugs sur une session distante (SSH/Telnet).
- terminal no monitor : Désactive l'affichage des logs/debugs.

### CONFIGURATION COMPLÈTE : ROUTEUR CISCO
#### 1. Sécurisation et Accès
```
enable
configure terminal
hostname R1
no ip domain-lookup                 ! Évite les blocages sur erreurs de frappe
enable secret class                 ! Mot de passe mode privilégié (chiffré)
service password-encryption         ! Chiffrer tous les mots de passe visibles
banner motd #ACCES PRIVE - R1#      ! Message d'avertissement

```
#### 2. Configuration SSH
```
ip domain-name lab.local            ! Requis pour générer les clés
crypto key generate rsa             ! Choisir 2048 bits
username admin secret cisco         ! Utilisateur local pour SSH
line vty 0 4                        ! Accès à distance
 transport input ssh                ! Désactive Telnet (insécure)
 login local                        ! Utilise la base de données locale
 exec-timeout 5 0                   ! Déconnexion auto après 5 min
exit

```
#### 3. Interfaces et Routage (IPv4/IPv6)
```
ipv6 unicast-routing                ! Activer le routage IPv6 globalement
interface GigabitEthernet0/0/0
 description LIEN_LAN
 ip address 192.168.1.254 255.255.255.0
 ipv6 address fe80::1 link-local
 ipv6 address 2001:db8:acad:1::1/64
 no shutdown                        ! ACTIVER LE PORT
exit

! Route par défaut vers le fournisseur d'accès (Internet)
ip route 0.0.0.0 0.0.0.0 [IP_SUIVANT]

```

### CONFIGURATION COMPLÈTE : SWITCH CISCO
#### 1. Sécurité de base
```
enable
configure terminal
hostname SW1
enable secret class
line console 0
 logging synchronous                ! Empêche les logs de couper votre saisie
exit

```
#### 2. Gestion des VLANs
```
vlan 10
 name COMPTA
vlan 20
 name INFORMATIQUE
exit

```
#### 3. Assignation des Ports
```
! Ports d'accès (Ordinateurs, Imprimantes)
interface range fastEthernet 0/1-10
 switchport mode access
 switchport access vlan 10
 no shutdown

! Port Trunk (Lien vers Routeur ou autre Switch)
interface fastEthernet 0/24
 switchport mode trunk
 description TRUNK_VERS_R1

```
#### 4. Adresse de Management (SVI)
```
! Permet de prendre la main sur le switch à distance via son IP
interface vlan 1
 ip address 192.168.1.10 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.1.254    ! Passerelle pour joindre le switch hors du LAN

```


## OS WINDOWS : COMMANDES ET POWERSHELL 

### 1. LIGNE DE COMMANDE (CMD)
#### Réseau et Connectivité
- ipconfig /all : Configuration complète (MAC, DNS, DHCP).
- ipconfig /displaydns : Affiche les entrées DNS mises en cache.
- ipconfig /release : Réinitialise (libère) l'adresse IP actuelle.
- ipconfig /renew : Demande une nouvelle adresse IP au serveur DHCP.
- nslookup [Nom] : Test de résolution de nom DNS.
- tracert [IP/Nom] : Trace la route vers une destination.
- route print ou netstat -r : Affiche la table de routage.
- netstat -n : Affiche les connexions actives (adresses numériques).
- arp -a : Affiche la mémoire cache ARP (liaisons IP/MAC).
- netsh interface ip delete arpcache : Efface le cache ARP.
#### Système et GPO
- gpupdate /force : Force l'application immédiate des stratégies de groupe.
- gpresult /r : Affiche le résumé des GPO appliquées (RSOP).
- sfc /scannow : Analyse et répare l'intégrité des fichiers système.
- net user [user] /domain : Vérifie les informations d'un compte dans l'AD.

### 2. POWERSHELL : ESSENTIELS & SYSTÈME
#### Gestion de l'exécution et Scripts
- Get-ExecutionPolicy : Voir le niveau de sécurité des scripts.
- Set-ExecutionPolicy [Restricted | Unrestricted | AllSigned | RemoteSigned] : Modifier les droits d'exécution.
- Lancer un script : .\script.ps1 ou powershell -File "chemin/vers/script.ps1".
#### Aide et Découverte
- Get-Help [Commande] : Afficher l'aide complète.
- Get-Command *[Mot]* : Trouver une commande par mot-clé.
- Get-History : Voir l'historique des commandes saisies.
#### Processus et Services
- Get-Service : Lister les services.
- Start-Service -Name [Nom] / Stop-Service : Gérer les services.
- Get-Process : Lister les processus actifs.
- Stop-Process -ID [N] ou -Name [Nom] : Arrêter un processus.
#### Diagnostic et Rapports
- Start-Transcript : Enregistre toute la session console dans un fichier texte.
- Test-NetConnection -ComputerName [IP] -Port [N] : Vérifie l'ouverture d'un port (TNC).
- ConvertTo-HTML :
	- Exemple : Get-Process | Select-Object Name, Status | ConvertTo-HTML > C:\processes.html (Génère un rapport web des processus).
   
### 3. ADMINISTRATION ACTIVE DIRECTORY (Module RSAT)
#### Gestion des Utilisateurs
- Get-ADUser -Identity [Login] -Properties * : Détails complets d'un utilisateur.
- New-ADUser : Créer un utilisateur.
	- Ex : New-ADUser -Name "Jean" -SamAccountName jdupont -Path "OU=Users,DC=lab,DC=local" -Enabled $true
- Set-ADUser -Identity [Login] -Replace @{title="Technicien"} : Modifier un attribut.
- Unlock-ADAccount -Identity [Login] : Déverrouiller un compte bloqué.
- Set-ADAccountPassword -Identity [Login] -Reset : Réinitialiser le mot de passe.
#### Gestion des Groupes et OU
- Get-ADGroup -Filter 'Name -like "*Compta*"' : Trouver un groupe.
- Add-ADGroupMember -Identity "[Groupe]" -Members [Login] : Ajouter un membre.
- Get-ADGroupMember -Identity "[Groupe]" : Lister les membres d'un groupe.
- Get-ADOrganizationalUnit -Filter * : Lister toutes les Unités d'Organisation.
- Get-ADComputer -Filter * : Lister les ordinateurs du domaine.
  
### 4. LOGIQUE POWERSHELL : PIPES ET FILTRES
PowerShell traite des objets et non du texte simple. On utilise le symbole | pour passer l'objet à la commande suivante.
- Le Filtre ( Where-Object) :
	- Get-Service | Where-Object {$_.Status -eq "Stopped"} (Affiche les services arrêtés).
- La Sélection ( Select-Object) :
	- Get-ADUser -Filter * | Select-Object Name, SamAccountName (Affiche uniquement deux colonnes).
- Le Tri ( Sort-Object) :
	- Get-Process | Sort-Object CPU -Descending (Classe par consommation CPU).
   
### 5. OUTILS GPO & DIAGNOSTIC AVANCÉ
- gpresult /h report.html : Génère un rapport HTML détaillé des GPO (très visuel).
- rsop.msc : Console graphique du "Jeu de stratégie résultant".

### 6. RÉSEAU & INTEROPÉRABILITÉ
- Infos IP : ipconfig /all
- Passerelle : route print
- Libérer DHCP : ipconfig /release
- Renouveler DHCP : ipconfig /renew
- DNS : nslookup
- Route : tracert
- Ports ouverts : netstat -an
- Test port TCP : tnc [IP] -p [Port] (PS)


## COMMANDES SYSTÈME UNIX / LINUX - BASH

### 1. SYSTÈME UNIX / LINUX & BASH
#### Commandes de Base
- echo $SHELL : Obtenir le shell actuel.
- whoami / id : Nom d'utilisateur courant / Détails (UID, GID).
- cd : Changer de répertoire ( ~ : home, .. : parent, - : précédent).
- ls -al : Lister tout (fichiers cachés + détails).
- touch / mkdir : Créer un fichier vide / un répertoire.
- rm -ri : Supprimer (récursif + confirmation).
- more / tail -f : Lire un fichier / Suivre les logs en temps réel.
- grep [chaîne] [fichier] : Rechercher un texte.
- find [nom] : Rechercher un fichier/dossier.
- ps aux / kill [PID] : Lister / Stopper les processus.
#### Compression (Tar)
- tar -cvzf output.tar.gz input : Compresser (c: create, z: gzip).
- tar -xvzf archive.tar.gz : Extraire (x: extract).
#### Gestion des Utilisateurs et Groupes
- su - : Passer en root.
- sudo useradd -m [user] : Créer utilisateur + home.
- sudo passwd [user] : Définir/Changer le mot de passe.
- sudo groupadd [groupe] / sudo usermod -aG [groupe] [user] : Gérer les groupes.
- getent group [groupe] : Lister les membres d'un groupe.
- sudo userdel [user] : Supprimer un utilisateur.
- sudo visudo : Seule méthode sûre pour modifier /etc/sudoers.
#### Permissions et Droits
- Octal : r=4, w=2, x=1 (7=rwx, 5=rx).
- chmod [droits] [fichier] : Modifier permissions (ex: 755 ou u+rwx).
- chown [user]:[group] [fichier] : Modifier propriétaire et groupe.
- Bits spéciaux :
	- s (setuid/gid) : Exécute avec droits propriétaire.
	- t (sticky bit) : Seul le propriétaire peut supprimer son fichier dans un dossier commun.
- Umask : Droits par défaut.
	- Calcul : 777 (rép) ou 666 (fichier) - Valeur Umask.
	- Ex: umask 022 -> Rép: 755 / Fich: 644.
   
### 2. SCRIPTING BASH & AUTOMATISATION
#### Caractères Spéciaux & Logique
- ; : Séparateur de commandes.
- | : Pipe (Résultat A devient entrée de B).
- && : Exécute B si A réussit.
- || : Exécute B si A échoue.
#### Variables et Fonctions
- Déclaration : nom=valeur (pas d'espace).
- Commande : var=$(commande).
- Utilisation : $var.
- Fonction : function nom { commandes; }.
#### Structures de Contrôle
- Condition :
	- If
		```
	 	if [ condition ]; then
	 	  # actions
		elif [ cond2 ]; then
	      # actions
		else
	      # actions
		fi
		```
- Boucles :
	- For
		```
	 	for i in $liste; do
	 	...
	 	done
	 	```
 	- While
		```
	  	while [ condition ]; do
	 	...
	 	done
	 	```
 	- Do while
	 	```
		until [ condition ]; do
	   	...
	  	done
	  	```
- Case :
	  ```
	  case $val in
	  cas1) action;;
	  cas2) action;;
	  esac
	  ```
  
#### Automatisation (CRONTAB)
- crontab -e : Modifier | -l : Lister | -r : Supprimer.
- Syntaxe : mm hh jj MMM JJJ tâche (> log)
	- mm : minutes (00-59).
	- hh : heures (00-23) .
	- jj : jour du mois (01-31).
	- MMM : mois (01-12 ou abréviation anglaise sur trois lettres : jan, feb, mar, apr, may, jun, jul, aug, sep, oct, nov, dec).
	- JJJ : jour de la semaine (1-7 ou abréviation anglaise sur trois lettres : mon, tue, wed, thu, fri, sat, sun).
	- * : Toutes les valeurs
  	- - : Plage de valeurs (1-5 : Plage de lundi à venredi)
  	- , : séparateur de valeurs (2,7 : Le mardi et le dimanche par exemple)
  	- / : ignore des séquences de valeurs (*/15 : Toutes les 15 min par exemple)

### 3. RÉSEAU & INTEROPÉRABILITÉ
			
- Infos IP : ip a
- Passerelle : ip route
- Libérer DHCP : sudo dhclient -r
- Renouveler DHCP : sudo dhclient
- DNS : dig ou host
- Route : traceroute
- Ports ouverts : ss -tunlp
- Test port TCP : nc -zv [IP] [Port]		
- Cache ARP : arp -a | arp -d (Suppression).
- Scan DHCP : sudo nmap --script broadcast-dhcp-discover -e [interface].


