# Proj4 with JNI
The most recent version of [OSGeo/proj.4](https://github.com/OSGeo/proj.4) may work better, but last time I tried to build it I couldn't get JNI to work properly with my Docker files.

## Pull the Docker image
Pull the docker image with the [tag](https://hub.docker.com/r/echoparklabs/proj.4/tags/) appropriate to your Java environment. In this case I'm pulling the JDK Alpine image
```bash
docker pull echoparklabs/proj.4:8-jdk-alpine
```

## Building for Mac
After cloning the repo locally, you'll need to install some build tools. These can be installed with homebrew (I can't remember if `mozjpeg` is necessary, someone please try building without it and let me know):
```bash
brew reinstall libtool
brew reinstall mozjpeg
brew reinstall autoconf
brew reinstall automake
brew install ant
```

Change directories into the proj.4 directory and execute the following to build proj.4 and copy the proj.4 jar into `/usr/local/lib`:
```bash
export PROJ_DIR=/usr/local/Cellar/proj/5.2.0
export JAVA_HOME="$(/usr/libexec/java_home)"
autoreconf -i
CFLAGS=-I$JAVA_HOME/include/darwin ./configure --with-jni=$JAVA_HOME/include --prefix=$PROJ_DIR
make -j 8
make install
cd jniwrap
ant -v
cp ./out/proj.jar $PROJ_DIR/lib/proj.jar
brew link proj
```

## Building your own Docker Image
The Docker images are based off of the [openjdk](https://hub.docker.com/_/openjdk/) images. You can build a jdk image or a jre image, you can use Java 8 or 10 (maybe 11, haven't tested), and you can use debian or alpine.

### Building Debian
To build the latest debian 8 jdk image:
```bash
docker build -t us.gcr.io/echoparklabs/proj.4:8-jdk-slim .
```
The latest debian 8 jre image
```bash
docker build --build-arg JRE_TAG=8-jre-slim -t echoparklabs/proj.4:8-jre-slim .
```
To build the latest debian 10 jdk:
```bash
docker build --build-arg JDK_TAG=10-jdk-slim -t echoparklabs/proj.4:10-jdk-slim .
```
To build the latest debian 10 jre:
```bash
docker build --build-arg JDK_TAG=10-jdk-slim --build-arg JRE_TAG=10-jre-slim \
       -t echoparklabs/proj.4:10-jdk-slim .
```


### Building Alpine
At this time, the resulting Alpine docker image is about 50% smaller than the slim debian images. The default Alpine image uses the `8-jdk-apline` image

To build the latest Alpine JDK 8 image:
```bash
docker build -t echoparklabs/proj.4:8-jdk-alpine -f Dockerfile.alpine .
```

To build the latest Alpine JRE image use the jre tag with a `--build-arg` (it will default to the latest JDK 8 alpine image):
```bash
docker build --build-arg JRE_TAG=8-jre-alpine \
       -t echoparklabs/proj.4:8-jre-alpine -f Dockerfile.alpine .
```


### Building with specific Debian JDK or JRE docker images:

To build a specific Alpine JDK 8 image use the `--build-arg`. For example if you wanted to build off of the `8u171-jdk-alpine3.8` openjdk image:
```bash
docker build --build-arg JDK_TAG=8u171-jdk-alpine3.8 \
       -t echoparklabs/proj.4:8u171-jdk-alpine3.8 -f Dockerfile.alpine .
```

And to build a specific jre image use the following `--build-args`. For example if you wanted to the `8u171-jre-alpine3.8`  you would need to also specifiy `8u171-jdk-alpine3.8` JDK:
```bash
docker build --build-arg JRE_TAG=8u171-jre-alpine3.8 \
       --build-arg JDK_TAG=8u171-jdk-alpine3.8 \
       -t echoparklabs/proj.4:8u171-jre-alpine3.8 -f Dockerfile.alpine .
```


# PROJ

[![Travis Status](https://travis-ci.com/OSGeo/proj.4.svg?branch=master)](https://travis-ci.com/OSGeo/proj.4)
[![AppVeyor Status](https://ci.appveyor.com/api/projects/status/584j49uguwoo5evi?svg=true)](https://ci.appveyor.com/project/OSGeo/proj-4)
[![Coveralls Status](https://coveralls.io/repos/github/OSGeo/proj.4/badge.svg?branch=master)](https://coveralls.io/github/OSGeo/proj.4?branch=master)
[![Gitter](https://badges.gitter.im/OSGeo/proj.4.svg)](https://gitter.im/OSGeo/proj.4)
[![Mailing List](https://img.shields.io/badge/PROJ-mailing%20list-4eb899.svg)](http://lists.maptools.org/mailman/listinfo/proj)

PROJ is a generic coordinate transformation software, that transforms
coordinates from one coordinate reference system (CRS) to another.
This includes cartographic projections as well as geodetic transformations.

For more information on the PROJ.4 project please see the web page at:

https://proj4.org/

The PROJ.4 mailing list can be found at:

http://lists.maptools.org/mailman/listinfo/proj

See the NEWS file for changes between versions.

The following command line utilities are included in the PROJ package:

- proj, for cartographic projection of geodetic coordinates
- cs2cs, for transformation from one CRS to another CRS.
- geod, for geodesic (great circle) computations.
- cct, for generic Coordinate Conversions and Transformations.
- gie, the Geospatial Integrity Investigation Environment.


## Installation
### Building with CMake

    cd proj
    mkdir build
    cd build
    cmake ..
    cmake --build .

On Windows, one may need to specify generator:

    cmake -G "Visual Studio 15 2017" ..

Tests are run with

    ctest

The test suite requires that the proj-datumgrid package is installed
in `PROJ_LIB`.

### Building on Unix/Linux

FSF's configuration procedure is used to ease installation of the
PROJ system.

The default destination path prefix for installed files is `/usr/local`.
Results from the installation script will be placed into subdirectories
`bin`, `include`, `lib`, `man/man1` and `man/man3`.
If this default path prefix is proper, then execute:

    ./configure

See the note below if you are building PROJ directly from the git repository.

If another path prefix is required, then execute:

    ./configure --prefix=/my/path

In either case, the directory of the prefix path must exist and be
writable by the installer.
After executing configure, execute:

    make
    make install

The install target will create, if necessary, all required sub-directories.

Tests are run with

    make check

The test suite requires that the proj-datumgrid package is installed
in `PROJ_LIB`.

#### Building from the git repository

If you are building from the git repository you have to first run

    ./autogen.sh

which will generate a configure script that can be used as described above.

### Building on Windows with NMAKE

PROJ can be built with Microsoft Visual C/C++ using the `makefile.vc`
in the `PROJ` directory.  First edit the `PROJ\nmake.opt` and
modify the `INSTDIR` value at the top to point to the directory
where the PROJ tree shall be installed.
If you want to install into `C:\PROJ`, it can remain unchanged.
Then use the `makefile.vc` to build the software e.g.:

    C:\> cd proj
    C:\PROJ> nmake /f makefile.vc
    C:\PROJ> nmake /f makefile.vc install-all

Note that you have to have the VC++ environment variables, and path
setup properly.  This may involve running the `VCVARS32.BAT`
script out of the Visual C++ tree.

The `makefile.vc` builds `proj.exe`, `proj.dll` and `proj.lib`.

On Windows you have to set the `PROJ_LIB` environment variable to make
sure that PROJ can find the resource files that it needs. For the
default install directory you can set `PROJ_LIB` with:

    C:\> set PROJ_LIB=C:\PROJ\share


### Distribution files and format

Sources are distributed in one or more files.  The principle elements
of the system are in a compress tar file named `proj-x.y.z.tar.gz` where
"x" will indicate major release number, "y" indicates minor release
number and "z" indicates the patch number of the release.

In addition to the PROJ software package, distributions of datum
conversion grid files and PROJ parameter files are also available.
The grid package is distributed under the name `proj-datumgrid-x.y.zip`,
where "x" is the major release version and "y" the minor release
version numbers. Similarly regional packages are distributed. The
regional packages contain resources that are not essential to the
functionality of PROJ but still of value to users in the region
specific to the package. All grids that were in proj-datumgrids-1.6
remain in proj-datumgrids-1.7; the regional datumgrid files contain
grids for datums not previously supported (prior to PROJ 5.0.0).

The resource packages can be downloaded from the PROJ.4 website.

More info on the contents of the various resource packages can be
found at the
[proj-datumgrid GitHub repository](https://github.com/OSGeo/proj-datumgrid).

The resource file packages should be extracted to `PROJ_LIB`
where PROJ will find them after installation. The default location of
`PROJ_LIB` on UNIX-based systems is `/usr/local/share/proj` but it may
be changed to a different directory. On Windows you have to define
`PROJ_LIB` yourself.

### Citing PROJ in publications

See [CITATION](CITATION)