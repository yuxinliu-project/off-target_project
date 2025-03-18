### OFF-PEAK检测HCC1395的12对WES

###### 数据：

**WES数据：** hg38-bam：/data/share/liuyuxin_tanrenjie/HCC1395_data/WES

**bed文件：**/data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_modify.bed

**FASTA：**/data/share/1000GP_data/technology/reference/GRCh38_full_analysis_set_plus_decoy_hla.fa

**Tools文件夹：**/data/liuyuxin/Project-7-tools/OFF-PEAK-updat

###### 执行步骤：

注意：bed和bam需统一chr格式

```
cd /data/liuyuxin/Project-7-tools/OFF-PEAK-publication #无需编译、配置，直接使用脚本
#检查bam文件chr格式
samtools view -H WES_EA_N_1.bwa.dedup.bam | grep "^@SQ"
```

输出：@SQ SN:chr1 LN:2489564223

Result：带chr，格式统一

#### Step1. 01_targets-processing.sh

```Shell
bash 01_targets-processing.sh \
--genome hg38 \
--targets /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_modify.bed \
--name HCC1395_WES_target-panel \
--ref /data/share/1000GP_data/technology/reference/GRCh38_full_analysis_set_plus_decoy_hla.fa \
--minOntarget 100 \
--maxOntarget 300 \
--minOfftarget 1 \
--maxOfftarget 50000 \
--paddingOfftarget 300 
```

###### bash步骤：

1. extracting exons from gene file and selected the targeted ones

   从data/里的预先下载的基因注释文件，中提取所有外显子信息，然后根据输入的 bed 判断 on-target 区域

2. extend on-targets smaller than --minOntarget

   在其两侧扩展长度小于参数 --minOntarget 所设定的最小阈值的靶向区域，使区域的长度达到该最小值，保证每个靶向区域具有足够的长度用于后续分析。

3. split targets larger than --maxOntarget

   若靶向区域的长度超过参数 --maxOntarget 设定的最大阈值，则将该区域拆分为多个等长的小区域。这样可以防止由于区域过大而导致的覆盖度不均或信号混淆问题。

4. padding off-targets with --paddingOfftarget

   在靶向区域的两侧添加一个固定长度的“边距”（由参数 --paddingOfftarget 指定），以确保捕获过程中可能混入的临近读取不会错误地归入非靶向区域。这个边距有助于更准确地区分靶向和非靶向区域的覆盖度。

5. spliting off-targets larger than --maxOfftarget

   对于非靶向区域，如果其长度超过参数 --maxOfftarget 指定的最大值，则将其拆分成多个较小的区域。这样可以使每个区域的覆盖度更均匀，便于后续统计和 CNV 检测。

6. annotating non-coding exons

   对于非编码区域中的外显子（即未直接捕获到的外显子或其他相关区域）进行标注。在输出的 BED 文件中为这些区域添加外显子信息，便于后续分析时区分编码与非编码区域。

7. removing off-targets smaller than --minOfftarget

   筛选非靶向区域，删除那些长度小于参数 --minOfftarget 指定最小阈值的区域。这些区域由于面积太小可能不可靠或没有足够的读取支持，因此会被过滤掉。

8. writing output

   将经过上述所有步骤处理后的靶向和非靶向区域整理成一个标准化的 BED 文件

9. annotating GC content

   对每个区域计算并注释其GC含量。GC含量信息可以用于后续对覆盖度数据进行归一化处理，因为测序过程中GC含量的差异可能会影响区域的读取深度。

10. cleaning temporary files

    删除在整个处理过程中生成的临时文件和中间结果文件，确保最终输出目录干净，只保留最终生成的BED文件等必要文件。

###### Output：

HCC1395_WES_target-panel.bed

#### Step2. 02_coverage-count.sh

```markup
bash 02_coverage-count.sh \
--listBAM HCC1395_WES_list.txt \
--mosdepth mosdepth-master/mosdepth \
--work HCC1395_WES_output \
--targetsBED data/HCC1395_WES_target-panel.bed
```

###### bash步骤：

1. Using mosdepth to extract coverage for each on-target and off-target

   使用 mosdepth 计算样本在靶向和非靶向区域的覆盖度

2. Merging individual coverage files into one file

   合并各样本的覆盖度文件到一个总文件

