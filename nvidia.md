### Verify You Have a CUDA-Capable GPU

To verify that your GPU is CUDA-capable, go to your distribution's equivalent of System Properties, or, from the command line, enter:

```
lspci | grep -i nvidia
```

If you do not see any settings, update the PCI hardware database that Linux maintains by entering `update-pciids` (generally found in `/sbin`) at the command line and rerun the previous `lspci` command.

If your graphics card is from NVIDIA and it is listed in <https://developer.nvidia.com/cuda-gpus>, your GPU is CUDA-capable.

### Verify the System has the Correct Kernel Headers and Development Packages Installed

#### Fedora/RHEL8/CentOS8

The kernel headers and development packages for the currently running kernel can be installed with:

```
sudo dnf install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```

#### OpenSUSE/SLES

The kernel development packages for the currently running kernel can be installed with:

```
sudo zypper install -y kernel-variant-devel=version
```

To run the above command, you will need the variant and version of the currently running kernel. Use the output of the uname command to determine the currently running kernel's variant and version:

```
uname -r
3.16.6-2-default
```

In the above example, the variant is `default` and version is `3.16.6-2`.
The kernel development packages for the default kernel variant can be installed with:

```
sudo zypper install -y kernel-default-devel=$(uname -r | sed 's/\-default//')
```

#### Ubuntu

The kernel headers and development packages for the currently running kernel can be installed with:

```
sudo apt install linux-headers-$(uname -r)
```

传统上，安装 NVIDIA Driver 和 CUDA Toolkit 的步骤是分开的，但实际上我们可以直接安装 CUDA Toolkit，系统将自动安装与其版本匹配的 NVIDIA Driver。下面我们讲述安装 CUDA Toolkit 的方法。

#### 1.单独安装NVIDIA驱动

```
sudo ./NVIDIA-Linux-xxx.run -no-x-check -no-nouveau-check -no-opengl-files
sudo sh NVIDIA-Linux-xxx.run -no-x-check -no-nouveau-check -no-opengl-files
```

只有禁用opengl这样安装才不会出现循环登陆的问题

```
-no-x-check：安装驱动时关闭X服务
-no-nouveau-check：安装驱动时禁用nouveau
-no-opengl-files：只安装驱动文件，不安装OpenGL文件
```

#### 2.Nvidia驱动与CUDA一起安装

安装CUDA时，通过选项是否安装驱动来安装Nvidia驱动。或者ppa源安装，安装完Nvidia驱动后nvidia-smi查询看一下多匹配的cuda版本。

```
sudo sh cuda_x.x.x_linux.run --no-opengl-libs
```

双显卡的注意，遇到提示是否安装openGL ，选择no。

在执行`sudo ./cuda_xxx_linux.run --no-opengl-libs`后，安装过程会提示是否安装driver？选择y “是”，进行GPU驱动和CUDA一起安装。

如果电脑是双显，且主显是非NVIDIA的GPU在工作需要选择no，否则可以yes，其他都选择yes或者默认即可。

如果电脑是双显卡，且在这一步选择了yes，那么你极有可能安装完CUDA之后，重启图形化界面后遇到登录界面循环问题：输入密码后又跳回密码输入界面。

这是因为电脑是双显，而且用来显示的那块GPU不是NVIDIA，则OpenGL Libraries就不应该安装，否则你正在使用的那块GPU（非NVIDIA的GPU）的OpenGL Libraries会被覆盖，然后GUI就无法工作了。

加`--no-opengl-libs`选项或者先不加`--no-opengl-libs`选项，在后面的交互模式下手动选择

sudo sh cuda_10.1.168_418.67_linux.run --no-opengl-libs

执行命令后，稍等一会儿，会出现交互界面，输入 accept，选择要安装的选项进行安装

注意：若没加参数 --no-opengl-libs，后面会遇到循环登陆的问题，原因就是 NVIDIA显卡 的 OpenGL 库覆盖了当前 Intel 显卡的库，解决办法是再登入到文本命令行模式，卸载 cuda 和 NVIDIA驱动，再按正确的步骤重新安装

