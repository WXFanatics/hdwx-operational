# Python HDWX
---
~ Modular, future-proofed, portable HDWX with API access for custom clients

## Installation

1. Python HDWX has been tested on CentOS 7, Ubuntu 20.04, and macOS 10.15 and 11.0. You'll need an operating system (duh), and at the time of writing this, I don't ever intend to add support for Windows.
2. Clone this repo (*important: with the* `--recursive` *flag*) to somewhere on your computer. This can realistically be anywhere, but on everything I've done, I've always just shoved it in my home folder, `~/hdwx-operational/`
3. You will need to install bash 5.0 or newer. At the time of this writing, the limiting factor is modelplotter's `generate.sh`'s method of tracking PIDs to limit the number of workers active. If you absolutely need bash 4.X or older, you'll definitely need to make changes there to get this to work, but you could just install bash 5.0 to avoid that whole problem.
4. You will also need to install python via a python package manager. Currently supported package managers are [mambaforge](https://github.com/conda-forge/miniforge#mambaforge) (recommended) and [miniconda](https://docs.conda.io/en/latest/miniconda.html) (not recommended). I like mambaforge better because it's generally faster and includes the community built conda-forge repository of python packages as a default source, while miniconda uses the anaconda repository which has packages that are considered to be "stable and known to work together" but are not always the latest version of thpse packages.
5. Once you've installed the python package manager of your choice, you'll need to import the HDWX environment. There's "supposed to be" a "nice" way of doing this where I can export my entire environment to a .yml file, and anyone can import this .yml file and instantly have my exact environment... however you may have noticed the quotes around the first part of this sentence. While writing this install guide, there was [an issue](https://github.com/conda-forge/cfgrib-feedstock/issues/25) with the versioning of an external library that completely broke this automatic import feature, and this is far from an isolated incident. ANYWAY... you can try the automated import by running `mamba env create -f hdwx-env.yml` (mambaforge installs) `conda env create -f hdwx-env.yml` (miniconda installs, can also be used on mambaforge if the first command didn't work) from the top-level directory of the clone of this repo. It might work, but if not, skip down to Troubleshooting^(1). Even if everything works perfectly, you will *definitely* still get an error related to pyxlma...
6. For `hdwx-hlma` to work, you'll need to set up an external method of providing 1-minute VHF source files from an LMA network to `hdwx-hlma/lightningin/`. I currently have a "once a minute" cronjob set up on `thor.geos.tamu.edu` that runs `hdwx-hlma/hlmaFetch.py`, which works well enough to satisfy me, but definitely could be faster using more advanced methods. You will also **need** to install my fork of [pyxlma](https://github.com/wx4stg/xlma-python) by cloning the repository (somewhere outside of your clone for this repository), checking out the furthest ahead branch (at the time of writing, `git checkout cf-lma-format`), then running `conda activate HDWX && pip install .` from inside your clone of pyxlma. **If you cannot complete these requirements** (*providing LMA data and installing pyxlma*) **it is recommended that you enitrely delete the hdwx-hlma directory from your clone of hdwx-operational.** Additionally, I have narcissistically named all of the files and code/etc. to make references to the "HLMA" or "Houston Lightning Mapping Array", but if you're an operator of an LMA other than HLMA, the code "should" still work fine for you also... but you'll probably want to find and replace all references to HLMA to the name of your network... for prettyness' sake.
6. Configure the options in `config.txt`, these options should be self-explanatory, and if they're not... well they have explanations in the comments.
7. If you want the API to work, you'll need to clone [hdwx-api](https://github.tamu.edu/samgardner4/hdwx-api) to a location that will be served to the internet by your website stack and edit `config.php` to contain the parameters it's asking for... again, self-explanatory
8. Optional, but recommended, try running `all.sh` once using `bash ./all.sh`. It *should* work fine at this point, but better to test it now than to set up a cronjob to run it once per minute... only to come back hours later to find it's done nothing... not speaking from experience or anything
9. Set up a once-a-minute cronjob to run `all.sh` at the top level of your hdwx-operational clone. This will begin generating products.

## What all.sh does

When executed, all.sh opens the parent directory of itself (hdwx-operational), looks for any directories inside (hdwx-activitymonitor, hdwx-adrad, hdwx-hlma... etc.), changes to them, runs the `generate.sh` file contained inside. Each submodule is expected to product an `output/` directory which is then rsynced to your target, defined in config.txt. There can be multiple instances of all.sh running simultaneously, generating multiple products at a time (or in some cases, generating in the same submodule simultaneously using multithreaded black magic). This is normal and expected, and improves performance.

## Troubleshooting
(1) Ok, the automatic environment install didn't work. Here's how to create the environment manually (use commands as-is if you're using mambaforge, else replace `mamba` with `conda` for miniconda installs):
- `mamba create --name HDWX`
- `mamba activate HDWX`
- `mamba install arm_pyart atomicwrites cartopy cfgrib imageio lxml matplotlib metpy natsort netcdf4 numpy pandas pillow pyepsg scikit-learn scipy xarray -c conda-forge`
- `pip install ecmwf-opendata`


~~(2) If you're feeling extra risky, you can try a `mamba update --all` to fetch the latest versions of all packages, but be aware that this may break your install and you may have to remove and recreate the environment or update the python code in the submodules to remove now-deprecated function calls that I had no idea would be deprecated when I wrote this file.~~ jk, don't do that, it'll break eccodes which will break cfgrib which will break xarray which will break modelplotter and mrms (and maybe others)
