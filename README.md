# README.MD

> Comment utiliser le programme, présentation de son fonctionnement et de son architecture
> 

L’objectif de ce projet est de contrôler une flottille de drone Tello EDU depuis un PC en les asservissant en position grâce à un retour sur leur position avec des camera Optitrack. L’ensemble du programme sera implémenté sur middleware ROS2.

Pour utiliser ce programme il faut [installer](https://docs.ros.org/en/foxy/Installation/Ubuntu-Install-Debians.html) et [configurer](https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Configuring-ROS2-Environment.html) ROS2 Foxy sur Ubuntu 20.04.

## Utilisation :

### Mise en place du réseau :

Il faut faire en sorte de placer le PC Optitrack et le PC de contrôle, de préférence par connexion filaire, sur le même réseau que les drones. Pour configurer les drones sur un nouveau réseau vous pouvez vous référer au [SDK 3.0 Tello](https://dl.djicdn.com/downloads/RoboMaster+TT/Tello_SDK_3.0_User_Guide_en.pdf) et envoyer les paquets UDP via le logiciel PacketSender par exemple. 

### Configuration d’Optitrack :

Pour utiliser le programme il faut d’abord lancer Optitrack. Il faut donc allumer les caméra Optitrack, lancer le logiciel Motive sur le PC Optitrack, correctement le configurer en suivant les indication de la doc Optitrack présente sur ce dépôt git. Attention à bien mettre “Z up” sur Motive.

Il faut ensuite ajouter les créer les rigid bodies de chaque drone en plaçant l’avant du drone vers **x positif** d’Optitrack (afin que les axe du drone et d’Optitrack correspondent)

Il faut aussi configurer les rigid body de sorte à ce que leur streaming id correspondent à l’ID entré dans

```python
ros2_ws/srx/swarm/swarm/config/Table.csv
```

> 🛑 Attention : le package ros2-mocap_optitrack a besoin qu’au moins un rigid body ait le **streaming id 1** donc soit toujours utiliser le drone 1, soit entrer un rigid body 1 qui n’existe pas sur Motive, soit adapter le fichier Table.csv
> 

### Configuration et exécution du package ros2-mocap_optitrack :

A nouveau se référer à la doc Optitrack présente sur ce dépôt git.

Bien penser à sourcer ROS2 dans chaque terminal utilisé : 

```bash
source /opt/ros/foxy/setup.bash
```

Puis lancer le build.sh présent à la racine du ROS2 workspace :

```bash
bash build.sh
```

 et laisser ce terminal tourner.

Afin de vérifier qu’il fonctionne correctement vous pouvez, dans un nouveau terminal dans lequel ROS2 est sourcé, lancer la commande :

```bash
ros2 topic echo /mocap_rigid_bodies
```

et constater si il y a bien des position de rigid body envoyé. Sinon vérifier toute la configuration et relancer (peut planter si latence sur le réseau)

Sur certain PC le programme plante à une fréquence de 10 minutes environs et lorsque des changement sur les rigid bodies sur Motive sont effectué. Il faut donc exécuter à nouveau le build.sh.

### Configuration du driver / package swarm :

S’assurer que la table d’adresse IP des drones est correcte dans le fichier :

```bash
trello_swarm/ros2_ws/src/swarm/swarm/config/Table.csv
```

Entrer l’adresse IP du PC sur réseau commun aux drone et Optitrack dans le programme driver_swarm.py ainsi que sont port de communication avec les Tello (nouveau utilisions entre 8890 et 8899 mais fonctionne avec n’importe quel port disponible sur le PC)

```bash
trello_swarm/ros2_ws/src/swarm/swarm/driver_swarm.py
```

Dans les variables correspondante :

```python
class TelloMasterNode (Node):
    self_ip = '192.168.1.227' #Set your own IP address on the routeur used to connect the tellos
    listening_port = 8894 #any available port works
```

Les valeurs sont mauvaise si le socket échoue au lancement du programme.

### Configuration de la démo exécutée par les drones :

Les drones vont suivre une démo entrée en CSV dans le fichier :

```python
trello_swarm/ros2_ws/src/swarm/swarm/demo/csv_demo_test.csv
```

Et dont le nombre de drone qui l’exécute est égale au **nombreDeDrone** entré dans le fichier JSON :

```python
trello_swarm/ros2_ws/src/swarm/swarm/demo/csv_demo_test.csv.json
```

Les position dans ce fichier n’importent pas réélement mais servent d’indication sur la démo et serons print avant sont exécution. De plus il faut qu’il y est un nombre égale ou supérieur de position que drones dans le JSON.

Le csv_demo_test.csv fonctionne de la manière suivante :

Chaque ligne correspond à une position que doit atteindre le drone, entre le temps indiqué en **starttime, endtime**.

les position **x,y,z** correspondent aux axes mis en place Motive. 

Si le champ Quaternions est **True**, alors l’orientation du drone prise sera dans les champs **qx, qy, qz, qw.**

Si le champ Quaternions est **False**, alors l’orientation du drone sera donc les angles d’Euler entré en **rx,ry,rz**

Pour l’instant pour l’instant l’orientation commandée est uniquement celle autour de **l’axe Z** donc **rz.**

Le champ Vobjectif n’est pour l’instant pas utilisé, donc le laisser à 0 (ou n’importe quelle valeur).

Le champ drone correspond au drone auquel l’ordre s’applique. Partir de 1 et incrémenter de 1 à chaque nouveau drone, le streaming ID correspondant sera ajouter dans le programme à partir des drones disponible.

<aside>
🛑 Les commandes doivent forcément être dans l’ordre : **les champs starttime doivent être dans l’ordre croissant.**

</aside>

---

Un exemple de programme permettant de générer une démo .csv de drone qui tourne autour du centre avec des variables modifiable (nombre de drone, vitesse, rayon, fréquence d’échantillonnage) est disponible en :

```python
trello_swarm/ros2_ws/src/swarm/swarm/demo/tornado_generator.py 
```

et d’autres exemple de démo sont également disponible dans ce dossier. 

Pour l’instant pour modifier les démo on vient simplement coller la démo dans le fichier 

```python
trello_swarm/ros2_ws/src/swarm/swarm/demo/csv_demo_test.csv
```

et adapter la la main le JSON.

### Compilation et exécution du package :

Aller dans un nouveau terminal à la racine du workspace ROS2 :

```bash
cd /trello_swarm/ros2_ws
```

puis build le workspace complet : 

```bash
colcon build
```

Et il faudra le sourcer dans chaque nouveau terminal :

```bash
. install/setup.bash
```

Il est maintenant possible d’exécuter le programme :

Si vous souhaiter monitorer les positions des drones et de leur référence c’est possible avec le logiciel rviz2, à lancer dans un terminal sourcé ROS2, à lancer de la manière suivante : 

```python
ros2 run rviz2 rviz
```

puis changer le champ de la fixed frame à **world.**

Ensuite ajouter l’observation des TF en faisant add → TF.

Si vous souhaitez simplement observer la démo sur RVIZ, c’est possible de bloquer le décollage des drones (mais ceux-ci doivent être allumer sinon la démo ne se lancera pas) en commentant la ligne suivante :

```python
self.master.send_command("takeoff", self.ip, False) #Comment this line if you don't want the drone to take off ----------
```

présente dans la classe TelloNode, fonction takeoff_land.

Pour exécuter le programme le lancer avec la commande :

```bash
ros2 run swarm swarm_command
```

Vérifier que le bon nombre de drone est présent dans les log **"Created ", i ,"tello nodes successfully”**  avec i == au bon nombre.

Une fois que vous vous ètes assurer que tout était bon, placement de drone etc, alors vous pouvez entrer

```bash
done
```

Dans le terminal, et la démo se lancera immédiatement.

🛑 Faire CTRL + C dans le terminal permet de couper le programme et d’ordonner au drone de se stopper et de se poser, malheureusement les drones sont capricieux et donc leur arrêt ne s’effectue pas toujours.   

Nous n’avons pas accès à leur logiciel interne donc il n’est pas possible de corriger se côté imprévisible des drones, sauf peut être en comprenant pourquoi le problème intervient et ainsi adapter le driver_swarm.py.

---

Pour concevoir une nouvelle commande (asservissement) des drones, voir la doc présente dans ce dépôt : **Doc : Comment modifier la commande pour asservir un essaim de drone**

Pour voir l’architecture du logiciel voir l’image architecture_logiciel.png

<aside>
🐞 Nous avons découvert un bug en fin de projet que nous n’avons pas encore eu le temps de résoudre, actuellement il est impossible de controller plus de 4 drones à la fois. En effet le calcul de la transformé de tello_abs_pos_ID à tello_ref_ID renvoie en permanence le même résultat pour un ou deux drone à la fois. Il ne sont plus asservi et maintienne leur commande initiale de manière infini. Le bug semble survenir dans avec l’utilisation de la transformé car les frames TF2 sont correctes sur RVIZ2 mais l’erreur calculé de correspond pas. De plus, la transformée n’échoue pas, elle renvoie toujours la même erreur.

</aside>

<aside>
📥 Le package pour connecter lire les données Optitrack est https://github.com/tud-cor-sr/ros2-mocap_optitrack

</aside>


