# EIPP 2019 Singularity group project

<a href="https://sylabs.io/singularity/"><img src="https://sylabs.io/guides/2.6/admin-guide/_static/logo.png" height="100"/></a> <img src="https://science.sciencemag.org/content/sci/287/5457/1401/F1.medium.gif" height="100"/>
[![https://www.singularity-hub.org/static/img/hosted-singularity--hub-%23e32929.svg](https://www.singularity-hub.org/static/img/hosted-singularity--hub-%23e32929.svg)](https://singularity-hub.org/collections/3751)


-   [Introduction to containers](#introduction-to-containers)
    -   [tl;dr](#tldr)
-   [What can I do with a container?](#what-can-i-do-with-a-container)
-   [How do I get a container?](#how-do-i-get-a-container)
    -   [Remote](#remote)
        -   [Docker Hub](#docker-hub)
        -   [Singularity Hub](#singularity-hub)
        -   [Singularity Library](#singularity-library)
        -   [Quay and BioContainers](#quay-and-biocontainers)
    -   [Build locally](#build-locally)
-   [Exercise 1](#exercise-1)
    -   [Task 1](#task-1)
    -   [Task 2](#task-2)
    -   [Task 3](#task-3)
-   [Sandbox development](#sandbox-development)
-   [Exercise 2](#exercise-2)
-   [`run` and serving applications](#run-and-serving-applications)
    -   [`singularity run`](#singularity-run)
    -   [Serving applications](#serving-applications)
-   [Workflow management systems](#workflow-management-systems)
-   [Programs requiring GPUs](#programs-requiring-gpus)
-   [Bonus](#bonus)

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

The above command will pull the container into the current directory and name it `miniconda3-4.6.14.sif`. If we wanted to call it instead `miniconda3.sif` we would use the `--name` argument

```sh
singularity pull --name miniconda3.sif docker://continuumio/miniconda3:4.6.14
```

When we want to use this image again in the future, rather than specifying the URI we
just point Singularity at our local copy

```sh
singularity exec miniconda3.sif <command>
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
singularity pull --name centrifuge.sif "$uri"
singularity exec centrifuge.sif centrifuge --help
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
singularity pull --name centrifuge.sif "$uri"
singularity exec centrifuge.sif centrifuge --help
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
not usable with v2. The container extension for v2 was `.simg`, so you may see this sometimes.
For instance, the cluster at EBI is still on v2 (the training VMs are v3). For those using
the Heidelberg cluster, your cluster has v3. Singularity v2 containers, with the `.simg` extension,
can be executed by Singularity v3. You will also find all of the recipe
files in that presentation in the [`recipes/`][recipes-dir] directory of this repository.

[build-slides]: https://slides.com/mbhall88/making-containers#/

[recipes-dir]: https://github.com/mbhall88/eipp-2019-singularity/tree/master/recipes

[sandbox-slides]: https://slides.com/mbhall88/making-containers#/2/4

## Exercise 1

Form two groups and complete the following tasks.

### Task 1

[Fork][fork] this repository on GitHub.

[fork]: https://help.github.com/en/github/getting-started-with-github/fork-a-repo

### Task 2

[Enable Singularity Hub][enable-shub] on your fork of this repository.

[enable-shub]: https://slides.com/mbhall88/remote-container-systems#/1/6

### Task 3

Each group should choose one of the following two GitHub issues to close:

-   `snakemake` recipe: [![GitHub issue/pull request detail](https://img.shields.io/github/issues/detail/state/mbhall88/eipp-2019-singularity/1)](https://github.com/mbhall88/eipp-2019-singularity/issues/1)
-   `shellcheck` recipe: [![GitHub issue/pull request detail](https://img.shields.io/github/issues/detail/state/mbhall88/eipp-2019-singularity/2)](https://github.com/mbhall88/eipp-2019-singularity/issues/2)

## Sandbox development

During the previous exercise, you may have noticed that errors in your build recipe require you to rerun the build all over again. When installing simple programs, this isn't too costly. However, when we want to build more complicated containers, it becomes time-consuming to rerun the entire build continually. In this section, we will look at how we can use Singularity's [`--sandbox`][sandbox] option to speed up the container recipe development cycle.

So what is a sandbox? Think of it as a directory that mimics the inside of a container. You can then start an interactive shell session in this sandbox and run commands in the same environment that they will run in when building the container. In this way, you can test out what commands you need to run to get your program(s) installed and executing correctly. This massively reduces your turnaround time for creating containers. In addition, as we make the sandbox writeable, any changes we make will stay saved.

Let's get into the sandbox and play!

Create a new directory where we will do our sandbox development.

```sh
mkdir sandbox-dev
cd sandbox-dev
```

Next, we will use the [template recipe][template-recipe] in this repository to build our sandbox from.

```sh
sudo singularity build --sandbox playground ../recipes/Singularity.template
```

You should now see a directory called `playground`. I've named the sandbox `playground`, but you can name it whatever you want.

Now we will start an interactive shell within the sandbox/container image.

```sh
sudo singularity shell --writable playground
```

_Note: If you don't use [`--writable`][writable] you won't be able to install anything or do anything that changes the size of the container._

You should now see the prompt change to something like

```sh
Singularity playground:~>
```

**IMPORTANT:  
The directory `/root` from your local machine will be mounted in the sandbox. So anything you do in the sandbox in that directory will also be reflected in the `/root` directory locally.
Ensure you move out of `/root` within the sandbox and do all of your work there. I tend to use `/usr/local`, but you could create a new directory altogether (but outside `/root`) e.g. `/sandbox`.**

```sh
cd /usr/local
```

Now we'll try and [install `conda`][install-conda] inside the sandbox.

```sh
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda3.sh
```

This will give us: `bash: wget: command not found`. A perfect example of why these sandboxes
are so useful. The OS installation is _very_ minimal and doesn't include a lot of programs.  

Let's install `wget` in our sandbox and try again.

```sh
apt install -y wget
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda3.sh
```

Now we'll install `conda`, specifying the prefix (`-p`) as a directory in `/usr/local`
called `miniconda`.

```sh
bash miniconda3.sh -b -p $(pwd)/miniconda
```

In order to run `conda` now, we need to ensure it's binary is in our `PATH`.

```sh
export PATH="$(realpath miniconda/bin):${PATH}"
```

_Remember from the [`%environment` slide][env-slide] that when writing the recipe for
this `conda` installation we would need to write the `export` line as:_

```sh
echo "export PATH=$(realpath miniconda/bin):${PATH}" >> "$SINGULARITY_ENVIRONMENT"
```

Lastly, we need to test `conda` is executable.

```sh
conda list
```

In order to convert these commands into a recipe I generally keep a text file open where
I paste (successful) commands into as I go so I don't have to search back through my
shell history later.

[sandbox]: https://sylabs.io/guides/3.4/user-guide/build_a_container.html#creating-writable-images-and-sandbox-directories

[template-recipe]: https://github.com/mbhall88/eipp-2019-singularity/blob/master/recipes/Singularity.template

[writable]: https://sylabs.io/guides/3.4/user-guide/build_a_container.html#writable

[install-conda]: https://conda.io/projects/conda/en/latest/user-guide/install/macos.html#install-macos-silent

[env-slide]: https://slides.com/mbhall88/making-containers#/1/7

## Exercise 2

Similar to [Exercise 1](#exercise-1), form two groups (can be different groups) and put
in a pull request each to close the following two issues:

-   `flye` recipe: [![GitHub issue/pull request detail](https://img.shields.io/github/issues/detail/state/mbhall88/eipp-2019-singularity/3)](https://github.com/mbhall88/eipp-2019-singularity/issues/3)
-   `nanopolish` recipe: [![GitHub issue/pull request detail](https://img.shields.io/github/issues/detail/state/mbhall88/eipp-2019-singularity/4)](https://github.com/mbhall88/eipp-2019-singularity/issues/4)

I chose more complicated programs this time so you can get some experience using a sandbox.

## `run` and serving applications

### `singularity run`

The [`run`][run-docs] directive will execute the [`%runscript`][runscript-slides] and
pass along all arguments to this script. The `run` directive is handy for when you want
to automate some common tasks using the programs installed within the container and be
able to handle user options. Refer to [the slide on `%runscript`][runscript-slides],
from the earlier section on [buiding containers locally](#build-locally), for
an example of using `singularity run`.  

[run-docs]: https://sylabs.io/guides/3.4/user-guide/cli/singularity_run.html

[runscript-slides]: https://slides.com/mbhall88/making-containers#/1/10

### Serving applications

It is also possible to serve applications through a port from a container. As an example
we will build a container to run a [`jupyter notebook`][jupyter] that we can access on
our local machine.

The recipe to do this can be found in the `recipe/` directory as [`Singularity.jupyter`][jupyter-recipe].
Of particular interest for this example, see the `%runscript` section.

```sh
%runscript
    PORT="${1:-8888}"
    echo "Starting notebook..."
    echo "Open browser to localhost:${PORT}"
    exec /usr/local/bin/jupyter notebook  --ip='*' --port="$PORT" --no-browser
```

We take the first option passed by the user and store it in a variable `PORT`, or use `8888`
if nothing is given. We print some logging to the screen with `echo` and then start
a `jupyter` session, passing the `PORT` to `jupyter`.  

Let's build this image and then fire it up.

```sh
sudo singularity build jupyter.sif recipes/Singularity.jupyter
# we will use the default port 8888
singularity run jupyter.sif  
```

You should get some output from `jupyter` indicating it has started running the notebook
and providing a location, which should look something like:

```
[I 11:40:28.948 NotebookApp] Serving notebooks from local directory: /home/vagrant/container-dev
[I 11:40:28.949 NotebookApp] The Jupyter Notebook is running at:
[I 11:40:28.949 NotebookApp] http://dev-vm:8888/?token=c8fe88de778120e5ccd42850d6d13712e27b125b0481d5b0
[I 11:40:28.949 NotebookApp]  or http://127.0.0.1:8888/?token=c8fe88de778120e5ccd42850d6d13712e27b125b0481d5b0
[I 11:40:28.949 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 11:40:28.953 NotebookApp]
```

Copy the URL (either one), and paste it into a web browser. You should now see the home
page for the notebook. Select the example notebook at `notebooks/plot.ipynb`.

Run the two cells in the notebook, and you should see some toy data plotted.

This is quite a simple use case for serving applications. You can do far more complicated
things like [running an RStudio server][rstudio] from a container and access it locally.

[jupyter]: https://jupyter.org/

[jupyter-recipe]: https://github.com/mbhall88/eipp-2019-singularity/blob/master/recipes/Singularity.jupyter

[rstudio]: https://divingintogeneticsandgenomics.rbind.io/post/run-rstudio-server-with-singularity-on-hpc/

## Workflow management systems

Containers and workflow management systems (WMSs), such as `snakemake` and [`nextflow`][nextflow],
are a match made in heaven. Containers add a crucial layer of reproducibility to these systems.  

Though this is not a project to teach you how to use WMSs, I would
encourage you to take a look at [this short slide deck][wms-slides] from the Singularity course I ran
as it shows you how easy it is to integrate Singularity containers into WMSs.

[nextflow]: https://www.nextflow.io/

[wms-slides]: https://slides.com/mbhall88/singularity-and-workflow-management-systems#/

## Programs requiring GPUs

Singularity also provides the ability to utilise GPU cards, without needing to install
the GPU drivers into your container. Currently, it can only use NVIDIA GPUs. To allow a
container to use the local GPU card and drivers all you need to do it pass the
[`--nv`][gpu] option. For example, to get a python shell with the GPU version of `tensorflow`
available, you would run the following (on a machine with an NVIDIA GPU).

```sh
singularity exec --nv docker://tensorflow/tensorflow:latest-gpu python
```

[gpu]: https://sylabs.io/guides/2.6/user-guide/appendix.html#a-gpu-example

## Bonus

If you have gotten to this point, then have a go at creating a container for a piece of
software you have had difficulties installing in the past. Alternatively, you could try 
and reduce the size of the containers we have already produced by using [Alpine][alpine] as the 
base OS.

[alpine]: https://www.alpinelinux.org/
