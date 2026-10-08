# ISCAD Command Reference

This page lists the feature commands, postprocessing actions and documentation statements of the InsightCAD script language. The language itself (symbols, expressions, operators, selections) is described in the [language guide](insightcad.md).

The descriptions of the feature commands are the same as those shown in the "Insert feature" dialog of the ISCAD editor (menu *Model → Insert feature...*, shortcut Ctrl+F).

In the syntax descriptions, `<type:name>` denotes an argument of a certain type (`scalar`, `vector`, `datum`, `feature`, `path`, `string`, or a selection such as `edgeSelection`). Arguments in square brackets `[ ]` are optional, `|` separates alternatives. Paths are enclosed in double quotes (`"file.stp"`), strings in single quotes (`'text'`).

## Feature Commands

### 1D Primitives (Curves and Wires)

#### Line

```cpp
Line( <vector:p0>, <vector:p1> )
```

Creates a line between point p0 and p1.

#### Arc

```cpp
Arc( <vector:p0>, <vector:et0>, <vector:p1> )
```

Creates an arc between point p0 and p1. At point p0, the arc is tangent to vector et0.

#### Arc3P

```cpp
Arc3P( <vector:p0>, <vector:pm>, <vector:p1> )
```

Creates an arc between point p0 and p1 through intermediate point pm.

#### Ellipse

```cpp
Ellipse( <vector:p0>, <vector:axmaj>, <vector:axmin> )
```

Creates an ellipse around point p0. The major axis has length and direction of axmaj. The minor axis length is that of axmin. Its direction is corrected to be orthogonal to axmaj.

#### Polyline, Polygon, ClosedPolyline

```cpp
Polyline( <vector:p0>, <vector:p1>, ..., <vector:pn> )
Polygon( <vector:p0>, <vector:p1>, ..., <vector:pn> )
ClosedPolyline( <vector:p0>, <vector:p1>, ..., <vector:pn> )
```

These commands create a wire from a number of corners, connected by straight edges. `Polyline` creates an open wire, `Polygon` a closed wire. `ClosedPolyline` also creates a closed polyline from the given list of points p0 to pn.

#### SplineCurve

```cpp
SplineCurve( <vector:p0>, ..., <vector:pn> [, der <vector:tan0> <vector:tan1>] )
```

Creates a spline curve through all given points p0 to pn. Optionally, the tangent vectors at the start and end point can be prescribed after the keyword `der`.

#### RapproximatedCurve

```cpp
RapproximatedCurve( <vector:p0>, <vector:p1>, <feature:orgCurve> [, der <vector:tan0> <vector:tan1>] )
```

Creates a spline starting at p0 and ending at p1. Both endpoints are projected onto orgCurve; the curve follows orgCurve between those projections, with smooth transition wings blending from the new endpoints to their respective projection points on the original curve. Optionally, tangent vectors tan0 and tan1 can be supplied with the `der` keyword to enforce departure/arrival directions at the new endpoints.

#### Helix

```cpp
Helix( <vector:p0>, <vector:p1>, <scalar:d>, <scalar:n> )
```

Creates a helical curve on a cylinder of diameter d. The cylinder axis runs from p0 to p1 and the curve makes n turns over this length. The start and end points are provided as `p0` and `p1`, the tangents at the ends as `et0` and `et1`.

#### Spring

```cpp
Spring( <vector:p0>, <vector:p1>, <scalar:d>, <scalar:n> )
```

Like `Helix`, but the result is a wire which additionally contains straight connections from the axis points p0 and p1 to the ends of the helix.

#### Spiral

```cpp
Spiral( <vector:p0>, <vector:axis>, <scalar:n>, <scalar:a> [, <scalar:P>] )
```

Creates an Archimedean spiral around center point p0 in the plane normal to axis. The spiral makes n revolutions, the radius grows with factor a per radian. If a pitch P is given (default 0), the spiral additionally advances along the axis by P per revolution.

#### SineWave

```cpp
SineWave( <scalar:l>, <scalar:A> )
```

Creates a sine curve (single period) with wave length l and amplitude A.

#### CoilPath

```cpp
CoilPath( <scalar:l>, <scalar:dcore>, <scalar:nr>, <scalar:d>, <scalar:R>, <scalar:rmin>, <scalar:nl>, <scalar:dr> )
```

Creates a wire which represents the path of a coil in an electric motor. The straight part of the windings has length l, the core width is dcore, the number of turns nr and the wire distance (approx. equal to wire diameter) is d. The coil is wound on a yoke of radius R. The smallest bending radius is rmin. Multiple radial layers are activated by giving a value for nl. Radial spacing between layers is then dr.

The motor axis is along EZ while the radial direction is EX.

#### Wire

```cpp
Wire( <edgeSelection> )
```

Creates a wire from a number of edges.

#### WireFillet

```cpp
WireFillet( <vertexSelection:vertices>, <scalar:r> )
```

Creates fillets at selected vertices of a wire. All vertices in the selection set vertices are rounded with radius r.

