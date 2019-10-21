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

In it's most basic form,

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
