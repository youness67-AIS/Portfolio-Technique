# Mon projet Nagios

## Introduction à Nagios Core :
Nagios Core est un logiciel de supervision open source permettant de surveiller l’état de vos systèmes, services et applications. 
Dans cet atelier, je vais installer **Nagios Core 4.5.11** sur une machine **Ubuntu 24.04**, ainsi qu'une **VM Windows et une autre Vm Ubuntu** pour simuler les agents.
Je vais également configurer des utilisateurs, des groupes, et des services pour me permettre de surveiller efficacement mon infrastructure.

## Etape 1 : Installation du Nagios Core
### Etape 1.1 :
- J'installe un CT Ubuntu 24.04 sur Proxmox, je mets mon système à jour avec les commandes
```
apt update && apt upgrade -y
```
### Etape 1.2 :
- Installation des dépendances : Nagios Core nécessite plusieurs dépendances, y compris Apache, PHP, et des outils de compilation. Pour les installer, j'utilise la commande suivante :
```
apt install -y build-essential libgd-dev openssl libssl-dev unzip apache2 php libapache2-mod-php libperl-dev libpng-dev
```
<img width="830" height="320" alt="image" src="https://github.com/user-attachments/assets/c844fa95-27a6-472a-923e-602109e769c9" />

### Etape 1.3 : Téléchargement de Nagios Core 4.5.11
- Pour continuer mon installation, je vais maintenant télécharger l'archive source de **Nagios Core** depuis le site officiel. Cette archive contient tous les fichiers nécessaires pour compiler et installer le logiciel sur votre système :
```
cd /tmp
wget https://go.nagios.org/get-core/4-5-11
```
<img width="1484" height="358" alt="Capture d’écran 2026-01-22 100738" src="https://github.com/user-attachments/assets/590663f1-a707-46e1-af42-eaf7d68539d1" />

### Etape 1.4 :  Décompression de l'archive et installation de Nagios Core
- Une fois l'archive téléchargée, je le décompresse et j'entre dans le dossier extrait :
```
tar -xvzf 4-5-11
cd nagios-4.5.11
```
- Je vais maintenant créer un utilisateur nagios et l'ajouter au groupe nagcmd. Ces commandes permettent de créer les utilisateurs et groupes nécessaires pour que Nagios fonctionne correctement avec les bonnes permissions.
```
useradd nagios
groupadd nagcmd
usermod -G nagcmd nagios
usermod -G nagcmd www-data
```
- Ensuite, je compile et j'installe Nagios Core en exécutant les commandes suivantes :
```
./configure --with-httpd-conf=/etc/apache2/sites-available --with-command-group=nagcmd
```
<img width="635" height="481" alt="Capture d’écran 2026-01-22 102329" src="https://github.com/user-attachments/assets/c14d238d-24ad-4a23-a240-fa24ab296de9" />

- Je peux maintenant lancer l'installation avec la commande :
```
make all
```
<img width="563" height="69" alt="Capture d’écran 2026-01-22 102632" src="https://github.com/user-attachments/assets/c1527edd-0bdb-496d-918b-8bdd4ef88fdd" />

- Si nous regardons la liste des commande que l’ont nous donne juste avant de cette enjoy, nous voyons la liste des commandes a lancer pour continuer notre installation :
<img width="625" height="681" alt="Capture d’écran 2026-01-22 102723" src="https://github.com/user-attachments/assets/b753ee04-1827-49ef-8bad-9b05491359fc" />

- L'étape suivante consiste à installer **Nagios** et ses fichiers de configuration en lançant les commandes suivantes :
<img width="345" height="242" alt="Capture d’écran 2026-01-22 103148" src="https://github.com/user-attachments/assets/fbd6b6b8-d6ee-4ec3-967a-0f012112b4d8" />

- cette série de commandes va installer **Nagios Core** et ses fichiers de configuration nécessaires.
### Etape 1.5 : Installation de l'interface web de **Nagios** 
* L'interface web de Nagios est un outil essentiel qui offre une vue d'ensemble détaillée et en temps réel de l'état de vos systèmes supervisés. 
* Cette interface permet de surveiller facilement les performances, les alertes et les métriques de tous vos équipements et services depuis un tableau de bord centralisé.
* Pour l'installer, je lance la commande suivante toujours dans le meme répertoire :
```
make install-webconf
```
* Ensuite j'active le module Apache nécessaire, ainsi que le site et redémarrez Apache avec les commandes suivantes :
```
a2enmod cgi
a2enite nagios
systemctl restart apache2
```
<img width="883" height="326" alt="Capture d’écran 2026-01-22 104340" src="https://github.com/user-attachments/assets/aedc9b65-5ab0-4c18-93d3-20e18a5f470a" />