#### ClipWire

```cpp
ClipWire( <feature:wire>, <scalar:Ls>, <scalar:Le> )
```

Modifies an open wire feature by clipping its ends. From the beginning, a segment of length Ls is removed and from the end a segment of length Le.

#### FaceIsoCurve

```cpp
FaceIsoCurve( <faceSelection>, u|v, <scalar:isovalue> )
```

Creates a curve in all selected faces along a constant parameter value.

#### MaxSurfaceCurvature

```cpp
MaxSurfaceCurvature( <faceSelection> )
```

Computes the maximum curvature line on a surface originating from the point of maximum curvature in the selected faces. Returns a compound.

#### SingleVertex

```cpp
SingleVertex( <vector:location> )
```

Creates a topological vertex at the specified location.

### 2D Primitives (Faces)

#### Quad

```cpp
Quad( <vector:p0>, <vector:Lx>, <vector:Ly> [, <scalar:thickness>] [, centered | (center [x][y]) ] )
```

Creates a quad face. The quad is located at point p0 and direction and edge lengths are defined by the vectors Lx, Ly.

Optionally, the edges are centered around p0. Either all directions (option `centered`) or only selected directions (option `center [x][y]` where x,y is associated with Lx, Ly respectively).

#### Tri

```cpp
Tri( <vector:p0>, <vector:L1>, <vector:L2> )
```

Creates a triangle face from point p0 and spanned by vectors L1 and L2.

#### Circle

```cpp
Circle( <vector:p0>, <vector:n>, <scalar:D> [, <scalar:d> ] )
```

Creates a circular face. The circle is centered at p0, the axis vector is n and the diameter D. Optionally, an internal hole with a diameter d is cut out.

#### RegPoly

```cpp
RegPoly( <vector:p0>, <vector:n>, <scalar:ne>, <scalar:a> [, <vector:ez>] )
```

Creates a regular polygon. Center of the polygon is at point p0. Normal vector is n. The number of edges is ne. Radius of the outer circle is a. The direction from p0 to the first edge center is ez.

#### SplineSurface

```cpp
SplineSurface( (<vector:p11>, ..., <vector:p1n>), ..., (<vector:pm1>, ..., <vector:pmn>) )
```

Creates a spline surface through all the given points. Note that all rows need to have the same number of columns.

#### BoundedFlatFace

```cpp
BoundedFlatFace( <feature> [, ..., <feature>] )
BoundedFlatFace( <edgesSelection> [, ..., <edgesSelection>] )
```

Creates a flat face from a number of edges. Edges are taken from one or more features or from one or more edge selection sets.

#### FillingFace

```cpp
FillingFace( <feature:e0>|<edgeSelection:e0>, <feature:e1>|<edgeSelection:e1> [, inverted] )
```

Creates an interpolated surface between two edges. The two edges e0 and e1 can be given either as edge features or edge selection sets.

#### Airfoil

```cpp
Airfoil( <string:name>, <vector:p0>, <vector:L>, <vector:ez>, <scalar:c>, <scalar:t> [, r_EK <scalar:rEK>] [, r_AK <scalar:rAK>] )
```

Creates an airfoil section with specified camber c and thickness t. The camber and thickness distribution are selected by the name argument (e.g. `'naca16'`, `'naca2412'`). Optionally, minimum leading and trailing edge radii can be enforced. The leading edge is positioned at point p0. Length and direction of the chord line are specified by vector L. The normal direction of the foil section, i.e. spanwise direction of the wing, is given by vector ez.

#### Naca4

```cpp
Naca4( <string:code>, <vector:p0>, <vector:L>, <vector:ez> )
```

Creates an airfoil section from the NACA four digit series. The four digit code is passed as a string. The leading edge is positioned at point p0. Length and direction of the chord line are specified by vector L. The normal direction of the foil section, i.e. spanwise direction of the wing, is given by vector ez.

### 3D Primitives (Solids)

#### Box

```cpp
Box( <vector:p0>, <vector:L1>, <vector:L2>, <vector:L3> [, centered | ( center [x][y][z] ) ] )
```

Creates a box volume. The box is located at point p0 and edge directions and lengths are defined by the vectors L1, L2, L3. By default, p0 is a corner and the other corners are found by translating p0 along combinations of L1, L2, L3. Optionally, the edges are centered around p0. Either all directions (option `centered`) or only selected directions (option `center [x][y][z]` where x,y,z is associated with L1, L2, and L3 respectively).

#### Cylinder

```cpp
Cylinder( <vector:p0>, [ax] <vector:p1_or_axis>, <scalar:Da> [, <scalar:Di>] [, centered] )
```

Creates a cylinder based on point p0. The cylinder extends up to point p1_or_axis or, if the keyword `ax` is given, along the length vector p1_or_axis. The outer diameter of the cylinder is Da. If an inner diameter Di is given, a hollow cylinder is created. The cylinder is centered with respect to p0, if the keyword `centered` is supplied.

#### Ring

