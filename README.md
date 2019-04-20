## Norma Lisa

Outil à base de règles et de ressources linguistiques pour normaliser automatiquement l'orthographe de textes français.
Ce script a été élaboré à l'occasion d'une collaboration de plusieurs mois avec l'équipe d'ARTFL (Université de Chicago) au printemps 2018.

### Objectifs

Rendre possible des études diachroniques variées (recherches d'occurrences, textométrie, _topic modeling_, collocations…) sur des corpus de vaste ampleur, datant du XVIe au XXe siècle.

Ce script, en cours de perfectionnement, est voué à être mis à disposition de la communauté scientifique en accès libre sur GitHub.
Il est actuellement appliqué au corpus FRANTEXT (3559 textes publiés entre le XVIe et le XXe siècle) via la plateforme d’analyse textuelle _Philologic4_, développée par ARTFL.
Il sert d’autres projets au sein du Labex OBVIL (Sorbonne Université), tel que le projet « Haine du théâtre », qui établit l'édition numérique savante d'une centaine de textes non modernisés du XVIe siècle.
Il a également aidé à la normalisation des textes en français du projet _Electronic Enlightenment_ de la Bibliothèque Bodleian (Oxford).

### Fonctionnement

_Norma Lisa_ s’appuie sur plusieurs bases lexicales ouvertes, développées par l’ATILF : _Morphalou_, lexique ouvert de formes fléchies du français ; et _LGeRM_, lexique morphologique répertoriant les flexions et variations orthographiques du moyen-français et du français des XVIe-XVIIe siècles. A partir de ces ressources, _Norma Lisa_ réalise sur chaque mot/token du texte, selon son étiquetage grammatical, une somme de transformations.