### Etape 1.6 : Création d'un utilisateur Nagios pour l'accès à l'interface web
- Pour accéder à l'interface web, je dois créer un utilisateur et un mot de passe. J'exécute la commande suivante pour configurer un mot de passe pour l'utilisateur **nagiosadmin** :
 ```
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```
- C'est l'utilisateur et le mot de passe qui va me permettre de me connecter à l'interface web de **Nagios**.
### Etape 1.7 : Démarrage de **Nagios**
- Pour démarrer **Nagios Core**, j'exécute les commandes suivantes :
```
systemctl start nagios
systemctl enable nagios
```
### Etape 1.8 : Vérification du bon fonctionnement de **Nagios**
- J'accéde à l'interface web de **Nagios** via mon navigateur en me rendant à l'adresse suivante : `http://mon-ip-nagios/nagios` et en utilisant **nagiosadmin** pour le nom d'utilisateur et mon mot de passe.
<img width="1822" height="863" alt="Capture d’écran 2026-01-22 105425" src="https://github.com/user-attachments/assets/45f7fd5b-02db-4560-9272-217cf47a657a" />

⚠️ Bien que j'ai configuré les composants de base, il me reste encore plusieurs étapes cruciales à accomplir pour avoir un système **Nagios** pleinement fonctionnel. 
Je dois notamment installer les plugins nécessaires, configurer la surveillance des hôtes et des services, et mettre en place les notifications. 
### Etape 1.9 : installation des plugins
💡 Les plugins **Nagios** sont essentiels car ils fournissent les fonctionnalités de base pour effectuer des vérifications sur les hôtes et services. 
- J'installe les prérequis :
```
apt install -y autoconf gcc libc6 libmcrypt-dev make libssl-dev wget bc gawk dc build-essential snmp libnet-snmp-perl gettext
```
- Pour les installer, je télécharge d'abord la dernière version des plugins **Nagios** et je lance l’installation :
```
cd /tmp
wget -O nagios-plugins.tar.gz $(wget -q -O - https://api.github.com/repos/nagios-plugins/nagios-plugins/releases/latest  | grep '"browser_download_url":' | grep -o 'https://[^"]*')
tar zxf nagios-plugins.tar.gz
cd /tmp/nagios-plugins-*/
./configure
make
make install
```
- Une fois les plugins installés, je vérifie l'état du **ping**
<img width="865" height="730" alt="Capture d’écran 2026-01-22 111606" src="https://github.com/user-attachments/assets/621a91cb-f130-4857-8d01-4e086ebb6446" />

### Etape 1.9 : Configuration du pare-feu
- Si j'ai un pare-feu actif sur mon serveur **Ubuntu**, je dois peut-être autoriser l'accès à **Apache**. je peux le faire avec la commande suivante :
```
ufw allow 'Apache Full'
```

## Etape 2 : Installation de l'Agent NCPA sur une VM Windows Client/Server
💡 NCPA est un agent Nagios universel qui simplifie la configuration et prend en charge Windows, Linux et macOS. 
Il expose une API RESTful pour l'interrogation des données, et son installation est plus simple et plus flexible que NRPE. 
### Etape 2.1 :  Télécharger l'agent NCPA pour Windows
- Je me rends sur la page officielle de NCPA et je télécharge le fichier d'installation NCPA pour Windows : [Agent NCPA](https://www.nagios.org/projects/ncpa/#download-ncpa-section)
### Etape 2.2 : Installation de l'Agent NCPA
- Une fois le fichier télécharger, je l'éxecute pour démarrer l'installation
- Lors de l'installation, un **Token** est demandé.
- **Rôle :** Il sert de clé d'authentification entre le serveur Nagios et l'agent.
- **Valeur choisie :** `mytoken`.
> [!WARNING]
> Ce token sera indispensable pour les connexions sécurisées à l'API de NCPA.
<img width="525" height="422" alt="Capture d’écran 2026-01-22 113834" src="https://github.com/user-attachments/assets/1c4e5a4f-ae12-467a-ae00-8167f2aebc99" />

### Etape 2.3 :  Vérification de l'installation de NCPA
- Une fois l'installation terminée, l'agent NCPA devrait être en cours d'exécution en tant que service sur mon serveur Windows. Pour vérifier cela :
 J'ouvre le Gestionnaire des tâches et je vais dans l'onglet Services.
 Je recherche le service NCPA. Il devrait être en "En cours d'exécution".
