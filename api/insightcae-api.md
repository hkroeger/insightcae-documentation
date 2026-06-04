# Structure of the InsightCAE Project

The core of InsightCAE consists of a number of libraries. There is a core library (libtoolkit) which contains the basic functions. Other, optional features are found in a number of other libraries which differ in their build dependencies.

Overview over the component libraries:

-   `toolkit` (depends on VTK, Boost, Armadillo, GSL)

    i.e. depends only on the basic dependencies (including VTK)

    -   `insightcad` (depends on toolkit, OpenCASCADE, DXFlib)

        i.e. depends on the basic features plus CAD kernel

        Provides all the CAD functions but excluding CAD-related GUI classes.

    -   `tookit_remote` (depends on toolkit, Wt)

        Contains classes for remote execution client that require Wt

    -   `tookit_cad` (depends on toolkit, insightcad)

        Contains extensions to basic classes that require CAD (e.g. visualization of block mesh templates) but not GUI-related classes.

    -   `toolkit_gui` (depends on toolkit, toolkit_cad, insightcad, toolkit_remote, Qt)

        Extends all the above with Qt-GUI support.

Since InsightCAE is a toolbox for the creation of custom workflows, it is therefore very natural to use its API in own extensions.

There are different developer perspectives:

1.  Creating extensions (own workflow modules).

    To write own workflow modules, just a plain binary installation of InsightCAE is required. It brings header files for its libraries and a CMake configuration which enables the developer to build his own extensions. It is recommended to create a custom library containing the additions (plug-in). See section [1](#sec:plugins) for a guide.

2.  Modifying and extending the InsightCAE base project itself.

    Since InsightCAE is free and open software, it is perfectly possible to take it and apply extensions or corrections or any other modification or improvement. Please note that silentdynamics GmbH develops InsightCAE primarily with its own applications in mind so there are almost certainly features missing in the API which third party users might need. If you implement something which might be of public interest, please think about sharing it with us and the rest of the community. We are very open to integrate foreign contributions into the project.

    Now, modifying the InsightCAE project requires to build the project from its sources. Please see section [2](#sec:insightcae_dev) for a guide to set up a self-built working copy of the sources.

# Plug-In Development {#sec:plugins}

# InsightCAE Development {#sec:insightcae_dev}

## Dependencies

The InsightCAE toolkit requires a number of dependencies. Not all versions have been tested, of course. The following list contains the major dependencies along with the versions, which are currently utilized in binary packages:

-   boost, <http://www.boost.org>, 1.65.0

-   python, 3.6

-   gsl

-   armadillo

-   OpenCASCADE

-   dxflib

-   Gmsh

-   Qt, 5.

-   Wt

-   OpenFOAM

-   Code_Aster

# Building InsightCAE from the Sources

The InsightCAE sources are prepared to be built in a Posix environment. The developers use Ubuntu Linux in the latest LTS version. The Windows version is cross-compiled using the MXE cross compiling suite.

## Building in Linux

Some prerequisites:

-   GCC 10 should be used. Newer version were found to have buggy optimizers.

-   For Ubuntu, there is a package containing all the important dependencies. Install it by

    ```bash
    $ sudo apt install insightcae-dependencies
    ```

### CMake Configuration and Building

An out-of-source build is recommended. If the aforementioned dependencies package is used, the following variables need to be added to the CMake configuration:

-   INSIGHT_SUPERBUILD=/opt/insightcae

Once configured, the project is built either by

```bash
$ make
```

or

```bash
$ ninja
```

depending on the chosen generator.

### Execution

Once built, the InsightCAE programs can be executed directly from the build folder, provided that the environment is set.

To setup the environment, add the appropriate script to the beginning of the  /.bashrc file:

```bash
source /path/to/build/directory/bin/insight_setenv.sh
```

## Building for Windows

The Windows version so far has only been created using a cross compiler.

### MXE Cross-Compiler

First, an installation of MXE is required. There is Ubuntu installation package in the InsightCAE development repository (see section [3](#obtaining_insightcae)). It contains all the required dependencies plus some additional packages (i.e. dxflib and python 3.6) and a number of patches. It can be installed using:

```bash
$ sudo apt install mxe
```

On other Linux distributions, MXE probably needs to be built from its sources. The following command should do the build of all required packages:

```bash
$ make -j10 MXE_TARGETS=i686-w64-mingw32.shared MXE_PLUGIN_DIRS="plugins/gcc10 plugins/boost_1_66_0" pe-util cgal gettext gcc glew glfw3 mesa harfbuzz armadillo boost dxflib freetype libgcrypt glib gsl hdf5 libiconv libidn libidn2 vtk oce openssl jpeg qt5 cryptopp poppler libntlm openssl wt tiff libgsasl
```

### Setup QtCreator IDE

First, edit the "Kit" settings in QtCreator and create a Kit for MXE. Therefore, add:

1.  GCC compiler "GCC MXE32 shared", select /opt/mxe/usr/bin/i686-w64-mingw32.shared-gcc

2.  C++ compiler "GCC MXE32 shared", select /opt/mxe/usr/bin/i686-w64-mingw32.shared-g++

3.  Qt installation "Qt MXE32 shared", select /opt/mxe/usr/i686-w64-mingw32.shared/qt5/bin/ qmake

4.  In section "CMake", add "CMake MXE shared", select /opt/mxe/usr/bin/i686-w64-mingw32.shared-cmake

5.  Finally add a Kit "Desktop MXE32 shared" and select all the previously added entities

### Configure the Project for MXE

Some variables need to be added to the CMake configuration:

-   PYTHON_INCLUDE_DIRS=/opt/mxe/usr/i686-w64-mingw32.shared/python36/include

-   PYTHON_INCLUDE_DIR=/opt/mxe/usr/i686-w64-mingw32.shared/python36/include

-   PYTHON_LIBRARIES=/opt/mxe/usr/i686-w64-mingw32.shared/python36/python36.dll

-   PYTHON_LIBRARY=/opt/mxe/usr/i686-w64-mingw32.shared/python36/python36.dll

-   VTK_ONSCREEN_DIR=/opt/mxe/usr/i686-w64-mingw32.shared/lib/cmake/vtk-8.2

-   CMAKE_WRAPPER=/opt/mxe/usr/bin/i686-w64-mingw32.shared-cmake

-   INSIGHT_BUILD_MEDREADER:BOOL=OFF

With these variables manually added, it should be possible to build the project with QtCreator.

### Executing the Built Binaries in a Windows Machine

Install a SAMBA server and export the following paths as shares:

-   the build directory (e.g. build-insight-Desktop_MXE32_shared-Release) as share "insightwin"

Make sure to include the option `acl allow execute always = True` for the share. Otherwise there will be an error 0xc0000022 when it is attempted to execute an EXE file from the share.

Prepare a windows machine with the following steps:

-   install python3.6.8rc1.exe (32bit version is required!)

    Make sure to enable the option "add Python executable to PATH"!

-   map the share "insightwin" → drive I:

Set the following environment variables:

-   INSIGHT_GLOBALSHAREDDIRS=i:\\share\\insight

-   append to PATH:

    -   i:\\bin

    -   i:\\lib

Once this is done, it should be possible to execute the EXE files directly from the network drive I:\\bin

To run flow simulations or finite element analyses, the Windows installation needs a linux environment since the backend tools are mostly only available for Linux. Therefore, a WSL container is required with an InsightCAE linux version inside. For development purposes, it is beneficial to have the InsightCAE linux build from the same source available inside the WSL container as was used to create the Windows version. The following steps can be made to achieve this goal:

1.  Export the build folder (when QtCreator is used, it may be named like "build-insight-Desktop_superbuild-Debug") of the Linux build as an additional share by the Samba server (e.g. named "insightlin").

2.  Create an Ubuntu WSL container, e.g. execute in a Power Shell:

    ```bash
    > wsl.exe --install --distribution=Ubuntu-22.04
    ```

3.  Launch a shell in the newly created container:

    ```bash
    > wsl.exe
    ```

    and install the `insightcae-dependencies` package (compare section [3.1](#sec:install_apt)).

    The following commands at the WSL containers bash prompt are required:

    ```bash
    $ sudo apt-key adv --fetch-keys http://downloads.silentdynamics.de/SD_REPOSITORIES_PUBLIC_KEY.gpg
    $ sudo add-apt-repository http://downloads.silentdynamics.de/ubuntu_dev
    $ sudo apt-get update
    $ sudo apt-get install insightcae-dependencies
    ```

    Then mount the share and add the environment setup script to the users' bashrc file:

    ```bash
    $ sudo mkdir -p /opt/build-insightcae
    $ echo '\\<server>\insightlin /opt/build-insightcae drvfs defaults 0 0' | sudo tee /etc/fstab
    $ sudo mount -a
    $ sed -i '1i source /opt/build-insightcae/bin/insight_setenv.sh' ~/.bashrc
    $ sed -i '1i source /opt/insightcae/bin/insight_setthirdpartyenv.sh' ~/.bashrc
    ```

Note: File I/O from within the WSL container is slowed down considerably by the Windows Defender Services. To improve performance, consider to disable this service and/or add exceptions where possible.

# Creating and Distributing Snapshots of a Build

## Linux Build

### Creation

Move into the build directory and execute CPack, e.g.:

```bash
$ cd ~/Projekte/build-insight-Desktop_superbuild-Debug
$ cpack -G TGZ
```

### Installation

The resulting archive has to be copied to the target and unpacked. For example:

```bash
$ tar xzf insightcae_5.0.139-gf41a2d2c~ubuntu24.04_amd64.tar.gz
```

Then change the path to the environment setup script in $HOME/.bashrc:

```bash
source <UNPACK PATH>/insightcae_5.0.139-gf41a2d2c~ubuntu24.04_amd64/usr/local/bin/insight_setenv.sh
```

(Replace \<UNPACK PATH\> with the path, in which the tar xzf command was executed. Also the version number in the archive will be different)

A new shell needs to be opened to apply the changes.

## Windows Build using MXE

### Creation

There is a script for creating the MSI installer which also copied the required dependencies from the MXE installation. It is intended to be executed from the build directory. For example:

```bash
cd ~/Projekte/build-insight-Desktop_MXE32_shared-Release
../insight/generateInsightCAEWindowsMSI.py  -s ../insight -v 5.0.139
```

(The correct version number should be inserted after "-v")

This will yield an installation package "insightcae.msi".

### Installation

The resulting MSI package has to be copied to the target and installed using the Windows tools (double click in the explorer window on the file).

# Creating Custom OpenFOAM Analyses
