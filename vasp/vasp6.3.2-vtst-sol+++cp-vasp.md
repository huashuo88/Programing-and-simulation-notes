好的，我将把您提供的第三个文件（CP-VASP 安装说明）与之前的两份内容（VASP 6.3.2 + VTST 编译，以及 VASPsol++ 安装说明）合并成一份完整的安装指南，并整理为适合上传 GitHub 的 Markdown 格式。

```markdown
# VASP 6.3.2 + VTST + VASPsol++ + CP‑VASP 完整安装指南

## 一、VASP 6.3.2 安装

参考链接：<https://www.bilibili.com/opus/682064175698018357>

VASP（Vienna Ab-initio Simulation Package）是进行电子结构计算以及分子动力学模拟的软件包。以下安装步骤在 Parallels Desktop 17 虚拟机中的 Ubuntu 20.04.2 系统上完成。

### Step1：安装 Ubuntu 20.04.2
直接通过 PD17 安装默认的虚拟机即可（VASP 官网推荐 18.xx，此处使用 20.04.2）。

### Step2：更新系统 & 安装编译器
1. 选择最快的下载源：`select best server`
2. 更新系统
   ```bash
   sudo apt update
   sudo apt upgrade
   ```
3. 安装编译所需包
   ```bash
   sudo apt install build-essential
   ```
4. 安装 gcc, g++, gfortran（若未安装）
   ```bash
   sudo apt install gcc      # g++、gfortran 同理
   ```

### Step3：安装 Intel oneAPI 编译器
访问 Intel 官网：[oneAPI Toolkits](https://www.intel.cn/content/www/cn/zh/developer/tools/oneapi/toolkits.html#gs.5kk0bn)

#### 1. 安装 Intel oneAPI Base Toolkit
使用命令行安装（注意修正命令）：
```bash
sudo sh ./l_BaseKit_p_2022.2.0.262.sh
```
按引导安装，只选择 MKL 即可。

#### 2. 安装 Intel oneAPI HPC Toolkit
类似方式，选择 Fortran、C++、MPI Library 安装。

#### 3. 声明环境（单次有效）
```bash
source /opt/intel/oneapi/setvars.sh
```

#### 4. 验证安装
使用 `which icc ifort mpirun mpiifort` 等命令查看路径。

### Step4：安装 VASP
解压 VASP 安装包后：
1. 配置 makefile
   ```bash
   cp arch/makefile.include.linux_intel makefile.include
   ```
   编辑 `makefile.include`，将 `MKLROOT =` 后面的内容删除。
2. 编译 MKL 接口
   ```bash
   cd /opt/intel/oneapi/mkl/2022.1.0/interfaces/fftw3xf
   sudo su
   source /opt/intel/oneapi/setvars.sh
   make libintel64
   exit
   ```

### Step5：编译 VASP
在 VASP 根目录执行：
```bash
make all
```
编译耗时约几十分钟到一小时。

### Step6：测试
```bash
make test
```
若部分测试失败，在 `~/.bashrc` 中添加：
```bash
ulimit -s unlimited
ulimit -c unlimited
```
重新 `make test`，全部通过。

### Step7：运行 VASP
编译后 `bin/` 目录下生成三个可执行文件。添加环境变量：
```bash
PATH=/home/parallels/vasp/vasp6.3.0/vasp.6.3.0/build/std:$PATH   # 按实际路径修改
source ~/.bashrc
```
并行运行命令：
```bash
mpirun -np 4 vasp
```
**Tips**：将 `source /opt/intel/oneapi/setvars.sh` 写入 `~/.bashrc` 避免每次手动声明。

---

## 二、VTST 编译

参考链接：<https://zhuanlan.zhihu.com/p/8591076280>

### 1. 下载 VTST 代码
```bash
tar -zxvf vtstcode-204.tgz
```

### 2. 修改 VASP 源码
进入 `vasp.6.3.2/src/`：
- 备份 `main.F` 和 `.objects`
- 编辑 `main.F`：
  - 第 3542 行附近，在 `CHAIN_FORCE` 调用中加入 `TSIF,`
  - 第 940 行附近，将 `IF (LCHAIN) CALL chain_init(...)` 改为 `CALL chain_init(...)`
- 编辑 `.objects`，在 `chain.o` 前面添加 VTST 所需的目标文件：
  ```
  bfgs.o \
  dynmat.o \
  instanton.o \
  lbfgs.o \
  sd.o \
  cg.o \
  dimer.o \
  bbm.o \
  fire.o \
  lanczos.o \
  neb.o \
  qm.o \
  pyamff_fortran/*.o \
  ml_pyamff.o \
  opt.o \
  ```
- 复制 `vtstcode-204/vtstcode6.3/` 下所有文件到 `vasp/src/`
- 修改 makefile 中的 `LIB` 变量：
  ```
  LIB = lib parser pyamff_fortran
  ```
  并添加依赖：`dependencies: sources libs`

### 3. 重新编译
```bash
make all
```

参考：<http://bbs.keinsci.com/thread-46112-1-1.html>

---

## 三、VASPsol++ 安装

> 以下内容来自 VASPsol++ 官方文档

VASPsol++ 主要实现于单个 Fortran 文件 `solvation.F`，同时需要针对特定 VASP 版本应用补丁文件（需邮件联系 Craig Plaisance 获取）。当前可用的补丁：
- VASP 5.4.4 (带/不带 VTST)
- VASP 6.3.2 (带/不带 VTST)

### 安装步骤
假设 VASP 源码目录为 `<VASP_SRC>`，已配置好 makefile。

1. 从最新 release 的 “Assets” 下载源码包（使用 `wget`），解压：
   ```bash
   tar -xzf vaspsol-pp-<version>.tar.gz
   ```
   解压后目录记为 `<VASPPSOL_SRC>`。

2. 复制 `solvation.F`：
   ```bash
   cp <VASPPSOL_SRC>/src/solvation.F <VASP_SRC>/src/
   ```

3. 复制对应补丁文件：
   - 若使用 VTST：`vaspsol++-vtst-vasp_<version>.patch`
   - 若不使用 VTST：`vaspsol++-vasp_<version>.patch`
   将补丁文件复制到 `<VASP_SRC>/`

4. 应用补丁：
   ```bash
   cd <VASP_SRC>
   patch -p1 < 补丁文件名
   ```

5. 正常编译 VASP（推荐使用 Intel Fortran 编译器）。

---

## 四、CP‑VASP 安装

> 以下内容来自 CP‑VASP 文档，适用于 VASP 6.x

CP‑VASP 需要先安装 VASPsol 或 VASPsol++：
- VASPsol: <https://github.com/henniggroup/VASPsol/tree/master>
- VASPsol++: <https://gitlab.com/cplaisance/vaspsol_pp>

兼容性说明：
- 使用 **VASPsol++** 时，CP‑VASP 适用于 VASP 6.3
- 使用 **VASPsol** 时，CP‑VASP 适用于 VASP 6.4

补丁文件对应：
- VASPsol++ 使用 `cp-vaspsol++.patch`
- VASPsol 使用 `cp-vaspsol.patch`

### 安装步骤
1. 进入 VASP 文件夹的 `/src` 目录。
2. 将对应的补丁文件复制到该目录。
3. 执行打补丁命令（以 VASP 6.3 + VASPsol++ 为例）：
   ```bash
   patch -p0 < cp-vaspsol++.patch
   ```
   - VASPsol++ 补丁会修改 `main.F` 和 `reader.F`
   - VASPsol 补丁会修改 `main.F`、`pot.F`、`reader.F` 和 `solvation.F`
4. `.objects` 文件无需手动编辑。
5. 重新编译 VASP：
   ```bash
   make all
   ```
   通常不需要 `make veryclean`，但若编译出错可尝试执行。

---

## 附录：测试与运行
- 测试 VASP：`make test`（若测试失败可检查 `ulimit` 设置）
- 并行运行：`mpirun -np N vasp`（N 为核心数）
- 环境变量：建议将 Intel oneAPI 环境声明和 VASP 路径写入 `~/.bashrc`

---

*本指南整合了 VASP 6.3.2 基础安装、VTST 插件、VASPsol++ 溶剂模型以及 CP‑VASP 恒电势方法的完整流程。*
```

