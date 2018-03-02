compte-rendu de log
===================
# à propos
Les batches de nuit, bien que produisant des journaux d'exécution (logs), ne sont actuellement pas surveillés par le système central.
Aussi est-il nécessaire de consulter individuellement ces logs pour vérifier le bon déroulement de la *nuit batch*. 
Ce document traite de quelques scripts qui présentent dans un tableau les éléments les plus pertinents trouvés dans ces logs.

# installation

# serveurs
Sur chacun des serveurs sur lesquels résident des logs à consulter, installer le script perl ***cr*** dans un répertoire quelconque, idéalement le répertoire ***./bin*** de l'utilisateur. Ce script doit être exécutable.
Au même niveau que ce répertoire, créer si nécessaire le répertoire ***./etc*** et y installer le fichier de configuration ***cr.conf***.
```sh
user@server1:~$ tree
.
|-- bin/
|   `-- cr
`-- etc/
    `-- cr.conf
```

## machine pour *"merge"*
Sur une machine disposant du programme ***xsltproc***, installer le script kornshell ***cr-merge*** dans un répertoire quelconque, de préférence le répertoire ***./bin*** de l'utilisateur. Ce script doit être exécutable.
Ce script nécessite deux autres scripts de transformation xsl, ***merge_step1.xslt*** et ***merge_step2.xslt***, qu'on installera idéalement dans un répertoire ***./lib***, au même niveau que le répertoire précédent.

```sh
user@devcompil:~$ tree
.
|-- bin/
|   `-- cr-merge
`-- lib/
    |-- merge_step1.xslt
    `-- merge_step2.xslt
```
__Nota 1__: La *machine pour merge* peut être l'ordinateur de l'utilisateur.
__Nota 2__: Le serveur ***devcompil*** dispose du programme ***xsltproc***.

## ordinateur de l'utilisateur
Sur l'ordinateur de l'utilisateur, créer un répertoire quelconque, par exemple le répertoire ***./cr*** et y installer le 
script de transformation xsl ***cr_log2html.xslt***

# exécution
Sur chacun des serveurs, lancer le script ***cr*** en redirigeant la sortie standard vers un fichier ***xml***.
```sh
user@server1:~$ cr > ~/cr_server1.xml
```
```sh
user@server2:~$ cr > ~/cr_server2.xml
```

__info__: En l'état, ces fichiers ***xml*** sont lisibles avec un navigateur web, pourvu que le script de transformation xsl ***cr_log2html.xslt*** soit dans le même répertoire.

Rassembler ces fichiers intermédiaires sur la machine pour merge, et lancer le script ***cr-merge*** avec en argument, tous les fichiers ***xml*** précédemment produits, en redirigeant la sortie standard vers le fichier ***xml*** final.
```sh
user@devcompil:~$ cr-merge cr_server1.xml cr_server2.xml > compte-rendu.xml
```
Copier le fichier final sur l'ordinateur de l'utilisateur dans le même répertoire que le script de transformation xsl ***cr_log2html.xslt***.
Ouvrir le fichier avec un navigateur web.

# automatisation
Les étapes de l'exécution détaillée précédemment sont scriptées dans le script bash ***cr***.
La machine de merge est un serveur différent de l'ordinateur de l'utilisateur.

__prérequis__
1) Si tous ces serveurs peuvent être isolés les uns des autres, l'ordinateur de  l'utilisateur doit pouvoir les atteindre tous, par le protocole ***SSH***.
2) Les programmes ***bash***, ***ssh*** et ***scp*** doivent être disponibles. Les environnements **Cygwin** ou **MobaXterm** sont donc conseillés sous Windows ^TM^.
## installation
Sur la machine pour merge, respecter les propositions de répertoires ***~/bin*** et ***~/lib***.
De plus, y créer, si nécessaire, le répertoire ***~/cr***. Ce répertoire recueillera les fichiers ***xml*** intermédiaires.
Sur l'ordinateur de l'utilisateur, installer le script bash ***cr*** dans un répertoire quelconque, idéalement le répertoire ***~/bin***. Ce script doit être exécutable.
Installer le script de transformation xsl ***cr_log2html.xslt*** dans un répertoire quelconque, par exemple le répertoire ***~/cr***.

