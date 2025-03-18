### Copywriter使用记录

#### 日期：2025-01-08

##### 工具：RSTudio、Xshell

##### 数据：/data/share/1000GP_data/1kgp_exome_bam里30个bam文件

##### 服务器：A3

##### 文件夹：/data/liuyuxin/Softwares/CopywriteR/output

###### Step1.准备

```markup
# 1.设置工作目录
setwd("/data/liuyuxin/Softwares/CopywriteR")

# 2.安装和加载必要的R包
if (!requireNamespace("BiocManager", quietly = TRUE))
  install.packages("BiocManager")
  
# 3.安装CopywriteR
BiocManager::install("CopywriteR")

# 4.安装其他依赖包
BiocManager::install(c(
  "matrixStats", "gtools", "data.table", "S4Vectors",
  "chipseq", "IRanges", "Rsamtools", "DNAcopy",
  "GenomicAlignments", "GenomicRanges", "GenomeInfoDb",
  "BiocParallel", "futile.logger"
))

# 5.加载CopywriteR包
library(CopywriteR)
```

==失败，R版本过高，放弃使用RSTudio==

###### Step2. 使用Conda环境安装CopywriteR

1. 进入condar40（R4.0.5版本）

   ```markup
   source ~/.bashrc
   conda activate r40
   ```

2. 进入R环境并安装R包

   ==问题：== BiocManager::install依赖包均失败，切换至官方、中科大、北大镜像均因网速问题安装失败，切换至清华时无依赖包

   ==解决方法：== Google Chrome进入bioconductor官网手动下载所需依赖包并安装

   ```markup
   #安装包均放置在/data/liuyuxin/Softwares/CopywriteR中
   install.packages("/data/liuyuxin/Softwares/CopywriteR/chipseq_1.40.0.tar.gz", repos = NULL, type = "source")
   install.packages("/data/liuyuxin/Softwares/CopywriteR/Biostrings_2.58.0.tar.gz", repos = NULL, type = "source")
   install.packages("/data/liuyuxin/Softwares/CopywriteR/GenomeInfoDbData_1.2.4.tar.gz", repos = NULL, type = "source")
   install.packages("/data/liuyuxin/Softwares/CopywriteR/GenomeInfoDb_1.26.7.tar.gz", repos = NULL, type = "source")
   install.packages("/data/liuyuxin/Softwares/CopywriteR/GenomicAlignments_1.26.0.tar.gz", repos = NULL, type = "source")
   install.packages("/data/liuyuxin/Softwares/CopywriteR/GenomicRanges_1.42.0.tar.gz", repos = NULL, type = "source")
   install.packages("/data/liuyuxin/Softwares/CopywriteR/Rhtslib_1.22.0.tar.gz", repos = NULL, type = "source")
   install.packages("/data/liuyuxin/Softwares/CopywriteR/Rsamtools_2.6.0.tar.gz", repos = NULL, type = "source")
   install.packages("/data/liuyuxin/Softwares/CopywriteR/ShortRead_1.48.0.tar.gz", repos = NULL, type = "source")
   install.packages("/data/liuyuxin/Softwares/CopywriteR/CopyhelpeR_1.38.0.tar.gz", repos = NULL, type = "source")
   #总之R提示缺什么就装什么，BiocManager装不了就自己下安装包安装
   #安装CopywriteR
   install.packages("/data/liuyuxin/Softwares/CopywriteR/CopywriteR_2.6.0.tar.gz", repos = NULL, type = "source")
   ```

   成功

###### Step2. CopywriteR检测test-CNV

```R
# 加载 CopywriteR 包
library(CopywriteR)

# 定义路径
output_folder <- "/data/liuyuxin/Softwares/CopywriteR/output" # 输出文件夹路径
reference_folder <- "/data/liuyuxin/Project_Data/hg19_reference" # hg19 参考基因组路径
capture_regions_file <- "/data/liuyuxin/Project_Data/hg19_bed/20130108.exome.targets.bed" # 目标 BED 文件路径

# BAM 文件列表
bam_list <- "/data/liuyuxin/Softwares/CopywriteR/30bam_test.list"

# 确保输出文件夹存在
if (!dir.exists(output_folder)) {
  dir.create(output_folder, recursive = TRUE)
}

# Step 1: 生成 helper 文件
# 使用 preCopywriteR 创建辅助文件（mappability 和 GC-content GRanges）
preCopywriteR(
  output.folder = output_folder,      # 输出路径
  bin.size = 10000,                   # 分箱大小，10kb
  ref.genome = "hg19",                # 使用 hg19 基因组
  prefix = ""                         # 染色体前缀（如 "chr1" 适配 "hg19"）
)

# Step 2: 定义 BAM 文件样本列表
samples <- scan(bam_list, what = "", sep = "\n", quiet = TRUE)  # 读取 BAM 文件路径
sample.control <- data.frame(samples = samples, controls = samples)  # 无参考样本，每个样本对应自身

# 串行计算配置
bp.param <- SnowParam(workers = 4, type = "SOCK")  # 使用 4 个线程

# Step 3: 运行 CopywriteR 分析
CopywriteR(
  sample.control = sample.control,                       # 样本列表
  destination.folder = output_folder,                    # 输出文件夹
  reference.folder = file.path(output_folder, "hg19_10kb"), # helper 文件生成的路径
  bp.param = bp.param,                                   # 并行计算参数
  capture.regions.file = capture_regions_file,           # 目标 BED 文件
  keep.intermediary.files = TRUE                         # 保留中间文件
)

# Step 4: 可视化生成 CNV 分段图
plotCNA(
  destination.folder = output_folder,  # 输出文件夹路径
  set.nchrom = 24                      # 染色体数量 (22 常染色体 + X, Y)
)

```

