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

In the above example, the container we used for `samtools` was remote.  

Remote containers are containers that have been pre-built and stored in "the cloud".
There are many benefits to this kind of set up. Firstly, it makes sharing containers
easy. Secondly, it saves users (and yourself) a lot of time in the future. As the
container is pre-built, we don't need to spend time waiting for the build to happen (more on this later). The only wait time we have is for the download of the remote
container to finish. Lastly, remote services are convenient for building images if we
don't have `sudo` access on the machine we are using. We will look at building containers
locally very soon, but for now, it suffices to know that to build them locally, you need
`sudo` access.

Now you might have noticed in the example above that the [URI][uri] for the `samtools`
container has the work 'docker' in it. This is one of the coolest things about Singularity: [it can convert Docker containers into Singularity containers][singularity-docker]! We now have
access to any Docker container _plus_ any Singularity container.

Let's take a look at some remote container registries in a little more detail and see
how we can use containers from them.

[singularity-docker]: https://sylabs.io/guides/3.4/user-guide/singularity_and_docker.html

#### [Docker Hub][dockerhub]

The official registry for Docker containers. Let's search for [`miniconda3`][miniconda3] on [Docker Hub][dockerhub] and select the option [`continuumio/miniconda3`][miniconda3-dockerhub]. On the right, there is a section **Docker Pull Command**. It
says `docker pull continuumio/miniconda3`. If we were using Docker, this would be the
command we would use to pull that container to our local machine. To use it in Singularity
we need to tweak it just a little. For `miniconda3` we would use the URI `docker://continuumio/miniconda3`. As we can see, you need to add `docker://` to the
beginning of the `repository/tag`.  
We can go one step further and unlock another great benefit of using remote containers. We're reproducibility warriors, right?! Of course, we are. So let's be specific
about the version of `miniconda3` we want to use. On the [`miniconda3` Docker Hub page][miniconda3-dockerhub], select the [**Tags**][miniconda3-tags] heading. On this
page, we see a whole bunch of different versions of `miniconda3` we can choose from. Any
version of this container that has been built is kept. If we wanted to use version `4.6.14`, then all we have to do is append this, with a colon, to our original URI  

```sh
docker://continuumio/miniconda3:4.6.14
```

Now, as we saw earlier, we can directly execute a container from it's URI. However, it
is likely you may want to use a container multiple times. In these circumstances, it is
more "economical" to pull a copy of the container onto our local machine, so we don't
have to try and retrieve it from the registry each time (images are usually cached though). To pull the `miniconda3` container from Docker Hub, we use Singularity's [`pull`][singularity-pull]
command and optionally specify a name.

```sh
singularity pull docker://continuumio/miniconda3:4.6.14
```

The above command will pull the container into the current directory and name it `miniconda3-4.6.14.simg`. If we wanted to call it instead `miniconda3.simg` we would use the `--name` argument

```sh
singularity pull --name miniconda3.simg docker://continuumio/miniconda3:4.6.14
```

When we want to use this image again in the future, rather than specifying the URI we
just point Singularity at our local copy

```sh
singularity exec miniconda3.simg <command>
```

[uri]: https://en.wikipedia.org/wiki/Uniform_Resource_Identifier

[dockerhub]: https://hub.docker.com/

[miniconda3]: http://conda.pydata.org/miniconda.html

[miniconda3-dockerhub]: https://hub.docker.com/r/continuumio/miniconda3

[miniconda3-tags]: https://hub.docker.com/r/continuumio/miniconda3/tags

[singularity-pull]: https://sylabs.io/guides/3.4/user-guide/quick_start.html#download-pre-built-images

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
