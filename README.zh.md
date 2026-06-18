# CoV2RBDAb — SARS-CoV2 RBD 抗体数据库

[![CC BY-NC 4.0][cc-by-nc-shield]][cc-by-nc]

**CoV2RBDAb** 是一个经人工筛选整理的 SARS-CoV2 刺突蛋白 RBD（野生型及突变型）抗体数据库。提供详细的结合证据、定量亲和力数据、序列注释、预测结构以及可直接用于机器学习的数据集。

## 主要功能

- **结合证据与序列记录** — 经人工筛选整理，标注完整来源
- **定量结合亲和力**（K<sub>D</sub>）— 来自 SPR/BLI 实验
- **可变区注释** — 使用 InterProScan 进行结构域注释，使用 ANARCI 进行 IMGT 编号
- **预测抗体结构** — 使用 IgFold 预测的 apoform 结构（PDB 格式，可直接可视化）
- **机器学习数据集** — 提供适用于 WT 和突变 RBD 的二分类和定量亲和力预测数据集

## 技术栈

| 组件 | 技术 |
|-----------|-----------|
| 网站框架 | [Jekyll](https://jekyllrb.com/) 静态站点（GitHub Pages） |
| 主题 | [Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/) v4.24.0 |
| 数据库 | SQLite3（`collected_0919.db`），同时导出为 CSV |
| 数据格式 | CSV（表格）、TSV（数据集） |
| 分析工具 | Jupyter notebook |
| 压缩格式 | 7-zip 压缩包 |

## 目录结构

```
antibodycov2db/
├── _config.yml          # Jekyll 站点配置
├── Gemfile              # Ruby 依赖
├── README.md            # 本文件（英文版）
├── README.zh-CN.md      # 本文件（中文版）
├── LICENSE              # CC BY-NC 4.0 许可证
├── _data/               # 站点数据
│   ├── navigation.yml   # 导航定义
│   ├── datasets/        # 机器学习数据集（TSV）
│   └── tables/          # 数据库表（CSV）
├── _pages/              # 站点页面（Markdown）
│   ├── home.md          # 首页 / 快速入门
│   ├── abdetail.md      # 抗体详情页
│   ├── analysis.md      # 数据分析和相关性
│   ├── browse.md        # 数据浏览
│   ├── download.md      # 下载页面
│   ├── search.md        # 搜索界面
│   ├── utilities.md     # 工具（突变RBD序列生成器、表位查看器）
│   └── about.md         # 关于项目
├── assets/
│   ├── db/              # SQLite3 数据库文件
│   ├── igfold/          # IgFold 预测的抗体结构（PDB）
│   ├── images/          # 站点图片
│   └── js/              # JavaScript 资源
├── compressed/          # 7z 压缩包（批量下载用）
├── notebooks/           # Jupyter notebook
├── scripts/             # 工具脚本
├── templates/           # Jekyll 文章模板
└── _site/               # Jekyll 生成输出
```

## 快速开始

```bash
# 克隆仓库
git clone https://github.com/Cov2RBDAb/antibodycov2db.git
cd antibodycov2db

# 安装依赖
bundle install

# 本地预览
bundle exec jekyll serve

# 或生成站点
bundle exec jekyll build
```

站点通过 GitHub Pages 部署在 `gh-pages` 分支。

## 数据库

**位置**：[`assets/db/collected_0919.db`](assets/db/collected_0919.db)（约 175 MB）

SQLite3 数据库包含 14 张表，定义了完整的外键约束和索引。`_data/tables/` 中的所有 CSV 文件均导出自该数据库。

### ER 关系图

```
┌────────────────────────────────────────────────────────────────────┐
│                        name  (核心实体)                             │
│  ab_idx (PK)  |  all_names                                         │
└────────┬───────────────────────────────────────────────────────────┘
         │ 1
         │
    ┌────┼──────────────────────────────────────────────────────┐
    │    │                                                      │
    │    │ * (FK: ab_idx)                                       │
    ▼    ▼                                                      ▼
┌──────────────┐  ┌──────────────┐  ┌─────────────────────────────┐
│  record      │  │  evidence    │  │  vgene                      │
│──────────────│  │──────────────│  │─────────────────────────────│
│*Hseq (FK)→seq│  │*ab_idx (FK)  │  │*ab_idx (FK) │ chain         │
│*Lseq (FK)→seq│  │*target (FK)  │  │ V gene      │ source        │
│ ab_idx (FK)  │  │ DOI, source  │  │(ab_idx+chain+source unique) │
│ Hsource_id   │  │ binding, KD  │  └─────────────────────────────┘
│ Lsource_id   │  │ Ab_type      │
│ source       │  │ pdb, evidence│
│ notab-like   │  │ update_date  │
└──────┬───────┘  └──────┬───────┘
       │                 │
       │(FK)             │(FK)
       │                 ▼
       │          ┌───────────────┐
       │          │ target_rbdseq │
       │          │───────────────│
       │          │ target (PK)   │
       │          │ rbd_seq       │
       │          │ lineage       │
       │          └───────────────┘
       │
       │ ┌─────────────────────────────────────────────┐
       │ │    sequence  (序列主表)                       │
       │ │─────────────────────────────────────────────│
       │ │ seq (PK)                                    │
       │ └──────────┬──────────────────────┬───────────┘
       │            │                      │
       │            │1                     │1
       │            ▼                      ▼
       │  ┌──────────────┐   ┌──────────────────────┐
       │  │ ab_type      │   │ region               │
       │  │──────────────│   │──────────────────────│
       │  │ Hseq (FK)    │   │*seq (FK)             │
       │  │ Lseq (FK)    │   │ region               │
       │  │ ab_type      │   └──────────────────────┘
       │  │(Hseq+Lseq PK)│
       │  └──────────────┘
       │  ┌──────────────┐   ┌──────────────────────┐
       │  │ num_domain   │   │ trunct2fv            │
       │  │──────────────│   │──────────────────────│
       │  │*seq (FK)     │   │*seq (FK)             │
       │  │ num_var      │   │ seq_vdomain          │
       │  │ num_cons     │   └──────────────────────┘
       │  └──────────────┘
       │
       │ ┌────────────────────────────────────────────┐
       │ │    pdb_chain_idmapping  (PDB 链 ID 映射)    │
       │ │────────────────────────────────────────────│
       │ │ instance (PK)  |  entity  |  paired        │
       │ │ chain                                      │
       │ └──────────┬──────────────────────┬──────────┘
       │            │                      │
       │            │1                     │1
       │            ▼                      ▼
       │  ┌──────────────────────┐  ┌──────────────────────────┐
       │  │ pdb_ab_rbd_pairing   │  │ pdb_nb_rbd_pairing       │
       │  │──────────────────────│  │──────────────────────────│
       │  │*HC_instance_id (FK)  │  │*HC_instance_id (FK)      │
       │  │*LC_instance_id (FK)  │  │*spike_instance_id (FK)   │
       │  │*spike_instance_id    │  │ rbd_valid_seq            │
       │  │ rbd_valid_seq        │  │ rbd_valid_resids         │
       │  │ rbd_valid_resids     │  │ rbd_fulllen_seq          │
       │  │ rbd_fulllen_seq      │  │ matched_lineage          │
       │  │ matched_lineage      │  │ rbd_provided_seq         │
       │  │ rbd_provided_seq     │  │ rbd_provided_resids      │
       │  │ rbd_provided_resids  │  └──────────────────────────┘
       │  └──────────────────────┘
       │  ┌───────────────────────────────────────────────┐
       │  │ epitope_group                                 │
       │  │───────────────────────────────────────────────│
       │  │*HC_instance_id (FK) → pdb_chain_idmapping     │
       │  │*LC_instance_id (FK) → pdb_chain_idmapping     │
       │  │*ab_idx (FK) → name                            │
       │  │ epitope | source | checked_epitope_group      │
       │  │ epitope_sites | epitope_class                 │
       │  │ predicted_epitopes                            │
       └──────────────────────────────────────────────────┘
```

### 数据表

| 表名 | 角色 | 关键字段 |
|-------|------|------------|
| `name` | 抗体名称主表 | `ab_idx` (PK), `all_names` |
| `sequence` | 序列主表 | `seq` (PK, 氨基酸序列) |
| `record` | 序列记录 | `Hseq`, `Lseq`, `ab_idx`, `source` |
| `ab_type` | 抗体类型 (Fv/scFv/Fab/…) | `Hseq`, `Lseq`, `ab_type` |
| `region` | 序列区域 (FR1–FR4, CDR1–CDR3) | `seq`, `region` |
| `num_domain` | IMGT 域编号 | `seq`, `num_var`, `num_cons` |
| `trunct2fv` | 全长到可变域截断映射 | `seq`（全长）, `seq_vdomain`（可变域） |
| `evidence` | 结合证据 | `ab_idx`, `target`, `KD`, `DOI`, `source` |
| `target_rbdseq` | RBD 靶标序列 | `target`, `rbd_seq`, `lineage` |
| `vgene` | V 基因注释 | `ab_idx`, `chain`, `V gene`, `source` |
| `pdb_chain_idmapping` | PDB 链 ID 映射 | `instance`, `entity`, `chain`, `paired` |
| `pdb_ab_rbd_pairing` | 抗体-RBD 复合物配对 | `HC_instance_id`, `LC_instance_id`, `spike_instance_id` |
| `pdb_nb_rbd_pairing` | 纳米抗体-RBD 复合物配对 | `HC_instance_id`, `spike_instance_id` |
| `epitope_group` | 表位组分类 | `HC/LC_instance_id`, `epitope`, `epitope_class` |

### 数据集

预构建数据集存放于 `_data/datasets/`（TSV 格式）：

| 文件 | 任务 |
|------|------|
| `classification_variantrbd.tsv` | 针对突变 RBD 的二分类预测 |
| `epitope_variantrbd.tsv` | 针对突变 RBD 的表位预测 |
| `regression_wtrbd.tsv` | 针对 WT RBD 的定量亲和力回归 |

## 使用方式

- **在线浏览**：访问 [网站](https://Cov2RBDAb.github.io) 浏览、搜索和下载数据。
- **批量下载**：获取 [`all_db_tables.7z`](compressed/all_db_tables.7z) 或 [`all_ds_tables.7z`](compressed/all_ds_tables.7z)。
- **SQLite3 数据库**：从 [Zenodo](https://doi.org/10.5281/zenodo.17627592) 下载完整数据库，或直接使用 `assets/db/collected_0919.db`。
- **搜索**：在[搜索页面](https://Cov2RBDAb.github.io/search/)通过 ID、名称或序列查找抗体。
- **分析**：在[分析页面](https://Cov2RBDAb.github.io/analysis/)查看数据分布、相关性及数据集划分。

## 许可证

本项目采用 [Creative Commons Attribution-NonCommercial 4.0 International License][cc-by-nc] 开源。

[cc-by-nc]: https://creativecommons.org/licenses/by-nc/4.0/
[cc-by-nc-image]: https://licensebuttons.net/l/by-nc/4.0/88x31.png
[cc-by-nc-shield]: https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg

## 引用

如果您在研究中使用了 CoV2RBDAb，请引用：

> *（待补充引用信息）*
