faire phrase disant on se concentre sur application principales actuelles de SamPy, i.e. mammiferes territoriaux.

faire phrase disant qu'on va se reposer sur le guide begginer (ou y faire des references)

# Il y a plein de choses possibles avec SamPy, mais ici on se concentre sur le cadre principal pour lequel SamPy a ete cree: le developpement d'ABM centres sur des mamiferes territoriaux.

Dans ce guide d'utilisations nous allons donc montrer comment aborder la modelisation d'un mamifere territorial sur un territoire donne en utilisant les outils fournis par SamPy, afin de creer un ABM.

## graph

Premiere etape necessaire, choisir/definir territoire

- territoire virtuel ou territoire "reel" ?
- valeur de K
- cellules / sommets
- matrice de connection
- HexGrid, SquareGrid, SquareGridWithDiag, (OrientedHexagonalGrid) 

Maintenant on a un territoire, il nous faut les agents.

## agents

pour l'instant, dans SamPy, on a que des Mammals qui sont deja construit / immediatement disponible. Pour ceux qui le souhaite, avec l'aide du guide developpeur vous pourrez construire d'autres types d'agents. En l'etat, il y a BasicMammal, BasicMammalWithResistance, BasicMammalPolygamous, BasicMammalPolygamousWithResistance. BasicMammal represente un mammifere territorial "generique" monogame (pour une periode de reproduction donnee). 

### natality
### movements
### mortality