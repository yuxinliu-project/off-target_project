创建conda环境

```markup
conda create -n copywriter r-base=4.0
```

进入环境

```markup
conda activate copywriter
```

安了无数R包都失败，换成r44

克隆conda环境r44，命名为copywriter

进入conda环境，进入R：

```markup
source ~/.bashrc
conda activate copywriter
cd /data/share/liuyuxin_tanrenjie/CopywriteR_HCC1395-WES_output
R
```

###### Step1. 加载 CopywriteR 包并设置路径

```markup
# 1. 加载必要的包
library(CopywriteR)        # 加载 CopywriteR 工具包

# 2. 设置各个文件和目录的路径
bam_path <- "/data/share/liuyuxin_tanrenjie/HCC1395_data/WES/"  # BAM 文件所在的目录
output_path <- "/data/share/liuyuxin_tanrenjie/CopywriteR_HCC1395-WES_output/"  # 输出结果的目录
reference_path <- "/data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/GRCh38/"  # 参考基因组所在目录（包含 .fa、.fai、.dict）
bed_file <- "/data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_modify.bed"  # 捕获区域 BED 文件
```

###### Step2. 设置样本和对照组

```markup
# 3. 定义肿瘤样本和对应的正常对照样本
tumor_samples <- c("WES_EA_Tumor_1",
                   "WES_FD_Tumor_1", "WES_FD_Tumor_2", "WES_FD_Tumor_3",
                   "WES_IL_Tumor_1", "WES_IL_Tumor_2", "WES_IL_Tumor_3",
                   "WES_LL_Tumor_1",
                   "WES_NC_Tumor_1",
                   "WES_NV_Tumor_1", "WES_NV_Tumor_2", "WES_NV_Tumor_3")

normal_samples <- c("WES_EA_Normal_1",
                    "WES_FD_Normal_1", "WES_FD_Normal_2", "WES_FD_Normal_3",
                    "WES_IL_Normal_1", "WES_IL_Normal_2", "WES_IL_Normal_3",
                    "WES_LL_Normal_1",
                    "WES_NC_Normal_1",
                    "WES_NV_Normal_1", "WES_NV_Normal_2", "WES_NV_Normal_3")

# 4. 构建样本-对照数据框
# 这里使用 file.path() 与 paste() 将目录和文件名组合为完整路径，
# 每一行表示一个肿瘤样本和其对应的正常样本
sample_control <- data.frame(
  samples = file.path(bam_path, paste(tumor_samples, ".bwa.dedup.bam", sep="")),
  controls = file.path(bam_path, paste(normal_samples, ".bwa.dedup.bam", sep=""))
)
```

###### Step3. 设置并行计算参数

```markup
# 设置并行计算参数（使用20个 CPU 核心）
bp.param <- SnowParam(workers = 20, type = "SOCK")
```

###### Step4. 准备参考基因组和 bin 大小

```markup
# 准备参考基因组的辅助文件
# 6. 生成辅助文件（helper files）用于后续的 CNV 分析
# preCopywriteR 根据参考基因组（hg38）和指定的 bin 大小（20000 bp）生成
# 包含 mappability、GC-content 和黑名单区域信息的文件，这些文件将存放在 output_path 下的子目录中
preCopywriteR(output.folder = output_path, bin.size = 20000, ref.genome = "hg38", prefix = "")
# 注意：这里“prefix”参数如果参考基因组中的染色体名称已经符合要求，则设置为空
```

生成/data/share/liuyuxin_tanrenjie/CopywriteR_HCC1395-WES_output/hg38_20kb/blacklist.rda和/data/share/liuyuxin_tanrenjie/CopywriteR_HCC1395-WES_output/hg38_20kb/GC_mappability.rda

###### Step5. 运行 CopywriteR 进行 CNV 检测

```markup
# 7. 运行 CopywriteR 进行 CNV 检测
# 这里 reference.folder 应该指向 preCopywriteR 生成的辅助文件目录，
# preCopywriteR 生成的目录为 output_path/hg38_20kb
CopywriteR(
  sample.control = sample_control,                           # 输入样本与对照数据
  destination.folder = output_path,                          # 结果输出目录
  reference.folder = file.path(output_path, "hg38_20kb"),      # 辅助文件目录（必须由 preCopywriteR 生成）
  bp.param = bp.param,                                       # 并行计算设置
  capture.regions.file = bed_file,                           # 捕获区域的 BED 文件（用于排除 on-target 读取）
  keep.intermediary.files = FALSE                            # 保留中间文件
)

# 8. 分段和绘制 CNV 图谱
plotCNA(destination.folder = output_path)
```

