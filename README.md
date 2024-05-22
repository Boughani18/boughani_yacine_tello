# boughani_yacine_tello
projet sur les drones (Tello edu)
Ce projet propose une solution complète pour le contrôle d'une flottille de drones Tello EDU à partir d'un PC de controle, en asservissant en position grace a un retour sur leur position avec des caméras Optitrack . L’implémentationt du  programme est basé sur le middleware ROS2 et vise à fournir une interface pratique et efficace pour le contrôle des drones en temps réel. Donc dans cette partie pratique dans un prmier temps nous allons passer par plusieur configuration et etape afin de pouvoir réaliser notre lancement .

Pré requis et etapes a suivre :

L'utilisation de ce programme nécessite l'installation et la configuration de ROS2 Foxy sur un système Ubuntu 20.04.Avoir une station optitrack pour controle et visionage des drones ,un pc de controle ,et un pc optitrack.et a l’aide de python et les definition de chaque fonction on pourais mieux comprendre chaque commande et  a l’aide du programme tornado on pouraient avoir aleatoirement des position . Une fois ces prérequis remplis et compris le programme peut être utilisé selon les étapes suivantes :
-Mise en place du réseau
-Configuration d’Optitrack
-Configuration et exécution du package ros2-mocap_optitrack
-Configuration du driver / package swarm
-Configuration de la démo exécutée par les drones
-Compilation et exécution du package

Mise en place du réseau :
Les PC Optitrack et de contrôle doivent être connectés au même réseau que les drones, de préférence via une connexion filaire. Les drones doivent être configurés sur ce réseau, ce qui peut être fait en utilisant le SDK 3.0 Tello et en envoyant des paquets UDP via des logiciels tels que PacketSender.

Utilisation de packetSender :
Pour ce qui est de l’utilisation de ce logiciel avant le commencement faudrait etre connecter au reseau du drone , Apres ca dans un premier temps dans l’encart «ASCII», saisissez le message que vous souhaitez envoyer dans notre cas en a 3message a envoye premierement commande pour le commander ensuite battery pour verifier l’etat de  la batterie et en fin configuration ap c’est ce qui va nous permettre de configurer le drone au routeur dans notre cas il sera relier au reseau speed wiffi , Ensuite entrez l’adresse IP et le port du serveur de destination. A la fin cliquez sur le bouton «envoyez» pour transmettre le paquet.
Pour ce qui est des fonctionnalités :
Protocoles supportés : TCP, UDP, SSL.
Génération de requetes HTTP/HTTPS.
Affichage de l’état et des ports pour les serveurs UDP, TCP et SSL.
Réponses facultatives peuvent etre envoyées.



Configuration d’Optitrack :
Avant d'utiliser le programme, Optitrack doit être lancé. Cela implique d'allumer les caméras Optitrack, de lancer le logiciel Motive sur le PC Optitrack, et de le configurer correctement . Avant de lancer on doit s’assurer que notre pc optitrack est bien connecter au bon réseau et qu’il a la bonne adresse IP car tout les appareil devraient etre connecter sure la méme adress. Ensuite en effectue plusieur configuration :
Configuration systemes :
Calibration du volume on prepare notre espace de capture (volume) en placant les caméras OptiTrack autour de la zone ou vous souhaitez suivre les mouvements.
Configuration des caméras :
Placement des caméras on positionne les caméras de maniére a couvrir toute la zone de capture.apres en effectue une calibration des caméras on utilisera le logiciel motive pour calibrer chaque caméra en fonction de son emplacement.
Placement des marqueurs :
On attache des pastilles ou marqueurs réfléchissants aux objets que vous souhaitez suivre. Ces marqueurs sont détectés par les caméras OptiTrack.pour ce qui du placement des marqueurs faut les mettre de facon qu’il soit visbles au moins par deux caméras simultanément.
Capture de mouvement :
On lance le logiciel Motive , On crée un projet et on  configure les paramétres de capture (fréquence d’image , résolution ,ect).ensuite en calibre le volume en utilisant les marqueurs de calibration. On commence la capture de mouvement en enregistrant les données des marqueurs. 
Analyse des données :
Apres la capture ,On analyse les données pour obtenir des information sur les mouvements des drones, Ensuite exportez les données au format souhaité (CSV,BVH,etc.) pour une utilisation ultérieure.

On fait une configuration de motive et du package de mocap : 
-Configuration motive :

Ensuite on va sur propreities et on verifie les propriété du drone et il faut s’assurer que le streaming ID de chaque drones doit etre different allant de 1 a 12 et on devrait toujours avoir un drone qui prend l’adresse 1 :
Pour ce qui est des données de chaque drones ils seront envoyé  vers le PC de controle avec un protocole NatNet. Et pour ce qui est de la configuration des rigid body on effectura les modification sur notre Table.csv .


Configuration et exécution du package ros2-mocap_optitrack :
Ce package permet de recevoir les données de positionnement des drones à partir d'Optitrack. Il est nécessaire de suivre les instructions et la configuration suivant afin d’exécuter le package ros2-mocap_optitrack :
Sourcer ROS2 :
Avant de commencer , on s’assure d’abord d’avoir bien sourcer ROS2 dans chaque terminal qu’on vas utiliser
             Source /opt/ros/foxy/setup.bash
Lancer le script build.sh :
A la racine de votre workspace ROS2, executez le script build.sh
             Bash build.sh
On laisse ce terminal en cours d’exécution



Vérification du fonctionnement :
Dans un nouveau terminal ou ROS2 est sourcé, on lance la commande suivante pour verifier si les position rigid bodies sont correctement envoyées
                       ros2 topic echo /mocap_rigid_bodies
