# Nettoyage & Optimisation W11
## Créer un point de restauration
Trouver "créer un point de restauration" depuis la recherche Windows, puis sélectionner le disque principal, cliquer sur "Configurer" et cocher "Activer la protection du système".) 
Maintenant on peut cliquer sur "Créer" pour créer un point de restauration, le nommer, et le garder si ça tourne mal ! 
## Ménage applicatif
Désinstaller toutes les applications inutiles avec [Bulk Crap Uninstaller](https://github.com/Klocman/Bulk-Crap-Uninstaller)
## Démarrage applications
Trier toutes les applications au démarrage (utiliser Microsoft SysInternals et lancer autoruns.exe) (https://learn.microsoft.com/en-us/sysinternals/downloads/)
Désactiver le bluetooth si on l'utilise pas
Ensuite on tape "**Services**" dans la barre de recherche Windows, et on essayes de désactiver tout ce qui sert pas
## Gestion de l'alimentation
Aller chercher "Choisir un mode de gestion d'alimentation" et choisir l'option "performances élevées" -> consomme plus d'énergie mais meilleurs perfs.
## Nettoyage des disques
Chercher "Nettoyage de disque" -> cocher tous les fichiers et lancer pour chaque disques (si plusieurs). Pensez à **vérifier la corbeille** avant de la vider !
## MAJ
Faire toutes les mises à jour disponibles depuis Windows Update, et depuis Windows Update, aller dans "**options avancées**", puis "**optimisation de la distribution**", et dans "Options de téléchargement" on vient cocher "**Limite d'arrière plan en tant que %**" qu'on vient mettre à **45**, et "**Limite de premier plan en tant que %**" et qu'on met à **90**.
Mettre a jour le BIOS de la carte mere -> on tape "**msinfo32**" dans la recherche et on ouvre "**Informations Systemes**". Le modele de la carte mere c’est la ligne "**Produit de la carte de base**" et la version du BIOS c’est la ligne "**Version du BIOS/Date**". On peut recup la nouvelle version du BIOS DEPUIS LE SITE OFFICIEL du constructeur, toujours. On formate une clé usb en FAT32, et on unzip le fichier qu’on glisse sur la clé. Ensuite on reboot l’ordi sur le BIOS, on cherche « flash » et on MAJ avec le fichier mis sur la clé
Toujours dans le BIOS, on va chercher à activer le "**profil XPM / EXPO**" pour faire tourner la RAM a sa plus haute frequence.
Depuis le site du constructeur de la **CM**, on vient **DL et install tous les pilotes** et outils dispo (audio, chipset, carte graphique, bluetooth…) **TOUT**
Si le PC possède une CG, **mettre a jour les pilotes de la carte graphique** toujours depuis le site officiel constructeur
## Affichage & Ecran
Paramétrer l'affichage -> Clic droit sur le bureau puis "**Paramètre d'affichage**", puis dans "**Paramètres avancés de l'écran**" régler "**choisir une fréquence d'actualisation**" au max pour bénéficier du meilleur taux de rafraichissement de l'écran.
Ensuite dans "**Systèmes**" et "**Ecran**" toujours, on vient chercher "**Graphiques**" et on coche "**Optimisations pour les jeux en mode fenetré**". On déroule juste en dessous "**Paramètres graphiques avancés**" et on vient cocher "**planification de processeur graphique à accélération matérielle**".
## Notifications
Désactiver toutes les notifications inutiles dans "**Système**" puis "**Notifications**".
## Fichiers Temporaires
Ensuite supprimer tout le cache -> **Windows + R** on tape "**%appdata%**", on revient d'un dossier en arrière, on va dans "**Local**" puis dans "**Temp**" et on supprime tout le contenu du dossier.
## Optimisation perfs
Dans bluetooth et appareil, aller dans "**Souris**, puis **paramètres de souris supplémentaires** choisir **aucun** dans modèle. Dans l’onglet "**options du pointeur**", on décoche **"améliorer la précision du pointeur"**.
Dans **Accessibilité** on vient chercher "**Effets Visuels**" et on peut désactiver les "**effets d’animation**". 
Ensuite dans la barre de recherche on peut taper "**Afficher les paramètres systèmes avancés**" et cliquer dans "**Paramètres**" sous performances pour sélectionner "**ajuster afin d’obtenir les meilleures performances**". Efficace pour les vieux PCs mais **ATTENTION** on perd en qualité !
Toujours dans "**Accessibilité**" on peut venir dans "**Clavier**" et désactiver les 3 options "**Touches rémanentes**", "**Touches Filtres**" et "**Touches bascules**"
Ensuite on optimise le processeur en téléchargeant le soft [UnparkCpu](https://coderbag.com/programming-c/disable-cpu-core-parking-utility) . Ca va venir activer les processeurs que Windows aura pu mettre en veille pour économiser de l’énergie. A exécuter en tant qu’admin. Une fois lancé, tout en bas on règle la jauge à 100% et on clique sur "**unpark all**" puis sur "**Apply**".
## Confidentialité & Sécurité
Ensuite dans "**Confidentialité et sécurité**" désactiver presque tout, sauf ce dont on a exclusivement besoin ! (Localisation, partage d’XP…) et dans les "**Autorisations des applications**" on désactive tout ce dont on a pas besoin (attention, laisser genre micro et camera qui peuvent être utilisé par ex)
[Win11Debloat](https://github.com/Raphire/Win11Debloat) => Désactivation des télémétrie : screenshots de l'écran à intervalle régulier pour analyse par l'IA, pour retrouver ce qu'on était en train de faire, applis par défaut, recherche bing, cortana etc... **Lancer le mode par défaut**
## Réseaux
Désactiver les cartes réseaux non utilisées. Dans "**réseau et internet**", aller dans "**paramètres réseau avancés**", et désactiver tout ce dont on se sert pas. 
Changer de serveur DNS pour ceux de CloudFare en DOT **1.1.1.1** et secondaire **8.8.8.8** / ou **9.9.9.9**
Télécharger [TCP Optimizer](https://www.speedguide.net/downloads.php) et l’executer en tant qu’admin : régler ces paramètres la : mettre la barre "**Connection Speed**" sur la valeur la plus haute (en **MBPS**). Cliquer tout en bas sur "**Custom**" pour modifier les valeurs, et mettre :
![[Screenshot from 2025-11-22 19-19-19.png]]
Ensuite, en allant dans l'onglet "**Advanced settings**", on vient mettre les réglages comme suivant : 
![[Pasted image 20251127155421.png]]
Une fois fait, on peut cliquer en bas sur "**Apply Changes**" en **COCHANT LA CASE "BACKUP"** pour sauvegarder les valeurs par défaut.
Si quelque chose va mal, on peut choisir en bas "**Windows Default**" pour restaurer les paramètres par défaut aussi
## Si Ordi Gamer
Dans la partie "**Jeux**" de Windows, dans Game Bar on désactive "**Autorisez votre manette a ouvrir la game Bar**". En revenant dans "**Jeux**" on vient chercher "**Mode Jeu**" et on le laisse activer !
- Optimiser Discord
- Optimiser Steam
- Optimiser ???
## Automated Tools 
- [The Ultimate Windows Utility](https://christitus.com/windows-tool/) 
Le 1er onglet "**Install**" permet d'installer / désinstaller des logiciels. Il permet aussi de tout mettre à jour, il nous suffit de cliquer sur ""**Upgrade all Applications**"" pour mettre toutes les applis à jour. 
Dans le 2ème onglet ""**Tweaks**"" on a plein d'options cool à fouiller (désactivation de pleins de services inutiles)
Lancer ""**O&O ShutUp10++**"" qui permet de désactiver la télémétrie Windows (= tout ce qui Microsoft collecte sur notre utilisation de Windows, plein d'informations personnelles, collecte d'historique du presse papier, etc). Si ça nous cause des problèmes, on peut annuler et revenir en arrière pour annuler les modifications.
- [AtlasOS](https://atlasos.net/) pour aller plus loin, un OS gratuit de Windows débarassé de tout ce qui sert à rien : bloatware, télémétrie, services & applications par défaut etc... Rend l'OS plus fluide !
- [WinToys](https://apps.microsoft.com/detail/9p8ltpgcbzxd?hl=fr-FR&gl=FR) à télécharger depuis le Microsoft Store directement pour réaliser toutes les optimisations présentées ! (applis démarrage, performances, applis en arrière-plan...)
## Pour finir
Mettre un coup de CCleaner
Installer AnyDesk pour pouvoir reprendre la main sur l'ordi en cas de besoin ! ;) 
Installer Brave Browser + Bloqueur de pub



# Anti Malware
## Connexion réseaux actives
From Windows CMD as Admin
```bash
# Afficher connexion réseaux actives
netstat -nbf

# En filtrant par port
netstat -nbf | find "4782"

# Afficher les connexions avec un serveur distant
netstat -ano | findstr "ESTABLISHED"

# Trouver le chemin d'execution d'un processus en fonction de son PID
wmic process where processid=10172 get ExecutablePath
```
## Windows Defender
Tout activer.
Vérifier qu'il ne se désactive pas tout seul après, ou qu'il est possible de le réactiver. 
Vérifier les exclusions exclues de l'analyse anti-virus
## Processus en cours d'execution
Checker dans le **gestionnaire des tâches** les processus en cours, et vérifier les noms + consommation des ressources
Utiliser [Microsoft SysInternals](https://learn.microsoft.com/en-us/sysinternals/downloads/) 
- Autoruns.exe pour vérifier tous les process lancés au démarrage
- Procexp.exe pour analyser en détail les processus actifs 
=> focus les process n'ayant pas de signatures vérifiées. 
Faire attention aux noms des process + leur emplacement (dossier temporaire ?)
Si on trouve un process louche, on peut faire un **clic-droit** sur le process puis **Process Explorer** pour avoir + de détails : ensuite en faisant **clic-droit** puis **propriétés** de l'executable on peut vérifier si y'a des connexions actives, et si oui vers quelle @IP (potentiellement celle de l'attaquant) dans l'onglet **TCP/IP**.
=> on peut également vérifier la **hiérarchie** des processus, en vérifiant si le process en a pas lancé d'autres ou s'il en génère d'autres. On fini par vérifier le nom du process en ligne pour voir s'il est connu ou pas.

- TCPView.exe pour voir les connexions réseaux actives
## VirusTotal
Scan sur VT
## Autres signes
Surveiller les pop-up, les fonds d'écrans changés, les paramètres du navigateur...
## Remédiation
Déconnecter le PC d'internet
Faire un scan + optimisation pour nettoyer complètement l'ordinateur et le désinfecter
Changer **TOUS** les mots de passe (en priorisant email + compte bancaire + réseaux sociaux) + utiliser un gestionnaire de MDP si possible + activer la 2FA sur tous les services

