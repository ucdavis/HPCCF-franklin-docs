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
All module operations will be of the form `module [SUBCOMMAND]`, which we will go over in the
following sections.

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

This loads the requested module into the active environment. 

### Searching

## Organization
