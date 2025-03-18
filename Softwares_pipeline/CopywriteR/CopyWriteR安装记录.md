##### CopywriteR使用实验记录

###### 目的：检测2692个cram的CNV，为论文准备好数据

###### 时间：2024/11/30

###### Github: https://github.com/PeeperLab/CopywriteR

###### 所需其他工具：CopyhelpeR

###### 所用服务器：A3

###### 所选环境：conda r44

###### 工具位置：/data/liuyuxin/miniconda3/envs/r44/lib/R/library

###### 输出文件位置：/data/liuyuxin/Softwares/CopywriteR/bam_output

1. **进入conda r44并安装所需依赖包：**

   ```markup
   source ~/.bashrc
   conda activate r44
   R
   options(repos = c(CRAN = "https://mirrors.tuna.tsinghua.edu.cn/CRAN/"))
   options(BioC_mirror = "https://mirrors.tuna.tsinghua.edu.cn/bioconductor/")
   ```

   安装依赖包：

   ```markup
   install.packages(c("matrixStats", "gtools", "data.table"))
   source("http://bioconductor.org/biocLite.R")
   biocLite(c("S4Vectors", "chipseq", "IRanges", "Rsamtools", "DNAcopy",
              "GenomicAlignments", "GenomicRanges", "GenomeInfoDb",
              "BiocParallel", "futile.logger"))
   ```

2. **下载并安装CopyhelpeR：**

   1. 安装chipseq

      ```markup
      BiocManager::install("chipseq")
      ```

      成功

   2. 安装其他依赖包：

      ```markup
      BiocManager::install(c(
        "matrixStats", "gtools", "data.table", "S4Vectors", "IRanges",
        "Rsamtools", "DNAcopy", "GenomicAlignments", "GenomicRanges",
        "GenomeInfoDb", "BiocParallel", "futile.logger"
      ))
      ```

      成功

3. 安装CopywriteR：

   ```markup
   install.packages("/data/liuyuxin/Softwares/CopywriteR_2.6.1.tar.gz", repos = NULL, type = "source")
   ```

   成功，可运行

