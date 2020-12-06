Requirements

- Fortran, C and C++ compilers.
- Message Passing Interface (MPI)
- Numerical libraries: `FFTW`, `BLAS`, `LAPACK`, and `ScaLAPACK`.
- When compiling with PGI Compilers and Tool: [QD library](https://github.com/scibuilder/QD)
- And for the GPU ports (CUDA and OpenACC) of VASP: NVIDIA's CUDA toolkit

install Intel Parallel Studio XE

即parallel studio xe 2020，里面有十几个安装包，建议都安；
直接把解压文件夹中的install.sh文件拉入终端程序按“Enter”即可开始安装；
安装完成后设置环境变量。

```
nano ~/.bashrc
--------------
source /path_to_folder/bin/ifortvars.sh intel64
source /path_to_folder/mklvars.sh intel64
```

source ~/.bashrc

上面的`path_to_folder`为相应文件的安装路径，64位用`intel64`,32位用`ia32`；最后一行命令使修改生效。

验证

```
which ifort
```

如果显示路径则安装成功。