<img width="817" height="610" alt="Capture d’écran 2026-01-22 114610" src="https://github.com/user-attachments/assets/22ea1321-adf7-4a00-8d24-bfa86194afc3" />

### Etape 2.4 :  Configuration du pare-feu Windows 
- Si le pare-feu est activé sur mon serveur Windows, je dois m'assurer d'ouvrir le port 5693 pour permettre la communication avec le serveur Nagios. Je peux le faire en exécutant les commandes suivantes dans une fenêtre PowerShell avec des privilèges administratifs :
```
New-NetFirewallRule -DisplayName "NCPA" -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 5693
```
<img width="991" height="406" alt="Capture d’écran 2026-01-22 114748" src="https://github.com/user-attachments/assets/4de9c84b-c6c1-4cd1-9dc2-5f0e6217d03a" />

### Etape 2.5 : Test de **l'Agent NCPA**
- L'agent NCPA expose une interface Web accessible via l'adresse IP de mon serveur Windows. Je peux tester cette interface en ouvrant un navigateur et en accédant à : `https://<IP_du_serveur_windows>:5693`
- J'entre le token que j'ai défini lors de l'installation de l'agent, et je me connecte à mon interface
<img width="1291" height="508" alt="Capture d’écran 2026-01-22 115237" src="https://github.com/user-attachments/assets/eca26191-b9da-4031-838a-1e728585177c" />

### Etape 2.6 : Configuration de NCPA sur le serveur Nagios
- Maintenant que l'agent NCPA est installé et fonctionne, je dois configurer mon serveur Nagios pour surveiller le serveur Windows.
- Je crée un fichier de configuration pour mon serveur Windows dans le dossier servers (par exemple, windows_server.cfg) avec les commandes suivantes :
```
cd /usr/local/nagios/etc/
mkdir servers
nano /usr/local/nagios/etc/servers/windows_server.cfg
```
- Et une fois dans l'éditeur nano, j'ajoute la configuration suivante :
  <img width="575" height="713" alt="Capture d’écran 2026-01-22 120601" src="https://github.com/user-attachments/assets/66d87adb-a435-4066-9a7c-ca642771a0f9" />

- Je recharge la configuration de **Nagios** pour appliquer les modifications :
```
systemctl restart nagios
```
- Pour configurer correctement la surveillance NCPA, je dois maintenant ajouter une définition de commande spécifique dans le fichier de configuration du serveur Nagios.
- J'ouvre le fichier `/usr/local/nagios/etc/objects/commands.cfg` et j'ajoute la configuration suivante qui permettra à Nagios d'interagir avec l'agent NCPA :
<img width="662" height="157" alt="Capture d’écran 2026-01-22 122114" src="https://github.com/user-attachments/assets/c0b32dad-9ff1-42fa-947d-fa7110f92497" />

-Une fois fait, je dois mettre à jour la configuration dans le fichier `/usr/local/nagios/etc/nagios.cfg`. Je décommente la ligne `#cfg_dir=/usr/local/nagios/etc/servers` pour permettre la découverte des nouveaux serveurs. 
⚠️ Sans cette étape, aucune vérification ni remontée d'information ne sera effectuée.
- Une fois ces modifications effectuées, il est nécessaire de redémarrer le service Nagios pour que les changements prennent effet.
```
systemctl reload nagios
```
### Etape 2.7 : Vérification dans l'interface Web de **Nagios**
- J'accéde à mon interface web Nagios : `http://mon-ip/nagios`
- Je vérifie bien que le serveur Windows est bien ajouté et que les services que j'ai configurés (comme la charge CPU et l'utilisation de la mémoire) sont correctement surveillés.
<img width="1440" height="100" alt="Capture d’écran 2026-01-22 124349" src="https://github.com/user-attachments/assets/4fa68a5a-39e5-4390-85de-3ff5462c651b" />
<img width="712" height="635" alt="Capture d’écran 2026-01-22 124405" src="https://github.com/user-attachments/assets/2a022129-70c9-4799-920f-13ac7b0ffe5a" />
<img width="798" height="608" alt="Capture d’écran 2026-01-22 124423" src="https://github.com/user-attachments/assets/b2b567a2-7b63-40e6-ab2e-7eddc9d04c98" />