Si on vois pas de psitions de rigid body, on vérifie toute la configuration et relance le processus (cela peut échouer en cas de latence sur le réseau).
Gestion des plantages :
Sur certains PC, le programme peut planter environ toutes les 10 minutes, surtout lorsque des modification sont apportées aux rigid bodies dans Motive.Dans ce cas en utilise a nouveau le script build.sh ou bien en peut utiliser aussi
                       colcon build

Configuration du driver / package swarm :
Pour configurer le packageswarm et le driver associé:
Table d’adresses IP des drones :
On dois d’abord s’assurer que la table d’adresses IP des drones est correcte dans le fichier
                         trello_swarm/ros2_ws/src/swarm/swarm/config/Table.csv  
Ce fichier contient les adresses IP des drones que vous souhaitez controler.
Configuration du driver  driver_swarm.py :
Ouvrire le fichier driver_swarm.py situé dans le répertoire 
                         Trello_swarm/ros2_ws/src/swarm/swarm/.
Dans les variables correspondantes ,on configure les paramétres suivant :
Self_ip : on entre l’adresse IP  de notre PC de controle sur le réseau aux drones et a OptiTrack.
Listening_port : choisire un port de communication avec les drones Tello (par exemple entre 8890 et 8899, mais tout port disponible sur notre PC fonctionne).
Verification des valeurs :
On s’assure que les valeurs sont correctes, car un échec du socket au lancement du programme peut etre du a des parametres incorrects.

En résumer Le fichier Table.csv doit être correctement configuré avec les adresses IP des drones. De plus, l'adresse IP du PC sur le réseau commun aux drones et à Optitrack doit être entrée dans le programme driver_swarm.py .

Configuration de la démo exécutée par les drones :
Fichier CSV de démo (csv_demo_test.csv) :
Chaque ligne de ce fichier correspond à une position que le drone doit atteindre pendant un intervalle de temps défini entre starttime et endtime.
Les positions sont spécifiées en coordonnées x, y et z, conformément aux axes définis dans Motive (le logiciel OptiTrack).
Si le champ Quaternions est défini comme True, l’orientation du drone sera déterminée par les champs qx, qy, qz et qw.
Si le champ Quaternions est défini comme False, l’orientation du drone sera déterminée par les angles d’Euler spécifiés dans les champs rx, ry et rz.
Actuellement, seule l’orientation autour de l’axe Z (rotation en radians) est prise en compte (champ rz).
Le champ Vobjectif n’est pas encore utilisé, donc vous pouvez le laisser à 0 ou à toute autre valeur.
Le champ drone indique à quel drone l’ordre s’applique. Commencez à partir de 1 et incrémentez d’un pour chaque nouveau drone. Le streaming ID correspondant sera ajouté dans le programme en fonction des drones disponibles.
Ordre des commandes :
Les commandes doivent être spécifiées dans l’ordre croissant des champs starttime.
Exemple de génération de démo :
Un exemple de programme permettant de générer une démo au format .csv où les drones tournent autour du centre avec des variables modifiables (nombre de drones, vitesse, rayon, fréquence d’échantillonnage) est disponible dans le fichier tornado_generator.py :
                                 trello_swarm/ros2_ws/src/swarm/swarm/demo/tornado_generator.py
D’autres exemples de démos sont également disponibles dans ce dossier.
Modification des démos :
Pour modifier les démos, collez simplement la nouvelle démo dans le fichier csv_demo_test.csv et adaptez manuellement le fichier JSON correspondant.

En résumé les drones exécutent une démo spécifiée dans un fichier CSV. Ce fichier doit être correctement configuré, en veillant à ce que le nombre de positions spécifiées corresponde au nombre de drones.Les drones vont suivre une démo entrée en CSV dans le fichier et dont le nombre de drone qui l’exécute est égale au nombreDeDrone entré dans le fichier JSON .Pour l’instant pour modifier les démo on vient simplement coller la démo dans le fichier
                                  trello_swarm/ros2_ws/src/swarm/swarm/demo/csv_demo_test.csv
et adapter la la main le JSON.

Compilation et exécution du package :
pour compiler et exécuter le package swarm :
Compilation du workspace ROS2 :  On ouvre un nouveau terminal et  on accéde à la racine de votre workspace ROS2 :
                                   cd /trello_swarm/ros2_ws
On compile l’ensemble du workspace avec la commande :
                                   colcon build
Sourcer le workspace :
Dans chaque nouveau terminal, on s’assure de sourcer le workspace ROS2 :
                                   . install/setup.bash
Exécution du programme :
Pour surveiller les positions des drones et de leurs références, on utilise le logiciel RVIZ2. Lancez-le dans un terminal où ROS2 est sourcé :
                                     ros2 run rviz2 rviz
Dans RVIZ2, on modifie le champ “Fixed Frame” en sélectionnant “world”.On ajoute l’observation des transformations (TF) en cliquant sur “Add” → “TF”.Si nous souhaitons simplement observer la démo dans RVIZ sans faire décoller les drones (mais les drones doivent être allumés sinon la démo ne se lancera pas), commentez la ligne suivante dans la classe TelloNode, fonction takeoff_land :
                                    self.master.send_command("takeoff", self.ip, False)
Pour exécuter le programme, utilisez la commande :
ros2 run swarm swarm_command
On vérifie que le bon nombre de drones est présent dans les logs affichant "Created ", i ,"tello nodes successfully”, où i correspond au nombre correct de drones.Une fois qu’on est  sûr que tout est configuré correctement (placement des drones, etc.), on entre la commande done dans le terminal, et la démo se lancera immédiatement.

Conclusion sur la partie pratique :
En résumé, ce projet offre une solution complète et robuste pour le contrôle de drones à partir d'un PC, en combinant les technologies ROS2 et Optitrack pour un fonctionnement précis et efficace.