### Installing cuDNN and NCCL

We recommend installing cuDNN and NCCL using binary packages (i.e., using apt or yum) provided by NVIDIA.

If you want to install tar-gz version of cuDNN and NCCL, we recommend installing it under the CUDA_PATH directory. For example, if you are using Ubuntu, copy *.h files to include directory and *.so* files to lib64 directory:

```
cp /path/to/cudnn.h $CUDA_PATH/include
cp /path/to/libcudnn.so* $CUDA_PATH/lib64
```

The destination directories depend on your environment.

If you want to use cuDNN or NCCL installed in another directory, please use CFLAGS, LDFLAGS and LD_LIBRARY_PATH environment variables before installing CuPy:

```
export CFLAGS=-I/path/to/cudnn/include
export LDFLAGS=-L/path/to/cudnn/lib
export LD_LIBRARY_PATH=/path/to/cudnn/lib:$LD_LIBRARY_PATH
```

### Working with Custom CUDA Installation

If you have installed CUDA on the non-default directory or multiple CUDA versions on the same host, you may need to manually specify the CUDA installation directory to be used by CuPy.

CuPy uses the first CUDA installation directory found by the following order.

1.`CUDA_PATH` environment variable.

2.The parent directory of `nvcc` command.

3.`/usr/local/cuda`

Note: CUDA installation discovery is also performed at runtime using the rule above. Depending on your system configuration, you may also need to set `LD_LIBRARY_PATH` environment variable to `$CUDA_PATH/lib64` at runtime.

### Fedora

Perform the pre-installation actions.

Address custom xorg.conf, if applicable

The driver relies on an automatically generated xorg.conf file at `/etc/X11/xorg.conf`. If a custom-built xorg.conf file is present, this functionality will be disabled and the driver may not work. You can try removing the existing xorg.conf file, or adding the contents of `/etc/X11/xorg.conf.d/00-nvidia.conf` to the xorg.conf file. The xorg.conf file will most likely need manual tweaking for systems with a non-trivial GPU configuration.

Install repository meta-data

```
sudo rpm --install cuda-repo-<distro>-<version>.<architecture>.rpm
```

Clean DNF repository cache

```
sudo dnf clean expire-cache
```

Install CUDA

```
sudo dnf module install nvidia-driver:latest-dkms
sudo dnf install cuda
```

The CUDA driver installation may fail if the RPMFusion non-free repository is enabled. In this case, CUDA installations should temporarily disable the RPMFusion non-free repository:

```
sudo dnf --disablerepo="rpmfusion-nonfree*" install cuda
```

It may be necessary to rebuild the grub configuration files, particularly if you use a non-default partition scheme. If so, then run this below command, and reboot the system:

```
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

Remember to reboot the system.
Add libcuda.so symbolic link, if necessary
The libcuda.so library is installed in the `/usr/lib{,64}/nvidia` directory. For pre-existing projects which use libcuda.so, it may be useful to add a symbolic link from libcuda.so in the `/usr/lib{,64}` directory.

Perform the post-installation actions.

### Fedora 33

- local local installer

```
wget https://developer.download.nvidia.com/compute/cuda/11.2.0/local_installers/cuda-repo-fedora33-11-2-local-11.2.0_460.27.04-1.x86_64.rpm
sudo rpm -i cuda-repo-fedora33-11-2-local-11.2.0_460.27.04-1.x86_64.rpm
sudo dnf clean all
sudo dnf module install nvidia-driver:latest-dkms
sudo dnf install cuda
```

- local network installer

```
sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/fedora33/x86_64/cuda-fedora33.repo
sudo dnf clean all
sudo dnf module install nvidia-driver:latest-dkms
sudo dnf install cuda
```

### Debian 10

Perform the pre-installation actions.

Install repository meta-data

```
wget https://developer.download.nvidia.com/compute/cuda/11.2.0/local_installers/cuda-repo-debian10-11-2-local_11.2.0-460.27.04-1_amd64.deb
sudo dpkg -i cuda-repo-debian10-11-2-local_11.2.0-460.27.04-1_amd64.deb
```

Installing the CUDA public GPG key

- When installing using the local repo:

```
sudo apt-key add /var/cuda-repo-debian10-11-2-local/7fa2af80.pub
```

- When installing using the network repo:

```
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/debian10/x86_64/7fa2af80.pub
```

Enable the contrib repository

- When installing using the local repo:

```
sudo add-apt-repository contrib
```

- When installing using the network repo:

```
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/debian10/x86_64/ /"
sudo add-apt-repository contrib
```

Update the apt repository cache

```
sudo apt update
```

Install CUDA

```
sudo apt install cuda
```

Perform the post-installation actions.

- local deb installer

```
wget https://developer.download.nvidia.com/compute/cuda/11.2.0/local_installers/cuda-repo-debian10-11-2-local_11.2.0-460.27.04-1_amd64.deb
sudo dpkg -i cuda-repo-debian10-11-2-local_11.2.0-460.27.04-1_amd64.deb
sudo apt-key add /var/cuda-repo-debian10-11-2-local/7fa2af80.pub
sudo add-apt-repository contrib
sudo apt update
sudo apt install cuda
```

- local network installer

```
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/debian10/x86_64/7fa2af80.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/debian10/x86_64/ /"
sudo add-apt-repository contrib
sudo apt update
sudo apt install cuda
```

### Disabling Nouveau

To install the Display Driver, the Nouveau drivers must first be disabled. Each distribution of Linux has a different method for disabling Nouveau.

The Nouveau drivers are loaded if the following command prints anything:

```
lsmod | grep nouveau
```

#### Fedora

Create a file at `/usr/lib/modprobe.d/blacklist-nouveau.conf` with the following contents:

```
blacklist nouveau
options nouveau modeset=0
```

Regenerate the kernel initramfs:

```
sudo dracut --force
```

Run the below command:

```
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

Reboot the system.

#### OpenSUSE

Create a file at `/etc/modprobe.d/blacklist-nouveau.conf` with the following contents:

```
blacklist nouveau
options nouveau modeset=0
```

Regenerate the kernel initrd:

```
sudo /sbin/mkinitrd
```

#### Ubuntu

Create a file at `/etc/modprobe.d/blacklist-nouveau.conf` with the following contents:

```
blacklist nouveau
options nouveau modeset=0
```

Regenerate the kernel initramfs:

```
sudo update-initramfs -u
```

#### Debian

Create a file at `/etc/modprobe.d/blacklist-nouveau.conf` with the following contents:

```
blacklist nouveau
options nouveau modeset=0
```

Regenerate the kernel initramfs:

```
sudo update-initramfs -u
```

### Remove CUDA Toolkit and Driver

Follow the below steps to properly uninstall the CUDA Toolkit and NVIDIA Drivers from your system. These steps will ensure that the uninstallation will be clean.

- Fedora

To remove CUDA Toolkit:

```
sudo dnf remove "cuda*" "*cublas*" "*cufft*" "*curand*" "*cusolver*" "*cusparse*" "*npp*" "*nvjpeg*" "nsight*"
```

To remove 3rd party NVIDIA Drivers:

```
sudo dnf remove "*nvidia*"
```

To remove NVIDIA Drivers:

```
sudo dnf remove nvidia-driver
```

To reset the module stream:

```
sudo dnf module reset nvidia-driver
```

- OpenSUSE/SLES

To remove CUDA Toolkit:

```
sudo zypper remove "cuda*" "*cublas*" "*cufft*" "*curand*" "*cusolver*" "*cusparse*" "*npp*" "*nvjpeg*" "nsight*"
```

To remove NVIDIA Drivers:

```
sudo zypper remove "*nvidia*"
```

- Debian and Ubuntu
To remove CUDA Toolkit:

```
sudo apt --purge remove "*cublas*" "*cufft*" "*curand*" \
 "*cusolver*" "*cusparse*" "*npp*" "*nvjpeg*" "cuda*" "nsight*"
```

To remove NVIDIA Drivers:

```
sudo apt --purge remove "*nvidia*"
```

To clean up the uninstall:

```
sudo apt autoremove
```
