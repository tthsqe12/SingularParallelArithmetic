# Parallel multivariate polynomial computations

## ... using FLINT and Singular

## About this document

> - Date: 2019-08-28
> - Authors: Erik M. Bray, Samuel Lelièvre, Daniel Schulz
> - Occasion: OpenDreamKit Final Reports Sprint
>   - website: https://opendreamkit.org/2019/08/26/FinalReportsSprint/
> - We acknowledge financial support from the OpenDreamKit Horizon 2020
>   European Research Infrastructures project (#676541).
> - License: CC-BY

## Introduction

OpenDreamKit members are preparing for the final review meeting,
where they are planning to set up a JupyterHub instance with
their demonstrators.

The present notes document setting up an environment with bleeding edge
versions of FLINT and Singular, a recent Jupyter, and the Singular
Jupyter kernel, so as to showcase Dan Schulz's work on parallel
multiplication of multivariate polynomials with integer coefficients
in FLINT, which also speeds up Singular when it is set to use FLINT.

Each section documents a particular step.

## Local installation

We first work locally on a laptop with 4 processors.

## Install prerequisites

We need a few prerequisites.

```
$ sudo apt-get update
```

The list of packages below might not be minimal.
```
$ sudo apt-get install autoconf autogen build-essential bzip2 clang cmake curl g++ gcc git libgmp-dev libgmp10 libgmpxx4ldbl libmpfr-dev libtool m4 make nano ninja-build patch pkg-config sudo unzip vim wget xsltproc
```

## Install FLINT trunk

The FLINT development page

- http://www.flintlib.org/development.html

says the development branch is called 'trunk'.

We follow the installation instructions in the 'INSTALL' file:

- https://github.com/wbhart/flint2/blob/trunk/INSTALL

Get the sources and run `configure` and `make`:
```
$ cd
$ git clone https://github.com/wbhart/flint2
$ cd flint2
$ ./configure --prefix=$HOME/.local
$ make -j12
```

Optional: run some tests:
```
$ make check MOD=fmpz_mpoly:mul
$ make profile MOD=fmpz_mpoly
```

Install:
```
$ make install
```

Quick check that the latest improvements to FLINT work well:
```
$ LD_LIBRARY_PATH=$HOME/.local/lib ./build/fmpz_mpoly/profile/p-mul 16 dense 30 30
```

## Install Singular

Follow instructions on Singular wiki page:

- https://github.com/Singular/Sources/wiki/Installation-from-GitHub-on-Debian

The only real change is `--disable-omalloc` and this is an important one. 
Get the sources and run `autogen`, `configure` and `make`:
```
$ cd
$ git clone https://github.com/Singular/Sources
$ cd Sources
$ ./autogen.sh
$ LDFLAGS="-Wl,-rpath,$HOME/.local/lib"
$ ./configure --disable-omalloc --with-flint=$HOME/.local --prefix=$HOME/.local
$ make -j12
```

Try it:
```
$ ~/Sources/Singular/Singular
> ring r = 0, (x, y, z, t), lp;
> poly p = (1 + x + y + z + t)^30; size(p);
46376
> size(p * p);
635376
```

It is using FLINT, which is good. Timings:
```
> system("--ticks-per-sec", 1000);
> int time1 = rtimer; size(p*p); rtimer - time1;
635376
5475
```

Finish up the installation of Singular:
```
$ make install
```

## Install Conda

In order to install Python, pip, and jupyter via Conda,
we first need to install Conda.

We use Minimamba which is a version of Miniconda that
also provides Mamba, a version of Conda with faster
dependency resolution.

- https://quantstack.net/mamba.html

Download the install script:
```
$ cd
$ wget https://github.com/QuantStack/mamba/releases/download/0.0.7/minimamba-0.0.7-Linux-x86_64-py37.sh
```
Run it:
```
$ minimamba-0.0.7-Linux-x86_64-py37.sh
```
Read the licence agreement and accept it.

Say yes to everything.

Create a file `~/.condarc` and write to it the following:
```
channels:
  - conda-forge
  - defaults
```

One way of creating the file and writing the above to it is:
```
$ cat > ~/.condarc <<EOF
channels:
  - conda-forge
  - defaults
EOF
```

For Conda / Mamba to be active, open a new terminal,
or source your `/.bashrc` file:
```
$ source ~/.bashrc
```

Once this is done, the prompt now start with `(base)`,
indicating the conda base environment.

To leave the conda base environment type
```
$ conda deactivate
```

To activate it again, type
```
$ conda activate
```

or you can specify which environment to activate:
```
$ conda activate base
```

Whether or not a conda environment is activated,
the command `conda` is now in your path so that
you can at least run `conda activate`.

Once the base environment is activated, the
command `mamba` is also available. You can use
`mamba create`, `mamba install` and `mamba update`
instead of `conda create`, `conda install` and
`conda update`.  You still use `conda activate`
and `conda deactivate`.

Update mamba:
```
$ mamba update mamba
```

## Install Python, pip, and Jupyter via Conda

Create a new conda environment called `jupyter`
and install Jupyter in it:
```
$ mamba create -n jupyter jupyter pip
```

Activate it:
```
$ conda activate jupyter
```

## Install the Singular Jupyter kernel

We follow the "Graphical interface" page of the Singular website:

- https://www.singular.uni-kl.de/index.php/graphical-interface.html

```
$ export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:$HOME/.local/lib/pkgconfig
$ export PATH=$PATH:$HOME/.local/bin
$ pip install --upgrade --user PySingular
$ pip install --upgrade --user jupyter_kernel_singular
```

## Customize the Singular Jupyter kernel

Check where the Singular Jupyter kernel is installed:
```
$ jupyter kernelspec list
```

Edit the file `kernel.json` in the singular Jupyter kernel.

This could be achieved by a command such as:
```
$ vim $HOME/.local/share/jupyter/kernels/singular/kernel.json
```

It is a json file containing a dictionary with entries `argv`,
`language`, `codemirror_mode`, etc.

One can add an entry to set the environment variables as above:

```
    "env": {
        "PATH": "$PATH:$HOME/.local/bin",
        "PKG_CONFIG_PATH": "$PKG_CONFIG_PATH:$HOME/.local/lib/pkgconfig"
        },
```

## Try it locally

Launch the Jupyter notebook server:
```
$ jupyter notebook
```

This should launch a notebook server (lines of log appear in the terminal)
and connect your default web browser to it.

It works! We can create a new Jupyter notebook with Singular kernel,
and we can compute in it.

```
In [1]  1 + 2;
Out[1]  3

In [2]  ring r = 0, (x, y, z, t), lp; r;
Out[2]  // coefficients: QQ
        // number of vars : 4
        //        block   1 : ordering lp
        //                  : names    x y z t
        //        block   2 : ordering C

In [3]  poly p = (1 + x + y + z + t)^30; size(p);
Out[3]  46376

In [4]  int time1 = rtimer; size(p*p); rtimer - time1;
Out[4]  635376
        4
```

We can rename our notebook by clicking the word "Untitled" at the top.

The corresponding file name will change from `Untitled.ipynb`
to our chosen name, still with the `.ipynb` extension.


## Remote installation

We use a remote computer with multiple processors.

To connect to the remote computer set up by Nicolas,
we ask Nicolas what to use instead of "user@example.com"
and we send him our public ssh key so he can allow us
to connect.

Connection is as follows:
```
$ ssh user@example.com
```

Check the operating system release:
```
$ cat /etc/os-release
```

Check the number and model of processors:
```
$ cat /proc/cpuinfo
```

We then perform all the same installation steps as we did
for the local installation.

## Remote use

For this we need to setup some port forwarding.

On the remote machine, edit the configuration file for the ssh daemon:
```
$ sudo vim /etc/ssh/sshd_config
```

and add or uncomment or change to yes the following line:
```
AllowTcpForwarding yes
```

then restart the ssh daemon:
```
sudo /etc/init.d/ssh restart
```

Most likely, when doing that, we lose your ssh connection;
but in rare cases we might be lucky and not lose it.

On the local machine, edit the ssh config file to add the distant machine:
```
cat >> ~/.ssh/config <<EOF
Host opendreamkit-hpc-demo
    Hostname 134.158.74.39
    User debian
EOF
```

and set up the local port forwarding:
```
ssh -N -L 8888:127.0.0.1:8888 opendreamkit-hpc-demo
```

where `-N` is for "don't open a remote shell", so that
command just blocks and waits until we hit ctrl-C.


On the remote machine:




## JupyterLab

We could also use JupyterLab:
```
$ mamba install jupyterlab
$ jupyter lab
```

## Notes

Our starting list of packages was extracted from the list
of prerequisites for OSCAR as found on OSCAR's install page:

- https://oscar.computeralgebra.de/install/

```
$ sudo apt-get install 4ti2 ant ant-optional autoconf autogen bliss build-essential bzip2 clang cmake curl debhelper default-jdk gfortran git graphviz language-pack-el-base language-pack-en libbliss-dev libboost-dev libboost-python-dev libcdd-dev libcdd0d libdatetime-perl libflint-dev libglpk-dev libgmp-dev libgmp10 libgmpxx4ldbl libjson-perl libmpfr-dev libncurses5-dev libnormaliz-dev libntl-dev libperl-dev libppl-dev libreadline6-dev libsvn-perl libterm-readkey-perl libterm-readline-gnu-perl libtool libxml-libxml-perl libxml-libxslt-perl libxml-perl libxml-writer-perl libxml2-dev libxslt-dev libzmq3-dev m4 make nano ninja-build patch pkg-config python-dev python3-pip sudo unzip vim wget xsltproc
```
