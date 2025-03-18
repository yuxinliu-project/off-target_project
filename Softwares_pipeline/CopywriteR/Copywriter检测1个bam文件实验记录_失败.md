###### Copywriter检测1个bam文件

sample：/data/share/liuyuxin\_tanrenjie/HCC1395\_data/WES/WES\_EA\_Tumor\_1.bwa.dedup.bam

control：/data/share/liuyuxin\_tanrenjie/HCC1395\_data/WES/WES\_EA\_Normal\_1.bwa.dedup.bam

bed：/data/share/liuyuxin\_tanrenjie/HCC1395\_data/reference\_genome/Exome\_Target\_bed/S07604624\_modify.bed

FASTA：/data/share/liuyuxin\_tanrenjie/HCC1395\_data/reference\_genome/GRCh38/GRCh38.d1.vd1.fa

code：

```markup
#Step1 
# 1. 加载必要的包
library(CopywriteR)        # 加载 CopywriteR 工具包

# 2. 设置各个文件和目录的路径
bam_path <- "/data/share/liuyuxin_tanrenjie/HCC1395_data/WES/"  # BAM 文件所在的目录
output_path <- "/data/share/liuyuxin_tanrenjie/CopywriteR_HCC1395-WES_output/"  # 输出结果的目录
reference_path <- "/data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/GRCh38/"  # 参考基因组所在目录（包含 .fa、.fai、.dict）
bed_file <- "/data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_modify.bed"  # 捕获区域 BED 文件

#Step2
sample_control <- data.frame(
  samples = "/data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Tumor_1.bwa.dedup.bam", 
  controls = "/data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Normal_1.bwa.dedup.bam"
)

#Step3
preCopywriteR(output.folder = output_folder, bin.size = bin_size, ref.genome = ref_genome, prefix = "chr")

#Step4
CopywriteR(
  sample.control = sample_control,                # 提供样本与对照数据框
  destination.folder = output_path,               # 结果输出目录
  reference.folder = reference_path,              # 辅助文件目录
  bp.param = bp.param,                            # 禁用并行计算
  capture.regions.file = bed_file,                # 捕获区域 BED 文件
  keep.intermediary.files = FALSE                 # 不保留中间文件
)
```

out：

```markup
The following samples will be analyzed: 
sample: WES_EA_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_EA_Normal_1.bwa.dedup.bam
The bin size for this analysis is 20000 
The capture region file is /data/share/liuyuxin_tanrenjie/
  HCC1395_data/reference_genome/Exome_Target_bed/
  S07604624_modify.bed 
This analysis will be run on 1 cpus 
INFO [2025-03-18 12:47:38] Paired-end sequencing for sample WES_EA_Tumor_1.bwa.dedup.bam: TRUE
INFO [2025-03-18 12:47:38] Paired-end sequencing for sample WES_EA_Normal_1.bwa.dedup.bam: TRUE
INFO [2025-03-18 14:12:08] filterBam("/data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Tumor_1.bwa.dedup.bam", "/data/share/liuyuxin_tanrenjie/CopywriteR_HCC1395-WES_output/CNAprofiles/BamBaiPeaksFiles/WES_EA_Tumor_1.bwa.dedup_properreads.bam", filter = filter, indexDestination = TRUE, param = param)
INFO [2025-03-18 14:12:08] filterBam("/data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Normal_1.bwa.dedup.bam", "/data/share/liuyuxin_tanrenjie/CopywriteR_HCC1395-WES_output/CNAprofiles/BamBaiPeaksFiles/WES_EA_Normal_1.bwa.dedup_properreads.bam", filter = filter, indexDestination = TRUE, param = param)
INFO [2025-03-18 14:16:30] CopywriteR has finished filtering low-quality and anomalous reads in the following (unique) samples:

[1] "WES_EA_Tumor_1.bwa.dedup_properreads.bam" 
[2] "WES_EA_Normal_1.bwa.dedup_properreads.bam"
Error: BiocParallel errors
  1 remote errors, element index: 1
  0 unevaluated and other errors
  first remote error:
Error in if (nrow(peaks) > 0) {: argument is of length zero

```
