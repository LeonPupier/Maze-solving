# Maze-solving made in high school by Léon Pupier
##### *The program was coded in Python. The document is entirely in French, no translation is available...*
## 1. Génération, affichage et parcours de labyrinthe
### 1.0.1 Programme de générération de labyrinthe:

Définition de la valeur représentative d’un mur ( ***WALL*** ) et d’un couloir ( ***VOID*** ) dans le labyrinthe:

```python
# Environment
VOID = 0
WALL = 1
```

La class **Cell()** définie la représentation d’une cellule dans le labyrinthe avec les arguments suivants:
- *list_state* : Une liste contenant l’état des bords ***Nords, Sud, Ouest, Est***, pouvant prendre comme valeur un murs ou le vide.
- *is_entry* : Vrai si la cellule est l’entrée du labyrinthe, sinon Faux.
- *is_exit* : Vrai si la cellule est la sortie du labyrinthe, sinon Faux.

```python
# A cell in the Labyrinthe
class Cell():
  def __init__(self, list_state=None):
    # States
    if list_state is None:
      self.list_state = [VOID, VOID, VOID, VOID] # up, down, left, right
    else:
      self.list_state = list_state
     
    # Entry and exit control
    self.is_entry = False
    self.is_exit = False
```

Fonction prenant un labyrinthe et le parcourant afin de retourner les coordonées de la cellule d’entrée:
```python
# Search the entry in a labyrinthe
def search_entry(labyrinthe):
  for cell in labyrinthe.items():
    if cell[ 1 ].is_entry:
      return cell[ 0 ]
```

Fonction prenant un labyrinthe et le parcourant afin de retourner les coordonées de la cellule de sortie:
```python
# Search the exit in a labyrinthe
def search_exit(labyrinthe):
  for cell in labyrinthe.items():
    if cell[ 1 ].is_exit:
      return cell[ 0 ]

```

Fonction de génération d’un labyrinthe prenant les arguments suivants:
- **SIZE** : Tuple de deux valeurs représentant la taille du labyrinthe en ***x,y***.
- **WALL_PROPORTION** : Proportion de murs dans le labyrinthe final.
```python
# External library
from random import randint

# Generation of the complete labyrinthe
def generation(SIZE=( 50 , 50 ), WALL_PROPORTION= 100 ):
  labyrinthe ={}

  # Cell generation in the labyrinthe
  for cell_x in range(SIZE[ 0 ]):
    for cell_y in range(SIZE[ 1 ]):
      coo =(cell_x, cell_y)
      labyrinthe[coo]= Cell()

      # Place a wall if the cell is an edge of the labyrinthe
      if cell_y == 0 :
        labyrinthe[coo].list_state[ 0 ]= WALL # up
      if cell_y == SIZE[ 1 ]- 1 :
        labyrinthe[coo].list_state[ 1 ]= WALL # down
      if cell_x == 0 :
        labyrinthe[coo].list_state[ 2 ]= WALL # left
      if cell_x == SIZE[ 0 ]- 1 :
        labyrinthe[coo].list_state[ 3 ]= WALL # right

  # Random wall position in the labyrinthe
  wall_count = 0
  while wall_count != WALL_PROPORTION*(SIZE[ 0 ]*SIZE[ 1 ])// 100 :
    coo =(randint( 0 , SIZE[ 0 ]- 1 ), randint( 0 , SIZE[ 1 ]- 1 ))
    border =randint( 0 , 3 )
    if labyrinthe[coo].list_state[border]== VOID:
      labyrinthe[coo].list_state[border] = WALL
      wall_count += 1

  # Place one entry and one exit
  coo_entry= coo_exit= None
  while coo_entry == coo_exit:
    coo_entry= (randint( 0 , SIZE[ 0 ]- 1 ), 0 )
    coo_exit= (randint( 0 , SIZE[ 0 ]- 1 ), SIZE[ 1 ]- 1 )
    
  labyrinthe[coo_entry].is_entry= True
  labyrinthe[coo_exit].is_exit = True
  
  return labyrinthe
```

### 1.0.2 Exemple de labyrinthe généré à la main:

