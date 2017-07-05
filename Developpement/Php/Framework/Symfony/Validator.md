Validator
=========

Un validateur est une classe php qui permet de valider la coherence de donnée d'une entité ou d'une propriété d'entité.
Un validateur peut contenir des options pour définir le comportement du validateur (ex: liste de choix / vérification de host pour email / ...).
Un validateur peut être déclarer comme une service afin d'y injecter des dépendences

On peut appliquer un ou plusieur validateur sur une entité ou une propriété d'entité et définir des options, pour cela on peut utiliser le format yml/xml/annotation/php.
Un entité peut être vérifié afin d'en resortir toutes les violations de contrainte en utilisant le service validator.
