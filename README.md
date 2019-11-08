# longhorn_spec

Welcome to the party, pal


## Overview

This is a temporary repository for spec files for Longhorn. It is based on the Life Sciences Computing group's spec file repo here: [https://github.com/TACC/lifesci_spec]. Please see the link for example spec files. Ultimately, we will merge this repo back in to the Life Sciences spec file repo once the compilers and module systems are more mature.

The naming convention for spec files in this repo follows this pattern:

`<application name>-<major version>.<minor version>.<patch>-<spec revision number>.spec`

## Packaging Environment

Building RPMs requires a specific directory structure and system specific include files. Navigate to an appropriate file system on Longhorn and create an `rpmbuild` directory. (At the time of this writing, `/work` is not yet mounted on Longhorn, so `/scratch` is probably the best place):

```
$ cds && mkdir rpmbuild && cd rpmbuild
```

This is where the entire packaging environment will live. Now checkout our rpm spec repository, which contains spec files, include files, and helper scripts.

```
$ git clone https://github.com/TACC/longhorn_spec.git SPECS
Cloning into 'SPECS'...
remote: Counting objects: 1568, done.
remote: Total 1568 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (1568/1568), 5.78 MiB | 3.61 MiB/s, done.
Resolving deltas: 100% (730/730), done.
```

Then, enter the `SPECS` directory, and build your directory tree for building RPMs in `$SCRATCH`.

```
$ cd SPECS

$ ./scripts/buildDirectories.sh
An rpmbuild directory already exists here:
/scratch/12345/user/rpmbuild

Would you like to have the directory tree under this existing directory? (y/n) [n]
y
Creating rpmbuild heirarchy in the /scratch/12345/user/rpmbuild directory.
```

Done! Notice that there is a single `SOURCE` directory that is linked to every system folder in the tree (only Longhorn for now). This enables you to download a source tarball once and use it everywhere.

## Building a module

In this section, we will be building the zlib module which is based on the `example.spec` template. If you are simply looking for a "good" example, please refer to the `zlib-1.2.8-1.spec` file.

### Downloading sources