Exemple de génération d’un labyrinthe à la main suivant le pattern ressorti par la fonction **generation()** :
```python
# Example
size_exemple = ( 4 , 4 )
exemple = {
( 0 , 0 ) : Cell([ 1 , 0 , 1 , 1 ]),
( 1 , 0 ) : Cell([ 1 , 0 , 1 , 1 ]),
( 2 , 0 ) : Cell([ 1 , 1 , 1 , 0 ]),
( 3 , 0 ) : Cell([ 1 , 0 , 0 , 1 ]),
( 0 , 1 ) : Cell([ 0 , 0 , 1 , 0 ]),
( 1 , 1 ) : Cell([ 0 , 1 , 0 , 0 ]),
( 2 , 1 ) : Cell([ 1 , 0 , 0 , 1 ]),
( 3 , 1 ) : Cell([ 0 , 0 , 1 , 1 ]),
( 0 , 2 ) : Cell([ 0 , 0 , 1 , 0 ]),
( 1 , 2 ) : Cell([ 1 , 1 , 0 , 1 ]),
( 2 , 2 ) : Cell([ 0 , 1 , 1 , 0 ]),
( 3 , 2 ) : Cell([ 0 , 0 , 0 , 1 ]),
( 0 , 3 ) : Cell([ 0 , 1 , 1 , 1 ]),
( 1 , 3 ) : Cell([ 1 , 1 , 1 , 0 ]),
( 2 , 3 ) : Cell([ 1 , 1 , 0 , 0 ]),
( 3 , 3 ) : Cell([ 0 , 1 , 0 , 1 ]),
}

exemple[( 0 , 0 )].is_entry= True
exemple[( 1 , 3 )].is_exit = True
```

### 1.0.3 Programme d’affichage de labyrinthe:

La fonction **display_labyrinthe()** permet de récupérer un labyrinthe représenté par un dictionnaire comme vu précédemment et de le sauvegarder sur le disque dans le répertoire courant et de l’afficher en prenant en compte les arguments suivants:
- *labyrinthe* : Le dictionnaire représentant le labyrinthe.
- *path_list* : Une liste de coordonées (***x,y***) qui seront coloriés à l’affichage graphique du labyrinthe pour symboliser par exemple un chemin.
- *scale* : Taille du labyrinthe multipliée par la taille initiale size pour un affichage plus grand et donc plus lisible.
- *mutl* : Multiplicateur de la taille de l’image finale du labyrinthe pour son enregistrement, comparable à un zoom intégré.
- *width* : Taille d’une cellule, représenté par un carré en (***x,y***).
- *mode* : Prend pour valeur **RGB** ou **RGBA**, sert à determiner le format d’enregistrement de l’image finale respectivement en ***.jpg*** ou ***.png***.
- *title* : Nom pour le fichier sauvegardé.
- *fg* : Couleur hexadécimal représentant les murs sur le labyrinthe.
- *bg* : Couleur hexadécimal représentant le fond de cellule vide sur le labyrinthe.
- *entry* : Couleur hexadécimal représentant le fond de cellule pour l’entrée du labyrinthe.
- *exit* : Couleur hexadécimal représentant le fond de cellule pour la sortie du labyrinthe.
- *path* : Couleur hexadécimal représentant le fond de cellule pour chaque coordonée fournie dans *path_list*.
```python
# External library
from PIL import Image, ImageDraw
from mathimport *

# Display the labyrinthe in a picture and save it in the current folder:
def display_labyrinthe(labyrinthe, path_list= None , scale= 100 , mult= 10 , width= 5 , mode='RGBA', title='maze', fg='black', bg='#E6E6E3', entry='#6C98FF', exit='#FF6666', path='#FFED91'):
  size = int(sqrt(len(labyrinthe)))
  SIZE = (size, size)

  # Create image
  size_draw = (SIZE[ 0 ]*scale+width, SIZE[ 1 ]*scale+width)
  image = Image.new(mode, size_draw, bg)
  draw = ImageDraw.Draw(image)

  # Draw the labyrinthe
  for cell in labyrinthe.items():
    # Recovery and conversion of cell data
    coo = cell[ 0 ]
    coo = [coo[ 0 ]*scale, coo[ 1 ]*scale, coo[ 0 ]*scale+scale, coo[ 1 ]*scale+scale]
    up, down, left, right = cell[ 1 ].list_state

    # Draw the path if it exists
    try :
      if cell[ 0 ] in path_list and labyrinthe[cell[ 0 ]].is_entry== False :
        draw.rectangle(coo, fill=path)
    except TypeError :
      pass

    # Draw the entry
    if cell[ 1 ].is_entry:
      draw.rectangle(coo, fill=entry)
    # Draw the exit
    elif cell[ 1 ].is_exit:
      draw.rectangle(coo, fill=exit)

    # Draw a wall if it's noted in the data
    if up == WALL:
      draw.line((coo[ 0 ], coo[ 1 ], coo[ 2 ], coo[ 1 ]), fill=fg, width=width)
    if down == WALL:
      draw.line((coo[ 0 ], coo[ 3 ], coo[ 2 ], coo[ 3 ]), fill=fg, width=width)
    if left == WALL:
      draw.line((coo[ 0 ], coo[ 1 ], coo[ 0 ], coo[ 3 ]), fill=fg, width=width)
    if right== WALL:
      draw.line((coo[ 2 ], coo[ 1 ], coo[ 2 ], coo[ 3 ]), fill=fg, width=width)

  # Save and output
  if mult > 1 :
    image= image.resize((SIZE[ 0 ]*mult, SIZE[ 1 ]*mult))

  if mode == 'RGB':
    image.save(f' { title } .jpg')
  else :
    image.save(f' { title } .png')

  display(image)
```

