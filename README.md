# EIPP 2019 Singularity group project

<a href="https://sylabs.io/singularity/"><img src="https://sylabs.io/guides/2.6/admin-guide/_static/logo.png" height="100"/></a> <img src="https://science.sciencemag.org/content/sci/287/5457/1401/F1.medium.gif" height="100"/>

## Introduction to containers

### tl;dr

A container is a standard unit of software that packages up code and all its
dependencies, so the application runs quickly and reliably from one computing
environment to another. That includes files, environment variables, dependencies and
libraries.

For those who would like more detailed information about what containers are, please
refer to [this fantastic slide deck from Josep Moscardo][josep-slides].

[josep-slides]: https://github.com/titansmc/singularity-training-2019/raw/master/1.-singularity-training-what-are-containers.odp

## What can I do with a container?

In it's most basic form, you can execute a software program, via a container, even
though you may not have that program installed on the system you are running it on.

Examples are the best teachers!

Firstly, let's clone this repository (and call it `eipp-singularity`) as we will use some files from it throughout this
project.

```sh
project="eipp-singularity"
git clone https://github.com/mbhall88/eipp-2019-singularity.git "$project"
cd "$project"
```

Now, there is a [BAM][bam] file in the repository that we sadly can't view as we do not have [`samtools`][samtools] installed (let's pretend). Thanks to Singularity we
don't have to worry about trying to install `samtools` and can instead use a pre-built container to view our BAM file with `samtools`.

```sh
img="docker://quay.io/biocontainers/samtools:1.9--h10a08f8_12"
singularity exec "$img" samtools view data/toy.bam
```

Magic ✨  

So what's going on here?

Let's work our way through the command.

1.  [`singularity exec`][exec] tells Singularity to execute a given command inside a
    given container.
2.  `"$img"` specifies the container for Singularity to operate on. We will look at this component in more detail later.
3.  `samtools view data/toy.bam` This is the command we want Singularity to execute inside the container. Notice how we can specify files that exist on our local file system?!

[bam]: https://samtools.github.io/hts-specs/SAMv1.pdf

[samtools]: https://github.com/samtools/samtools

[exec]: https://sylabs.io/guides/3.4/user-guide/quick_start.html#executing-commands

## How do I get a container?

### Remote

Singularity Hub, Docker Hub, quay.io etc.

### Build locally

Show sample with simple program from binary

## Exercise 1

PRs for two recipes (simple programs with binaries)

## Sandbox developmenet

More complicated installations or multiple programs

## Exercise 2

PR for two recipes required some sandbox usage.

## `run` and serving applications

Example of the `run` command and also serving things (jupyter/RStudio) from a container

## Workflow management systems

Very short examples of using in `snakemake` and `nextflow`

## Programs requiring GPUs

Example of GPU usage with `tensorflow`
