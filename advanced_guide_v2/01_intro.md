faire phrase disant on se concentre sur application principales actuelles de SamPy, i.e. mammiferes territoriaux.

faire phrase disant qu'on va se reposer sur le guide begginer (ou y faire des references)

# Il y a plein de choses possibles avec SamPy, mais ici on se concentre sur le cadre principal pour lequel SamPy a ete cree: le developpement d'ABM centres sur des mamiferes territoriaux.

Dans ce guide d'utilisations nous allons donc montrer comment aborder la modelisation d'un mamifere territorial sur un territoire donne en utilisant les outils fournis par SamPy, afin de creer un ABM.

On va commencer par [detailler] les elements principaux qu'il faut choisir pour construire l'ABM, puis nous verrons comment implementer ces choix dans un script.

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

Comment se reproduisent les BasicMammals (pour les polygame, les differences sont mineures et seront misent en evidence).

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
- une fois le nombres d'enfants determine pour chaque femelle mettant bas, des nouveaux agents d'age 0 sont cree (un pour chaque enfants). Leurs atributes lies a ceux de leur mere, telles la position et le territoire, sont directement copie de leur mere, et les autres utilisent les valeurs par defauts stockees dans l'objet `agent` (voir section [cite] pour plus de details sur les attributs). Les enfants gardent en attribut l'ID de leurs deux parents.
- si lors de la creation des enfants, l'utilisateur souhaite utiliser d'autres valeurs que les valeurs par defauts pour certains (ou tous) attributs, il peut utiliser l'argument optionnel `dico_default_value` (voir doc).


### mortality

On presente ici les differentes formes de mortalite non liees a une maladie. Notez qu'un modele peut avoir plusieurs de ces mortalites incluses si elles ne sont pas mutuellement incompatibles. Toutes ces methodes se retrouvent dans le module agent/mortality.py .

#### Kill too old


La methode presentee ici modelise une mortalite densite (d'agents) et ressource dependante. Cette methode se nomme `kill_too_old` et elle est definie dans la classe `MortalityKillTooOld`. Comme son nom l'indique, a chaque fois qu'elle est appelee, cette methode efface completement les agents qui ont atteint ou depasse un age limite fournit par l'utilisateur avec l'argument `limit_age`. L'age dans le modele etant compte en nombre de time-step, il faut faire attention a la valeur fournie ici. Par exemple, dans le cas d'un modele ou les timesteps correspondent a des semaines avec des annees de 52 semaines, pour tuer tous les agents ayant plus de 3 ans, l'utilisateur doit appeler la methode comme suit:
```python
agents.kill_too_old(52 * 3)
```
L'utilisateur peux utiliser l'argument optionnel `condition` pour restreindre l'application de cette methode a un sous-ensemble d'agents.

#### Density and ressource dependant mortality

La methode presentee ici modelise une mortalite densite (d'agents) et ressource dependante. Cette methode se nomme `natural_death_orm_methodology` et elle est definie dans la classe `NaturalMortalityOrmMethodology`. Elle reprend une approche utilisee dans le modele "Ontario Rabies Model" [cite] utilise pour modeliser la rage chez le raton laveur en Ontario, qui associe une probabilite de mourrir (a chaque timestep) a chaque agent conditionne par la valeur de K de sa cellule, le nombre d'agents sur cette cellule, ainsi que de probabilites de deces par age fourni par l'utilisateur. Dans SamPy, cette approche est modelisee comme suit.

- l'utilisateur fournit deux arrays numpy 1D de meme longueur : `array_death_proba_male` et `array_death_proba_female`. Ces arrays contiennent les probabilites de mourrir par age, ou l'age est compte en nombre de timestep pour males et femelles respectivement. Par exemple, `array_death_proba_male[10]` contient la probabilite de mourrir d'un agent male age de 10 timesteps.
- shuffle
- le test et la formule
- les agents decedes sont effaces par defaut
- parametre optionnel `kill_too_old`
- les 2 parametres optionnels `condition` et `condition_count`
- kecece ORM ?

#### offspring mortality

- classe `OffspringDependantOnParents`
- method `kill_children_whose_mother_is_dead`
- cherche si mere existe. Si non, l'agent meurt quand trop jeune (param `age_limit`)
- Cette methode ne tient pas compte des positions respective des jeunes et de leurs mere. Ainsi, si la mere est presente mais a plusieurs cellules de distance, les jeunes survive malgre l'abandon.

#### Seasonal mortality

[FAIRE LES DOCS]

- class `SeasonalMortality`
- plusieurs methodes pour initialiser les mortalite saisonnieres : `initialize_seasonal_mortality_uniform`, `initialize_seasonal_mortality_normal`, `initialize_seasonal_mortality_custom_proportions` selon [...].
- une methode pour gerer la mortalite saisonnier en tant que tel : `deal_with_seasonal_mortality`.
- test 

#### Kill percent

- classe `KillPercentagePop`
- method `kill_proportion_of_population`
- argument condition
- test

### movements


### add new attributes

## disease

## intervention

## implementation