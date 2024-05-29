# 4DO-Specification
The official specification for the 4DO Geometry File Format

# 4D OBJ File Format Specification
### Specification Version 1
April 7, 2024

## Background
In recent years, four-dimensional software and video games have seen a steady rise in popularity and interest. Games like Miegakure, 4D Miner, 4D Golf, 4D Toys, and Moena all have growing fan bases and some have seen viral success with their devlogs and status updates. Other software such as Stella4D, Polychora, and HoxelDraw are geared toward 4D content creation and visualization. So far, there seems to be little or no collaboration or coordination between any 4D software. As of April 2024, there is no consensus on a standard file format for 4D geometric objects. Some software (at least Stella4D and Polychora) have used a modified version of the .OFF file format, originally designed for 3D polytopes. However, the extensions to 4D seem out-of-place within the file format. Additionally, the geometric structure of the .OFF format (arbitrary cells composed of arbitrary polygons) requires too much post-processing to transform the data into a tetrahedral mesh. Other software uses either a proprietary format, or opts to generate geometric models on-the-fly.

I propose a new 4D geometry file format that is designed specifically for 4D computer graphics applications.

### Design Goals
- Enable 4D geometric data transfer between 4D applications
- Use the most basic, common building blocks of 4D computer graphics ("least common denominator")
- Human readable
- Contain mostly primary data, very little metadata, and no secondary data (i.e. data that can be generated from primary data)
- Describe only geometry, not a full scene
- Minimize data duplication

This format might not be the most efficient, nor the most flexible (like .glTF). It does not cover all possible use-cases of 4D geometric computation. This format is intended to be simple to read and write, both by humans and computers. Its features are unapologetically skewed toward those needed for 4D Computer Graphics applications.

### Core Features
- Vertices
  - Position
  - Normal
  - Texture coordinate
  - Color
- Polylines
- Tetrahedral cells
- Compound cells (contiguous groups of tetrahedral cells)
- Multiple named objects
- Model handedness

### Format Design Inspiration: Wavefront OBJ
The 4DO format is mostly patterned after the ubiquitous Wavefront OBJ file format for 3D geometry. The OBJ format is popular for its human readability, simplicity, and its ability to reuse vertex data for multiple faces. The 4DO format will feel very similar to the OBJ format with a few key differences:

1. It's 4D instead of 3D (obviously)
2. It supports only tetrahedral cells (3-simplex) whereas OBJ supports n-sided convex and concave polygons
3. The indices are Zero-based, as opposed to OBJ which is One-based
4. Relative indices are NOT supported in 4DO (no negative indices)
5. 4DO provides a mechanism for defining the coordinate frame of the model (model handedness)
6. Indices are relative to the current object, rather than global
7. It supports vertex colors in addition to texture coordinates

## About this document
**OPTIONAL**, **REQUIRED**, **MAY**, **MAY NOT**, **SHOULD**, **SHOULD NOT**, **MUST**, **MUST NOT**

**OPTIONAL**: The inclusion and/or exclusion of valid **OPTIONAL** elements does not affect the validity of the file.

**REQUIRED**: If a given file does not include a required element or does not satisfy a required specification, it is Invalid. The application is not expected to process such a file and can reject it. When possible, the reason for rejection should be given.

**MAY**/**MAY NOT**: The practice and/or convention is allowed, but not required.

**SHOULD**/**SHOULD NOT**: Denotes a convention or best practice, but is not strictly required for a valid 4DO file.

