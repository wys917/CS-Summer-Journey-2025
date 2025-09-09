# 迷你计算集群搭建及HPL性能测试
!!! info "提示"
    具体细节请点击下方按钮进入我的GitHub仓库查看
    
    [-> GitHub 源码](https://github.com/wys917/My_HPC/tree/main/迷你计算集群搭建及HPL性能测试){ .md-button .md-button--primary }

**实验者：** 苏易文 (学号: 3240103466)  
**日期：** 2025年6月  

## 📌 项目概述

本实验通过VMware虚拟机从零搭建了一个含4个计算节点的迷你高性能计算集群，并进行了全面的HPL (High-Performance Linpack) 基准测试。项目涵盖了从集群架构设计、软件环境搭建、并行计算框架配置到性能优化的完整实践流程。

## 🎯 实验目标

- 掌握高性能计算集群的搭建流程
- 理解并行计算框架OpenMPI的工作原理
- 学习HPL基准测试的配置与优化
- 分析不同参数对计算性能的影响
- 培养系统性能调优的实践能力

## ⚙️ 技术栈与环境

### 硬件环境
- **虚拟化平台**: VMware Workstation Pro
- **集群规模**: 4个计算节点
- **节点配置**: 每节点2核CPU + 2GB内存
- **网络配置**: VMware NAT模式，静态IP分配

### 软件环境
- **操作系统**: Debian 12.2.0
- **编译器**: GCC 12.2.0, GFortran
- **并行框架**: OpenMPI 5.0.3
- **数学库**: BLAS 3.12.0, CBLAS
- **基准测试**: HPL 2.3
- **开发工具**: Make, Python 3.x, matplotlib

## 🏗️ 项目结构

```
迷你计算集群搭建及HPL性能测试/
├── README.md                    # 项目说明文档
├── report/                      # 实验报告
│   ├── report.md               # Markdown格式报告
│   ├── 《迷你计算集群搭建及HPL性能测试》项目报告.pdf
│   └── image.png               # 报告中的图表
├── code/                       # 数据可视化代码
│   ├── hpl_n_vs_gflops_plot.py       # N vs Gflops 性能图表
│   ├── plot_nb_vs_gflops.py          # NB vs Gflops 性能图表
│   └── plot_n_and_nb_vs_gflops.py    # 对比图表
├── result/                     # 测试结果
│   └── hpl_output.txt          # HPL完整测试输出
└── assets/                     # 生成的图表资源
    ├── hpl_n_vs_gflops_plot.png
    ├── plot_nb_vs_gflops.png
    └── compare.png
```

## 🚀 核心成果

### 最优性能指标
- **峰值性能**: **3.1339 Gflops**
- **最优配置**: N=2000, NB=224, P×Q=2×5
- **运行时间**: 1.70秒
- **测试通过率**: 100% (72/72测试用例通过)

### 性能分析结果

#### 1. 问题规模N的影响分析
测试配置: 固定NB=128, P=2, Q=5

| 问题规模 N | 性能 (Gflops) | 性能提升 |
|-----------|--------------|---------|
| 500       | 0.24072      | 基准     |
| 1000      | 0.85795      | +256%   |
| 1500      | 1.6817       | +96%    |
| 2000      | 2.5377       | +51%    |

**分析**: 随着问题规模增大，计算密度提高，通信开销相对减少，性能显著提升。

#### 2. 分块大小NB的影响分析
测试配置: 固定N=2000, P=2, Q=5

| 分块大小 NB | 性能 (Gflops) | 相对最优性能 |
|-----------|--------------|-------------|
| 32        | 2.0376       | 65.0%       |
| 64        | 2.2678       | 72.4%       |
| 96        | 2.6805       | 85.5%       |
| 128       | 2.5053       | 79.9%       |
| 160       | 2.6498       | 84.6%       |
| 192       | 2.8955       | 92.4%       |
| **224**   | **3.1339**   | **100%**    |
| 256       | 3.0896       | 98.6%       |

**分析**: 分块大小存在最优点，NB=224时达到峰值性能。过小的分块导致计算效率低，过大的分块可能影响缓存效率。

## 📊 可视化分析

项目包含三个主要的数据可视化脚本：

### 1. `hpl_n_vs_gflops_plot.py`
- 生成问题规模N与性能的关系图
- 展示计算规模对性能的影响趋势

### 2. `plot_nb_vs_gflops.py`  
- 生成分块大小NB与性能的关系图
- 识别最优分块大小参数

### 3. `plot_n_and_nb_vs_gflops.py`
- 生成N和NB对比分析图表
- 便于整体性能分析和参数优化

## 🔧 使用方法

### 环境准备
```bash
# 安装Python依赖
pip install matplotlib

# 或使用项目配置的Python环境
python -m pip install matplotlib
```

### 运行可视化脚本
```bash
# 切换到lab1目录
cd lab1

# 生成N vs Gflops图表
python code/hpl_n_vs_gflops_plot.py

# 生成NB vs Gflops图表  
python code/plot_nb_vs_gflops.py

# 生成对比图表
python code/plot_n_and_nb_vs_gflops.py
```

生成的图表将自动保存到 `assets/` 目录下。

## ⚠️ 关键技术挑战与解决方案

### 挑战1: HPL编译链接失败
**问题**: `undefined reference to _gfortran_...` 错误
**原因**: mpicc链接BLAS时缺失Fortran运行库
**解决**: 在Makefile中添加 `-lgfortran` 链接选项

### 挑战2: CBLAS测试程序编译报错
**问题**: testing/目录代码与gfortran/BLAS版本不兼容
**解决**: 跳过测试程序，直接使用生成的 `cblas_LINUX.a` 库

### 挑战3: 集群网络配置
**问题**: SSH免密登录配置和主机名解析
**解决**: 配置 `/etc/hosts` 静态解析，生成并分发SSH密钥

## 📈 性能优化建议

1. **内存配置**: 根据问题规模合理分配内存，避免内存不足影响性能
2. **网络优化**: 使用高速网络互联，减少通信延迟
3. **负载均衡**: 优化进程网格配置(P×Q)，平衡计算与通信
4. **缓存友好**: 选择合适的分块大小，提高缓存命中率

## 📚 学习资源

- [HPL官方文档](http://www.netlib.org/benchmark/hpl/)
- [OpenMPI用户指南](https://www.open-mpi.org/doc/)
- [BLAS/LAPACK参考手册](http://www.netlib.org/lapack/)

## 📧 联系信息

如有问题或建议，请联系：
- **姓名**: 苏易文
- **学号**: 3240103466
- **邮箱**: 3240103466@zju.edu.cn

---

**注**: 本项目仅用于学术研究和教学目的，请遵守相关软件的许可协议。