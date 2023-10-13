# CellTools
This python package provides the tools to build and manipulate unit cells and supercells and to calculate diffraction 
patterns from these cells. Using ```pyqtgraph``` the cells and supercells can be drawn in 3D. 

***This package is under active development and collaborates are welcome. I will try to ensure consistency so that old code
can run on newer version, but cannot guarantee it at this point.*** 

*At some point a readthedocs will also come up, so far you have to work with the docstrings of the functions, 
unfortunately. All classes and functions are well documented.*

## Installation
Either download the whole package from here and install into any environment by executing 
```shell
python3 -m pip install -e celltools
```
inside the download folder, or use
```shell
python3 -m pip install git+git@github.com:HammerSeb/celltools.git
```
to install directly from Github. 

### Known issues
For drawing the unit cells the package uses the opengl module of ```pyqtgraph```, which can be hard to get to work 
sometimes. Before you start, make sure that you can run all the 3D examples of pyqtgraph. Run
```shell
python3 -m pyqtgraph.examples
```
to start the example module and try all 3D examples. Unfortunately I cannot help you if you have problems there. What 
has been known to work is to install ```pyqt5```, ```pyopengl``` and ```pyqtgraph``` into a clean environment and start 
from there. Good Luck!

## How do I use it
The package comes in two parts, ```celltools``` and ```simulation```. The former contains all tools to build a unit cell
and manipulate it, while the latter contains diffraction and pair distribution function simulations. 

### celltools
```celltools``` cotains a linear algebra part ```linalg```, a unit and supercell part ```cell``` and a drawing part ```
draw```. A quick explanation follows of the important features follows

#### linalg
```Basis```: class describing a basis system spanned by three linear independent vectors
```python
from celltools.linalg import Basis
standard_basis = Basis([1, 0, 0], [0, 1, 0], [0, 0, 1])
```

```Vector```: class representing a vector defined by coordinates in a given ```Basis```
```python
from celltools.linalg import Basis, Vector
basis = Basis([2, 0, 0], [1,1,0], [1, 0, -3])
v = Vector([1,0,0], basis)
w = Vector([1,1,1], basis)
v.
z = 3*v + w
print(f"coordinates: {z.vector}")
print(f"global coordinates: {z.global_coord}, global length: {z.abs_global}")
```
vector addition is possible for vectors in the same basis. Scaler multiplication is possible as well. Coordinates in 
basis system, coordinates  and length in global reference frame can be returned by ```Vector.vector```, 
```Vector.global_coord``` and ```Vector.abs_global```, respectively. 
*If no basis is given, vectors are defined in the standard basis.*

```Line``` and ```Plane```: Classes representing lines and planes in 3D space. They are defined by a point of origin and
a directional/normal vector. Can be used to define rotational axis for example. 
```python
from celltools.linalg import Vector, Line, Plane

origin = Vector([1, 1, 1])
vec = Vector([1, 2, 3])

# Line through origin in direction to vec
line = Line(origin, vec)

# Plane through origin, normal to vec
plane = Plane(origin, vec)
```

#### cell
```Latice```, ```Atom``` and ```Cell```: Classes representing lattice vectors, atoms and unit cells. 

```python
from celltools.linalg import Basis, Vector
from celltools.cell import Lattice, Atom, Cell

#Aluminum lattice
lattice = Lattice(Basis([4.59, 0, 0], [0, 4.59, 0], [0, 0, 4.59]))
origin = Vector([0, 0, 0], lattice)
a, b, c = Vector([1, 0, 0], lattice), Vector([0, 1, 0], lattice), Vector([0, 0, 1], lattice)
#Atom positions in an fcc lattice
atoms = [Atom('Al', origin), Atom('Al', a), Atom('Al', b), Atom('Al', a+b), #bottom corners
         Atom('Al', c), Atom('Al', c+a), Atom('Al', c+b), Atom('Al', c+a+b), #top corners
         Atom('Al', 0.5*(a+b)), Atom('Al', 0.5*(a+c)), Atom('Al', 0.5*(b+c)), # faces
         Atom('Al', c+0.5*(a+b)), Atom('Al', b+0.5*(a+c)), Atom('Al', a+0.5*(b+c))
         ]

al_unit_cell = Cell(lattice, atoms)
```
```SuperCell```: Class contructing a super cell of given size from a ```Cell``` object.
```python
from celltools import SuperCell

supercell = SuperCell(al_unit_cell, (3,3,3))
```

```Molecule```: Container class for atoms, can also be content of a cell
```python
from celltools.linalg import Vector
from celltools.cell import Atom, Molecule
#Atoms of CO2 molecule along x-axis of standard basis
atoms = [Atom('C', Vector([0,0,0])), Atom('O', Vector([-1.16,0,0])), Atom('O', Vector([1.16,0,0]))]
#Define molecule
carbondioxide = Molecule(atoms)
carbondioxide.auto_bonds() #add bonds between atoms automatically
```

#### Moving and Rotation
There are explicit functions to move atoms and molecules in ```celltools.cell.tools```. Using the CO2 molecule from 
above as an example shows how it's done:

```python
from celltools.linalg import Line
from celltools.cell import move, rotate
#Move atom 2 Angstrom along x-axis
move(carbondioxide, Vector([2,0,0]))

#Rotate molecule, now centered at (2,0,0) 45 degrees around its z-axis
zaxis = Line(Vector([2,0,0]), Vector([0,0,1]))
rotate(carbondioxide, zaxis, 45, mode="deg")
```

#### Draw
Drawing is done using ```pyqtgraph```. There are a number of function provided in ```draw``` to facilitate the 3D 
depiction of the cells. We will give an example with the aluminum supercell from above
```python
import pyqtgraph as pg
from celltools import make_figure, draw_cell

w = make_figure()
draw_cell(w, al_unit_cell)

pg.exec()
```
Using ```draw_supercell``` instead of ```draw_cell``` draws a given super cell. With ```draw_line``` and ```draw_plane```
Line and Plane objections can be drawn, to e.g. show molecular planes or rotation axis.

### simulation
So far only the electron diffraction simulation has been implemented. *I plan to implement electron pair distribution 
function and x-ray diffraction simulations as well.* 



### Example Code
