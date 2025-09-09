# 🔄 基于MPI+OpenMP混合并行的BiCGSTAB算法实现与调优

[![Language](https://img.shields.io/badge/Language-C%2B%2B-blue.svg)](https://en.cppreference.com/)
[![OpenMP](https://img.shields.io/badge/OpenMP-4.5+-green.svg)](https://www.openmp.org/)
[![MPI](https://img.shields.io/badge/MPI-OpenMPI%204.0+-orange.svg)](https://www.open-mpi.org/)

!!! info "提示"
    具体细节请点击下方按钮进入我的GitHub仓库查看
    
    [-> GitHub 源码](https://github.com/wys917/My_HPC/tree/main/基于MPI+OpenMP混合并行的BiCGSTAB算法实现与调优){ .md-button .md-button--primary }
本项目实现了BiCGSTAB（双共轭梯度稳定化）迭代求解器的多层次并行优化，通过系统性的性能分析和优化策略，探索了从串行到OpenMP、MPI以及混合并行模型的完整性能调优过程。

## 📊 核心成果

| 优化阶段 | 配置 | 运行时间 | 加速比 | 性能特点 |
|----------|------|----------|--------|----------|
| **串行版本** | 1核心 | 379.384s | 1.0x | 初始基准 |
| **编译器优化** | 1核心 (-O2) | 93.877s | 4.04x | 编译器自动优化 |
| **OpenMP并行** | 96线程 | **3.488s** | **108.8x** | 🏆 最优性能 |
| **MPI+OpenMP混合** | 2进程×48线程 | 12.5s | 30.4x | 通信开销明显 |

## 🎯 项目亮点

- **🔍 系统性性能分析**: 使用Intel VTune Profiler定位性能瓶颈，99%计算时间集中在`gemv`函数
- **⚡ 多层次并行优化**: 从编译器优化到OpenMP、MPI及混合并行的完整实践
- **📈 量化性能评估**: 详细的可扩展性分析和并行效率计算
- **🛠️ 实际工程挑战**: 解决了MPI环境配置、SLURM调度等实际部署问题

## 🏗️ 项目结构

```
基于MPI+OpenMP混合并行的BiCGSTAB算法实现与调优/
├── README.md                    # 项目说明文档
├── assets/                      # 图表和可视化结果
│   ├── image-20250909214805195.png  # VTune性能分析-初始版本
│   ├── image-20250909214825105.png  # VTune性能分析-编译器优化后
│   ├── e4290bf6-2957-4838-85fb-46c52c9d77d2.png  # OpenMP可扩展性分析图表
│   ├── image-20250909214914518.png  # OpenMP性能对比
│   └── image-20250909214930548.png  # ITAC通信分析报告
├── code/                        # 源代码和构建配置
│   ├── src/                     # C++源代码
│   │   ├── main.cpp            # 主程序入口
│   │   ├── judger.cpp          # 性能测试模块
│   │   └── bicgstab/           # BiCGSTAB算法实现
│   ├── include/                 # 头文件
│   │   └── judger.h
│   ├── CMakeLists.txt          # CMake构建配置
│   ├── compile.sh              # 编译脚本
│   ├── run.sh                  # 运行脚本
│   ├── run_benchmark.sh        # 性能基准测试
│   ├── run_mpi.sh             # MPI运行脚本
│   └── run_vtune.sh           # VTune性能分析脚本
├── report/                      # 实验报告
│   └── 基于MPI+OpenMP混合并行的BiCGSTAB算法实现与调优.md
└── result/                      # 实验结果和数据
    ├── data/                   # 测试数据集
    │   ├── case_2001.bin      # 2001维稀疏矩阵
    │   ├── case_4001.bin      # 4001维稀疏矩阵
    │   └── case_6001.bin      # 6001维稀疏矩阵
    ├── reports/               # VTune性能分析报告
    └── reports_time/          # 运行时间统计结果
```

## ⚙️ 技术栈

- **编程语言**: C++14/17
- **并行编程模型**: OpenMP 4.5+, MPI (OpenMPI)
- **构建系统**: CMake 3.15+
- **性能分析工具**: Intel VTune Profiler, Intel ITAC
- **作业调度系统**: SLURM
- **编译器**: GCC 9+ 或 Intel ICC

## 🚀 快速开始

### 环境要求

```bash
# 必需依赖
sudo apt-get install build-essential cmake
sudo apt-get install libopenmpi-dev openmpi-bin
sudo apt-get install libomp-dev

# 可选：性能分析工具
# Intel VTune Profiler (需要单独安装)
# Intel ITAC (Intel MPI工具链的一部分)
```

### 编译与运行

```bash
# 编译项目
cd code
mkdir build && cd build
cmake ..
make

# 串行运行
./bicgstab_solver

# OpenMP并行运行 (96线程)
export OMP_NUM_THREADS=96
./bicgstab_solver

# MPI运行 (16进程)
mpirun -np 16 ./bicgstab_solver

# MPI+OpenMP混合运行 (2进程 × 48线程)
export OMP_NUM_THREADS=48
mpirun -np 2 ./bicgstab_solver
```

### 性能基准测试

```bash
# 运行完整的性能基准测试
cd code
./run_benchmark.sh

# VTune性能分析
./run_vtune.sh

# SLURM集群环境运行
sbatch run_mpi.sh
```

## 📈 性能分析结果

### OpenMP可扩展性分析

| 线程数 | 运行时间 | 加速比 | 并行效率 |
|--------|----------|--------|----------|
| 1 | 93.076s | 1.00x | 100.0% |
| 8 | 12.423s | 7.49x | 93.6% |
| 16 | 6.559s | 14.60x | 91.3% |
| 32 | 4.626s | 20.12x | 62.9% |
| 64 | 3.692s | 25.21x | 39.4% |
| 96 | **3.488s** | **26.04x** | 27.1% |

### 关键发现

1. **🎯 性能瓶颈定位**: 99%的计算时间集中在矩阵向量乘法(`gemv`)函数
2. **🚀 OpenMP优势**: 在单节点内实现近似线性加速，最高达到108.8倍加速
3. **🌐 通信开销挑战**: MPI通信成为性能瓶颈，`MPI_Allgatherv`耗时巨大
4. **⚖️ 混合模型权衡**: "少进程、多线程"策略缓解通信压力但无法超越纯OpenMP

## 🔬 算法实现细节

### BiCGSTAB核心算法
```cpp
// OpenMP并行化的矩阵向量乘法
#pragma omp parallel for
for (int i = 0; i < n; i++) {
    double sum = 0.0;  // 线程私有累加器
    for (int j = row_ptr[i]; j < row_ptr[i+1]; j++) {
        sum += values[j] * x[col_idx[j]];
    }
    y[i] = sum;
}
```

### MPI分布式策略
- **数据分布**: 按行分解策略，使用`MPI_Scatterv`分发数据
- **全局通信**: `MPI_Allreduce`实现点积归约，`MPI_Allgatherv`收集向量
- **负载均衡**: 根据稀疏矩阵非零元分布进行动态负载分配

## 📚 BiCGSTAB算法数学基础

### 算法描述

BiCGSTAB (Biconjugate Gradient Stabilized) 是一种用于求解非对称线性系统 $Ax = b$ 的Krylov子空间迭代方法。

#### 算法步骤:

1. $r_0 = b - \mathbf{A}x_0$
2. Choose an arbitrary vector $\hat{r_0}$ such that $(\hat{r_0}, r_0)\ne 0$, eg. $\hat{r_0} = r_0$
3. $\rho_0 = (\hat{r}_0, r_0)$
4. $p_0 = r_0$
5. For $i = 1, 2, 3, \ldots$
    1. $y = \mathbf{M^{-1}} p_{i-1}$
    2. $v = \mathbf{A}{y}$
    3. $\alpha = \rho_{i-1} / (\hat{r}_0, v)$
    4. $h = x_{i-1} + \alpha y$
    5. $s = r_{i-1} - \alpha v$
    6. If $s$ is within the accuracy tolerance then $x_{i-1} = h$ and quit.
    7. $z = \mathbf{M^{-1}} s$
    8. $t = \mathbf{A}z$
    9. $\omega = (t, s) / (t, t)$
    10. $x_i = h + \omega z$
    11. $r_i = s - \omega t$
    12. If $r_i$ is within the accuracy tolerance then quit.
    13. $\rho_i = (\hat{r}_0, r_i)$
    14. $\beta = (\rho_i / \rho_{i-1}) (\alpha / \omega)$
    15. $p_i = r_i + \beta (p_{i-1} - \omega v)$

## 🛠️ 开发环境配置

### Intel VTune Profiler集成
```bash
# 使用VTune进行性能分析
vtune -collect hotspots -app-args ./bicgstab_solver
vtune -report hotspots -r r000hs
```

### SLURM作业配置
```bash
#!/bin/bash
#SBATCH --job-name=bicgstab_hybrid
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=48
#SBATCH --time=01:00:00

export OMP_NUM_THREADS=48
mpirun -np 2 ./bicgstab_solver
```



**数据文件**：
- 测试数据位于 `/river/hpc101/2025/lab4/data`
- 可通过 `ln -s /river/hpc101/2025/lab4/data data` 引用
- 提交时请勿包含数据文件



## 📄 许可证

本项目采用 MIT License 开源协议。

---

**作者**: 苏易文 (学号: 3240103466)  
**课程**: 高性能计算 - 浙江大学  
**完成时间**: 2025年9月
