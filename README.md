# codex_MLIP

面向合金、晶界与异质界面能研究的机器学习原子势（MLIP）工作流。

> **项目状态（2026-11-19）**：本仓库当前仅包含本 README。下文的目录、命令和接口是建议的工作流与规划中的脚本组织方式，并不表示对应脚本已经提交。请在使用前以仓库实际内容和各脚本的 `--help` 为准。

## 目录

- [概述](#概述)
- [研究边界与可复现性](#研究边界与可复现性)
- [支持的势能与上游项目](#支持的势能与上游项目)
- [规划中的仓库结构](#规划中的仓库结构)
- [使用教程（规划工作流）](#使用教程规划工作流)
- [质量控制与界面能后处理](#质量控制与界面能后处理)
- [引用说明](#引用说明)
- [许可证与开源声明](#许可证与开源声明)
- [English summary](#english-summary)

## 概述

本项目旨在为金属及多组元合金建立一条可追踪的计算流程：

1. 生成体相、缺陷、晶界和异质界面结构；
2. 使用密度泛函理论（DFT）计算能量、力和应力，形成训练/验证数据集；
3. 训练或接入机器学习原子势；
4. 在 LAMMPS 中进行结构弛豫、分子动力学（MD）和有限温度采样；
5. 从总能、面积和参考态中计算晶界能、界面能及其收敛性。

目标体系包括 FCC、BCC 和 HCP 金属及其置换/间隙合金。这里的“支持”表示为相应势能预留数据格式、训练入口或 LAMMPS 接口；在具体势能和脚本真正加入仓库前，不应将其视为已实现功能。

### 适用范围

- 合金体相、表面、空位、间隙原子和短程有序结构；
- 对称/非对称倾斜晶界、扭转晶界及晶界弛豫；
- 金属/金属异质界面和相界面；
- 0 K 能量、有限温度采样，以及 LAMMPS 中的动力学性质。

### 不在默认保证范围内

- 超出训练分布的元素组合、压力、温度或相结构；
- 未经收敛测试的界面能绝对值；
- 将模型预测当作 DFT 或实验结果的替代；
- 未核对上游许可证、模型权重许可证和数据集条款的再分发。

## 研究边界与可复现性

每次计算建议记录：

- 元素、晶体结构、晶格常数、组分和超胞尺寸；
- 晶界 CSL/取向关系、界面法向、真空层或周期边界设置；
- DFT 程序、交换-相关泛函、赝势、截断能、k 点、展宽和收敛阈值；
- 训练集版本、数据划分、随机种子、模型超参数和软件版本；
- 势能文件校验和、LAMMPS 版本、时间步长、温控/压控方法；
- 几何弛豫阈值、重复计算和尺寸/厚度收敛结果。

训练集必须覆盖目标体系的局域环境，而不应只包含理想体相。至少应评估能量、力、应力以及晶格常数、弹性常数、缺陷形成能和界面结构等与研究问题相关的量。

## 支持的势能与上游项目

下表列出计划接入或可作为参考的主流开源项目。链接指向原始 GitHub 仓库；许可证仅概括软件仓库当前公开的许可证，模型权重、训练数据、依赖和论文可能有独立条款。使用或再分发前请阅读上游仓库中的完整许可证和 NOTICE 文件。

| 项目 | 主要特点 | 原始 GitHub 仓库 | 上游软件许可证提示 |
| --- | --- | --- | --- |
| **MACE** | 基于等变消息传递/原子簇展开思想的高精度势；适合能量、力和应力拟合 | [ACEsuit/mace](https://github.com/ACEsuit/mace) | MIT（软件仓库；模型与数据另行核对） |
| **NequIP** | E(3) 等变图神经网络；适用于从第一性原理数据学习原子相互作用 | [mir-group/nequip](https://github.com/mir-group/nequip) | MIT（软件仓库） |
| **Allegro** | NequIP 生态中的局域、可扩展等变模型；面向较大体系和长时间 MD | [mir-group/allegro](https://github.com/mir-group/allegro) | MIT（软件仓库） |
| **DeepMD-kit** | Deep Potential 系列训练、评估和部署工具；支持多种元素与 LAMMPS 耦合 | [deepmodeling/deepmd-kit](https://github.com/deepmodeling/deepmd-kit) | LGPL-3.0（软件仓库；依赖和模型另行核对） |
| **GAP / QUIP** | Gaussian Approximation Potential 及 SOAP 等方法的实现生态 | [libAtoms/QUIP](https://github.com/libAtoms/QUIP) | 以仓库 LICENSE、子目录和构建依赖的实际条款为准；不要据此推定为某一单一许可证 |
| **SevenNet** | 面向材料模拟的等变图神经网络势，并提供 LAMMPS 使用路径 | [MDIL-SNU/SevenNet](https://github.com/MDIL-SNU/SevenNet) | MIT（软件仓库；模型与数据另行核对） |

### 势能选择原则

- **金属合金与界面**：优先选择训练数据明确覆盖目标元素、晶相、缺陷和界面局域环境的模型；模型架构本身不能替代数据覆盖。
- **大体系/长时间 MD**：比较推理速度、并行效率、能量漂移和热力学稳定性，而不只比较测试集 RMSE。
- **界面能**：必须检查界面处的力、应力、局域结构和厚度收敛；只报告体相测试误差是不充分的。
- **LAMMPS 部署**：确认上游版本、编译选项、模型格式、单位制和元素映射与生产计算一致。

## 规划中的仓库结构

以下是建议的目录契约。当前仓库尚未包含这些目录；新增脚本时应同步补充输入格式、输出文件、依赖和示例。

```text
codex_MLIP/
├── README.md
├── LICENSE                         # 本仓库许可证（待明确并提交）
├── environment.yml                 # 或 pyproject.toml/requirements.txt
├── configs/
│   ├── elements.yaml              # 元素、晶格和组分定义
│   ├── dft/                        # DFT 参数模板
│   └── mlip/                       # 各势能训练/部署参数
├── structure_generation/
│   ├── bulk/                       # 体相与合金随机结构
│   ├── defects/                    # 空位、间隙、替位和表面
│   ├── grain_boundaries/            # CSL/取向与晶界建模
│   └── interfaces/                 # 异质界面与相界面
├── dft_labeling/
│   ├── prepare.py                  # 生成 DFT 输入
│   ├── submit.py                   # 集群提交（不包含站点凭据）
│   └── collect.py                  # 解析能量、力和应力
├── datasets/
│   ├── raw/                        # 原始输出（建议由外部存储管理）
│   ├── processed/                  # 统一后的训练格式
│   └── splits/                     # train/validation/test 清单
├── mlip_training/
│   ├── mace/
│   ├── nequip_allegro/
│   ├── deepmd/
│   └── scripts/                    # 训练、评估和主动学习入口
├── lammps/
│   ├── input/                      # 弛豫、MD 和采样输入
│   ├── potentials/                 # 不提交私有/未授权权重
│   └── postprocess/
├── interface_energy/
│   ├── calculate.py                # 面积、参考态和能量核算
│   └── convergence.py              # 厚度、面积和尺寸收敛
└── tests/                          # 小型回归测试与数据格式测试
```

建议脚本采用命令行接口（CLI），至少提供 `--help`、输入文件路径、输出目录、随机种子和日志路径；不要把集群账号、API token 或私有数据写入仓库。

## 使用教程（规划工作流）

> 以下命令展示接口约定。由于对应脚本尚未提交，命令目前不能直接执行。

### 1. 创建环境

```bash
git clone https://github.com/Wqeeeeeeee/codex_MLIP.git
cd codex_MLIP
# 按实际提交的环境文件选择其一
conda env create -f environment.yml
conda activate codex-mlip
```

固定代码、环境和数据版本，并记录 `python --version`、LAMMPS/DFT 程序版本以及 GPU/CUDA 信息。

### 2. 生成结构

```bash
python structure_generation/bulk/generate_alloy.py \
  --elements Ni Al --composition Ni3Al --phase fcc \
  --supercell 4 4 4 --output datasets/raw/bulk

python structure_generation/grain_boundaries/generate_csl.py \
  --lattice fcc --axis 001 --sigma 5 --output datasets/raw/gb_sigma5
```

结构生成后应检查元素计数、周期性、原子最小距离、晶界法向、重复边界和界面面积，并保留生成参数。

### 3. DFT 打标与数据整理

```bash
python dft_labeling/prepare.py \
  --input datasets/raw \
  --config configs/dft/pbe.yaml \
  --output dft_jobs
# 在集群上完成计算后
python dft_labeling/collect.py \
  --input dft_jobs --output datasets/processed/alloy_gb.extxyz
python datasets/make_splits.py \
  --input datasets/processed/alloy_gb.extxyz \
  --seed 2024 --output datasets/splits
```

避免随机拆分导致同一条 MD 轨迹或同一界面构型同时出现在训练集和测试集；对晶界/界面数据应按结构家族或轨迹分组拆分。

### 4. 训练和验证 MLIP

按实际接入的后端填写配置，例如：

```bash
python mlip_training/train.py \
  --backend mace \
  --config configs/mlip/mace.yaml \
  --train datasets/splits/train.extxyz \
  --valid datasets/splits/validation.extxyz \
  --output runs/mace-alloy-gb

python mlip_training/evaluate.py \
  --model runs/mace-alloy-gb/model.model \
  --data datasets/splits/test.extxyz \
  --output runs/mace-alloy-gb/test.json
```

至少报告能量、力、应力的 MAE/RMSE 及其单位，并单独报告体相、缺陷、晶界和异质界面子集。若使用预训练模型，应记录来源、版本、权重许可证和再训练过程。

### 5. LAMMPS 弛豫与 MD

```bash
lmp -in lammps/input/relax_gb.in \
  -var structure datasets/processed/gb_sigma5.data \
  -var potential runs/mace-alloy-gb/deployed.model
```

实际命令取决于 LAMMPS 编译的 pair style 和上游部署方式。运行前核对单位（通常为 `metal`）、元素映射、邻居表、时间步长、温控/压控和模型加载日志。生产计算应先用小体系完成能量、力和并行一致性测试。

## 质量控制与界面能后处理

### 界面能定义

对于具有两个等价界面的周期性晶界超胞，常用 0 K 晶界能为

$$
\gamma_{\mathrm{GB}} = \frac{E_{\mathrm{GB}} - N E_{\mathrm{bulk}}}{2A},
$$

其中 `E_GB` 是弛豫后的超胞总能，`N` 是原子数，`E_bulk` 是同组分、同计算设置下的每原子体相参考能，`A` 是一个界面的面积，分母中的 2 对应两个周期性界面。单界面、非等价界面、表面或异质界面必须按实际几何和参考态重新推导，不能机械套用此式。

对异质界面，应明确化学势/参考相。例如可采用

$$
\gamma_{\mathrm{int}} = \frac{E_{\mathrm{interface}} - \sum_i N_i\mu_i}{A_{\mathrm{interface}}},
$$

并在方法部分说明 `μ_i` 的来源、界面数、面积定义、组分偏聚和是否含应变能。有限温度下由能量得到的量通常不是严格的界面自由能；若要报告自由能，应说明熵项和采样方法。

### 必做检查

- 体相参考与界面计算使用同一势能、元素映射、单位和数值精度；
- 对法向厚度、平行尺寸、真空层/周期边界和界面间距做收敛测试；
- 检查弛豫后的残余力、应力、晶格变化和界面是否发生重构；
- 比较不同初始平移、终止面、组分和磁性设置；
- 用 DFT 或实验可得量对关键构型交叉验证；
- 对统计采样报告均值、标准差和独立重复次数；
- 保存原子结构、总能、面积、参考能和脚本版本，使结果可复算。

## 引用说明

使用本仓库时，请同时引用：

1. 本仓库实际使用的版本（建议引用 Git commit 或 release DOI）；
2. 采用的 MLIP 软件及其方法论文；
3. DFT 程序、赝势/泛函、LAMMPS 和数据集；
4. 晶界/界面结构生成算法及界面能定义的来源。

上游项目通常在各自仓库的 README、`CITATION.cff` 或论文列表中给出推荐引用。不要只引用本仓库而省略实际执行训练和生产 MD 的底层软件。

示例（请按实际版本补全）：

```bibtex
@software{codex_mlip,
  title  = {codex_MLIP},
  author = {Project contributors},
  url    = {https://github.com/Wqeeeeeeee/codex_MLIP},
  note   = {Accessed: YYYY-MM-DD}
}
```

## 许可证与开源声明

本仓库的许可证文件尚未在当前初始版本中提交。维护者应在发布代码前选择并提交适用于本仓库脚本的许可证，并在此处更新 SPDX 标识。除非仓库根目录明确声明，不能据此推定本仓库代码采用 MIT、GPL 或其他许可证。

本仓库不自动重新许可第三方软件、模型权重、训练数据、DFT 输入输出或 LAMMPS 插件。分发包含上游项目代码、权重或数据的版本时：

- 保留上游版权、LICENSE 和 NOTICE；
- 明确区分本仓库代码、上游代码、模型权重和数据集；
- 遵守各项目的署名、源代码提供、修改标注和再分发条件；
- 检查依赖的许可证兼容性，尤其是 LGPL/GPL 组件与闭源分发场景；
- 不提交私有计算数据、集群凭据、访问令牌或未经授权的权重。

## English summary

`codex_MLIP` is intended to host a reproducible machine-learning interatomic-potential workflow for metallic alloys, grain boundaries and heterointerfaces: structure generation, DFT labeling, MLIP training, LAMMPS simulation and interface-energy analysis.

**Repository status:** at the time of writing, the repository contains only this README. The directory tree, commands and CLI names above are proposed interfaces, not evidence that the corresponding scripts already exist. Check the repository and each script's `--help` before use.

The planned workflow should document composition, crystal structure, interface geometry, DFT settings, dataset splits, model/version identifiers, LAMMPS settings and convergence tests. For a periodic cell containing two equivalent grain boundaries, a common 0 K expression is `γ_GB = (E_GB − N E_bulk)/(2A)`. The number of interfaces, reference chemical potentials and finite-temperature/free-energy assumptions must be stated explicitly for other cases.

The upstream projects listed above retain their own licenses. This repository does not relicense their code, model weights, datasets or dependencies. A root `LICENSE` file and a precise SPDX identifier should be added before distributing project code.
