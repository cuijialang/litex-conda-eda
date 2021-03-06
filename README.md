# litex-conda-eda

Conda recipes for FPGA EDA tools (for synthesis, place and route and bitstream
generation).

# Synthesis

 * [Yosys](https://github.com/YosysHQ/yosys)
 * TODO: [GHDL Synthesis](https://github.com/tgingold/ghdlsynth-beta)

# Place and Route

 * [arachne](https://github.com/cseed/arachne-pnr)
 * [nextpnr (iCE40, ECP5, generic)](https://github.com/YosysHQ/nextpnr)
 * [Versatile Place and Route (vpr)](https://github.com/verilog-to-routing/vtr-verilog-to-routing)

# Bitstream

 * [Project Icestorm](https://github.com/cliffordwolf/icestorm)
 * [Project Trellis](https://github.com/SymbiFlow/prjtrellis)
 * TODO: [Project X-Ray](https://github.com/SymbiFlow/prjxray)

# Simulation

 * TODO: [GHDL](http://ghdl.free.fr/)
 * [Icarus Verilog](http://iverilog.icarus.com/)
 * [Verilator](https://www.veripool.org/wiki/verilator)

# Formal

 * [Symbiyosys](https://github.com/YosysHQ/SymbiYosys)

# Building

This repository is set up to be built by Travis CI, using the GitHub
integration to Travis CI.

See [`.travis.yml`](.travis.yml) for the build configuration given to
Travis CI, and the [`.travis`](.travis) directory for scripts referenced.

The Travis CI output can be found on the https://travis-ci.com/ for the
GitHub account and GitHub repository.  For instance, for the main:

https://github.com/litex-hub/litex-conda-eda

GitHub repository, the Travis CI results can be seen at:

https://travis-ci.com/litex-hub/litex-conda-eda

On a successful build in the `litex-hub` Travis CI, the resulting packages
are uploaded to:

https://anaconda.org/litex-hub/repo

and can be installed with:

```
conda install --channel "LiteX-Hub" package
```

These packages are mostly used by
[`litex-buildenv`](https://github.com/litex-hub/litex-buildenv).

## Building via Travis CI in your own repository

If you [enable Travis CI on your GitHub
fork](https://travis-ci.com/getting_started) of `litex-conda-eda`
then your Travis CI results will be at:

```
https://travis-ci.org/${GITHUB_USER}/litex-conda-eda
```

Since the repository includes `.travis.yml` and all the other Travis CI
setup, you should just need to turn on the Travis CI integration at GitHub,
and push changes to your GitHub repo, to trigger a Travis CI build, then
watch the https://travis-ci.org/ site for the build progress.

A full build of everything will take the Travis CI infrastructure a few
hours if it all builds successfully.

## Common Travis CI build failures

If the build fails, see the [common Travis CI build
problems](https://docs.travis-ci.com/user/common-build-problems/)
for assistance investigating the issues.  Common issues with this
repository include package dependencies (eg, where Conda has changed),
output log file size (Travis CI has a 4MB maximum, and some package
builds like `gcc` generate a *lot* of output), and builds timing out
(either for maximum time allocation, or for ["no output" for more than
10 minutes](https://docs.travis-ci.com/user/common-build-problems/#build-times-out-because-no-output-was-received)).

## Testing conda builds locally

Given a fairly empty *disposable* Ubuntu x86-64 test environment (eg,
created with Docker, or Vagrant), it is possible to simulate *part* of
what Travis CI will do to test building individual toolchain architectures
locally.

This can be done with something like:

```shell-session
sudo apt-get update
sudo apt-get install wget git

# Packages from ~/.travis.yml; realpath is in coreutils in Ubuntu 18.04
# Plus libtool and pkg-config, which are needed for openocd
#
# sudo apt-get install realpath
sudo apt-get install coreutils

git clone https://github.com/litex-hub/litex-conda-eda.git
cd litex-conda-eda
./.travis/conda-get.sh

# Adapted from .travis/common.sh
get_built_package() {
   .travis/conda-env.sh render --output "$@" 2>/dev/null | grep conda-bld | grep tar.bz2 | tail -n 1 | sed -e's/-[0-9]\+\.tar/*.tar/' -e's/-git//'
}

# Anchor the build date/time, so we have predictable versions and filenames
# amongst related packages, and to make it easy to do package installs.
#
# Either to current date/time at the start of the build:
#
# export DATE_NUM="$(date -u +%Y%m%d%H%M%S)"
# export DATE_STR="$(date -u +%Y%m%d_%H%M%S)"
#
# Or lock to date/time of the last commit on git, as Travis CI config does
# (see .travis/common.sh)
#
export DATE_TS="$(git log --format=%ct -n1)"
export DATE_NUM="$(date --date=@${DATE_TS} -u +%Y%m%d%H%M%S)"
export DATE_STR="$(date --date=@${DATE_TS} -u +%Y%m%d_%H%M%S)"

# Combinations taken from .travis.yml
TOOLCHAIN_ARCH=lm32
export PACKAGE TOOLCHAIN_ARCH

for PACKAGE in bit/* syn/* pnr/* sim/*; do
  .travis/conda-env.sh build --check "${PACKAGE}"   # Downloads and caches stuff
  .travis/conda-env.sh build         "${PACKAGE}"   # Actually build package
  CONDA_OUT="$(get_built_package ${PACKAGE})" # Calculate output package
  .travis/conda-env.sh install       "${CONDA_OUT}"
done
```

Expect packages like `binutils` to take 3-5 minutes to build, packages
like `gcc/nostdc` to take 10-15 minutes to build, and packages like
`gcc/newlib` to take 25-40 minutes to build, on a relatively fast
build system (eg, SSD, i7, reasonable amount of RAM).  Beware that
`gcc/newlib` wants to see `gcc/nostdc` *of the same version* already
installed before it will build; this means that `gcc/newlib` is
non-trivial to build individually.

To build one architecture of tools, without any cleanup will need a
VM with maybe 12-15GiB of space available (a 10GiB disk image is not
quite big enough). Building more architectures at once will need more
disk space.

**NOTE**: By preference only packages built by Travis CI should be
uploaded to the Anaconda repository, so that the externally visible
packages have consistent package versions (and do not conflict).  But
it can be useful to build locally to debug `conda-build` config issues
without waiting for a full Travis CI cycle.
