# cwl-udocker-tests
A repository for testing udocker, cwl and toil

# Step by step

## Assumptions

We assume you have available:

- Installed pyenv
- Have python version 2.7.12 available
- Have exported `PYENV_ROOT`

We will install:

- a virtualenv for toil based on python 2.7.12
- toil v3.16.0
- udocker under your home directory
- udocker devel branch (due to problems with udocker 1.1.0's string handling and cwl)
- The udocker binaries (including a precompiled proot)

## Instaling toil using pyenv

```bash
if [ ! -e $PYENV_ROOT/versions/toil ]; then
  pyenv virtualenv 2.7.12 toil
  export PYENV_VERSION=toil
  pip install -U pip setuptools wheel toil[cwl]==3.16.0
fi
```

## Installing udocker in your userspace/home directory

Because of the way udocker handles images at the moment it seems safer to use a local install of udocker versus a shared instance. We will install udocker in `$HOME/udocker-install-dev`.

```
unset UDOCKER_TARBALL
export UDOCKER_BASE=$HOME/udocker-devel
export UDOCKER_DIR=$UDOCKER_BASE/udocker
udocker_binary=$UDOCKER_BASE/bin/udocker

if [ -d $UDOCKER_BASE ]; then
  rm -rf $UDOCKER_BASE
fi

mkdir -p $HOME/udocker-install-dev
mkdir -p $UDOCKER_BASE/bin
curl 'https://raw.githubusercontent.com/indigo-dc/udocker/devel/udocker.py' > $udocker_binary
chmod ugo+x $udocker_binary
$udocker_binary install

echo "export UDOCKER_DIR=$UDOCKER_DIR" > $UDOCKER_BASE/env.sh
echo "export PATH=$UDOCKER_BASE/bin:$UDOCKER_DIR/bin:$PATH" >> $UDOCKER_BASE/env.sh
echo "export PYENV_VERSION=toil" >> $UDOCKER_BASE/env.sh
```

# Testing this has all worked

## Sourcing the environment onto your 

```bash
source $UDOCKER_BASE/env.sh
```

## Setup the job (run the example Docker CWL workflow) 

```
export TOIL_TEST=$HOME/toil-test
mkdir -p $TOIL_TEST
cd $TOIL_TEST
curl -s 'https://raw.githubusercontent.com/common-workflow-language/common-workflow-language/master/v1.0/examples/docker.cwl' > docker.cwl
curl -s 'https://raw.githubusercontent.com/common-workflow-language/common-workflow-language/master/v1.0/examples/docker-job.yml' > docker-job.yml
curl -s 'https://raw.githubusercontent.com/common-workflow-language/common-workflow-language/master/v1.0/examples/hello.js' > hello.js
```

## Executing toil locally

Will create a work directory. Remove these options when in production

```
export TOIL_WORKDIR=$PWD/workdir-local
mkdir -p $TOIL_WORKDIR
cd $TOIL_WORKDIR
toil-cwl-runner --tmpdir-prefix $PWD --tmp-outdir-prefix $PWD --jobStore $PWD/scratch-local --clean=never --cleanWorkDir=never --user-space-docker-cmd=udocker ../docker.cwl ../docker-job.yml
```

Inspect the content of the workdir for output, temporary files and everything else.

## Executing toil on LSF

```
export TOIL_WORKDIR=$PWD/workdir-lsf
mkdir -p $TOIL_WORKDIR
cd $TOIL_WORKDIR
toil-cwl-runner --batchSystem=lsf --disableCaching --tmpdir-prefix $PWD --tmp-outdir-prefix $PWD --jobStore $PWD/scratch-local --clean=never --cleanWorkDir=never --user-space-docker-cmd=udocker ../docker.cwl ../docker-job.yml
```
