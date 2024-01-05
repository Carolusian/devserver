# Develop Server Environment Hosted on HomeLab

## Linux distribution

- Choose `Manjaro` Arch based linux distrbution and more friendly for starters
- See: https://bishwarup-paul.medium.com/why-i-love-manjaro-144841982ece
- The installation is quite straight forward with `iso` file
- Can update the network address to manual later
- Common commands:
  - `sudo pacman -Syu`
  - `sudo pacman -Ql packagename`

## RDP options

- No good option for the time being, tried `nomachine`, `anydesk`, `tigervnc`, all without luck

## Code Editor

- `code-server`: basically `vscode` via browser
- `pamac build code-server`
- `settings.json` location: `/home/cs/.local/share/code-server/User/settings.json`

## Source code directory

- Mount NFS directory: `/mnt/sample-pool/nfs`
- Use systemd to auto mount on startup
- `sudo systemd-escape -p --suffix=mount '/mnt/nfs'`: Where= must be matched to the unit name
- `sudo systemctl enable mnt-nfs.mount --now`

```
# /etc/systemd/system/mnt-nfs.mount
[Unit]
Description=NFS from TrueNAS
After=network.target

[Mount]
What=192.168.x.x:/mnt/path/nfs
Where=/mnt/nfs
Type=nfs
Options=_netdev,auto

[Install]
WantedBy=multi-user.target
```

## Setup static IP

- https://wiki.manjaro.org/index.php/Networking

## Disable sleep and hibernate

```
sudo systemctl mask suspend.target hibernate.target sleep.target
sudo systemctl unmask suspend.target hibernate.target sleep.target
```

## Install `rstudio-server`

```
sudo pamac build rstudio-server-bin
sudo useradd --system rstudio-server
sudo rstudio-server verify-installation
sudo systemctl start rstudio-server
```

In rstudio can use `py_config()` to check who the python path is determined, e.g.

- by conda
- by environment variable `RETICULATE_PYTHON_FALLBACK`

Then login with your default linux user

## Install tensflow with CUDA support

```
sudo pacman -S tensorflow-cuda python-tensorflow-cuda
```

Sometimes, you may want to benchmark again CPU, can use `Sys.setenv(CUDA_VISIBLE_DEVICES = -1)` to disable CUDA support

NOTE: 

- ArchLinux update is rolling based, which makes the dependencies in relation to CUDA unstable; in this case, conda is preferred. 
- However, conda doesn't work well with tensorflow[and-cuda] for 2.11-2.15
- So, elder version tensorflow-gpu==2.10.0 is more stable, can test again when tensorflow 2.16 is released

Quick start with `conda`:

```
conda_create(env_name, python_version = "3.10")  # python 3.10 is latest version compatible with tensorflow 2.10
use_condaenv(condaenv = env_name)
reticulate::conda_install(envname = env_name, packages = c("tensorflow-gpu==2.10.0"))
reticulate::import("tensorflow")
tf$sysconfig$get_build_info()
```

`reticulate`'s conda can be used in shell after `init`, e.g. `~/.local/share/r-miniconda/bin/conda init zsh`

## Install torch with CUDA support

- In rstudio, can combine `use_virtualenv` and `py_install(..., pip = TRUE)` to install pytorch with CUDA support
- then, set CUDA version via `sys.setenv` accordingly
- then, install mlverse/torch
- If certain `.so` files cannot be found, just use `find` and copy them over

## Free disk space

```
sudo pacman -Sc
sudo pacman -Qdt
sudo pacman -Rns $(pacman -Qtdq)
sudo journalctl --vacuum-size=50M
```
