# TODO

- [X] Gestion de la mise à jour de version de nxlog (Linux ET Windows)
- [X] Suppression du dossier {{ nxlog_linux_temp_dir }} après installation
- [X] Définir où mettre "become: true" pour faire une élévation de privilège seulement quand nécessaire
- [X] Explique-moi pourquoi graylog est dans les template de conf
- [X] Dans notre cas, supprimer le fichier download.yml. On gérera les sources différement
- [X] Fichier install.yml en trop ?
- [X] Voir mes commentaires (commencent par "# PMA - ")
- [X] Etudier la possibilité de vérifier version nxlog sans script mais avec un module Ansible (package_facts ?)
- [X] Supprimer la partie de vérification des prérequis en mode hors-ligne (on a accès à un repo de packages Debian local)
- [X] daemon_reload dans le handler plutôt que dans les tasks (car reload seulement si nouvelle install ou modif de conf)
- [X] Ne pas checker les prérequis si NXlog déjà installé et à la version souhaithée
- [X] Récupérer/utiliser l'adresse IP de l'hôte via Ansible (facts ou fichier d'inventaire) - Linux et Windows
- [X] Vérifier s'il exite un module Ansible pour récupérer les infos d'applis ou services installés (évite d'utiliser le module shell ou similaire)
- [X] Essayer de tester les chemins via un module Ansible (task - Détecter le chemin d'installation NXLog) - bonus : mettre cela dans une boucle

Choses faites/rajoutées :

Les scripts PowerShell ont été retirés pour utiliser uniquement des modules Ansible lorsque nécessaire.
Récupération des adresses IP via l'inventaire
become 'no' retiré lorsque inutile pour optimiser le code

Fichier "download.yml" et "install.yml" supprimés au profit de "windows.yml" & "debian.yml"

Explications :

Pourquoi Graylog est-il dans les templates?

Au tout début du projet, les logs étaient affichés sous forme de message complet.

Il était très difficile de récupérer les champs interessants ne serai-ce qu'un utilisateur ou un code d'erreur, ou même son adresse IP.

Un changement de format était donc nécessaire. Après avoir choisi JSON comme format, NXLog envoyait bien les logs mais Grafana n'arrivait pas a les afficher.

Après avoir fouillé sur Internet, le format GELF a retenu mon attention. C'est un format très proche du JSON mais qui est mieux parsé, et pris en charge par NXLog et Grafana. Les essais ont été concluants, les logs étaient correctements affichés et recupérer une information était très facile a faire. C'est donc le format GELF qui a été retenu pour ce projet.