We will be building zlib version 1.2.8, so lets grab that from their [release page](http://zlib.net).

```
$ cd ../SOURCES
$ wget http://zlib.net/fossils/zlib-1.2.8.tar.gz
$ cd ../SPECS
```

> note: Source files **must** live in the SOURCES directory

### Writing the spec file

For a new module, start by modifying the `example.spec` file. To build the `zlib` RPM, we will use the existing `zlib-1.2.8-1.spec` file.


### Compiling the RPMs

To take advantage of `compiler-defines.inc`, we need to use the `build_rpm.sh` script and also specify which compiler to use. While this will restrict your module to only loading with a specific compiler environment, it also means much less work writing system-specific compiler arguments. Since we will be compiling on Longhorn, we should use the latest version of the xl* compilers (in the system path for now).

```
$ scripts/build_rpm.sh -xl16 zlib-1.2.8-1.spec
```

To help decide which directories you want to include in your modulefile, inspect your built RPM with

```
rpm -qlp ../longhorn/RPMS/ppc64le/tacc-zlib-1.2.8-package-1.2.8-1.ppc64le.rpm
```

and match or delete the

```
prepend_path("PATH",		"%{INSTALL_DIR}/bin")
prepend_path("LD_LIBRARY_PATH",	"%{INSTALL_DIR}/lib")
prepend_path("MANPATH",		"%{INSTALL_DIR}/share/man")

setenv("%{MODULE_VAR}_DIR",     "%{INSTALL_DIR}")
setenv("%{MODULE_VAR}_BIN",	"%{INSTALL_DIR}/bin")
setenv("%{MODULE_VAR}_LIB",	"%{INSTALL_DIR}/lib")
setenv("%{MODULE_VAR}_INC",	"%{INSTALL_DIR}/include")
```

variables as necessary.

If all goes well, you should be left with two RPMS:

- The RPM that contains all LMOD module files
  - tacc-zlib-1.2.8-modulefile-1.2.8-1.ppc64le.rpm
- The RPM that contains all program files
  - tacc-zlib-1.2.8-package-1.2.8-1.ppc64le.rpm

## Testing the RPM

Now that you have functional RPMs, you can test them by installing them locally using the `scripts/myRpmInstall` script. Use it as follows:

```
scripts/myRpmInstall <install directory> <module RPM> <package RPM>
```

We can install our zlib RPMs as follows

```
$ mkdir -p $SCRATCH/public/apps
$ scripts/myRpmInstall $SCRATCH/public/apps ../longhorn/RPMS/ppc64le/tacc-zlib-1.2.8-modulefile-1.2.8-1.ppc64le.rpm ../longhorn/RPMS/ppc64le/tacc-zlib-1.2.8-package-1.2.8-1.ppc64le.rpm
Installing RPMs
WARN: Modulefile RPM was installed while using non-standard RPM database location.
WARN: Package RPM is being installed while using non-standard RPM database location.
WARN: You're off the map! Good luck, friend.
INSTALLED! - ignore the warnings
Checking the $MODULEPATH variable.
looks like the $MODULEPATH environment variable needs updating.
Check the README.md file if you aren't sure how to do that.
```

The warnings look a little scary, but this is a successful installation. Update your module path to include the new location and then load the zlib module.

```
staff.longhorn(102)$ ml use $SCRATCH/public/apps/xl16/modulefiles
staff.longhorn(103)$ ml zlib
staff.longhorn(104)$ ml show zlib
--------------------------------------------------------------------------------------------------------------
   /scratch/12345/user/longhorn/public/apps/xl16/modulefiles/zlib/1.2.8.lua:
--------------------------------------------------------------------------------------------------------------
help([[The zlib module file defines the following environment variables:

 - TACC_ZLIB_DIR
 - TACC_ZLIB_LIB
 - TACC_ZLIB_INC

for the location of the zlib distribution.

For static linking on Linux* OS,

  gcc -O3 -o zpipe_ipp.out zpipe.c -I$TACC_ZLIB_INC $TACC_ZLIB_LIB/libz.a

For static linking on Linux* OS,

  gcc -O3 -o zpipe_ipp.out zpipe.c -I$TACC_ZLIB_INC -L$TACC_ZLIB_LIB -lz

Documentation: http://zlib.net

Version 1.2.8
]], [[
]])
whatis("Name: tacc-zlib-1.2.8-xl16")
whatis("Version: 1.2.8")
whatis("Category: applications, compression")
whatis("Keywords: compressino, deflate")
whatis("Description: A Massively Spiffy Yet Delicately Unobtrusive Compression Library")
whatis("URL: http://zlib.net")
prepend_path("PATH","/work/12345/user/longhorn/public/apps/xl16/zlib/1.2.8/bin")
prepend_path("LD_LIBRARY_PATH","/scratch/12345/user/longhorn/public/apps/xl16/zlib/1.2.8/lib")
prepend_path("MANPATH","/scratch/12345/user/longhorn/public/apps/xl16/zlib/1.2.8/share/man")
setenv("TACC_ZLIB_DIR","/scratch/12345/user/longhorn/public/apps/xl16/zlib/1.2.8")
setenv("TACC_ZLIB_BIN","/scratch/12345/user/longhorn/public/apps/xl16/zlib/1.2.8/bin")
setenv("TACC_ZLIB_LIB","/scratch/12345/user/longhorn/public/apps/xl16/zlib/1.2.8/lib")
setenv("TACC_ZLIB_INC","/scratch/12345/user/longhorn/public/apps/xl16/zlib/1.2.8/include")
```

If everything goes well with your tests, you can use the `moveRpm` helper script to deposit the RPMs in their appropriate location.

```
staff.longhorn(105)$ scripts/moveRpm ../longhorn/RPMS/ppc64le/tacc-zlib-1.2.8-modulefile-1.2.8-1.ppc64le.rpm ../longhorn/RPMS/ppc64le/tacc-zlib-1.2.8-package-1.2.8-1.ppc64le.rpm

Move:
 - ../longhorn/RPMS/ppc64le/tacc-zlib-1.2.8-xl16-modulefile-1.2.8-1.ppc64le.rpm
to
 - /admin/build/admin/rpms/longhorn/RPMS/ppc64le/?
[Y/n] y

Move:
 - ../longhorn/RPMS/ppc64le/tacc-zlib-1.2.8-xl16-package-1.2.8-1.ppc64le.rpm
to
 - /admin/build/admin/rpms/longhorn/RPMS/ppc64le/?
[Y/n] y

Please request the following for installation:

  rpm -ivh --nodeps --relocate /tmpmod=/opt/apps /admin/build/admin/rpms/longhorn/RPMS/ppc64le/tacc-zlib-1.2.8-xl16-modulefile-1.2.8-1.ppc64le.rpm
  rpm -ivh --nodeps --relocate /tmprpm=/home1/apps /admin/build/admin/rpms/longhorn/RPMS/ppc64le/tacc-zlib-1.2.8-xl16-package-1.2.8-1.ppc64le.rpm

```

Done! You can now submit a collab ticket for the RPMs to be installed. After you submit your collab ticket, remember to commit your new spec file back to the repo.

```
git pull
git add your-new.spec
git commit -am "Just completed a new spec file"
git push
```