### 1.0.4 Programme de recherche de chemin:

La fonction **search_path()** cherche un chemin sans condition et s’arrête une fois bloquée par un
murs ou quand elle trouve la sortie avec pour argument:
- *labyrinthe* : Un labyrinthe sous la forme d’un dictionnaire comme vu précédemment.
- *coo* : Un tuple de valeur (***x,y***) dans le labyrinthe par lequel la recherche commence.
- *list_visited* : Une liste de coordonées déjà parcourue par la recherche de la fonction.
```python
# Search a path in a labyrinthe without other condition
def search_path(labyrinthe, coo, list_visited):
  path = []
  up, down, left, right = labyrinthe[coo].list_state
  cost_up = cost_down = cost_lef t= cost_right = None

  if labyrinthe[coo].is_exit:
    return coo

  # Check the openings of a cell
  if coo not in list_visited:
    list_visited.append(coo)

    coo_up =(coo[ 0 ], coo[ 1 ]- 1 )
    coo_down= (coo[ 0 ], coo[ 1 ]+ 1 )
    coo_left= (coo[ 0 ]- 1 , coo[ 1 ])
    coo_right= (coo[ 0 ]+ 1 , coo[ 1 ])

  if up == VOID and labyrinthe[coo_up].list_state[ 1 ] == VOID:
    if coo_up not in list_visited:
      cost_up= search_path(labyrinthe, coo_up, list_visited)

  if down == VOID and labyrinthe[coo_down].list_state[ 0 ] == VOID:
    if coo_down not in list_visited:
      cost_down=search_path(labyrinthe, coo_down, list_visited)

  if left == VOID and labyrinthe[coo_left].list_state[ 3 ] == VOID:
    if coo_left not in list_visited:
      cost_left=search_path(labyrinthe, coo_left, list_visited)

  if right== VOID and labyrinthe[coo_right].list_state[ 2 ]== VOID:
    if coo_right not in list_visited:
      cost_right =search_path(labyrinthe, coo_right, list_visited)
```

### 1.0.5 Programme de recherche du plus court chemin:

La fonction **voisin_de()** cherche toutes les cellules voisines ouvertes à une cellule dans le labyrinthe
avec pour arguments:
- *labyrinthe* : Un labyrinthe dictionnaire.
- *coo* : Un tuple (***x,y**) représentant la case à explorer les voisins.
```python
def voisin_de(labyrinthe, coo):
  list_voisin = []
  up, down, left, right = labyrinthe[coo].list_state

  coo_up = (coo[ 0 ], coo[ 1 ]- 1 )
  coo_down = (coo[ 0 ], coo[ 1 ]+ 1 )
  coo_left = (coo[ 0 ]- 1 , coo[ 1 ])
  coo_right = (coo[ 0 ]+ 1 , coo[ 1 ])

  if up == VOID and labyrinthe[coo_up].list_state[ 1 ] == VOID:
    list_voisin.append(coo_up)
    
  if down == VOID and labyrinthe[coo_down].list_state[ 0 ] == VOID:
    list_voisin.append(coo_down)
    
  if left == VOID and labyrinthe[coo_left].list_state[ 3 ] == VOID:
    list_voisin.append(coo_left)
    
  if right == VOID and labyrinthe[coo_right].list_state[ 2 ] == VOID:
    list_voisin.append(coo_right)

  return list_voisin
```