## Etape 3 : Installation de **l'Agent NCPA** sur un Nouveau Serveur **Ubuntu 24**
- **L'agent NCPA** est compatible avec Ubuntu 24 et peut être installé facilement via un fichier `.deb.` Nous allons suivre les étapes nécessaires pour installer **l'agent NCPA** et le configurer pour la surveillance via **Nagios**.
### Etape 3.1 : Télécharger **l'agent NCPA** pour **Ubuntu**
<img width="864" height="334" alt="Capture d’écran 2026-01-22 125429" src="https://github.com/user-attachments/assets/1da43c0f-c407-415f-94fe-f32b2ae5fa6a" />

### Etape 3.2 : Installation de NCPA sur Ubuntu
- Une fois que le dépôt a été correctement ajouté et mis à jour dans mon système, je peux procéder à l'installation de l'agent NCPA.
- J'exécute la commande suivante dans mon terminal :
```
apt install ncpa
```
### Etape 3.3 : Configuration de NCPA sur Ubuntu
L'agent NCPA est maintenant installé, mais avant de pouvoir l'utiliser pour la surveillance via Nagios, je dois le configurer. Cela inclut la configuration du mot de passe de l'API et la vérification des paramètres de connexion.
- Configurer le mot de passe de l'API : À l'installation de NCPA, un mot de passe d'API est généré automatiquement. Pour définir mon propre mot de passe sécurisé, je modifie le fichier de configuration : `nano /usr/local/ncpa/etc/ncpa.cfg` puis je remplace la ligne `community_string = mytoken` par mon mot de passe.
- Ouvrir le port sur UFW, vous pouvez exécuter la commande suivante :
```
ufw allow 5693/tcp
```
- Je démarre les services, ensuite je vérifie le status du service :
```
systemctl start ncpa
systemctl enable ncpa
systemctl status ncpa
```
<img width="953" height="471" alt="Capture d’écran 2026-01-22 130335" src="https://github.com/user-attachments/assets/3084d139-07e1-4f4e-872f-4dca90aafceb" />

-### Etape 3.4 : Tester l'interface Web de NCPA
- Pour y accéder j'ouvre mon navigateur et je tape `https://mon-ip-ubuntu:5693`
<img width="1369" height="508" alt="Capture d’écran 2026-01-22 131204" src="https://github.com/user-attachments/assets/c44f67df-148f-4512-a420-7472b8e357fb" />

### Etape 3.5 : Configuration de NCPA pour Nagios
Maintenant que l'agent NCPA est installé et fonctionne sur mon serveur Ubuntu, il est temps de le configurer sur mon serveur Nagios pour qu'il puisse surveiller ce serveur. Je dois ajouter ce serveur à la configuration de Nagios et commencer à surveiller ses services.
- Je vais dans l'editeur nano sur le chemin suivant `/usr/local/nagios/etc/servers/ubuntu_server.cfg`et ajouter la configuration suivant :
<img width="571" height="718" alt="Capture d’écran 2026-01-22 142131" src="https://github.com/user-attachments/assets/eb81376b-9e09-48d0-9a81-b8473d6055a1" />

- Je recharge la configuration de **Nagios** pour appliquer les modifications :
```
systemctl reload nagios
```
- Je dois maintenant installer le script check_ncpa.py, qui est un composant essentiel permettant à Nagios de communiquer avec l'agent NCPA et d'effectuer les vérifications des services. Ce script agit comme une interface entre Nagios Core et l'agent NCPA, facilitant la collecte et la transmission des données de surveillance. j'utilise les commandes suivantes :
```
wget https://raw.githubusercontent.com/NagiosEnterprises/ncpa/master/client/check_ncpa.py -O /usr/local/nagios/libexec/check_ncpa.py
chmod 777 /usr/local/nagios/libexec/check_ncpa.py
apt install python-is-python3
```
## Résultat final : Vérification de la surveillance dans l'interface Web de **Nagios**
- J'accéde à l'interface web de **Nagios** pour vérifier que mon serveur Ubuntu et ses services sont maintenant surveillés. J'ouvre un navigateur et je vais à l'adresse suivante : `https://<mon_ip>/nagios`
<img width="1384" height="169" alt="Capture d’écran 2026-01-22 141153" src="https://github.com/user-attachments/assets/272f0b0f-2a6a-435e-bccb-add9f15bc181" />
<img width="1477" height="346" alt="Capture d’écran 2026-01-22 141734" src="https://github.com/user-attachments/assets/27b3838d-1041-4847-9b6b-1dd3749741b9" />

🎉🎉🎉 je vois bien mes deux serveurs Windows et Ubuntu dans la liste des hôtes surveillés.