###### Output：

/data/liuyuxin/Project-7-tools/OFF-PEAK-publication/HCC1395_WES_output

1. ALL.target.tsv

2. 各Sample的tsv格式的coverage

#### Step3. 03_OFF-PEAK.R

准备：

```markup
R
url <- "https://cran.r-project.org/src/contrib/Archive/ExomeDepth/ExomeDepth_1.1.16.tar.gz"
pkgFile <- "ExomeDepth_1.1.16.tar.gz"
download.file(url = url, destfile = pkgFile)
untar(pkgFile)
install.packages(c("Biostrings","IRanges","Rsamtools","GenomicRanges","aod","VGAM","GenomicAlignments","dplyr","magrittr"))
install.packages(pkgs=pkgFile, type="source", repos=NULL)
unlink(pkgFile)
```

code：

```markup
Rscript 03_OFF-PEAK.R \
--output HCC1395_output_directory \
--data HCC1395_WES_output/ALL.target.tsv \
--databasefile data/data-hg38.RData \
--mincor 0.9 \
--minsignal 2500 \
--maxvar -0.2 \
--leaveoneout 1 \
--downsample 20000 \
--nbFake 500 \
--stopPC 0.0001 \
--minZ 4 \
--minOfftarget 1000 \
--chromosome-plots \
--genome-plots \
--nb-plots 10
```

**Rscript步骤：**&#x20;

```markup
[1] "Analyzing sample: WES_EA_N_1"
[1] "  Step 1: selecting samples with correlation > 0.9"
[1] "WARNING: less than 15 other samples with R2>0.9, taking 15 samples with highest correlation. This can lead to decreased performances."
[1] "    Samples selected excluding the one analyzed: 15 out of 23"
[1] "  Step 2: normalize by sample and filtering of targets/antitargets"
[1] "    Intervals selected: 418363 out of 507812"
[1] "  Step 3: optimization of PC removal with fake CNVs"
[1] "  Step 4: computation of CNVs"
[1] "  Step 5: genome and chromosome plots"
[1] "  Step 6: merging of consecutive CNVs for all targets"
[1] "  Step 7: merging of consecutive CNVs for targets only"
```

1. **数据读取与预处理：**

   1. 读取覆盖度数据：通过 ==read.table== 函数，将Step2生成的覆盖度数据文件（ALL.target.tsv）读入 R 中

   2. 过滤目标区域：根据设定的最低信号（==--minsignal==）和最大噪声（==--maxvar==）的阈值，对目标区域进行筛选。只有当一个目标区域的平均信号高于 minsignal 且其信号的标准差与均值的比值满足要求时，该目标区域才会被保留。目的是效剔除噪声过大或信号不足的区域

   3. 提取 GC 含量：脚本将目标区域数据中的 GC 含量信息分离出来，后续用于对覆盖度数据进行归一化处理，以校正由 GC 含量引起的测序偏差。

2. **数据归一化与降噪：**

   1. 样本间归一化：先计算每个目标区域在所有样本中的平均覆盖度，然后将每个样本的数值与平均值比对，得到一个归一化比率

   2. GC校正：利用前面提取的 GC 含量信息，对覆盖度数据进行回归校正。通过建立 GC 与覆盖度的关系模型，将覆盖度数据进行校正，从而减少由于 GC 含量差异造成的系统性误差

   3. 降噪处理与 PCA：通过 ==--leaveoneout== （0 or 1）参数决定留一法 PCA（LOO-PCA）或标准 PCA，以排除待测样本自身的异常信号对 PCA 模型的影响。并域数据进行下采样（==--downsample== ）以及通过植入一定数量的假 CNV（--nbFake）来评估和优化最佳移除主成分的数量。 参数 ==--stopPC== 用于设置停止条件，当移除的主成分达到一定标准后，不再继续降噪，以免过度移除信号。