**MUST**/**MUST NOT**: Failure to follow the convention or practice will render the file Invalid.

## File Structure

A 4DO file **SHOULD** begin with "4DO" followed by a space and then the file specification version, which is currently "1".

A 4DO file is made up of keywords and data.

Each line of the file contains a separate command, which starts with a keyword and is followed by data.

Each command is on its own line.

Each command starts with the keyword.

Blank lines are ignored.

Commands are processed sequentially. A Command **MUST NOT** reference data that is further down the file.

Floating point numbers can have arbitrary precision. Use the period instead of the comma to denote fractional values. Floating point numbers **MUST** be real numbers. Negative numbers **MUST** use the `-` sign. Positive numbers **MAY** use the `+` sign but any number without a sign is assumed to be positive.

Floating point numbers **MAY** be written as integers, without a decimal point (e.g. `1`)

Floating point numbers **MAY** be expressed in scientific notation.

Indices are zero-based (as opposed to Wavefront OBJ where indices are 1-based)

Indices **MUST** be non-negative real integers and **MUST NOT** refer to out-of-bounds data positions (i.e. if there are 10 vertices defined, a Tetrahedron **MUST NOT** attempt to refer to a vertex at index 10 (remember that indices are zero-based))

Keywords are case-insensitive.

4DO files **MUST** use Unicode characters and **MUST** be encoded using UTF-8.

## Geometric Structure
There are several ways to define 4D geometric entities. Vertices and vertex data are the foundation of this file format. All other structures build upon collections of vertex data (or collections of collections of vertex data).

Cell => Tetrahedron => Vertex

Tetrahedron => Vertex

Path => Vertex

Vertex

[TODO: it'd be nice to have a diagram to really drive this point home]
[TODO: also maybe provide use-cases for each of these four structures. Point clouds, tet meshes, cell meshes, wireframe meshes]

## Comments

Comments are denoted with the `#` symbol. Any text after a `#` **MUST** be ignored until a newline is encountered. Comments **MUST NOT** affect the geometric content of the file.

Comments **MUST** be on their own lines. Comments **MUST NOT** share a line with any other command, even if the comment comes after the command (i.e. you can't do `v 0 0 0 0  #invalid comment`).

#### Example
```
# This is a comment
```


## Model Orientation (Handedness)

An Orientation is **OPTIONAL**. If one is not provided, the orientation is assumed to be X-right, Y-up, Z-forward, W-over/away.

If an Orientation is provided, it **MUST** be listed only once and before any Vertices are listed.

Model Orientation is defined using the `orient` command, followed by a space-separated list of signed axes that correspond to the Right, Up, Forward, and Away directions. Each of the X, Y, Z, and W axes **MUST** be included exactly once.

#### Example
```
# default orientation
orient X Y Z W


# alternate orientation (-Y right, +W up, X forward, -Z away)
orient -Y W X -Z
```

## Vertex Position
Vertex Positions are **REQUIRED**.

Each vertex **MUST** be defined on its own line.

A vertex is denoted using the `v` keyword, followed by 4 floating point numbers, denoting the x, y, z, w coordinates. Coordinate values are separated by spaces.

#### Example
```
v 1.0 1.0 -1.0 -1.0
```

## Vertex Normal
Vertex Normals are **OPTIONAL**.

A Vertex Normal is denoted using the `vn` keyword, followed by 4 floating point numbers, denoting the x, y, z, w components of the normal vector.

Vertex Normals **SHOULD** be normalized.

#### Example
```
vn 0.5 0.5 0.5 0.5

# this normal vector is not normalized, but not invalid
vn 1.0 1.1 1.2 1.3
```

## Vertex Texture Coordinate

Vertex Texture Coordinates are **OPTIONAL**.

A Vertex Texture Coordinate is denoted using the `vt` keyword, followed by 3 floating point numbers, denoting the i, j, k components of the texture coordinate.

Texture Coordinate values **SHOULD** be in the range [0,1]. Values **MAY** be outside the [0,1] range, but it is up to the application to decide how to handle values outside this range.

#### Example
```
vt 0.25 0.75 0.1

# values outside [0,1] are also allowed
vt -0.2 1.6 0.9
```

## Color Data

Color Data is **OPTIONAL**.

Color Data can be assigned to Vertices, Tetrahedra, Paths, and Cells in the same way that custom Data can be assigned to them.

A Color is denoted using the `co` keyword, followed by 3 unsigned integer numbers. These numbers **MUST** be in the [0,255] range. The 3 values represent the red, green, and blue components of the color.

#### Example
```
# RGB color
co 114 255 66
```

## Tetrahedron

Tetrahedra are **OPTIONAL**.

A tetrahedron is denoted using the `t` keyword. A Tetrahedron command is an ordered list of vertex data for each of the four vertices of the tetrahedron. By default, tetrahedra are defined as a list of four Vertex Position indices, separated by spaces. Other data, such as Vertex Texture Coordinate, Vertex Normal, and Vertex Color, may be associated with a tetrahedron's vertices by customizing the Tetrahedron Format (see below).

### Tetrahedron Format Customization

Before listing any Tetrahedron commands, you must specify the line format (with one exception). The line format command tells the parser what data types are listed for each vertex in the tetrahedron and in what order.

The command keyword is `tformat`, followed by a forward slash-separated list of vertex data keywords that will be included for each vertex. For example, to attach Vertex Texture Coordinate data to the tetrahedron, first call the following command: `tformat v/vt`. Then, tetrahedra may be listed like this: `t v0/vt0 v1/vt1 v2/vt2 v3/vt3`. Each vertex is a list of the vertex data indices in the order defined by the most recent `tformat` command, separated by `/`.
The default Tetrahedron Format is one that includes only Vertex Position data. You may list the command `tformat v`, which has the same effect.

### Tetrahedron-level Data
You may also attach data to an entire tetrahedron, as opposed to attaching to the individual vertices. For example, you may want to assign a color to the whole tetrahedron, instead of assigning different colors to each vertex. To do this, call the `tformat` command and list the keyword for the tetrahedron-level data first, followed by a space. For example, to attach color data to a tetrahedron (as well as vertex position and normal data), call `tformat co v/vn`. Then, to list the tetrahedron, call `t co0 v0/vn0 v1/vn1 v2/vn2 v3/vn3`.

### General Tetrahedron Requirements

Every tetrahedron **MUST** include Vertex Position data.

You **MAY** switch tetrahedron formats within a single object, but if two sequential `t` commands are called with differing formats, and the `tformat` command was not called to define the new format, the file is **INVALID**.

The indices provided by each tetrahedron entry **MUST** be non-negative integers. They also **MUST** correspond to vertex entries that have previously been listed in the file.

Vertex commands and Tetrahedron commands **MAY** be mixed. You do not need to list all vertex data strictly before any tetrahedron commands. For example, you may list the vertex positions for a tetrahedron, then list the tetrahedron itself, then list more vertex positions for the next tetrahedron.

You **MAY** define a custom Tetrahedron Format that includes both vertex color data and texture coordinates, but it's up to the application to decide what to do if both are present.

Each vertex in a single tetrahedron **MUST** have the same format. (E.g. you couldn't do `t 1 2/4 5/6/8 6//6` because each of these four vertices uses a different format)

#### Example
```
# Tetrahedron with only Vertex Position
# Note that a tformat command is not required for this simple case
t 0 1 2 3

# Tetrahedron with Vertex Position (indices 0,1,2,3) and Vertex Normal (indices 4,5,6,7)
tformat v/vn
t 0/4 1/5 2/6 3/7

# Tetrahedron with Vertex Position (indices 0,1,2,3), Normal (indices 4,5,6,7), and Texture Coordinates (indices 8,9,10,11)
tformat v/vn/vt
t 0/4/8 1/5/9 2/6/10 3/7/11

# Tetrahedron with Vertex Position (indices 0,1,2,3), Texture Coordinates (indices 4,5,6,7), and Color (indices 8,9,10,11)
tformat v/vt/co
t 0/4/8 1/5/9 2/6/10 3/7/11

# Two tetrahedra in the same object but with different formats
tformat v/vn
t 0/4 1/5 2/6 3/7
tformat v/vn/vt
t 4/8/0 5/9/1 6/10/2 7/11/3

# Two tetrahedra in the same object with their vertex position data listed just before the tetrahedron commands
# (the tformat command is not necessary in this case since only vertex position data is provided)
v 0 0 0 0
v 1 0 0 0
v 0 1 0 0
v 0 0 1 0
t 0 1 2 3
v 2 2 2 2
v 4 2 2 2
v 2 4 2 2
v 2 2 4 2
t 4 5 6 7

# A tetrahedron with a single color (at index 9), rather than a color for each vertex
tformat co v
t 9 0 1 2 3 
```

### Tetrahedron Vertex Winding Order
[TODO: this is just a convention and is not enforced by the format, but deserves an explanation here]

## Polylines

Polylines are **OPTIONAL**.

Polylines are lists of vertex indices. A polyline is denoted using the `p` keyword, followed by an ordered list of 2 or more Vertex Position indices, separated by spaces.

Additional data **MAY** be added to the vertices in the Polyline using Polyline Format Customization.

There is no limit to the length of a polyline, except those naturally imposed by file size limits, maximum values of counter variables, max memory available, etc.

Polylines **MUST** be defined within the scope of an object, or within the global scope.

Polylines can visit vertices and travel along edges that are not part of any tetrahedron.

### Polyline Format Customization

The format of the Polyline command **MAY** be customized using the `pformat` command. This command uses the same rules as the `tformat` command (see above).

### Polyline-level Data

Vertex Data **MAY** be assigned to an entire Polyline, not just the individual vertices in the polyline. Before listing a Polyline with polyline-level data, the Polyline command format must be defined. Define polyline-level data in the pformat command like this: `pformat co v`. Like with Tetrahedron Format Customization, first list the polyline level data keyword, followed by a space, then provide a slash-separated list of vertex data keywords.

#### Examples
```
# A polyline that visits the first 8 vertices
p 0 1 2 3 4 5 6 7

# A polyline with vertex position (indices 2,3,4) and vertex color (indices 6,8,9)
pformat v/co
p 2/6 3/8 4/9

# A polyline with polyline-level color data (index 10), vertex position (indices 1,2,3,4), and vertex texture coordinate (indices 2,4,6,8)
pformat co v/vt
p 10 1/2 2/4 3/6 4/8
```

## Cells

Cells are **OPTIONAL**.

A cell is denoted by the `c` keyword, followed by a list of 1 or more tetrahedron indices, separated by spaces.

Like Paths, there is no artificial limit to the number of tetrahedra assigned to a cell.

The tetrahedra in cells **SHOULD** be contiguous.

The tetrahedron indices **SHOULD NOT** be repeated within a single cell.

### Cell-level Data

Vertex Data **MAY** be assigned to an entire Cell, not just the individual vertices in the cell. Before listing a Cell with cell-level data, the Cell command format must be defined. Define cell-level data in the cformat command like this: `cformat co t`. Like with Tetrahedron Format Customization, first list the cell-level data keyword, followed by a space, then provide a slash-separated list of vertex data keywords.

#### Example
```
# A cell comprised of tetrahedra at indices 0, 2, and 3
c 0 2 3

# A cell with cell-level color data (index 9), and the tetrahedra at indices 0, 5, and 6
cformat co t
C 9 0 5 6
```

## Objects

Objects are **OPTIONAL**.

An object is started with the `o` keyword
Objects **MAY** be named (e.g. `o name_of_object`). To name an object, use the `o` keyword, followed by a space. Anything after the space until the end of the line is considered the object's name. If an object is unnamed, it is up to the application to decide how to differentiate between distinct objects.

When the `o` keyword is used, all subsequent keywords apply to this object until the `o` keyword is used again (i.e. a new object is defined). When the `o` keyword is used again, a new object begins and the vertex indices reset to 0. Objects organize vertices, vertex normals, vertex texture coordinates, vertex colors, lines, tetrahedra, and cells into a separate index space.

All objects within a file **MUST** have unique names (case-insensitive).

Objects **MUST** be contiguous (i.e. you cannot start object1, then define object2, then redefine object1 to add additional data)

#### Examples
```
# Object names can include spaces and numbers: "object 1"
o object 1
v 2 2 3 3
v 3 3 4 4
v 4 4 5 5
v 5 5 6 6
t 0 1 2 3


# A new object definition resets the vertex data and tetrahedron data indices
o object_2
v 7 7 8 8
v 8 8 9 9
v 9 9 10 10
v 10 10 11 11

# vertex 0 here refers to the one at (7,7,8,8), NOT the one at (2,2,3,3)
t 0 1 2 3

# You cannot restart "object 1" after "object_2" has been declared
# o object 1  <= **INVALID**

# You cannot name a new object "Object_2". Object names are case-insensitive
# o Object_2  <= **INVALID**
```

# Material Library
## Material Library File Format

[BIG TODO]

[TODO: similar to how OBJ reads materials, but all the material model will probably be based on the glTF PBR model]

## Using Materials
To import a material library, use the `mtllib` keyword, followed by the name of an external .pbr file.

The material library command **MUST NOT** contain directory names and/or directory symbols.

Material libraries **SHOULD** be in the same directory as the 4DO file, however, it is up to the application to implement features for finding material library files in other directories.

Materials are assigned to tetrahedra. 

A tetrahedron **MUST** have either one or zero materials assigned to it.

To assign a material to an object or part of an object, use the `usemtl` command. Any tetrahedra that are defined after this command will be assigned the material. An object **MAY** have multiple materials, but an individual tetrahedron **MUST** have only one material.

When a new object is defined, the `usemtl` command is deactivated and must be called again for the new object.

```
[TODO: examples]
Object with one material
Object with two materials
New object: no material until usemtl is called again
```

# Invalid 4DO Files

If a file is deemed **INVALID**, it **MAY** be rejected outright. However, a parser **MAY** attempt to fix the contents of the file, but it's under no obligation to do so. Optionally, it may disregard the broken/inconsistent data and attempt to properly load the rest of the data. If this is done, a parser **SHOULD** inform the user of the problem and of which data wasn't loaded.
