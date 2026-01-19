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
- [ ] Ne pas checker les prérequis si NXlog déjà installé et à la version souhaithée
- [ ] "become: no" est le comportement par défaut --> mettre "become: yes" seulement là où c'est nécessaire
- [ ] Récupérer/utiliser l'adresse IP de l'hôte via Ansible (facts ou fichier d'inventaire) - Linux et Windows
- [ ] Vérifier s'il exite un module Ansible pour récupérer les infos d'applis ou services installés (évite d'utiliser le module shell ou similaire)
- [ ] Essayer de tester les chemins via un module Ansible (task - Détecter le chemin d'installation NXLog) - bonus : mettre cela dans une boucle

Choses faites/rajoutées :

Fichier "download.yml" et "install.yml" supprimé pour ne garder que "windows.yml" & "debian.yml"

Ajout d'une section dans les "tasks" Windows & Debian.yml pour récupérer l'adresse IP de la machine et l'ajouter dans les template.

Un dashboard Grafana utilise une variable avec l'IP de la machine pour filtrer les résultats. Au lieu de la taper a la main dans la configuration NXLog, celle-ci est ajoutée automatiquement.
Par exemple, on peut retrouver cela a la ligne 100 du template pour Windows.

Explication : pourquoi graylog est-il dans les template ? 

Au tout début du projet les logs étaient affichés sous forme de message complet. 

Il était très difficile de récupérer les champs interessants ne serai-ce que un utilisateur ou un code d'erreur. Après avoir choisi JSON comme format, Grafana recevait mal les logs et ne voulait pas les afficher. 

Au final, c'est le format GELF qui est choisi car il est mieux parsé que du JSON, NXLog arrive a l'envoyer correctement et Grafana arrive a l'afficher et a récupérer les informations très simplement.