```cpp
Ring( <vector:p0>, <vector:p1>, <scalar:Da>, <scalar:Di> )
```

Creates a hollow cylinder (ring) between points p0 and p1 with outer diameter Da and inner diameter Di.

#### Shoulder

```cpp
Shoulder( <vector:p0>, <vector:dir>, <scalar:d> [, <scalar:Dmax>] )
```

Creates a semi-infinite annular volume starting at p0 and extending in direction dir. The inner diameter is d, the outer diameter is Dmax (default: 1e6, i.e. practically unbounded).

#### Cone

```cpp
Cone( <vector:p0>, <vector:p1>, <scalar:D0>, <scalar:D1> )
```

Creates a cone between point p0 and p1. At point p0, the diameter is D0 and at p1 it is D1.

#### Sphere

```cpp
Sphere( <vector:p0>, <scalar:D> )
```

Creates a sphere around point p0 with diameter D.

#### Torus

```cpp
Torus( <vector:p0>, <vector:axisTimesD>, <scalar:d> )
```

Creates a torus around center point p0. The axis is specified by the direction of axisTimesD and the torus diameter by its magnitude. The tube diameter is d.

#### Pyramid

```cpp
Pyramid( <feature:base>, <vector:ptip> )
```

Creates a pyramid from the planar base feature and the tip point ptip.

#### Bar

```cpp
Bar( <vector:p0> [ext <scalar:ext0>] [vmiter <scalar:vmiter0>] [hmiter <scalar:hmiter0>],
     <vector:p1> [ext <scalar:ext1>] [vmiter <scalar:vmiter1>] [hmiter <scalar:hmiter1>],
     <feature:xsec>, <vector:up> [, <vector:attractPt>] )

Bar( <feature:spine>, [start [ext ...] [vmiter ...] [hmiter ...],] [end [ext ...] [vmiter ...] [hmiter ...],]
     <feature:xsec>, <vector:up> [, <vector:attractPt>] )
```

This command creates a straight extruded volume from a planar section.

The location of the bar is specified by its start point p0 and end point p1. Alternatively, the end points can be taken from a straight edge feature spine; the end modifications then follow the keywords `start` and `end`. Also needed is a face feature xsec defining the cross section. For orienting the cross section an upward direction up has to be given. If attractPt is given, the cross section is shifted so that the section vertex which is farthest from attractPt lies on the bar axis.

The end points can optionally be shifted along the bar axis by offsets ext0 and ext1. Furthermore, the bar ends can be mitered around the vertical axis (vmiter0, vmiter1) and the horizontal axis (hmiter0, hmiter1).

A bar provides the reference points `p0`, `p1` (shifted end points), `start`, `end`, the vector `ex` (axis direction), the value `L` (length) and the subshapes `xsec` and `spine`.

#### SheetBar

```cpp
SheetBar( <vector:p0> [ext <scalar:ext0>] [vmiter <scalar:vmiter0>] [hmiter <scalar:hmiter0>],
          <vector:p1> [ext <scalar:ext1>] [vmiter <scalar:vmiter1>] [hmiter <scalar:hmiter1>],
          <feature:xsec>, <scalar:thickness>, <vector:up> [, <vector:attractPt>] )
```

Like `Bar`, but the section is expected to consist only of lines and the resulting bar will be made of sheets with some added thickness. The sheets are accessible as subshapes (`shell`) for possible use in e.g. FEM models.

#### SpurGear, BevelGear

```cpp
SpurGear( <scalar:m>, <scalar:z>, <scalar:t> [, <scalar:clearance>] )
BevelGear( <scalar:m>, <scalar:z>, <scalar:t> [, <scalar:clearance>] )
```

Creates a gear with thickness t. The diameter follows from module m and tooth number z. Optionally, the clearance (as a fraction of module) can be specified. If not specified, 0.2 is used.

#### Coil

```cpp
Coil( <vector:p0>, <vector:b>, <vector:l>, <scalar:r>, <scalar:d>, <scalar:nv>, <scalar:nr> )
```

Creates the wire path of a coil wound around a rectangular core. The coil is located at p0, the width and direction of the core are given by vector b and its length and direction by vector l. The bending radius of the windings is r (must not exceed half the core width), the wire diameter is d. nv is the number of layers in vertical direction and nr the number of layers in radial direction.

### Construction from Sections and Paths

#### Extrusion

```cpp
Extrusion( <feature:sec>, <vector:L> [, centered] )
```

Creates an extrusion of the planar feature sec. The direction and length of the extrusion is given by the vector L. If the keyword `centered` is given, the extrusion is centered around the base feature.

#### Revolution

```cpp
Revolution( <feature:xsec>, <vector:p0>, <vector:axis>, <scalar:angle> [, centered] )
```

Creates a revolution of the planar feature xsec. The rotation axis is specified by origin point p0 and the direction vector axis. Revolution angle is specified as angle. By giving the keyword `centered`, the revolution is created symmetrically around the base feature.

#### Pipe

