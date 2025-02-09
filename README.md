# wine-runner-sc

_Most users will just want to download a runner from the releases. The rest of this document covers running the wine runner builder._

_https://hub.docker.com/r/molotovsh/wine-builder-ubuntu will soon replace most/all of this codebase for building wine runners_

This code is for building a bi-arch wine runner (i.e. lutris compatible) via Ubuntu containers.

It is currently aimed at Star Citizen support, but will most likely support other games (and hopefully make it easier to do so).

This is version 2 - hugely rewritten to avoid as much rebuilding and recompilation as possible. Containers are built, wine sources downloaded (and patched if necessary) and then compiled (with ccache so recompiles are much less painful) into a runner. You could then package and release the runner.

This version of the builder currently uses Ubuntu 20.04 containers, so older distros can struggle with binary incompatibilities with the produced runners.

## Prerequisites

1) Install docker. This is often in your package management system.
2) Ensure you're in the docker group, e.g. `sudo usermod -a -G docker yourusername`. User group changes usually require a logout/login to apply.
3) (Optional) Install `pigz` for faster packaging if you need it.

## Usage

- Add any patches you require into the appropriate `patches` subfolder
  - e.g. for wine 5.9 it's `patches/5.9/your-patch-here.patch`, if staging requires a different patch then `patches/5.9/staging/your-patch-here.patch` is supported, which will use patches in the subfolder for staging builds.
- Setup environment variables (described below)
- Run `setup.sh` to setup some folders and build the containers
- Run `wine-git.sh` to download wine sources and optionally staging and any *.patch in patches.
- Run `build-wine.sh` to run the two build scripts. It will tell you where the finished build is (build/wine-runner-$version typically).
  - You can use that runner directly (move/copy the runner folder to `~/.local/share/lutris/runners/wine/`)
- (Optional) Run `package.sh` to tgz up your runner for distribution.

#### Wizard

New - a batch job which will given a particular wine version as a parameter build vanilla and staging runners (patches chosen by `wine_staging_list` - by default all) and package them.

```
./wizard.sh 5.9
```

### Known Issues

`build` can become quite large over time, especially the `ccache` subfolders which are deliberately unrestricted in size. `ccache` is cached compiled objects to make running `build-wine.sh` faster for future runs, it can be cleared but note build times will suffer. It is pruned for files older than 14 days in `build-wine.sh` (unless disabled).

If you encounter `wine-builder*` is already running errors, use `docker kill ${container_name}` to stop it and `docker rm ${container_name}` to remove it. `docker ps --all` will tell you what containers are running/configured.

### Environment variables

These are not necessary using the `wizard.sh` which is aimed at distribution builds, though `wine_staging_list`should be at least considered in said scenario.

These should be set with export, e.g.

```
export wine_version=5.2
export do_wine_staging=yes
```

#### Required

- `wine_version`: MUST (unless using wizard) be set to a wine version like 5.2 - see tags on wine sources for valid examples. The `wine-` is added in for you.

#### Recommended

- `do_wine_staging`: I recommend this is set to `yes` (unless using wizard) but that is up to you. You will need some `wine-staging` for esync if that matters to you.

#### Optional

- `build_cores`: Manually choose the core count for builds, used in `make -j$build_cores install`. Default is number of `processor` found in `/proc/cpuinfo` + 1.
- `wine_staging_list`: A list of patches to load from `wine-staging`, see `wine-git.sh` for the default list. The default option is all, which is probably overkill, wildcards are accepted, e.g. `eventfd*`
- `http_proxy`: For caching files retrieved over HTTP.
- `do_prune`: During `setup.sh` whether to do a `docker prune images -f` to clear out any unused/untagged images.

# Copyright etc

All copyrights and trademarks belong to their respective owners.

The code in this repository is freely available for anyone to use or modify without any restriction, though no warranty or suitability or fitness for any use is given.
