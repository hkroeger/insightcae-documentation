# Extensions to Backend Tools

## OpenFOAM

InsightCAE bundles a number of extensions to OpenFOAM which are used in cases created by the toolkit.

### FieldDataProvider {#sec:FieldDataProvider}

The FieldDataProvider class provides a swiss-army-knife-like source for field data. It is used in a number of places, where field data is required as a parameter. Some examples are described in the next section.

The field data is described by a single line of text. The provided data can be steady or unsteady and homogeneous or inhomogeneous.

The syntax of the input line is:

```cpp
<provider type> [additional input for some provider types] <steady|unsteady>   <time instance 1 data>  <time instance 2 data> ...
```

The different provider types together with their required value data are described in the following.

The keywords `unsteady` or `steady` select whether the input is time dependent. The `steady` keyword is the default and can be omitted. For unsteady input, the time instance data is a pair of a scalar time value and the value data. In the steady case, only the value data is required.

These provider types are known:

#### uniform

The value data is single value.

Example:

```cpp
uniform (1 0 0)
```

#### nonuniform

The value data is a list of values. The required number of entries is dependent on the context in which the FieldDataProvider is used. For a boundary condition for example, it needs to match the number of faces.

Example:

```cpp
nonuniform 7(0 0 0 1 1 2 3)
```

#### linearProfile

The value data is the name of a file which contains table of coordinate/value pairs, one per row. Between the lines is linearly interpolated.

This type requires additional input data:

1.  the origin of the coordinate axis,

2.  the direction of the coordinate axis.

Note: For vector and tensor values, the table file needs to contain one column per component in addition to the first column which contains the coordinate. The components are used as they are and are not transformed. For coordinates beyond the bounds of the table, the corresponding values at the bounds of the table are used ("clamp" behaviour).

Example:

```cpp
linearProfile (0 0 0) (0 1 0) "$FOAM_CASE/umean.txt"
```

This defines a linear profile starting at the global origin and the coordinate running along the y axis. The table file content could look like this (for a vector field):

```cpp
1 0 0 0
0.9 1 0 0
-0.9 1 0 0
-1 0 0 0
```

#### radialProfile

#### circumferentialProfile

The input for each instant is a file containing a table of components vs. angle. The coordinate system specification requires the base point, the axis direction and the direction of angle 0. The mapping is done this way: for each face, the angle is computed, the values for the angle are interpolated from the table and applied to the boundary. If the value is a tensor, it is transformed according to the current angle.

Example:

```cpp
circumferentialProfile (0 0 0) (0 0 1)  (1 0 0) steady "$FOAM_CASE/T_vs_phi.txt"
```

In this example, die coordinate system has its origin at (0 0 0)^T, the axis points along the global Z direction and the angle 0 is along the global X direction. The profile points are read from the file `T_vs_phi.txt` in the case directory.

#### fittedProfile

#### vtkField

The data is read from a VTK file and interpolated to the CFD grid. The VTK grid needs to geometricall coincide with the target CFD mesh. There may be many fields in the VTK dataset and the one to be used has to be specified.

Example:

```cpp
vtkField "$FOAM_CASE/data.vtk" "U"
```

### extendedFixedValue

The `extendedFixedValue` boundary condition uses the `FieldDataProvider` class (see section [4.1.1](#sec:FieldDataProvider)) to apply the provided fields as a Dirichlet boundary condition in OpenFOAM cases.

The following settings need to be set in an OpenFOAM cases that uses this BC:

1.  the library containing the BC needs to be added in the `system/controlDict`

    ```cpp
    libs ( "libextendedFixedValueBC.so" );
    ```

2.  use the `extendedFixedValue` boundary condition type in some field file:

    ```cpp
    boundary
    {
       dirichletPatch
       {
    	type extendedFixedValue;
    	source uniform 3;
    	value uniform 3;
       }
    }
    ```

The `source` entry in the boundary section of the field file defines the actual FieldDataProvider input (see [4.1.1](#sec:FieldDataProvider) for the syntax of the input statement).

The `value` statement needs to be present but will be replaced upon solver start with the value(s) evaluated from the FieldDataProvider machinery.