La fonction **search_short_path()** s’aide de l’algorithme de [Dijkstra](https://fr.wikipedia.org/wiki/Algorithme_de_Dijkstra) pour trouver LE plus court
chemin entre deux points, dans le labyrinthe représenté par l’entrée et la sortie.
- coo_entry : Un tuple (***x,y***) dans le labyrinthe par lequel la recherche commence.
```python
def search_short_path(labyrinthe, coo_entry=( 0 , 0 )):
  size =int(sqrt(len(labyrinthe)))

  G= {}
  for i in range(size):
    for j in range(size):
      G[i,j] ={}
      for v in voisin_de(labyrinthe, (i,j)):
        G[(i,j)][v]= 1

  # Dijkstra
  sdeb =coo_entry
  P= []
  d= {}

  infini = 1000000

  for a in G:
    d[a]= infini

  d[sdeb] = 0
  predecesseur= {}

  while len(P) < len(G):
    # Choisir un sommet a hors de P de plus petite distance d[a]
    dmin = infini
    dpos = None

    for a in G:
      if a in P:
        continue

      if d[a] <= dmin:
        dmin = d[a]
        dpos = a

    # on a trouvé : dpos
    a = dpos
    P.append(a)

    # Pour chaque sommet b hors de P voisin de a
    for b in G[a]:
      if b in P:
        continue

      if d[b] >d[a] +G[a][b]:
        d[b] = d[a]+ G[a][b]

      predecesseur[b]= a

  # Reverse path from output to input
  list_short = []
  coo_return = search_exit(labyrinthe)

  try :
    while coo_return != search_entry(labyrinthe) and search_exit(labyrinthe) in predecesseur:
      coo =predecesseur[coo_return]
      list_short.append(coo)
      coo_return =coo
  except KeyError :
    list_short =[]

  return list_short
```

### 1.0.6 Tests de fonctionnement de génération et d’affichage:
```python
# Display the labyrinthe with the example
display_labyrinthe(exemple, mult= 50 , title='exemple')
```
![image-000](https://user-images.githubusercontent.com/100092382/209581467-85b35ffa-4dda-4978-a7dc-a95096e1dd69.png)


```python
# Default configuration
display_labyrinthe(generation())
```
![image-001](https://user-images.githubusercontent.com/100092382/209581414-59917131-e5bd-424a-9795-43f3e7dd5d09.png)


### 1.0.7 Tests de fonctionnement de recherche de chemin:
```python
list_v = []
search_path(exemple, ( 0 , 0 ), list_v)
display_labyrinthe(exemple, list_v, mult= 50 , title='exemple')
```
![image-002](https://user-images.githubusercontent.com/100092382/209581391-0f21e6fd-db53-47c9-aa43-fa79664e3326.png)

```python
list_v = []
labyrinthe = generation()
search_path(labyrinthe, search_entry(labyrinthe), list_v)
display_labyrinthe(labyrinthe, list_v)
```
![image-003](https://user-images.githubusercontent.com/100092382/209581385-5c919e61-010e-40f9-9f12-59ba7a1388c7.png)

### 1.0.8 Tests de fonctionnement du plus court chemin:
```python
labyrinthe = generation(SIZE=( 20 , 20 ))
parcours = search_short_path(labyrinthe, search_entry(labyrinthe))
display_labyrinthe(labyrinthe, parcours)
```
![image-004](https://user-images.githubusercontent.com/100092382/209581372-e10c3244-5881-4ab9-843d-081d75327056.png)


### 1.0.9 Test de génération d’un labyrinthe possédant obligatoirement un chemin:
```python
parcours = []
while parcours == []:
  labyrinthe =generation(SIZE=( 20 , 20 ))
  parcours= search_short_path(labyrinthe, search_entry(labyrinthe)) 
display_labyrinthe(labyrinthe, parcours)
```
![image-005](https://user-images.githubusercontent.com/100092382/209581354-55cfe85f-c153-466a-a113-e3876e29c803.png)
