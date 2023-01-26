---
title: Module System
summary: An overview of the software module system and how to use it.
---

# Module System

## Intro

High performance compute clusters usually have a variety of software with sometimes conflicting
dependencies.
Software packages may need to make modifications to the [user environment](shells-ref), or the same
software may be compiled multiple times to run efficiently on differing hardware within the cluster.
To support these use cases, software is managed with a module system that prepares the user
environment to access specific software on load and returns the environment to its former state when
unloaded.
A _module_ is the bit of code that enacts and tracks these changes to the user environment, and the
module _system_ is software that runs these modules and the collection of modules it is aware of.
Most often, a module is associated with a specific software package at a specific version, but they
can also be used to make more general changes to a user environment; for example, a module could
load a set of configurations for the [BASH](shells-ref) shell that set color themes.

The two most commonly deployed module systems are [environment
modules](https://modules.readthedocs.io/en/latest/) (or `envmod`) and
[lmod](https://lmod.readthedocs.io/en/latest/index.html).
Franklin currently uses `lmod`, which is cross-compatible with `envmod`.

## Usage

The `module` command is the entry point for users to manage modules in their environment.
All module operations will be of the form `module [SUBCOMMAND]`. Usage information is available
on the cluster by running `module --help`.

The basic commands are: `module load [MODULENAME]` to load a module into your environment; `module unload [MODULENAME]` to remove that module; `module avail` to see modules available for loading; and `module list` to see which modules are currently loaded.
We will go over these commands, and some additional commands, in the following sections.

### Listing

#### `module avail`

Lists the modules **currently available** to load on the system. Some example output would be:

```bash
$ module avail

----------------------- /share/apps/franklin/modulefiles ------------------------
   conda/base/4.X                conda/cryolo/1.8.4-cuda-11 (D)    testmod/1.0
   conda/cryolo/1.8.4-cuda-10    conda/rockstar/0.1

---------------------- /share/apps/spack/modulefiles/Core -----------------------
   StdEnv                    (L)    mash/2.3
   abyss/2.3.1                      masurca/4.0.9
   amdfftw/3.2+amd                  mcl/14-137
   aragorn/1.2.38                   meme/5.3.0
   bedtools2/2.30.0                 metaeuk/6-a5d39d9
   blast-plus/2.12.0                minced/0.3.2
   blast2go/5.2.5                   miniasm/2018-3-30
   blat/35                          minimap2/2.14
   bowtie/1.3.0                     mirdeep2/0.0.8
   bowtie2/2.4.2                    mmseqs2/14-7e284
   bwa/0.7.17                       mothur/1.48.0
   bwtool/1.0                       motioncor2/1.5.0
   canu/2.2                         mummer/3.23
   cap3/2015-02-11                  mummer4/4.0.0rc1
   clustal-omega/1.2.4              muscle/3.8.1551
   clustalw/2.1                     ncbi-rmblastn/2.11.0
   corset/1.09                      ncbi-toolkit/26_0_1
```

Each entry corresponds to software available for [load](modules.md#loading-and-unloading).
Each section is a different directory of module files; this is largely unimportant for end users.

Note that this does not necessarily list every possible module on the system.
Some software is compiled with a specific compiler and compiler version, and the relevant compiler
module must be loaded first.
To see all **possible** modules, use the [`module spider`](modules.md#module-spider) command.

#### `module spider`

Lists **all possible** modules that could be loaded. Some modules require a specific
compiler version to be loaded, and these modules will not be listed in `module avail`
unless that module is loaded. Module spider will list these modules anyway.
If you run the command with a specific module, it will list the prerequisite modules
required to make said module available. For example, if we try to run:

```bash
$ module load megahit

Lmod has detected the following error:  These module(s) or extension(s) exist but cannot be loaded as requested: "megahit"
   Try: "module spider megahit" to see how to load the module(s).
```

...the load fails. If we then use `spider`:

```bash
$ module spider megahit                                                  130 ↵

-----------------------------------------------------------------------------
  megahit: megahit/1.1.4
-----------------------------------------------------------------------------

    You will need to load all module(s) on any one of the lines below before the "megahit/1.1.4" module is available to load.

      gcc/4.9.4
 
    Help:
      MEGAHIT: An ultra-fast single-node solution for large and complex
      metagenomics assembly via succinct de Bruijn graph
```

...we see that we have to first load `gcc/4.9.4`. Let's do that:

```bash
$ module load gcc/4.9.4
gcc/4.9.4: loaded.

$ module avail

-------------------- /share/apps/spack/modulefiles/gcc/4.9.4 --------------------
   megahit/1.1.4

----------------------- /share/apps/franklin/modulefiles ------------------------
   conda/base/4.X                conda/cryolo/1.8.4-cuda-11 (D)
   conda/cryolo/1.8.4-cuda-10    conda/rockstar/0.1

---------------------- /share/apps/spack/modulefiles/Core -----------------------
   StdEnv                    (L)    libevent/2.1.12        (L)
   abyss/2.3.1                      mash/2.3
```

We are now presented with a new section for those modules that require `gcc/4.9.4`,
with the `megahit` module listed there. Now, we can load it:

```bash
$ module load megahit

megahit/1.1.4: loaded.
```

Running `module spider` without any arguments will list all the modules that could be loaded,
unlike `module avail`, which will list only the modules available for load given any currently-loaded
prerequisites.

#### `module list`

Lists the modules **currently loaded** in the user environment. By default, the output should be
similar to:

```bash
$ module list

Currently Loaded Modules:
  1) cuda/11.8.0   3) libevent/2.1.12   5) slurm/22-05-6-1   7) openmpi/4.1.4
  2) hwloc/2.8.0   4) pmix/4.1.2        6) ucx/1.13.1
```

Additional modules will be added or removed as you load and unload them.

#### `module overview`

This will give a **condensed overview** of available modules.
It is most useful when the list of modules is very large, and there are multiple versions of each
module available.
The output is a list of each module name followed by the number of versions of the module.

```bash
$ module overview

----------------------- /share/apps/franklin/modulefiles ------------------------
conda/base (1)   conda/cryolo (2)   conda/rockstar (1)   testmod (1)

---------------------- /share/apps/spack/modulefiles/Core -----------------------
StdEnv         (1)   hmmer            (1)   ncbi-toolkit  (1)
abyss          (1)   homer            (1)   ncbi-vdb      (1)
amdfftw        (1)   hwloc            (1)   nextflow      (1)
aragorn        (1)   igv              (1)   openmpi       (1)
bedtools2      (1)   infernal         (1)   orthofinder   (1)
blast-plus     (1)   intel-oneapi-mkl (1)   orthomcl      (1)
blast2go       (1)   interproscan     (1)   parallel      (1)
blat           (1)   iq-tree          (1)   patchelf      (1)
```

### Loading and Unloading

#### `module load`

This **loads** the requested module into the active environment.
Loading a module can edit environment variables, such as prepending directories to `$PATH` so that
the executables within can be run, set and unset new or existing environment variables, define shell functions,
and generally, modify your user environment arbitrarily.
The modifications it makes are tracked, so that when the module is eventually unloaded, any changes can be returned
to their former state.

Let's load a module.

```bash
$ module load bwa/0.7.17
bwa/0.7.17: loaded.
```

Now, you have access to the `bwa` executable. If you try to run `bwa mem`, you'll get its help output.
This also sets the appopriate variables so that you can now run `man bwa` to view its manpage.

Note that some modules have multiple versions. Running `module load [MODULENAME]` without specifying a version
will load the latest version, unless a default has been specified.
When there are multiple versions, a `(D)` will be printed next to the default version when using [`module avail`](modules.md#module-avail).

Some modules are nested under a deeper hierarchy. For example, `relion` has six variants, two under `relion/cpu` and four under `relion/gpu`.
To load these, you must specify the second layer of hierarchy: `module load relion` will fail, but `module load relion/cpu` will load the default module under `relion/cpu`, which has the full name `relion/cpu/4.0.0+amd`.
More information on this system can be found under [Organization](modules.md#organization).

The modules on Franklin are all configured to set a `$NAME_ROOT` variable that points to the installation prefix.
This will correspond to the name of the module, minus the version. For example:

```bash
$  ls -R $BWA_ROOT
/share/apps/spack/spack-v0.19/opt/spack/linux-ubuntu22.04-x86_64_v3/gcc-9.5.0/bwa-0.7.17-3ogkbh2ixha52dxps2letankhc2dbeax:
bin  doc  man

...
```

Usually, this will be a very long pathname, as most software on the cluster is managed via the
[spack](https://spack.readthedocs.io/en/latest/) build system.
This would be most useful if you're [developing software](developing.md) on the cluster.

### Searching

## Organization