3. **CNV 计算与注释：**

   1. 计算 Z-score：脚本根据归一化并降噪后的覆盖度数据，为每个目标区域计算 Z-score，即当前样本与对照样本间覆盖度的标准差差异。对于单个目标区域，会根据设定的阈值 --minZ 来决定是否认为该目标区域存在 CNV 信号

   2. CNV 识别：结合连续目标区域的异常信号，通过HMM方法（调用 ExomeDepth 包中相关函数）进一步识别和调用跨多个目标的 CNV 事件。 HMM 可以帮助将散布在连续目标区域上的异常信号归纳为一个 CNV 事件，并确定其边界和类型（缺失或扩增）

   3. 注释 CNV：使用提供的数据库文件（==--databasefile== 参数），将识别到的 CNV 与外部数据库（ ClinVar、gnomAD）进行比对，注释每个 CNV 的已知临床意义和频率信息。 此外，还可以 CNV 进行进一步过滤（例如设置最低质量、最低样本支持数等），以获得高质量的 CNV 列表。（OFF-PEAK-HQ）

   4. 统计与质量评分：计算一些统计指标，如每个样本检测到的 CNV 数量、覆盖度变异的分布以及信号与噪声比等。通过这些统计，评估 CNV 检测的可靠性和整体质量

4. **绘图：**

   1. CNV 图形绘制：为每个样本（或每个 CNV）生成pdf，展示了目标区域的覆盖度变化、异常信号位置以及降噪前后的对比。--chromosome-plots 和 --genome-plots 可以控制是否生成染色体级别或全基因组的覆盖度图形

   2. 绘图参数：--nb-plots 控制每个样本生成多少个 CNV 图形。图形中通常包括原始信号、归一化信号以及通过降噪后计算得到的 Z-score 曲线，帮助用户直观评估 CNV 调用结果

5. **可选参数：**

   | 选项                 | 默认值    | 取值范围       | 描述                                                                 |
   | :----------------- | :----- | :--------- | :----------------------------------------------------------------- |
   | --mincor           | 0.9    | 0 ~ 0.99   | 控制样本与待分析样本之间最低相关性阈值。只有与待分析样本相关性高于该值的对照样本会被使用                       |
   | --minsignal        | 2500   | 1 ~ ∞      | 目标区域的最小信号值。信号低于此值的目标区域将不参与分析                                       |
   | --maxvar           | -0.2   | -1 ~ 1     | 目标区域的最小信号值。信号低于此值的目标区域将不参与分析                                       |
   | --leaveoneout      | 1      | 0 或 1      | 如果设置为 1，则采用留一法 PCA（LOO-PCA）进行降噪；若为 0，则采用标准 PCA                     |
   | --downsample       | 20000  | 100 ~ ∞    | 优化主成分移除时下采样目标区域的数量。下采样可以提高计算效率和优化降噪效果。                             |
   | --nbFake           | 500    | 10 ~ 10000 | 用于优化主成分移除过程中的假 CNV 数量。通过植入假 CNV 来评估最佳的主成分移除数                       |
   | --stopPC           | 0.0001 | 0 ~ 0.1    | 主成分移除优化过程的停止标准。当移除的主成分数量达到该标准时停止进一步移除                              |
   | --minZ             | 4      | 2 ~ 10     | 单个目标区域 CNV 处理时要求的最小绝对 Z-score。只有达到该阈值的区域才会被认为有 CNV 信号              |
   | --minOfftarget     | 1000   | 1 ~ ∞      | 非靶向区域（没有外显子）的最小尺寸，低于该值的区域将被忽略（针对非靶向区域设置了最小尺寸要求，防止太小的区域因数据不充分而干扰分析） |
   | --chromosome-plots | -      | -          | 如果指定该选项，将生成每条染色体的覆盖度图                                              |
   | --genome-plots     | -      | -          | 如果指定该选项，将生成全基因组范围内的覆盖度图                                            |
   | --nb-plots         | 10     | 0 ~ ∞      | 每个样本最多生成的 CNV 图数量                                                  |

