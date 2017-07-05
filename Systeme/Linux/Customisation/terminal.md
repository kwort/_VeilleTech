Terminal
========

Pour modifier l'aspect du terminal, modifier la variable PS1 (inclure l'export dans le bashrc pour du permanent)

exemple : 
		

Generateur
----------

http://bashrcgenerator.com/

Variable
--------

\u : nom de l'utilisateur.
\h : nom de la machine.
\W : nom du dossier courant.
\$ : affiche $ pour un utilisateur et # pour root.
\w : chemin complet du répertoire de travail.
\d : date format texte ("sam. janv. 31").
\A : heure format 24h sans secondes.
\t : heure format 24h avec les secondes.
\T : heure format 12h avec les secondes.
\@ : heure format 12H sans secondes.
\D{%d-%m-%Y %H:%M:%S%z} : Date et heure dans un format personnalisable (ici jour-mois-année sur 4 chiffres heure:minute:seconde fuseau horaire).
$? : code de retour de la dernière commande (0 si OK, 1 si erreur).
`commande_ou_fonction` : lance la commande ou fonction.
\j : nombre de tache en cours dans le terminal (pratique si vous lancez des tâches en arrière plan).
\# : le numéro de la commande.
\v : version de bash.
\n : nouvelle ligne.

Code couleur
------------

\e[1;37

le premier nombre correspond au type de la couleur : 0 foncé, 1 clair, 4 souligné, 7 fond, 9 barré.
le second à la couleur

Noir 0;30
Rouge 0;31
Vert 0;32
Jaune 0;33
Bleu 0;34
Violet 0;35
Cyan 0;36
Blanc 0;37