```cpp
Pipe( <feature:spine>, <feature:xsec> [, fixedbinormal <vector>] [, orient] [, reapprox] )
```

Sweeps the planar section xsec along the curve feature spine. The xsec is expected at global origin [0,0,0] and is moved to the beginning of the spine. By fixing the binormal direction using keyword `fixedbinormal` and a fixed direction, problems with erratic twisting of the section on spiral paths can be avoided. By default, the section is not rotated. If keyword `orient` is given, the z-axis of the section is aligned with the tangent of the spine. The keyword `reapprox` triggers a reapproximation of the spine wire into a single b-spline curve (experimental).

#### Sweep

```cpp
Sweep( <feature:xsec0>, ..., <feature:xsecn> )
```

Interpolates a solid through the planar sections xsec0 to xsecn.

#### RotatedHelicalSweep

```cpp
RotatedHelicalSweep( <feature:xsec>, <vector:p0>, <vector:axis>, <scalar:P> [, <scalar:revoffset>] )
```

Sweeps the planar section xsec helically around the axis through p0 with direction axis. P is the pitch (axial advance per revolution). Optionally, an angular offset revoffset can be given (default 0).

#### Sheet

```cpp
Sheet( <feature:base>, <scalar:t> )
```

Creates a solid from a shell feature by adding thickness t.

#### Offset

```cpp
Offset( <feature:base>, <scalar:t> )
```

Creates an offset surface from a shell feature by displacing the surfaces in normal direction by distance t.

#### Stringer

```cpp
Stringer( <feature:spine>, <vector:n>, <scalar:t>, <scalar:w> [, ext0 <scalar:ext0> ] [, ext1 <scalar:ext1> ] )
```

Creates a stringer along a curve or a wire on a surface.

#### Tongue

```cpp
Tongue( <feature:spine>, <vector:direction>, <vector:insidePt>, <scalar:t>, <scalar:w>, <scalar:overlap>, <scalar:delta> )
```

Creates a tongue extension along a section curve between two surface parts.

### Sketches

#### Sketch

```cpp
Sketch( <datum:pl>, <path:file> [, <string:name>] [, <identifier>=<scalar>, ... ] [, <tolerance>] )
```

Reads a sketch (i.e. a singly closed contour) from a file and places it on the datum plane pl. The format is recognized from the file name extension: `.dxf` and `.fcstd` (FreeCAD) are supported. The name is interpreted as layer name in DXF and sketch name in FreeCAD files (default: `'sketch'`). For FreeCAD sketches, parameter values can be supplied; the sketch is then regenerated through FreeCAD with these values. The optional last number is the tolerance for joining the contour (default 1e-3). If the file does not exist relative to the current directory, it is searched in the model search path (see `loadmodel`).

#### ConstrainedSketch

```cpp
ConstrainedSketch( <datum:plane>,
    [layer <name> [<parameters>] ...]
    <entity>, <entity>, ... )
```

Creates a parametric 2D sketch on a datum plane, consisting of geometric entities and constraints which are solved numerically. The sketch is usually created and modified interactively in the sketch editor of ISCAD; the script text is generated by the editor. It can however also be written by hand. Each entity has a unique integer label as its first argument, by which other entities refer to it. Points can be given either by label or as explicit coordinates `[x, y, z]`. All entities accept an optional `layer <name>` argument.

| Entity | Description |
|---|---|
| `SketchPoint(<label>, <x>, <y>)` | point in sketch coordinates |
| `Line(<label>, <point>, <point>)` | line segment between two points |
| `Arc(<label>, <point>, <point>, <point>)` | circular arc |
| `FixedPoint(<label>, <point>)` | fixes a point at its current location |
| `HorizontalConstraint(<label>, <line>)` | line is parallel to sketch x-axis |
| `VerticalConstraint(<label>, <line>)` | line is parallel to sketch y-axis |
| `TangentConstraint(<label>, <curve>, <curve>)` | two curves are tangent |
| `PointOnCurveConstraint(<label>, <point>, <curve>)` | point lies on a curve |
| `DistanceConstraint(<label>, <point>, <point> [, along <vector>])` | distance between points (value stored in sketch) |
| `Distance(<label>, <point>, <point>, <scalar>)` | distance between points, given by a model expression |
| `AngleConstraint(<label>, <point>, <point>\|toHorizontal, <point>)` | angle (value stored in sketch) |
| `Angle(<label>, <point>, <point>, <point>, <scalar>)` | angle, given by a model expression |

The result is a wire/face feature whose edges can be selected as usual. An example (from `test/manual/cad/constrainedsketch.iscad`):

```cpp
L=15;
sk=ConstrainedSketch(
 XY,
 SketchPoint(0, 0, 0),
 SketchPoint(1, 10, 0),
 SketchPoint(2, 10, 10),
 Line(10, 0, 1),
 Line(11, 1, 2),
 Line(12, 0, 2),

 Distance(100, 0, 2, L),
 FixedPoint(101, 0),
 HorizontalConstraint(103, 10),
 Distance(104, 0, 1, 0.3*L),
 Angle(105, 2, 0, 1, 90*deg)
);

ex:Extrusion(BoundedFlatFace(sk?alledges), 5*EZ);
```