6. **Output：**

   /data/liuyuxin/Project-7-tools/OFF-PEAK-publication/HCC1395_output_directory

   1. **01_general_stats 文件夹：**

      | 文件名                                | 描述                                                                             |
      | :--------------------------------- | :----------------------------------------------------------------------------- |
      | Pairwise-correlations-all.tsv      | 包含每对样本之间的相关性系数矩阵（两两样本的Pearson相关系数等），帮助评估哪些样本的覆盖度分布相似                           |
      | Heatmap-correlations-all.pdf       | 将上述样本间相关性矩阵绘制成热图的PDF文件，一目了然地展示样本之间相关性的高低情况。关性越高的样本其热图颜色越接近，方便发现异常样本或样本分组       |
      | Maximal-correlation-per-sample.tsv | 记录每个样本与其他样本的最高相关性值及对应样本。便于样本最相似的对照样本，评估该样本数据质量                                 |
      | Maximal-correlation-per-sample.pdf | 将每个样本的最大相关性以图形方式展示的PDF文件。通常每个样本对应一个点或条形，显示其与最相近样本的相关系数，方便直观判断哪些样本相关性偏低（可能质量较差） |
      | log_parameters.tsv                 | 数日志文件，记录运行CNV检测时使用的参数设置。通过查看该文件，可以追溯分析过程中采用的阈值、过滤标准、PCA成分数等设定，保证结果的可重复性和可追溯性   |

   2. **02_BED-files 文件夹：**

      | 文件名                           | 描述                                                                                |
      | :---------------------------- | :-------------------------------------------------------------------------------- |
      | CNVs-all-IGV.bed              | 包含所有检测到的CNV区域的BED文件，可直接载入IGV进行可视化                                                 |
      | CNVs-targets-only-IGV.bed     | 仅包含目标区域内（on-target）CNV的BED文件，可用于IGV可视化                                            |
      | CNVs-all-AnnotSV.bed          | 含所有CNV的BED文件，格式适配AnnotSV工具以进行CNV注释。将此文件输入AnnotSV，可以获得每个CNV的详细注释信息（如与已知致病变异的重叠情况等） |
      | CNVs-targets-only-AnnotSV.bed | 包含目标区域CNV的BED文件，用于AnnotSV注释。通过该文件可以注释在设计靶区域内发生的CNV，排除掉在非靶区域的CNV干扰                 |

   3. **03_Samples-info 文件夹：**

      | 文件名                      | 描述                                                                                                     |
      | :----------------------- | :----------------------------------------------------------------------------------------------------- |
      | Samples_quality_info.tsv | 样本质量信息表。每个样本一行，包含多种质量指标，例如平均测序覆盖度、覆盖度标准差、与其它样本的相关性等。通过该表可以比较各样本的数据质量，发现异常值（如覆盖度过低或变异噪声过高的样本）           |
      | Samples_low-quality.tsv  | 低质量样本列表。根据预先设定的标准（如与任何其它样本的相关性低于某阈值、覆盖度异常等），该文件列出被标记为质量欠佳的样本ID。分析时通常会重点关注这些样本，或在下游分析中过滤掉它们，以免影响CNV检测结果 |

   4. **04_CNVs-results 文件夹：**

      | 文件名                   | 描述                                                                                                                                 |
      | :-------------------- | :--------------------------------------------------------------------------------------------------------------------------------- |
      | CNVs-all.tsv          | 所有检测到的CNV列表（汇总所有样本）。每条记录代表一个CNV事件，包含字段说明CNV的位置、类型、大小、质量评分以及受影响的基因等（具体字段含义见 8. CNV文本文件中的字段解释）                                       |
      | CNVs-targets-only.tsv | 仅限于目标区域的CNV列表。即从上述所有CNV中筛选出发生在捕获靶区域上的CNV（on-target CNV）。这些变异通常更可靠（因为目标区域测序深度和设计更有保障）                                               |
      | 过滤后的CNV结果子集           | 该文件夹中还根据不同条件提供了筛选后的CNV列表                                                                                                           |
      | HQ                    | 满足高质量标准的CNV列表                                                                                                                      |
      | unique                | 在其他任何样本中不存在重叠的CNV（独有变异）的列表。即这些CNV是特定于某一个样本的，没有在队列中其他sample到相似的拷贝数改变。这样的CNV可能是稀有的个体特异性变异或与疾病相关的候选变异                                 |
      | HQ-sample             | 仅来自高质量样本的CNV列表。首先根据样本质量（03_Samples-info文件夹中的Samples_quality_info.tsv）筛除低质量样本，然后汇总这些可靠样本中的CNV。这样可以避免因样本数据不佳而产生的可疑CNV，将分析聚焦在可信样本的结果上 |

   5. **05_RData-files 文件夹：**

      | 文件名（ID为sample名）       | 描述                                                                  |
      | :-------------------- | :------------------------------------------------------------------ |
      | data-ID.RData         | 目标和非目标的RData                                                        |
      | data-plot-ID.RData    | 用于绘制CNV图形的相关数据RData文件。绘图脚本会使用其中的数据生成每个CNV的覆盖度图、Z分数图等，可加快绘图过程而不必重新计算 |
      | data-targets-ID.RData | 仅包含目标区域数据的RData                                                     |

   6. **06_PC-plots 文件夹：**

      | 文件名（ID为sample名）           | 描述                                                                                                                  |
      | :------------------------ | :------------------------------------------------------------------------------------------------------------------ |
      | PC-removal-ID.pdf         | 主成分去除效果评估图（AUC曲线）。                                                                                                  |
      | Variance-explained-ID.pdf | PCA主成分方差解释图。展示了按主成分序号的累计方差贡献率，反映每个主成分对覆盖度数据变异的解释程度。通常曲线会显示前几个主成分解释了主要变异，用于决定需要去除多少个主成分来消除噪音（如测序批次效应），同时保留尽可能多的真正信号。 |

   7. **每个样本的独立文件夹：** 每个样本都会有一个以该样本ID命名的文件夹，包含该样本的具体CNV检测结果和可视化图表

      | 文件名（ID为sample名）              | 描述                                                                                      |
      | :--------------------------- | :-------------------------------------------------------------------------------------- |
      | ID-merged.tsv                | 列出该样本检测到的所有CNV事件​，格式与04_CNVs-results 文件夹中的 CNVs-all.tsv 相同，每行一个CNV，包括位置、类型、影响基因、质量评分等字段 |
      | ID-merged_targets-only.tsv   | 列出该样本中限定在捕获目标区域内的CNV                                                                    |
      | 子文件夹                         | 每个样本文件夹内还包含若干子目录，存放该样本的CNV图形输出                                                          |
      | plots_on-targets_off-targets | 包含该样本最佳CNV（通常是按显著性排序的前若干个CNV，包括发生在on-target域及off-target的CNV）的pdf（all及HQ）                |
      | plots_on-targets-only        | 只有on-target区域的最佳CNV的pdf                                                                 |
      | plots_genome                 | 提供该样本全基因组范围的覆盖度可视化图表（全部区域及off-target区域）                                                 |
      | plots_chromosomes            | 提供逐条染色体的覆盖度分布图（全部区域及off-target区域）                                                       |

   8. CNV文本文件中的字段解释

      | 字段                        | 解释                                                                                                                                                            |
      | :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------ |
      | ID                        | Sample名称                                                                                                                                                      |
      | Chromosome                | 染色体                                                                                                                                                           |
      | Begin                     | CNV起始坐标                                                                                                                                                       |
      | End                       | CNV结束坐标                                                                                                                                                       |
      | Begin-min                 | 检测到的CNV最小起始位置。如果CNV起始区域有低质量的捕获目标，这个值可能比Begin更左（更小），表示在考虑那些低质量点时CNV可能开始的位置                                                                                     |
      | End-max                   | 检测到的CNV最大结束位置。类似地，如果CNV末端包含一些低质量目标位点，此值可能比End更右（更大）。 Begin-min 和 End-max 提供了CNV边界的不确定范围                                                                       |
      | Type                      | CNV类型，缺失(deletion) 或 扩增uplication)​                                                                                                                           |
      | Ploidy                    | 倍性，即根据Z-score计算出的该CNV区域最可能的拷贝数状态                                                                                                                              |
      | Nb_targets                | 该CNV覆盖的捕获目标数量。                                                                                                                                                |
      | Targets                   | CNV所包含的on-target ID 列表                                                                                                                                        |
      | Genes                     | 受到该CNV完全覆盖或部分影响的基因列表                                                                                                                                          |
      | Exons                     | 受到CNV影响的外显子编号或列表                                                                                                                                              |
      | Genes-possible            | 可能受影响的基因列表，基于Begin-min到End-max的最大全边界范围（包括低质量目标）得到                                                                                                             |
      | ncRNAs                    | CNV影响到的非编码RNA（如miRNA, lncRNA等）的列表，这些非蛋白编码的功能元件在变异报告中单独列出，帮助评估CNV对调控元件或非编码基因的影响                                                                                |
      | RefSeq_Functional_Element | CNV涉及的RefSeq功能元件（Functional Element）列表。RefSeq功能元件包括一些已注释的调控序列、增强子、启动子等非基因区功能元件                                                                                |
      | Nb_overlapping_samples    | 在所有Sample中有多少其他样本携带了与此CNV重叠的变异                                                                                                                                |
      | gnomAD-CNV_1%             | 来自gnomAD数据库中频率大于1%的CNV列表，与此CNV区域相重叠的条目                                                                                                                        |
      | gnomAD-CNV_ALL            | 来自gnomAD数据库的所有重叠CNV列表（不论频率）                                                                                                                                   |
      | Counts                    | 该CNV区域内所有目标的总测序覆盖深度                                                                                                                                           |
      | Average_counts            | 对照组样本在相同目标区域的平均总覆盖深度                                                                                                                                          |
      | SD_counts                 | 对照样本在该区域覆盖度的标准差                                                                                                                                               |
      | Ratio                     | 样本覆盖度与对照平均覆盖度的比值，计算方法通常是 Ratio = Counts / Average_counts                                                                                                      |
      | Z-score_cor               | 校正后的Z分数，这是根据样本覆盖度相对于对照均值的偏差，结合总体变异情况计算的Z值，并经过校正（例如去除了主成分噪声）得到更准确的显著性评分。Z-score数值表示该区域偏离正常的程度（以标准差计）：例如-4或+4表示显著低于或高于对照均值。校正后Z分数用于判定CNV的统计显著性。                 |
      | PQ                        | 倍性质量（Ploidy Quality）评分，用来评价拷贝数倍性预测的可靠性                                                                                                                        |
      | CQ                        | CNV质量评分，根据覆盖度分布形状一致性给出的评分                                                                                                                                     |
      | QUAL                      | 综合质量评分，结合PQ和CQ等因素给出的总体质量值，用于总体评价该CNV的可靠程度，越高越好                                                                                                                |
      | Rank_sample               | 按绝对Z分数对本样本内CNV排序的名次。Rank=1表示这是该样本中Z-score绝对值最大的事件，即最显著的CNV                                                                                                    |
      | Nb_CNV_HQ                 | 该样本中高质量CNV（HQ）事件的数量                                                                                                                                           |
      | ClinVar-patho_included    | ClinVar数据库中记录的、完全包含于此CNV区域的所有致病/可能致病CNV列表                                                                                                                     |
      | ClinVar-patho_overlap     | ClinVar数据库中记录的、与此CNV部分重叠的致病CNV列表                                                                                                                              |
      | Nb-exon-before-chr        | 此CNV起始位置之前，该染色体上剩余的（在捕获设计中的）外显子数量。例如，值为0表示该CNV起点已经是该染色体所覆盖的第一个目标外显子；值为5表示在CNV起点之前还有5个已捕获的外显子目标没有受影响。这个指标可以推测CNV是否可能延伸到捕获区域之外（如果为0，说明CNV可能还向染色体更前端延伸但未被捕获区域覆盖） |
      | Nb-exon-after-chr         | 此 CNV 结束位置之后，该染色体上仍剩余的捕获外显子数量。值为0表示CNV已经延伸到该染色体设计捕获的最后一个目标；非0表示在CNV末端之后还有若干目标未受影响。结合前一指标，可以判断CNV是否位于捕获范围中间，还是靠近两端（靠近两端时需考虑实际CNV可能超出捕获区域）                      |

#### Step4. 04_OFF-PEAK-plot.R

在 Step3 中，每个样本的 20 个最佳 CNV 会被自动绘制出来。此脚本用于进一步绘制其他 CNV。主脚本 04_OFF-PEAK-plot.R 以 Step3 的中间数据为输入，生成额外的图形输出

**code示例（未尝试）：**

```markup
Rscript 04_OFF-PEAK-plot.R \
  --ID 样本名称 \
  --chr chr编号 \
  --begin chr起始位置 \
  --end chr结束位置 \
  --batch HCC1395_output_directory \
  --out 定义输出目录 \
  --databasefile data-hg38.RData 
```

1. 参数：

   | 选项     | 默认值 | 取值范围  | 描述             |
   | :----- | :-- | :---- | :------------- |
   | --side | 20  | 1-100 | 在该区域左右各绘制的目标数量 |

