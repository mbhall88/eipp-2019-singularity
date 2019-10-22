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

#### [Singularity Hub][shub]

Set up and maintained by a collaboration between Stanford University and Singularity,
Singularity Hub is Singularity's "semi-official" version of Docker Hub. We will dig
into how to set this up for yourself a little later in [Exercise 1](#Exercise-1).

As with Docker Hub, we can search for containers uploaded by users and then use them in
the same way. However, it will ask us to log in using GitHub first. Login with your
GitHub account and then search for [`centrifuge`][centrifuge]. The first result should
be for [`mbhall88/Singularity_recipes`][shub-recipes] - click on this. This will take
you to a page listing all of the Singularity containers I maintain in a [recipes repository on GitHub][shub-recipes-github]. Scroll through these and look for the
[`centrifuge`][centrifuge-shub] one and then click on the green **Complete** button.
The resulting screen will have the Build Specs (more on this soon) plus a bunch of
build metrics. Additionally, at the top of this screen, you will see the core piece of
the URI that we need: `mbhall88/Singularity_recipes:centrifuge`. So to use this container,
we add the `shub://` scheme to the front.

```sh
uri="shub://mbhall88/Singularity_recipes:centrifuge"
singularity pull --name centrifuge.simg "$uri"
singularity exec centrifuge.simg centrifuge --help
```

Due to Singularity Hub be generously hosted as no charge by Google Cloud, and also due
to a recent malicious attack, it is [recommended][shub-limits] to `pull` containers from Singularity and
then execute them, rather than running directly from the URI.

Again, we can go one step further and specify a particular build of the container we
want to use. In the **Build Metrics** section, there is a field called 'Version (file hash)'. For reproducibility purposes, it is advisable to use this hash as it makes it
clear to others who may read your code exactly which container you used. So to pull the
latest centrifuge container, we would do the following (**don't run this if you already
pulled the container above**).

```sh
hash="13bc12f41b20001f17e6f8811dc3eeea"
uri="shub://mbhall88/Singularity_recipes:centrifuge@${hash}"
singularity pull --name centrifuge.simg "$uri"
singularity exec centrifuge.simg centrifuge --help
```

[shub]: https://singularity-hub.org/

[centrifuge]: https://github.com/DaehwanKimLab/centrifuge

[shub-recipes]: https://singularity-hub.org/collections/685

[shub-recipes-github]: https://github.com/mbhall88/Singularity_recipes

[centrifuge-shub]: https://singularity-hub.org/containers/5461

[shub-limits]: https://singularityhub.github.io/singularityhub-docs/docs/interact

#### [Singularity Library][library]

This is the official container registry for Singularity. However, all images built on
this service are Singularity v3+ compatible. At EBI we only have Singularity v2.6, but
 EMBL Heidelberg's cluster does use Singularity v3+. This service works similarly to Singularity and Docker Hubs, using the scheme `library://` for its URIs.  

One additional feature that Singularity Library has is a [remote builder][remote-builder]. This builder allows you to dump a recipe for a container, it will build the
container for you, and then you can download it on to your local machine. Very handy
when working on a computer you do not have `sudo` access on.

See the slides _below_ [this][slides-library] for more information about Singularity
Library.

[library]: https://cloud.sylabs.io/library

[remote-builder]: https://cloud.sylabs.io/builder

[slides-library]: https://slides.com/mbhall88/remote-container-systems#/2/1

#### [Quay][quay] and [BioContainers][biocontainers]

Quay is a container registry for Docker and [rkt][rkt] containers. We won't talk much
about this service outside how to use the BioContainers builds hosted on it.  

BioContainers is an open-source and community-driven framework for reproducibility in
bioinformatics[<sup>1</sup>][biocontainers-paper]. They build and maintain containers for a large suite of bioinformatics
tools. In particular, any tool that has a [Bioconda][bioconda] recipe automatically has
a BioContainers image built and stored on Quay.

To see an example of how to find and use these BioContainers images check out the slides
below [here][quay-slides].

[quay]: https://quay.io/

[rkt]: https://coreos.com/rkt/

[biocontainers]: https://biocontainers.pro/

[biocontainers-paper]: https://doi.org/10.1093/bioinformatics/btx192

[bioconda]: https://bioconda.github.io/

[quay-slides]: https://slides.com/mbhall88/remote-container-systems#/4/1i

* * *

For more details on remote container systems, refer to [my slides][remote-containers-slides] from a one-day
[Singularity course][singularity-course-repo] I was involved in running at EMBL in early 2019.

[remote-containers-slides]: https://slides.com/mbhall88/remote-container-systems

[singularity-course-repo]: https://git.embl.de/grp-bio-it/singularity-training-2019

### Build locally

We've talked a lot about how to use containers that others have been kind enough to
construct for us. But what happens if an image doesn't exist for the software tool you
want to use? Or if you want to combine multiple programs into a single container? You
guessed it; we can build containers locally from definition/recipe files.

Rather than reinvent the wheel, please refer to (and work your way through) [these slides][build-slides] from the [Singularity course][singularity-course-repo] I was involved in running at EMBL in early 2019. Once you get to slide titled ["Playing in a sandbox with a shell"][sandbox-slides] you can move on to [Exercise 1](#Exercise-1).

**Note:** As the course was aimed at users of Singularity v3+ you will see the container
extension `.sif` used. This was a new container file format introduced in v3 that is
not usable with v2. So as you work through the slides on the machines here at EBI,
replace all occurrences of `.sif` with `.simg`. For those not from EBI, `.sif` is the
format you will work with on your cluster though. You will also find all of the recipe
files in that presentation in the [`recipes/`][recipes-dir] directory of this repository.

[build-slides]: https://slides.com/mbhall88/making-containers#/

[recipes-dir]: https://github.com/mbhall88/eipp-2019-singularity/tree/master/recipes

[sandbox-slides]: https://slides.com/mbhall88/making-containers#/2/4

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
