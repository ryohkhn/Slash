# Projet

Le but du projet est de programmer un interpréteur de commandes (aka shell) interactif reprenant quelques fonctionnalités plus ou moins classiques des shells usuels : outre la possibilité d'exécuter toutes les commandes externes, `slash` devra proposer quelques commandes internes, permettre la redirection des flots standard ainsi que les combinaisons par tube, adapter le prompt à la situation, et permettre l'expansion des chemins contenant certains jokers décrits ci-dessous.

## Contributeurs

| Nom       | Prénom  | Github            |
|:----------|:--------|:------------------|
| RODRIGUEZ | Lucas   | @ryohkhn          |
| VYSHKA    | Tedi    | @tedivyshka       |
| MARTINEAU | Clément | @clementmartineau |

## Installation et utilisation

1. Cloner le dépôt

```bash
git clone https://github.com/ryohkhn/Slash.git
# or
git clone git@github.com:ryohkhn/Slash.git
```

2. Build

```shell
make
```

3. Exécution

```shell
./slash
```

4. Suppression des fichiers compilés

```shell
make clean
```

## Architecture logicielle

Le programme Slash se compose de 7 fichiers en langage C et un fichier makefile pour la compilation.  
L'exécution de `make` à la racine du dépôt crée (dans ce même répertoire) l'exécutable slash qui peut être lancé avec `./slash`,
`make clean` efface tous les fichiers compilés.

Entre les 7 fichiers C, `slash.c` est le fichier qui initialise les variables et lance la boucle principale du programme.  
Le fichier `utilities.c` contient des fonctions utilitaires utilisées pour gérer les erreurs d'allocation de mémoire ainsi que les tableaux de strings et les structures.

Les autres fichiers sont utilisés par slash pour répondre au sujet, respectivement:  
l'étoile et double étoile dans lineTreatment.c,  
les commandes cd, pwd, exit et commandes externes dans commands.c,  
les signaux dans signal.c,  
les pipeline dans pipeline.c,  
les redirections dans redirection.c.

## Structures de données

Les structures de données que le programme utilise se trouvent dans le fichier `utilities.c` sous cette représentation :  
`struct cmd_struct` représente une commande, cette structure contient une liste (char**) des parties de la commande et la taille (size_t) de cette liste.  
`struct cmds_struct` représente une liste de cmd_struct, cette structure contient une liste (cmd_struct*) des commandes et la taille (size_t) de cette liste. La séparation des commandes entre plusieurs `cmd_struct` s'effectue sur les pipes.  

## Algorithmes implémentés

Après que la ligne de commande ait été récupérée par readline, l'expansion des jokers est effectuée puis la ligne est transformée en structure `cmds_struct` contenant chaque commande et ses arguments.  
Une fonction `interpreter()` dans `lineTreatment.c` va traiter les différents cas possibles de la ligne de commande écrite :   
    - Une ligne de commande contenant des pipes  
    - Une commande interne ou externe avec redirection, ou non  

`handle_pipe()` dans `pipeline.c` va traiter itérativement tous les pipes dans la ligne ainsi que les redirections éventuelles de l'entrée standard, sortie standard et des sorties d'erreur.  
Chaque fils va alors exécuter sa commande et le père va attendre leur fin pour récupérer les valeurs de retour ainsi que l'éventuelle interception d'un signal.  

`handle_redirection_extern()` et `handle_redirection_intern()` dans `redirection.c` sont semblables.  
Les redirections de l'entrée, sortie et erreur standard sont traitées puis la commande est executée.  
Pour les commandes externes, toute la fonction s'effectue dans le processus fils et le père agit comme dans la fonction `handle_pipe()`.

L'algorithme utilisé pour implémenter l'extension du joker '**' peut s'apparenter au parcours en profondeur d'un arbre. Le répertoire courant en serait la racine, les répertoires qu'il contient des sous arbres. Les fichiers ainsi que tous les répertoires sont considérés comme de potentielles feuilles. Attention ! Un répertoire peut-être racine d'un sous arbre ET feuille d'un chemin ! 
À chaque répertoire parcouru, on essaie le chemin constitué de `[chemin vers le répertoire]/[Entrée dans ce répertoire]/[Suffixe]` en l'envoyant à la fonction qui gère '*' simple. Si le chemin est correct alors on renvoie ce chemin combiné avec tous les autres chemins possibles.

La gestion de '*' peut s'expliquer de cette manière : 
avant tout le chemin étudié est de cette forme -> `[prefixe]/[mot *]/[suffixe]`
On commence par ouvrir le répertoire correspondant au `[prefixe]`, ensuite on compare les entrées avec `[mot *]`, puis on combine tous les chemins de la forme `[prefixe]/[entrée]/[suffixe]`.
