# AbaqusUnitCell.jl
AbaqusUnitCell.jl provides functions for applying **periodic boundary conditions** on a **repeating unit cell** modelled in **Abaqus**.
The package contains functions for the following tasks:
- reading an Abaqus input-file,
- defining periodic boundary conditions,
- defining homogeneous displacement boundary conditions, and
- saving a new input-file with the defined boundary conditions.

**The following sections are currently in progress and therefore incomplete.**

## Model requirements
- The planes, which are defined by the periodic surfaces of the structure, should form a cuboid.
- The surfaces have to be perpendicular or parallel to the coordinate axes.
- The mesh has to be similar on opposing surfaces.
- If the structure is cuboid, the vertices can be found automatically, otherwise, the vertices have to be modelled as Reference-Points in the Assembly
- When using tied surfaces or interactions:
	- if the coupled surfaces intersect with a periodic surface, the slave nodes have to be collected in a set called "Slaves"
	- These nodes will then be ignored in the equations to prevent overconstraints.

## Global and local coordinate system
For onedimensionally and twodimensionally periodic structures the applied boundary conditions depend on the directions of the periodicity.
While the structure is modelled in the **global** x-y-z-coordinate system, the periodic boundary conditions as well as the load boundary conditions are defined in a **local** I-II-III-coordinate system.
This distinction between global and local coordinates simplifies the definition of the PBCs.

Hereby, the local coordinate system is defined by choosing on axis of the global coordinate system as *reference axis*.
Direction I is then oriented in the same direction as the reference axis.
Similar to the global coordinates, the axis I, II, and III form an orthogonal right-hand-sided coordinate system.

| ref. axis | I   | II  | III |
| --------- | --- | --- | --- |
| x         | x   | y   | z   |
| y         | z   | x   | y   |
| z         | y   | z   | x   |

## Periodicity
In `AbaqusUnitCell.jl` onedimensional, twodimensional, or threedimensional periodicity can be defined for the 3D structure.
The global direction in which the structure is periodic is defined by the choice of the reference axis as follows:
- onedimensional periodicity: the structure is periodic in direction of the reference axis, which means, the coupled surfaces are perpendicular to the local direction I.
- twodimensional periodicity: the reference axis is perpendicular to the free surface. Therefore, the periodic surfaces are perpendicular to directions II and III.
- threedimensional periodicity: Each surface is couppled to the opposite one.

## Name convention for surfaces, edges and vertices
For the definition of the periodic boundary conditions the six surfaces of the repeating unit cell are named *North*, *South*, *East*, *West*, *Top*, and *Bottom*.
The assignment of the surface names depends on the definition of the local coordinate system.
*Top* and *Bottom* are always perpendicular to axis I, *East* and *West* are perpendicular to axis II, *North* and *South* are perpendicular to axis III (see figure ??).

Figure with name convention

The edges and vertices are named after the surfaces intersecting in this edge or vertex; for a shorter notation, only the first letters are taken into account (e.g. *SW* for the edge of *South* and *West* or **NWB** for the vertex of *North*, *West*, and *Bottom*).

## Periodic Boundary Conditions
- [ ] Tolerance
- [ ] Why are exceptions needed?
- [ ] Why is setVertexFinder necessary?


## Load boundary conditions
The loads can be defined by the macroscopic strain tensor $\langle\varepsilon\rangle$.
This macroscopic strain tensor has to be formulated in the **local** coordinates!
For 1D periodicity:
$$\langle\varepsilon\rangle_{I\;I}$$
For 2D periodicity:
$$\langle\varepsilon\rangle = \begin{pmatrix}
\langle\varepsilon_{II\;II}\rangle & \langle\varepsilon_{II\;III}\rangle \\
\langle\varepsilon_{III\;II}\rangle & \langle\varepsilon_{III\;III}\rangle \\
\end{pmatrix}$$
For 3D periodicity:
$$\langle\varepsilon\rangle = \begin{pmatrix}
\langle\varepsilon_{I\;I}\rangle & \langle\varepsilon_{I\;II}\rangle & \langle\varepsilon_{I\;III}\rangle \\
\langle\varepsilon_{II\;I}\rangle & \langle\varepsilon_{II\;II}\rangle & \langle\varepsilon_{II\;III}\rangle \\
\langle\varepsilon_{III\;I}\rangle & \langle\varepsilon_{III\;II}\rangle & \langle\varepsilon_{III\;III}\rangle \\
\end{pmatrix}$$

It should be noted that only macroscopic strains in the direction of periodicity can be reasonably applied on the repeating unit cell. 

## Exported functions
### `AbqModel(path::String)`
Reads the Input-File in path `path` and parses all the necessary information of the file for computing the boundary conditions.
The functions return value is of type `AbqModel`.

### `setRefAxis!(abq::AbqModel, axis::String)`
Changes the reference axis in the Model `abq` to the given `axis`.
If the reference axis is not explicitly set, the default value `x` is used.

### `setTolerance!(abq::AbqModel, tol::Number)`
Sets the tolerance used for finding corresponding nodes to the value `tol`.
If the tolerance is not explicitly set, the default value `0.001` is used.

### `setExceptions!(abq::AbqModel, exc::Array{String,1})`
Sets exceptions for defining the periodic boundary equations.
For each nodal pair, the program checks, if the instance, on which the node is located, is set as exception.
If so, the equation will not be written to the model.
Therefore, the array `exc` has to contain all the instance names, which should be excluded from the equations.

### `setVertexFinder!(abq::AbqModel, val::Bool)`
Defines, if the vertices should be found automatically (`val == true`) or if they have already been defined as Sets in the Assembly (`val == false`).
If the VertexFinder is not set explicitly, the vertices will be defined automatically by default.

### `setPBCdim!(abq::AbqModel, dim::Float64)`
Sets the dimension of periodicity in the model to the valu `dim`.
If the dimension is not set explicitly, threedimensional periodicity is used by default.

### `nodeDesignation!(abq::AbqModel)`
Searches for all the nodes, which are located on te surface of the cuboid unit cell and writes the definition of these nodes (essentially combinations of name and node number) to the given Model `abq`.

### `pbc!(abq::AbqModel)`

### `LoadCase(strain::Array{<:Number,2}, abq::AbqModel)`

### `update!(abq::AbqModel)`
