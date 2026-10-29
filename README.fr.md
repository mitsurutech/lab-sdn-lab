# Lab SDN pour la classe : Open vSwitch, OpenFlow et un switch qui apprend, dans une seule VM

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md) | [Deutsch](README.de.md)

[![check](https://github.com/mitsurutech/lab-sdn-lab/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-sdn-lab/actions/workflows/check.yml)

Un **lab de réseau défini par logiciel (SDN)** prêt à l’emploi pour enseigner comment un switch fait circuler le trafic, issu de la [plateforme d’apprentissage MitsuruTech](https://mitsurutech.co.uk/fr/learning), où les établissements scolaires donnent à chaque élève ses propres machines et son terminal dans le navigateur. Chaque élève reçoit une **VM** Ubuntu avec **Open vSwitch**, **Mininet** et la bibliothèque de contrôleur OpenFlow **os-ken**, et un switch `br0` relié à trois hôtes, `h1`, `h2` et `h3`. Le switch démarre avec une table de flux vide : il ne transmet rien. L’élève écrit des **flux OpenFlow à la main** avec `ovs-ofctl`, puis termine un **contrôleur de switch à apprentissage** qui apprend sur quel port se trouve chaque adresse MAC et installe un flux pour elle. Rien n’est installé sur l’ordinateur de l’élève et aucun matériel réseau n’est nécessaire.

## Ce que fait l’élève

Le lab écrit un `README.md` dans le dossier personnel, qui explique quoi faire sans donner aucune réponse, et deux fichiers à modifier, `/etc/sdnlab/flows.txt` et `/etc/sdnlab/switch.py`, chargés à chaque démarrage du lab. Les hôtes sont des espaces de noms réseau (`h1` est 10.0.0.1, `h2` 10.0.0.2, `h3` 10.0.0.3, avec des adresses MAC finissant par 01, 02 et 03) et le switch est en fail-mode secure : sans flux, il rejette tous les paquets. En trois parties :

1. **Mininet** (non noté) : `sudo mn --mac --test pingall`, puis une invite Mininet pour observer les nœuds, les liens et les flux que son contrôleur installe.
2. **Écrire la table de flux à la main** dans `flows.txt` : h1 et h2 doivent se joindre dans les deux sens (ARP comme l’écho), un flux qui correspond à h3 doit le rejeter, et aucun flux ne doit utiliser l’action `NORMAL`.
3. **Terminer un contrôleur** dans `switch.py` : trois étapes TODO dans `packet_in` pour apprendre l’adresse source, choisir le port et installer un flux, afin que les paquets suivants d’une conversation n’arrivent jamais au contrôleur.

Le lab ajoute une commande `sdnlab` (`status`, `flows`, `controller`, `reset`, `shell h1`) pour reconstruire le lab, démarrer le contrôleur, remettre les deux fichiers comme fournis et ouvrir un shell dans un hôte.

## Comment il est noté

Le lab comporte une **vérification**, un script qui note une VM rendue sans que l’enseignant ait à la lire. Elle s’exécute sur une copie de la machine enregistrée, après un démarrage à neuf, et charge elle-même les deux fichiers de l’élève : un flux saisi à la main et jamais enregistré ne compte pas.

- **Partie 2, sans contrôleur :** h1 et h2 doivent se joindre, h3 ne doit joindre personne et personne ne doit joindre h3, et un flux qui correspond à h3 doit le rejeter et avoir été touché par des paquets de h3. Un flux avec l’action `NORMAL` ne passe pas.
- **Partie 3, le contrôleur de l’élève :** la vérification le démarre, donne à h2 une adresse MAC que personne n’a vue, de sorte qu’une table écrite d’avance ne sert à rien, puis envoie des pings entre toutes les paires. Le switch doit envoyer les paquets inconnus au contrôleur, et le contrôleur doit avoir installé un flux pour chaque paire d’adresses, écrit `priority=1,in_port=PORT,dl_src=SOURCE_MAC,dl_dst=DESTINATION_MAC` avec l’action de sortie vers le port appris, et aucun des flux installés ne doit inonder.

La première ligne affichée dit ce qui ne va pas.

Ce qu’elle ne fait pas : elle teste avec ping et lit la table de flux, pas d’autres types de trafic, et un élève qui a les droits root sur sa propre VM pourrait encore trafiquer les outils sur lesquels la vérification s’appuie. C’est une vérification pour la classe, pas une preuve.

## Ce que contient le fichier

`sdn-lab.lab.yaml` est une **définition de lab VM** complète, exactement ce que la plateforme exécute :

| Clé | Ce que c’est |
| --- | --- |
| `cpus`, `memory`, `disk` | la taille de la VM de chaque élève : 1 CPU, 1024 Mo et 10 Go |
| `open` | les ports qui s’ouvrent dans le navigateur : aucun, le lab est le terminal |
| `setup` | un script exécuté une fois, en tant que root, au premier démarrage de la VM de chaque élève : il installe Open vSwitch, Mininet, os-ken, `iproute2`, `ping` et `tcpdump` depuis l’archive d’Ubuntu, et écrit la commande `sdnlab`, un petit lanceur pour le contrôleur, les deux fichiers à modifier, une unité systemd et le README |
| `check` | le script de notation, exécuté en tant que root sur une copie du disque enregistré après un démarrage à neuf ; son code de sortie est le résultat et `0` signifie réussi |

## L’utiliser sur la plateforme

**Pour le donner comme travail à une classe :**

1. Dans votre module, ajoutez une tâche et choisissez la carte **Lab VM Ubuntu**.
2. Appuyez sur l’exemple Software-defined networking, ou collez le contenu de `sdn-lab.lab.yaml`. Utiliser un exemple remplace le texte, et Annuler rétablit le vôtre.
3. Publiez et essayez d’abord votre propre copie. Chaque élève a sa propre VM, conservée d’une session à l’autre. Le premier démarrage prend quelques minutes, le temps que le script installe les paquets.
4. Distribuez-le comme n’importe quelle tâche. Une fois rendu, lancez la vérification sur la ligne d’un élève dans la liste de la classe : elle démarre une copie de sa machine enregistrée, qui n’est pas sauvegardée en retour, et indique si elle a réussi et ce qu’elle a affiché.

Une leçon des bases de la plateforme, SDN Basics, se travaille dans ce lab. La [page de ce lab](https://mitsurutech.co.uk/fr/help/lab-sdn-lab) dans le manuel, son [guide des labs VM Ubuntu](https://mitsurutech.co.uk/fr/help/labs) et sa [page des exemples de labs](https://mitsurutech.co.uk/fr/help/sample-labs) disent le reste.

Pas encore de compte pour votre établissement ? [Votre première année est gratuite](https://mitsurutech.co.uk/fr/pricing) jusqu’à 50 élèves, et [l’inscription](https://mitsurutech.co.uk/fr/signup) ne demande qu’une adresse e-mail.

## Vérifier le fichier

À chaque push, `.github/workflows/check.yml` vérifie que le fichier est un YAML valide avec les clés ci-dessus et que `setup` et `check` tiennent chacun dans la limite de 16 Ko de la plateforme. Il ne démarre pas de VM : cela se fait sur les machines de la plateforme.

## Ce qui ne fonctionne pas dans un lab

La VM atteint l’archive de paquets d’Ubuntu, PyPI, npm et GitHub, et rien d’autre. Le réseau entre les hôtes est virtuel et reste dans la VM : rien n’en sort. La bibliothèque os-ken dépend d’eventlet, qui est obsolète et affiche un avertissement dans le journal du contrôleur ; c’est pourquoi le lab enseigne d’abord `ovs-ofctl` et Mininet. La commande `sudo mn -c` de Mininet supprime tous les ponts Open vSwitch, `br0` compris : lancez ensuite `sudo sdnlab flows` pour reconstruire le lab. Le script de préparation et la vérification ont été exécutés dans un conteneur qui tient lieu de VM ; un essai sur une vraie VM d’élève est utile avant un cours.

## À propos

Réalisé par [Mitsuru Technologies](https://mitsurutech.co.uk/fr) pour la [plateforme d’apprentissage MitsuruTech](https://mitsurutech.co.uk/fr/learning) : Python, Java, Go, Node, SQL et Linux dans le navigateur pour les écoles et les établissements, avec des labs de conteneurs, des labs VM, des applications web déployées depuis GitHub et des quiz qui se corrigent tout seuls. Essayez du code sans compte dans le [bac à sable](https://mitsurutech.co.uk/fr/playground).

D’autres labs du même éditeur : [lab-netlab](https://github.com/mitsurutech/lab-netlab), [lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop), [lab-dvwa](https://github.com/mitsurutech/lab-dvwa), [lab-webgoat](https://github.com/mitsurutech/lab-webgoat), [lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin), [lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea), [lab-sample](https://github.com/mitsurutech/lab-sample), [lab-reverse-lab](https://github.com/mitsurutech/lab-reverse-lab), [lab-formal-lab](https://github.com/mitsurutech/lab-formal-lab), [lab-hdl-lab](https://github.com/mitsurutech/lab-hdl-lab), [lab-k3s-lab](https://github.com/mitsurutech/lab-k3s-lab), [lab-tofu-lab](https://github.com/mitsurutech/lab-tofu-lab), [lab-ansible-roles](https://github.com/mitsurutech/lab-ansible-roles) et [lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics).

## Licence

Cette œuvre est sous licence [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/). Vous pouvez la partager et l’adapter pour apprendre et enseigner, à condition de citer Mitsuru Technologies Ltd, de ne pas l’utiliser commercialement et de partager tout ce que vous en tirez sous la même licence. Le texte complet est dans le fichier `LICENSE`.

[Open vSwitch](https://www.openvswitch.org), Mininet et os-ken sont installés depuis l’archive d’Ubuntu au démarrage d’une VM ; rien n’en est distribué ici.
