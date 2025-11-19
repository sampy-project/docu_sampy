faire phrase disant on se concentre sur application principales actuelles de SamPy, i.e. mammiferes territoriaux.

faire phrase disant qu'on va se reposer sur le guide begginer (ou y faire des references)

# Il y a plein de choses possibles avec SamPy, mais ici on se concentre sur le cadre principal pour lequel SamPy a ete cree: le developpement d'ABM centres sur des mamiferes territoriaux.

Dans ce guide d'utilisations nous allons donc montrer comment aborder la modelisation d'un mamifere territorial sur un territoire donne en utilisant les outils fournis par SamPy, afin de creer un ABM.

## graph

Premiere etape necessaire, choisir/definir territoire

- territoire virtuel ou territoire "reel" ?
- valeur de K
- cellules / sommets
- position / territory
- matrice de connection
- HexGrid, SquareGrid, SquareGridWithDiag, (OrientedHexagonalGrid) 

[on choisit OrientedHexagonalGrid / SquareGridWithDiag]

On retrouve ces classes dans graph/builtin_graph et on les appelles avec 
```python
from sampy.graph.builtin_graph import SquareGrid, etc
```

Maintenant on a un territoire, il nous faut les agents.

## agents

pour l'instant, dans SamPy, on a que des Mammals qui sont deja construit / immediatement disponible. Pour ceux qui le souhaite, avec l'aide du guide developpeur vous pourrez construire d'autres types d'agents. En l'etat, il y a BasicMammal, BasicMammalWithResistance, BasicMammalPolygamous, BasicMammalPolygamousWithResistance. BasicMammal represente un mammifere territorial "generique" monogame (pour une periode de reproduction donnee), alors que BasicMammalPolygamous represente des mammiferes territoriaux pour lesquels les males peuvent avoir plusieurs partenaires par periode de reproduction. Les version avec Resistance ajoute des fonctionalite de mouvement qui sont decrite [LA].

[on choisit BasicMammal]

On retrouve ces classes dans agent/builtin_agent et on les appelles avec 
```python
from sampy.agent.builtin_agent import BasicMammal, etc
```

### natality

Comment les agents se reproduisent les BasicMammals (pour les polygame, les differences sont mineures et seront misent en evidence).

on a deux methodes pour modeliser la reproduction.

#### finding mate and get pregnant
La premiere que nous allons voir est `find_random_mate_on_position`, qui se trouve dans la classe `FindMateMonogamous` qui est definie dans agent/reproduction.py (pour plus de details, voir la documentation dans le code). Son action se decompose en deux etapes.

##### find mate

- A un semaine specifique, la femelle en age recherche un mate sur sa position (diff monogame polygame). Cette recherche ne tiens pas compte de la famille des agents (i.e. l'inceste est autorise).
- sur une cellules, les femelles et les males sont ordonnees dans deux listes. La liste des males est melangee. Les paires sont formees par les positions dans les listes, i.e. femelle en position 0 avec male en position 0, etc... Les paires sont fixees durant l'appel de la methode.

##### get pregnant

Une fois les couples formes, cette methode effectue un test pour determiner quelles sont les femelles qui tombent effectivement enceinte.
Ce test s'effectue comme suit:
- l'utilisateur fournis une probabilite `prob_get_pregnant` (float entre 0 et 1) que les femelles tombent enceinte.
- pour chaque femelle en couple, un nombre aleatoire uniforme entre 0 et 1 est tire. la femelle devient enceinte si ce nombre est inferieur a `prob_get_pregnant`.

#### create offsprings

La deuxieme methode s'occupe de creer les baybay. Elle se nomme `create_offsprings_custom_prob` et se trouve dans la classe `OffspringCreationWithCustomProb` qui est definie dans agent/reproduction.py (pour plus de details, voir la documentation dans le code). Elle fonctionne comme suit.

- l'utilisateur fournit deux arrays numpy 1D de meme longueur. La premiere, `arr_nb_children` contient des entiers ou chaque valeur correspond a une taille de portee. La deuxieme, `arr_prob_nb_children` contient des float representant les probabilites d'avoir la taille correspondante de portee (remarque: cette array est normalisee par la methode de sorte que la somme des valeurs soit egale a 1). C'est a dire que l'utilisateur etablie que la probabilite que la portee soit de `arr_nb_children[i]` offsprings est de `arr_prob_nb_children[i]`. REMARQUE: l'utilisateur peut choisir de mettre 0 dans la liste des taille de portee possible pour simuler des echecs de mise-bas, ou bien utiliser l'argument optionnel `prob_failure` de la methode (voir doc).
- l'utilisateur peux utiliser l'argument optionnel `condition` pour restreindre l'application de cette methode a un sous-ensemble de femelles enceintes (par exemple, celles qui n'ont pas atteind un certain age et pour lesquels les probabilites des tailles de portees seraient differentes). La valeur fournie doit-etre une array numpy 1D de booleens et de longueur le nombre total d'agents. (donner exemple avec agents.df_population ?).
- Pour chaque femelle mettant bas, la taille de portee est determinee en faisant un tirage aleatoire de `arr_nb_children` pondere par `arr_prob_nb_children`. (a detailler)
- une fois le nombres de rejetons determine pour chaque femelle mettant bas, des nouveaux agents d'age 0 sont cree (un pour chaque rejetons). Leurs atributes lies a ceux de leur mere, telles la position et le territoire, sont directement copie de leur mere, et les autres utilisent les valeurs par defauts stockees dans l'objet `agent` (voir section [cite] pour plus de details sur les attributs). Les rejetons gardent en attribut l'ID de leurs deux parents.
- (mentionner a la fin) dico_default_value


### movements

### mortality

### add new attributes