失败，提示线程设置问题

更改：

```R
library(BiocParallel)
library(CopyhelpeR)
library(CopywriteR)

#Step1. 生成辅助文件
# 设置参数
output_folder <- "/data/liuyuxin/Softwares/CopywriteR/output"
bin_size <- 20000  # 20 kb
ref_genome <- "hg19"  # 参考基因组名称
prefix <- ""  # 不添加 'chr' 前缀
bam_list_file <- "/data/liuyuxin/Softwares/CopywriteR/30bam_test.list"
capture_regions_file <- "/data/liuyuxin/Project_Data/hg19_bed/20130108.exome.targets.bed"

# 运行 preCopywriteR 生成辅助文件
preCopywriteR(
    output.folder = output_folder,
    bin.size = bin_size,
    ref.genome = ref_genome,
    prefix = prefix
)

#Step2. 准备样本与对照数据
# 读取 BAM 文件列表
bam_files <- readLines(bam_list_file)

# 创建 sample.control 数据框
sample_control <- data.frame(
    samples = bam_files,
    controls = bam_files,  # 每个样本使用自身作为对照
    stringsAsFactors = FALSE
)

# 查看前几行以确认
head(sample_control)

#Step3. 设置并行计算参数
# 设置并行计算参数为30线程
desired_workers <- 30

# 使用 MulticoreParam 因为在 Linux 系统上
bp_param <- MulticoreParam(workers = desired_workers)

#Step4. 设置参考文件夹
# 设置参考文件夹，指向 preCopywriteR 生成的辅助文件所在目录
reference_folder <- file.path(output_folder, paste0(ref_genome, "_", bin_size, "kb"))

#Step5. 运行 CopywriteR 分析
CopywriteR(
    sample.control = sample_control,
    destination.folder = output_folder,
    reference.folder = reference_folder,
    bp.param = bp_param,
    capture.regions.file = capture_regions_file,
    keep.intermediary.files = FALSE  # 是否保留中间文件
)

#Step6. 运行 plotCNA 进行拷贝数分割和绘图
plotCNA(destination.folder = output_folder)
```

失败，提示capture.regions.file丢失

改进：发现只需选择hg19版本，无需输入bed文件和fa参考基因组，会自动生成、调用

```R
library(BiocParallel)
library(CopyhelpeR)
library(CopywriteR)

# 1. 运行 preCopywriteR 生成辅助文件
preCopywriteR(
  output.folder = file.path("./data/liuyuxin/Softwares/CopywriteR/output"),
  bin.size = 20000,
  ref.genome = "hg19",
  prefix = ""
)

# 2. 设置输出、参考文件目录
output_folder <- "/data/liuyuxin/Softwares/CopywriteR/output"
reference_folder <- file.path(output_folder, "hg19_20kb")

# 3. 创建 sample.control 数据框
bam_list_file <- "/data/liuyuxin/Softwares/CopywriteR/30bam_test.list"
bam_files <- readLines(bam_list_file) # 读取该文件以获得所有 BAM 文件路径
  # 每个样本使用自身作为对照
>sample_control <- data.frame(
    samples = bam_files,
    controls = bam_files,
    stringsAsFactors = FALSE
)

# 4. 运行Copywriter之前先检查下BAM文件
missing_bams <- bam_files[!file.exists(bam_files)]
if (length(missing_bams) > 0) {
    stop(paste("以下 BAM 文件不存在：", paste(missing_bams, collapse = ", ")))
} else {
    message("所有 BAM 文件均存在。")
}

# 5. 并行计算参数
bp.param <- SnowParam(workers = 20, type = "SOCK")

# 6. 运行 CopywriteR 分析
CopywriteR(
    sample.control = sample_control,
    destination.folder = output_folder,   # 分析结果的输出目录
    reference.folder = reference_folder,  # 指向 preCopywriteR 生成的 hg19_20kb 文件夹
    bp.param = bp.param,
    keep.intermediary.files = TRUE      # 可以保留中间文件
)

```

**==运行成功==**