### Import and Submodels

#### import

```cpp
import( <path> )
```

Imports a feature from a file. The format is recognized from the filename extension. Supported formats are IGS, STP, BREP.

#### STL

```cpp
STL( <path:filename> [, <feature:other transform feature> ] )
```

Imports a triangulated surface for display. The result can only be used for display, no operations can be performed on it. Transformations can be reused from other transform features: the name of another transformed feature can be provided optionally.

#### FreeCADModel

```cpp
FreeCADModel( <path:filename>, <string:solidname> [, <identifier>[.<identifier>] = <scalar:value>, ... ] )
```

Rebuilds a model in FreeCAD and imports the result. An arbitrary number of parameter/value pairs can be passed into FreeCAD.

#### SailCut3D

```cpp
SailCut3D( <path> )
```

Imports a SailCut `*.panel3d` XML file and returns a compound containing one face per sail panel. Each panel face is constructed from the four boundary curves (left/luff, right/leech, top/head, bottom/foot) stored in the file.

#### loadmodel

```cpp
loadmodel( <identifier:modelname> | <path:file> [, <identifier> = <value>, ... ] )
```

Imports a submodel. Either a model name is given, which is read from the file `<modelname>.iscad`, or an explicit file path. The model file is searched in the following locations:

1.  the directories listed in the environment variable `ISCAD_MODEL_PATH` (separated by ":")
2.  the subdirectory `iscad-library` in the shared file directories of InsightCAE
3.  the current directory

Only the features which were marked as components in the submodel (defined with ":" instead of "=") are included in the resulting compound. An arbitrary number of parameters are passed from the current model into the submodel and override symbols defined there with `?=`. The type of each value is deduced from the expression:

| Value syntax | Type |
|---|---|
| `<vector>` | point |
| `!<vector>` | direction |
| `<scalar>` | scalar |
| `<feature>` | feature |
| `%<datum>` | datum |

All symbols of the submodel can be accessed from the resulting feature, e.g. `sub$L`, `sub@p0`, `sub%plane`, `sub.part`.

#### for

```cpp
for( <identifier:var>, <scalar:start>, <scalar:end>, [inc <scalar:increment>,] <submodel> )
```

Creates multiple instances of a submodel in a loop. The loop variable var runs from start while it is smaller than end, incremented by increment (default 1). The submodel is either an in-situ submodel `{ ... }` or a model name/path with optional parameters like in `loadmodel`. In each instance, the loop variable is set to its current value. Inside the loop body, the feature symbol `PREV` is replaced by the previous instance, which allows chained constructions. The instances are accessible as `instance0`, `instance1`, ..., the last one as `instanceFinal`.

```cpp
row = for(i, 0, 5, { c: Cylinder(i*20*EX, ax 10*EZ, 5); });
```

#### asModel

```cpp
asModel( <verticesSelection>|<edgesSelection>|<facesSelection>|<solidSelection> )
```

Creates a new feature from selected entities of an existing feature. The short form is `*<selection>`.

### Boolean Operations and Modification