## lancement
Exécuter le script en redirigeant la sortie standard vers un fichier ***xml*** dans le répertoire contenant le script de transformation xsl ***cr_log2html.xslt*** 
```sh
user@mobaxterm$  cr > ~/cr/cr.xml
```
## consultation
Ouvrir le fichier avec un navigateur web.

# manpage

## cr (script bash)
__Nota__: Ce script appelle les commandes ***ssh*** et ***scp*** en mode batch. Aucun mot de passe ne sera demandé. Il convient donc de préparer des clefs **SSH** pour permettre la communication entre l'ordinateur de l'utilisateur et les différents serveurs.

`syntaxe : cr [-s "[user@]server .."] [-i -u "user .."] [-t ref]`
  * `-s`    Liste des serveurs à contacter, séparés par des virgules ou des espaces.
            Le dernier est le serveur pour merge. 
            Le login peut être précisé devant le nom de chaque serveur, séparé par un @.
            Par défaut, les serveurs contactés sont ***parvs4114420***, ***parva4117029*** et ***devcompil***.
  * `-i`    Utilise les clefs **SSH** qui doivent être présentes dans le répertoire ***~/.ssh*** et nommées ***user@server*** pour chaque serveur.
  * `-u`    Liste des logins, séparés par des virgules ou des espaces.
            Le dernier est le login pour le serveur pour merge.
            Par défaut, le script utilise le nom de l'utilisateur courant auquel est préfixé un *l*.
  * `-t`    permet de préciser un script de transformation xsl alternatif.
            Par défaut : ***cr_log2html.xsl***.
            
__Attention__, le serveur ***devcompil*** nécessite un login qui ne peut pas être deviné. L'utilisateur Foo BAR utilisera donc a minima la commande suivante :
```sh 
$ cr -u ,fbar > ref.xml
```
## cr-merge (script kornshell)
`syntaxe : cr-merge [-l dir] [-t ref.xsl] [-o ref.xml] ref1.xml ref2.xml..`
  * `-l`    Permet de préciser le répertoire dans lequel trouver les scripts de transformation xsl.
            Par défaut, ***.\./lib***.
  * `-t`    Permet de préciser un script de transformation xsl alternatif.
  * `-o`    Permet de préciser un fichier de sortie.

## cr.conf (fichier texte)
Ce fichier permet de paramétrer la recherche des logs.

* Une ligne commençant par un croisillon (#) est un commentaire et n'est pas prise en compte.
* Une ligne vide n'est pas prise en compte.
* Une ligne contenant uniquement un mot entre crochets définit le début de la section des batches pour l'application dont le nom est entre crochets.
* Une ligne de six champs séparés par des deux-points (:) définit un batch et ses paramètres
 1) ***nom du batch***
 2) ***serveur***. Si ce serveur n'est pas identique au hostname, la ligne est ignorée.
 3) ***fréquence***. Si la fréquence n'est pas quotidienne (Q), hebdomadaire (H) ou mensuelle (M), alors la ligne est ignorée.
 4) ***fréquence***. Permet de calculer l'âge des fichiers à rechercher.
 5) ***chemin***. Les fichiers de logs sont recherchés dans ce répertoire
 6) ***fichiers et formats***. Liste de fichiers et leur format. Les fichiers et leur format sont séparés par des barres verticales (|), les fichiers sont séparés de leur format par une virgule. Les noms de fichiers peuvent contenir des jokers ([], * et ?). Les formats pris en compte sont : ***stats***, ***log***, ***log-ok***, ***npcddb***, ***nprdb***, ***prgrlvop*** et ***sqlldr***.

Exemple
```[quid]
batchpurge:parvs4114420:Q:LMMJV_D:/var/log:batchpurge.log.*,log|stats.[0-9]*.csv,stats
quidpurgebatch:parvs4114420:Q:LMMJV_D:/var/lo:quidpurgebatch.log.*,log|stats.[0-9]*.csv,stats
quid-relance-batch:parvs4114420:Q::/exploit/Hercule/log:quid-relance-batch.log.[0-9.\:-]*,log-ok
```
