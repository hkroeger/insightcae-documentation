# Building from Source

## Getting the Source Code

The source code is hosted on GitHub. Cloning is the recommended approach as it simplifies
receiving updates and bug fixes:

```bash
git clone https://github.com/hkroeger/insightcae.git insight-src
```

Alternatively, download a snapshot archive from GitHub and unpack it in place of the
clone step.

## Dependencies

InsightCAE depends on a number of external projects. Some dependencies are optional and
only required for specific features.

| Project | Tested version | Required for |
|---|---|---|
| Armadillo | 9.800.4 | always |
| GNU Scientific Library | 2.6 | always |
| Boost | 1.65.1 | always |
| VTK | 9 | always |
| Python (Lib) | 3.6 | always |
| OpenCASCADE | 7.4 | always |
| DXFlib | 3.7.5 | always |
| libpoppler | 0.86 | always |
| Qt | 5.15 | GUI components |
| Gmsh | 4.8 | analyses with tet/tri meshing |
| Wt toolkit | 4.1 | client/server execution |
| OpenFOAM | ESI2112, 4.1 extend | OpenFOAM-based analyses |
| Code_Aster | 14 | FEM analyses |
| SWIG | 4 | Python bindings |

On Ubuntu, a convenience package installs all important dependencies:

```bash
sudo apt-key adv --fetch-keys http://downloads.silentdynamics.de/SD_REPOSITORIES_PUBLIC_KEY.gpg
sudo add-apt-repository http://downloads.silentdynamics.de/ubuntu_dev
sudo apt-get update
sudo apt-get install insightcae-dependencies
```

## Building

CMake is used to manage the build. An **out-of-source build** is required.

```bash
mkdir insight-build && cd insight-build
ccmake ../insight-src   # interactive configuration
```

Key CMake options:

| Option | Purpose |
|---|---|
| `INSIGHT_SUPERBUILD` | Path to pre-built dependencies (e.g. `/opt/insightcae`) |
| `INSIGHT_BUILD_TOOLKIT` | Core library |
| `INSIGHT_BUILD_CAD` | CAD module (requires OpenCASCADE 7.2+) |
| `INSIGHT_BUILD_OPENFOAM` | OpenFOAM extensions |
| `INSIGHT_BUILD_WORKBENCH` | Qt5 GUI |
| `INSIGHT_BUILD_PYTHONBINDINGS` | Python bindings via SWIG |
| `INSIGHT_BUILD_TESTS` | Test executables |

After configuration, build with:

```bash
make -j$(nproc)
# or, if the Ninja generator was selected:
ninja
```

## Setting up the Environment

Once built, source the environment script before running any InsightCAE tool. Add the
following line to the _beginning_ of your `~/.bashrc`:

```bash
source /path/to/insight-build/bin/insight_setenv.sh
```

Replace `/path/to/insight-build` with the actual path to your build directory.
Open a new shell to apply the change.

## Running Tests

Tests are only built when `-DINSIGHT_BUILD_TESTS=ON` is set during configuration.

```bash
ctest              # run all tests
ctest -N           # list tests without running
ctest --verbose    # verbose output
ctest -R <pattern> # run tests matching a pattern
```

## Creating Distribution Packages

### Linux (DEB/TGZ)

```bash
cd insight-build
cpack -G DEB   # Debian/Ubuntu package
cpack -G TGZ   # tarball
```

To install the tarball on a target machine:

```bash
tar xzf insightcae_<version>_amd64.tar.gz
```

Then update `~/.bashrc` to source the environment script from the unpacked location.

### Windows (via MXE Cross-Compiler)

The Windows installer is created with a cross-compiler on Linux using MXE. Install the
MXE package from the InsightCAE development repository:

```bash
sudo apt install mxe
```

Then generate the installer from the build directory:

```bash
cd insight-build
../insight-src/generateInsightCAEWindowsMSI.py -s ../insight-src -v <version>
```

This produces `insightcae.msi`, which can be installed on Windows with a double-click.
