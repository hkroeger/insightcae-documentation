# Obtaining InsightCAE

## Installation using Linux Package Manager

We provide binary installation packages for Ubuntu-based Linux distributions. We specifically support the latest Ubuntu LTS distibution.

The packages are held in our repository. To install them, first add the silentdynamics apt repository and the associated key by executing:

```
$ sudo apt-key adv --fetch-keys http://downloads.silentdynamics.de/SD_REPOSITORIES_PUBLIC_KEY.gpg
$ sudo add-apt-repository http://downloads.silentdynamics.de/ubuntu
$ sudo apt-get update
```

Then, you can install the community-edition package by executing

``` 
$ sudo apt-get install insightcae
```

Continue with preparing the environment according to section [Setting up the Environment](#xx)

Note: for customers who receive supported, tailored simulation apps from silentdynamics, these apps are bundled into add-ons and the add-ons are included in a tailored installation package. For each customer, a separate package repository is maintained (with a unique URL, different from the one above). This needs to be included additionally to the one above. These packages are then named `insighcae-<customername>` and the install command changes to:

``` 
$ sudo apt-get install insightcae-<customername>
```

Note: besides the standard repository at <http://downloads.silentdynamics.de/ubuntu> which includes the package built from the _master_ branch in the git source code version management system, silentdynamics provides a second package repository with a development package built form the branch _next-release_. To use this repository, just switch the URL in the instructions above to <http://downloads.silentdynamics.de/ubuntu_dev>.

## Installation on Windows

Our software as well as OpenFOAM and Code_Aster are currently pure Linux software. However, it can also be run under Windows 10 (64 bit only) or later with the help of the _Linux Subsystem for Windows_ (WSL). Virtualization is not used. The processes share the memory with the Windows processes and the files are stored in the Windows file system.

Graphical output from the WSL environment has turned out to be error-prone and poor, thus the software is split into a frontend part, which runs natively in Windows and a backend part which controls the simulation solvers and runs in the WSL environment. The communication between both is over the network stack.

For the comfortable installation of InsightCAE and OpenFOAM under Windows we provide an installer. The installer for the stable version (master branch in the git repository) can be found in this directory:

<http://downloads.silentdynamics.de/ubuntu/>

or for the development version (next-release branch in the repository) the installer is located here:

<http://downloads.silentdynamics.de/ubuntu_dev/>

The installers for the different versions X.Y.Z are named in the form

`InsightCAEInstaller-X.Y.Z.exe`

Note: for customers who receive supported, tailored simulation apps from silentdynamics, the installer is downloaded from the customers dedicated repository URL instead of the one above.

Download the latest EXE file from this link and execute it:

* It will activate the Windows subsystem for Linux, if it is not already activated
* The installer bundles the necessary third party programs (ParaView, gnuplot, MiKTeX, Putty) 

  Note: it is required that the python runtime library (python36.dll) is in the library search path (PATH environment variable). There is an option in the python installer which should be enabled by the user (it is disabled by default).
  
  If this is missed, the PATH variable needs to be manually edited after the installation.
* Please note:
  For the installation, administrator rights are required

# Usage

Appropriate start menu entries are created.

To complete the setup, perform a system reboot and launch the InsightCAE "Workbench_ from the start menu.

Upon the first launch, two things need to be completed:

1. Set paths to commonly required executables.

    The executables are searched in the executable search path defined by the system. Some will not be found since their location is not
    included in the executable search path environment variable PATH. If some of the required executables are not found, the configuration dialog shown below is brought up. Click on a row, where no "Path to executable_ is printed and click on _Select path\..._. In the dialog box, find the appropriate executable and click _ok_. Repeat this for every executable, which was not found.

    ![Configuration dialog to set the paths to commonly used third party executables](set_paths.png){#fig:set_paths}

2. The presence and the version of the WSL backend is checked.

    The installer package includes a WSL image matching the version of the installer. This image is installed, when the installation is
    performed. Thus this check should be silently successful.

    If there is no WSL existing, a message appears and the user should select _Create_ to create one. The form as shown in the figure below should appear.

    ![Create WSL backend form](create_wsl.png){#fig:create_wsl}

    The URL should be properly preset according from which source the installer was downloaded. For support customers, it is important to enter their credentials into the appropriate fields.

    When everything is set, click the button _Ok_. Then the image is downloaded from the server and installed. The image is approximately 1.5GB so the download takes some time and also the unpacking takes several minutes. When the creation is done, the creation dialog is closed and the workbench GUI is shown.

# Updating

To update to a newer version, just download the newer installer from the location described above and execute it.

Only the last two components ("WSL Distribution" and "InsightCAE Windows Client") needs to be checked in the installation wizard. If the third party tools were already installed, they should be skipped.

## Updating Dependencies -- MikTeX

MikTeX is independent from InsightCAE and used by InsightCAE for generating PDF reports and also for rendering charts. Sometimes it might be required to update it. This is done using the following steps:

1. Open the MikTeX console, e.g. by typing the Windows key and then the command _miktex console_ until the icon appears.

  ![Updating MikTeX, finding the MikTeX console](miktex1.png){#fig:update_miktex1}

2. Change to the tab _Updates_. First click on _Search for Updates_. When updates are found, click on _Update Now_ and
    follow the instructions. 
    
    ![Updating MikTeX, executing the update](miktex3.png){#fig:update_miktex2}

# Building from Sources

## Getting the Source Code

The source code of InsightCAE is hosted at Github. Cloning a working copy constitutes the best way to get updates and the latest bug fixes.

Alternatively, you can download snapshot archives from github, if you don't want to bother with a git client. In this case, just replace the cloning step below with unpacking the archive.

First, get the the sources by cloning the git repository:

``` 
$ git clone https://github.com/hkroeger/insightcae.git insight-src
```

## Dependencies

The InsightCAE project depends on a number of other projects. Some of the dependencies are optional and only required, if certain features are enabled. The dependencies are listed in the table below.

| Project | tested version | required for |
|---|---|---|
| Armadillo | 9.800.4 | (always) |
| Gnu Scientific Library | 2.6 | (always) |
| Boost | 1.65.1 | (always) |
| VTK | 9 | (always) |
| Python (Lib) | 3.6 | (always) |
| OpenCASCADE | 7.4 | (always) |
| DXFlib | 3.7.5 | (always) |
| libpoppler | 0.86 | (always) |
| Qt | 5.15 | for GUIs |
| Gmsh | 4.8 | only for analyses with tet/tri meshing |
| Wt toolkit | 4.1 | only for client/server execution |
| OpenFOAM | ESI2112, 4.1 extend | only for OpenFOAM based analyses |
| Code_Aster | 14 | only for FEM analyses |
| SWIG | 4 | only for python bindings |

## Building

CMake is utilized for managing the build. The preferred way is to build the software out of source in a separate build directory. Create a build directory, then configure the build using e.g. ccmake and finally build using make:

``` 
$ mkdir insight && cd insight
$ ccmake ../insight-src
```

In the CMake GUI, you may need to change paths or adapt settings. Please refer to the CMake documentation[^4] on how to use CMake. The biggest difficulty will probably be, to provide all the software packages, on which InsightCAE depends.

The final step in CMake is, to generate the Makefiles. Once this is done without errors, start the compilation of the project by executing:

```
$ make
```

To work with InsightCAE, some environment variables are needed to be set up. A script is provided therefore. It can be parsed e.g. in your _~/.bashrc_ script by adding to the end:

```
source /path/to/insight/bin/insight_setenv.sh
```

# Setting up the Environment

To set up the environment variables and search paths required by the InsightCAE components, a script needs to be parsed. This is conveniently added to the shell startup script file `~/.bashrc`. Add the following line to the _beginning_ of the `~/.bashrc` file:

```
$ source /opt/insightcae/bin/insight_setenv.sh
```

Note that the default contents of this file differs between linux distributions. Sometimes it contains statements which prematurely end
its evaluation for non-interactive shells. This