Boolean operations are usually written using the operators `|`, `-` and `&` (see the [language guide](insightcad.md#features)).

#### MergeSolids

```cpp
MergeSolids( <feature> )
```

Creates a boolean union of all (possibly intersecting) volumes of the given feature.

#### Compound

```cpp
Compound( <feature:c0> [, <feature:c1>, ..., <feature:cn> ] )
```

Creates a compound (assembly) of multiple features c0 to cn.

#### Fillet

```cpp
Fillet( <edgesSelection:edges>, <scalar:r> )
```

Creates fillets at selected edges of a solid. All edges in the selection set edges are rounded with radius r.

#### Chamfer

```cpp
Chamfer( <edgeSelection:edges>, <scalar:l> )
```

Creates chamfers at selected edges of a solid. All edges in the selection set edges are chamfered with width l.

#### Cutaway

```cpp
Cutaway( <feature:base>, ( <vector:p0>, <vector:n> ) | ( <datum:plane> [, inverted] ) )
```

Removes a halfspace from a feature. The halfspace is either specified by a point p0 and the normal vector n or by a datum plane. In the latter case, the removal side of plane can be toggled by giving the option `inverted`.

#### CutUp

```cpp
CutUp( <feature:base>, <vector:n>, <scalar:t>, <vector:p0> [, ..., <vector:pn>] )
```

Cuts up the base feature into several pieces with planar cuts at p0 to pn with normal n and cut thickness t. The result pieces are stored in subshapes of name `cut_<i>`.

#### Split

```cpp
Split( <feature:source>, <feature:tool> )
```

Splits the source feature by the tool feature. The result contains all pieces.

#### Partition

```cpp
Partition( <feature:base>, <feature:tool> )
```

Partitions the base feature by the tool feature.

#### SplitSurface

```cpp
SplitSurface( <feature:base>, <feature:tool1> : <identifier:name1> [, <feature:tool2> : <identifier:name2>, ...] )
```

Splits the outer surface of the (single) solid base by the given tool features and rebuilds the solid. For each tool, the faces of the result which are part of the tool solid are provided as a named face selection and as a subshape, e.g. `res?faces inlet` or `res.inlet`. This is useful for defining boundary patches for meshing.

#### GlueFaces

```cpp
GlueFaces( <feature> [, <scalar:tol>] )
```

Glues coincident faces of the solids in the feature, so that adjacent solids share common faces (tolerance tol, default 1e-3).

#### StitchedShell, StitchedCompound

```cpp
StitchedShell( <faceSelection> [ <scalar:tol> ] )
StitchedCompound( <faceSelection> [ <scalar:tol> ] )
```

Create a stitched shell from selected faces. The default tolerance is 0.001.

#### StitchedSolid

```cpp
StitchedSolid( <feature:f1> [, <feature:f2> [, ...] ] [, <scalar:tol>] )
```

Creates a stitched solid from all faces of the provided features. The default tolerance is 0.001.

#### FixShape

```cpp
FixShape( <feature:in> )
```

Runs some repair operations on the output shape of feature in.

#### Projected

```cpp
Projected( <feature:base>, <feature:target>, <vector:dir> )
```

Projects the feature base onto the feature target. The direction has to be prescribed by vector dir.

#### ProjectedOutline

```cpp
ProjectedOutline( <feature:base>, <datum:plane> )
```

Projects the outline (silhouette) of the base feature onto the datum plane. If the base feature consists only of edges, the edges are projected.

### Transformation and Patterns

#### Transform

```cpp
Transform( <feature:base>, <vector:trans>, <vector:rot> [, <scalar:scale>] )
```

Transforms the base feature by translating it by vector trans and rotating it around vector rot (magnitude gives angle, axis goes through global origin). Optionally, the base feature is scaled by the factor scale.

#### Rotate

```cpp
Rotate( <feature:base>, <vector:rot>, <vector:origin> )
```

Rotates the base feature around vector rot (magnitude gives angle), the axis goes through point origin.

#### Place

```cpp
Place( <feature:base>, ( <vector:p0>, <vector:ex>, <vector:ez> [, <vector:refpt> ] ) | <feature:other_place> )
```

Places the feature base in a new coordinate system. The new origin is at point p0, the new x-axis along vector ex and the new z-direction is ez. Optionally, the point refpt is made coincident with p0 instead of the origin. Alternatively, the placement is copied from another Place-feature other_place.

#### RefPlace

```cpp
RefPlace( <feature:base>, [ <placement condition>, ...] )
```

Places the feature base by solving a set of placement conditions numerically (experimental). Available placement conditions are:

- `<vector> == <vector>` (same point coordinates or vector components)
- `<vector> parallel <vector>` (same vector orientation)
- `<datum> aligned <datum> [inverted]` (direction properties of two datums are parallel or antiparallel. The latter holds, if keyword `inverted` is present)
- `<datum> inclined <datum> <scalar>`
- `<datum> coaxial <datum> [inverted]`
- `<vector> inplane <datum>`
- `<vector> onaxis <datum>`

#### AlignWithBoundingBox

```cpp
AlignWithBoundingBox( <feature:base>,
    <feature:other>, <vector:direction>, min|center|max, min|center|max
    [, <feature:other2>, <vector:direction2>, min|center|max, min|center|max, ... ] )
```

Translates the base feature along direction such that the extremities of its bounding box line up with those of other, measured along direction. The first location keyword refers to the bounding box of other, the second to that of base. Several alignments can be given; they are applied one after another.

```cpp
# put b on top of a
b2 = AlignWithBoundingBox(b, a, EZ, max, min);
```

#### InhomScale

```cpp
InhomScale( <feature:base>, <vector:scalefactor> )
```

Scales the base feature by independent factors for each coordinate direction (components of scalefactor).

#### Mirror, FlipX, FlipY, FlipXY

```cpp
Mirror( <feature:base>, <datum:plane> )
FlipX( <feature> )
FlipY( <feature> )
FlipXY( <feature> )
```

`Mirror` mirrors the base feature over the given datum plane. `FlipX` mirrors over the YZ plane, `FlipY` over the XZ plane and `FlipXY` over the diagonal plane with normal n=[1,1,0].

#### LinearPattern

```cpp
LinearPattern( <feature:base>, <vector:delta_l>, <scalar:n> )
```

Copies the base feature into a linear pattern. The copies of the base feature are shifted in increments of delta_l. The number of copies is n.

#### CircularPattern

```cpp
CircularPattern( <feature:base>, ( <vector:p0>, <vector:deltaphi>, <scalar:n> [, centered] [, not '<i>,<j>,...'] ) | <feature:other_pattern> )
```

Copies the base feature into a circular pattern. The copies are rotated around the axis which goes through the point p0 and has the direction of deltaphi. The angular step between successive copies is given by the length of deltaphi and the number of copies is n. If the keyword `centered` is given, the pattern is created symmetrically in both directions of rotation (the total number of elements is kept). With `not` followed by a comma separated list of (one-based) instance numbers, single instances can be omitted. Alternatively, the settings can be copied from an existing CircularPattern-feature other_pattern.

#### CurvePattern

```cpp
CurvePattern( <feature:base>, <feature:curve>, <scalar:delta>, <scalar:n> )
```

Copies the base feature into a linear pattern along a curve feature (curve). The distance between subsequent copies is delta and n copies are created.

#### Exploded

```cpp
Exploded( <datum:refaxis>, <feature:c0> [, ..., <feature:cn> ] )
```

Creates an exploded state from the supplied list of features. The components are translated along the direction of refaxis.

## Postprocessing Actions

Postprocessing actions can be placed anywhere in the script after the symbols they use.

### File Export

#### saveAs

```cpp
saveAs( <path:file> ) << <feature> [ <identifier:name> = <faceSelection> ... ];
```

Exports the feature geometry to a file. The format is determined by the extension:

| Extension | Format |
|---|---|
| `.brep` | OpenCASCADE BREP |
| `.stp`, `.step` | STEP |
| `.igs`, `.iges` | IGES |
| `.stl` / `.stlb` | STL (ASCII / binary) |
| `.emesh` | OpenFOAM edge mesh of all edges |
| `.dot` | Graphviz graph of the feature dependencies |

The syntax accepts a list of named face groups after the feature, but they are currently not written to the file.

#### exportSTL

```cpp
exportSTL( <path:file> [, <scalar:accuracy>] [, ascii] ) << <feature>;
```

Exports a triangulation of the feature to an STL file. Optionally, the triangulation accuracy can be specified. By default, a binary STL file is written; the keyword `ascii` selects ASCII format.

#### exportEMesh

```cpp
exportEMesh( <path:file>, <scalar:accuracy>, <scalar:maxlen> ) << <edgeSelection>;
```

Exports the selected edges as a discretized edge mesh (OpenFOAM eMesh format, e.g. for snappyHexMesh feature edge snapping). Accuracy and maximum segment length control the discretization.

### Drawing Export

#### DXF

```cpp
DXF( <path:outputfile> ) << <feature:f1> <view_definition> [ <view_definition> ... ]
                         [, <feature:f2> <view_definition> ... ];
```

Creates a DXF file for further use in drawings. Several views can be derived from one or more features. A view definition takes the following form:

```cpp
<identifier:viewname> (
 <vector:p_0>, <vector:n> [, up <vector:e_up>]
  [, section]
  [, poly]
  [, skiphl]
  [, add [l] [r] [t] [b] [k] ]
  )
```

It defines a view on the point vector p_0 with normal direction vector n of the view plane. The upward direction (Y-direction) is aligned with vector e_up.

The keyword `section` toggles whether only the outline is projected or if the view plane creates a section through the geometry. If keyword `poly` is given, the DXF geometry will be discretized. This is more robust but creates much larger DXF files. Keyword `skiphl` toggles whether hidden lines are output. The keyword `add` followed by the key letters l, r, t, b and/or k enables creation of additional projections from the left, right, top, bottom and/or back, respectively.

```cpp
c: Cylinder(O, 100*EX, 20);

DXF("c.dxf") << c
     top ( O, EX, up EY )
     front ( O, EZ, up EY )
;
```

A view definition can also be created interactively with *Insert drawing...* (Alt+D) in the ISCAD editor.

### Mesh Generation

#### gmsh

```cpp
gmsh( <path:outputfile> ) << <feature>
  L = ( <scalar:Lmax> <scalar:Lmin> )
  [linear]
  [vertexGroups( <name> = <vertexSelection> [@ <scalar:size>] ... )]
  [edgeGroups( <name> = <edgeSelection> [@ <scalar:size>] ... )]
  [faceGroups( <name> = <faceSelection> [@ <scalar:size>] ... )]
  [volumeGroups( <name> = <solidSelection> [@ <scalar:size>] ... )]
  [vertices( <name> = <vector:location> ... )]
  [meshSizes( <vector:center>, <scalar:radius>, <scalar:size> ... )]
  [screwHeads( <name> = <feature> [sub <identifier>] [@ <scalar:size>] ... )]
  [screwBases( <name> = <feature> [sub <identifier>] [@ <scalar:size>] ... )]
  [screws( <name> = <feature> [sub <identifier>] ... )]
  [keepTmpDir]
;
```

Generates a (triangular or tetrahedral) mesh of the feature using Gmsh, e.g. for an FEA analysis. The mesh format is determined by the output file extension (e.g. `.med`). Group names may be identifiers or strings.

- `L`: maximum and minimum element size.
- `linear`: switches from quadratic to linear elements.
- `vertexGroups`, `edgeGroups`, `faceGroups`, `volumeGroups`: named groups (see the [language guide](insightcad.md#lower-dimensional-shape-selection) for selection sets). An optional local mesh size is appended after `@`.
- `vertices`: creates additional named nodes at the given locations.
- `meshSizes`: local refinement inside spheres, given by center, radius and element size.
- `screwHeads`, `screwBases`, `screws`: define screw connections for structural analysis. The geometry is taken from the given feature (or from its subshape after `sub`).
- `keepTmpDir`: keeps the temporary Gmsh working directory for inspection.

```cpp
c:
Cylinder(O, 100*EX, 25)
 |
Cylinder(100*EX, ax 100*EX, 50)
;

gmsh("c.med") << c
     L = (10 0.1)
     linear
     faceGroups(
      lo_f = c?faces('isPlane&&minimal(CoG.x)')
      up_f = c?faces('isPlane&&maximal(CoG.x)')
     )
;
```

#### gmshExtrusion

```cpp
gmshExtrusion( <path:outputfile> ) << <feature>
  L = ( <scalar:Lmax> <scalar:Lmin> ) h = <scalar:h> nLayers = <scalar:n>
  [linear]
  [vertexGroups( ... )]
  [edgeGroups( ... )]
  [baseFaceGroups( ... )]
  [topFaceGroups( ... )]
  [volumeGroups( ... )]
  [vertices( ... )]
  [keepTmpDir]
;
```

Creates a 2D mesh of a planar (sheet) feature using Gmsh and extrudes it in normal direction by the height h with nLayers element layers. Faces on the base and top side of the extrusion are named with `baseFaceGroups` and `topFaceGroups`; edge groups name the lateral faces generated from the edges. The other options are as for `gmsh`.

#### frame

```cpp
frame( <path:outputfile> ) << L = <scalar:L> <feature:bar1> [, <feature:bar2>, ...];
```

Creates a beam (frame) mesh in MED format from the given bar features (e.g. created by `Bar`). Each feature has to provide the reference points `p0` and `p1`, between which a beam is created. Crossing beams are split at their intersections. L is the element length. Requires InsightCAE to be compiled with MED support.

### Measurements and Reports

The results of the following actions are shown in the ISCAD editor and written to the console.

#### Distance

```cpp
Distance( <identifier:name>, <vector:p1>, <vector:p2> );
```

Measures the distance between the points p1 and p2 and displays a dimension.

#### Angle

```cpp
Angle( <identifier:name>, <vector:p1>, <vector:p2>, <vector:pCtr> );
```

Measures the angle between the points p1 and p2 as seen from the center point pCtr.

#### Diameter

```cpp
Diameter( <identifier:name>, <edgeSelection> );
Diameter( <identifier:name>, center <vector:pc>, <vector:n>, <vector:p> );
Diameter( <identifier:name>, points <vector:p1>, <vector:p2>, <vector:p3> );
```

Measures the diameter of a circle, defined either by a circular edge, by a center point, the normal vector and a point on the curve, or by three points on the curve.

#### SolidProperties

```cpp
SolidProperties( <identifier:name> ) << <feature>;
```

Computes the properties of the feature: mass (using the density assigned to the feature), surface area, center of gravity, inertia tensor with respect to global axes and origin, and bounding box.

#### Hydrostatics

```cpp
Hydrostatics( <identifier:name>, <vector:psurf>, <vector:nsurf>, <vector:elong>, <vector:evert> ) << ( <feature:hull>, <feature:ship> );
```

Computes the hydrostatic properties of a floating body. The water surface is defined by point psurf and normal nsurf, elong is the longitudinal and evert the vertical direction of the ship. The hull volume feature is cut at the water surface to obtain the displaced volume; the mass and center of gravity are taken from the ship feature (assembly). Results are displaced volume, center of buoyancy, center of gravity, metacentre and the principal moments of inertia of the waterplane area.

#### BOM

```cpp
BOM << <feature>;
```

Creates a bill of materials of the feature (assembly) and writes it to the console. The descriptions of the parts are set using `@description` in the part models or by the `BOMDescription` property (see [Feature properties](insightcad.md#feature-properties)).

#### CuttingStock

```cpp
CuttingStock( <scalar:cutWidth>, <scalar:length1> [, <scalar:length2>, ...] ) << <feature1> [, <feature2>, ...];
```

Solves a cutting stock problem: all linear edges of the given features (e.g. bars of a frame) are assigned to stock bars of the given lengths so that scrap is minimized, taking a saw cut width cutWidth into account. The required number of bars of each stock length is computed. Two tables are written: one row per input edge with its assigned stock item, and one block per used stock item listing the pieces cut from it and the remaining length.

## Documentation Statements

After the keyword `@doc` at the end of a script, statements which add dimensions to the model documentation can be given (postprocessing actions are also allowed there):

```cpp
@doc
diameter( '<label>', <identifier>, <vector:p1>, <vector:p2>, inside|outside );
length( '<label>', <identifier>, <vector:p1>, <vector:p2> );
```