失败：

```markup
# 提示
The capture region file is /data/share/liuyuxin_tanrenjie/
  HCC1395_data/reference_genome/Exome_Target_bed/
  S07604624_modify.bed 
This analysis will be run on 12 cpus 
Stop worker failed with the error: reached elapsed time limit
Error: BiocParallel errors
  0 remote errors, element index: 
  24 unevaluated and other errors
  first remote error:
> Error in serialize(data, node$con) : ignoring SIGPIPE signal
Calls: local ... .send -> <Anonymous> -> sendData.SOCKnode -> serialize
Execution halted
Error in serialize(data, node$con) : ignoring SIGPIPE signal
Calls: local ... .send -> <Anonymous> -> sendData.SOCKnode -> serialize
Execution halted
Error in serialize(data, node$con) : ignoring SIGPIPE signal
Calls: local ... .send -> <Anonymous> -> sendData.SOCKnode -> serialize
Execution halted
#本次分析将在 12 个 CPU 上运行。
停止工作节点失败，错误原因：已达到允许的运行时间限制。
错误：BiocParallel 出现错误
0 个远程错误，任务索引 24 有未求值的错误和其他错误。
  第一个远程错误：
   > 在 serialize(data, node$con) 中出错：忽略 SIGPIPE 信号
   调用栈：local ... .send -> <匿名函数> -> sendData.SOCKnode -> serialize
   执行中止

  > 在 serialize(data, node$con) 中出错：忽略 SIGPIPE 信号
   调用栈：local ... .send -> <匿名函数> -> sendData.SOCKnode -> serialize
   执行中止

  > 在 serialize(data, node$con) 中出错：忽略 SIGPIPE 信号
   调用栈：local ... .send -> <匿名函数> -> sendData.SOCKnode -> serialize
   执行中止
```

改成不保存中间文件试试：keep.intermediary.files = FALSE

失败

The following samples will be analyzed:
sample: WES_EA_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_EA_Normal_1.bwa.dedup.bam
sample: WES_FD_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_FD_Normal_1.bwa.dedup.bam
sample: WES_FD_Tumor_2.bwa.dedup.bam ; 	 matching control: WES_FD_Normal_2.bwa.dedup.bam
sample: WES_FD_Tumor_3.bwa.dedup.bam ; 	 matching control: WES_FD_Normal_3.bwa.dedup.bam
sample: WES_IL_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_IL_Normal_1.bwa.dedup.bam
sample: WES_IL_Tumor_2.bwa.dedup.bam ; 	 matching control: WES_IL_Normal_2.bwa.dedup.bam
sample: WES_IL_Tumor_3.bwa.dedup.bam ; 	 matching control: WES_IL_Normal_3.bwa.dedup.bam
sample: WES_LL_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_LL_Normal_1.bwa.dedup.bam
sample: WES_NC_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_NC_Normal_1.bwa.dedup.bam
sample: WES_NV_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_NV_Normal_1.bwa.dedup.bam
sample: WES_NV_Tumor_2.bwa.dedup.bam ; 	 matching control: WES_NV_Normal_2.bwa.dedup.bam
sample: WES_NV_Tumor_3.bwa.dedup.bam ; 	 matching control: WES_NV_Normal_3.bwa.dedup.bam
The bin size for this analysis is 20000
The capture region file is /data/share/liuyuxin_tanrenjie/
HCC1395_data/reference_genome/Exome_Target_bed/
S07604624_modify.bed
This analysis will be run on 12 cpus
Stop worker failed with the error: reached elapsed time limit
Error: BiocParallel errors
0 remote errors, element index:
24 unevaluated and other errors
first remote error:

> Error in serialize(data, node$con) : ignoring SIGPIPE signal
> Calls: local ... .send -> <Anonymous> -> sendData.SOCKnode -> serialize
> Execution halted
> Error in serialize(data, node$con) : ignoring SIGPIPE signal
> Calls: local ... .send ->  -> sendData.SOCKnode -> serialize
> Execution halted
> Error in serialize(data, node$con) : ignoring SIGPIPE signal
> Calls: local ... .send ->  -> sendData.SOCKnode -> serialize
> Execution halted

