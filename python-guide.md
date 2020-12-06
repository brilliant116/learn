
install miniconda

```
wget https://mirrors.bfsu.edu.cn/anaconda/miniconda/Miniconda3-py38_4.8.3-Linux-x86_64.sh
```

### Miniconda

```
conda config --set auto_activate_base false
```

### uninstall miniconda

1.remove the entire miniconda install directory

```
rm -rf ~/miniconda
```

2.optional:

```
rm -rf ~/.condarc ~/.conda ~/.continuum
```

set BFSU conda mirror

```
~/.condarc
----------
channels:
  - defaults
show_channel_urls: true
channel_alias: https://mirrors.bfsu.edu.cn/anaconda
default_channels:
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/main
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/free
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/r
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/pro
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.bfsu.edu.cn/anaconda/cloud
  msys2: https://mirrors.bfsu.edu.cn/anaconda/cloud
  bioconda: https://mirrors.bfsu.edu.cn/anaconda/cloud
  menpo: https://mirrors.bfsu.edu.cn/anaconda/cloud
  pytorch: https://mirrors.bfsu.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.bfsu.edu.cn/anaconda/cloud
```

清除索引缓存，保证用的是镜像站提供的索引

```
conda clean -i
```

```
conda update conda
```

create a new environment named `python_env` and install packages

```
conda create --name python_env package_name
conda create -n python_env package_name
```

```
conda activate python_env
```

list all the environment

```
conda info --envs
conda info -e
```

change the current environment to the default

```
conda activate
```

create a new environment with a specific Python version

```
conda create -n python_env python=3.8
```

update all packages in the environment. Before it, `conda update conda` is recommended.

```
conda update --all
```

### remove an environment

```
conda remove -n env_name --all
# or
conda env remove -n env_name
```

verify the environment was removed

```
conda info -e
```
