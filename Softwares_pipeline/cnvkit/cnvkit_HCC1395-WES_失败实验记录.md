### CNVkit检测检测HCC1395的12对WES

###### data：

**WES数据：** hg38-bam：/data/share/liuyuxin_tanrenjie/HCC1395_data/WES

**bed文件：**/data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_modify.bed

**FASTA：**/data/share/1000GP_data/technology/reference/GRCh38_full_analysis_set_plus_decoy_hla.fa

**Tools文件夹：**/data/liuyuxin/Project-7-tools/cnvkit/cnvkit_HCC1395_WES_output

进入指令位置：

```markup
source ~/.bashrc
conda activate r44
cd /data/share/liuyuxin_tanrenjie/OFF-PEAK_HCC1395-WES_output
```

#### Step1. 生成target和off-target区域bed

1. 生成基因组可访问域 BED

   ```markup
   cnvkit.py access /data/share/1000GP_data/technology/reference/GRCh38_full_analysis_set_plus_decoy_hla.fa \
   -o GRCh38_access.bed
   ```

   output：

   | 文件                | 解释                                                                                   | 用处                                                                                                                    |
   | :---------------- | :----------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------- |
   | GRCh38_access.bed | 记录了基于 GRCh38中那些可被测序覆盖的区域的坐标。简单来说，它包含了基因组中“可及”的区域，即那些不被长序列“N”（通常代表缺失、重复或难以测序的区域）填充的区域 | 1. 生成off-targrt bins：在 WES 或靶向捕获数据中，CNVkit 使用这个文件来确定基因组中哪些区域是可靠的测序区域，并在这些区域内生成非目标区域的 bins，用以补充目标区域的信息，从而提高拷贝数变异检测的准确性 |
   |                   |                                                                                      | 2.排除无法测序的区域：例如着丝粒、端粒和极高重复的区域通常在参考基因组中被用“N”填充，这些区域在文件中会被排除，从而避免在这些区域内生成噪音较大的 bins                                      |

2. 根据on-target 区域BED 生成优化的目标区域 BED，并注释基因名称

   ```markup
   cnvkit.py target /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_modify.bed \
   --annotate /data/liuyuxin/Project_Data/refFlat.txt \
   --split -o Exome_targets.bed
   ```

   output：

   | 文件                | 解释                                                                                                              | 操作                                                                                |
   | :---------------- | :-------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------- |
   | Exome_targets.bed | 处理 WES（外显子组测序）数据时生成的 目标区域 BED 文件，用于指定 CNV 分析时感兴趣的基因组区域（主要是外显子区域）。这个文件是从原始捕获 BED 文件（S07604624_modify.bed）经过优化得到的 | 1. 去除过长的靶区，将过长的靶区拆分成更均匀的小段（bins）；2. 添加基因注释refFlat；3. 优化 bin 长度 以适配 CNVkit 的统计分析需求 |

3. 根据可及区域和目标区域，生成off-target区域bed

   ```markup
   cnvkit.py antitarget Exome_targets.bed -g GRCh38_access.bed -o Exome_off-targets.bed
   ```

   output：Exome_off-targets.bed

#### Step2. 计算覆盖度

计算每个样本在目标区域和非目标区域的测序覆盖深度。对于每个正常和肿瘤 BAM 文件，使用cnvkit.py coverage分别计算其在目标区域和非目标区域的读段深度计数

```Shell
# WES_EA_N_1-normal1
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_N_1.bwa.dedup.bam Exome_targets.bed -o WES_EA_N_1_normal1.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_N_1.bwa.dedup.bam Exome_off-targets.bed -o WES_EA_N_1_normal1.off-target_coverage.cnn
# WES_EA_T_1-tumor1
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_T_1.bwa.dedup.bam Exome_targets.bed -o WES_EA_T_1_tumor1.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_T_1.bwa.dedup.bam Exome_off-targets.bed -o WES_EA_T_1_tumor1.off-target_coverage.cnn
# WES_FD_N_1-normal2
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_N_1.bwa.dedup.bam Exome_targets.bed -o -o WES_FD_N_1_normal2.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_N_1.bwa.dedup.bam Exome_off-targets.bed -o -o WES_FD_N_1_normal2.off-target_coverage.cnn
# WES_FD_T_1-tumor2
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_T_1.bwa.dedup.bam Exome_targets.bed -o WES_FD_T_1_tumor2.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_T_1.bwa.dedup.bam Exome_off-targets.bed -o WES_FD_T_1_tumor2.off-target_coverage.cnn
# WES_FD_N_2-normal3
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_N_2.bwa.dedup.bam Exome_targets.bed -o WES_FD_N_2_normal3.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_N_2.bwa.dedup.bam Exome_off-targets.bed -o WES_FD_N_2_normal3.off-target_coverage.cnn
# WES_FD_T_2-tumor3
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_T_2.bwa.dedup.bam Exome_targets.bed -o WES_FD_T_2_tumor3.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_T_2.bwa.dedup.bam Exome_off-targets.bed -o WES_FD_T_2_tumor3.off-target_coverage.cnn
# WES_FD_N_3-normal4
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_N_3.bwa.dedup.bam Exome_targets.bed -o WES_FD_N_3_normal4.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_N_3.bwa.dedup.bam Exome_off-targets.bed -o WES_FD_N_3_normal4.off-target_coverage.cnn
# WES_FD_T_3-tumor4
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_T_3.bwa.dedup.bam Exome_targets.bed -o WES_FD_T_3_tumor4.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_T_3.bwa.dedup.bam Exome_off-targets.bed -o WES_FD_T_3_tumor4.off-target_coverage.cnn
# WES_IL_N_1-normal5
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_N_1.bwa.dedup.bam Exome_targets.bed -o WES_IL_N_1_normal5.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_N_1.bwa.dedup.bam Exome_off-targets.bed -o WES_IL_N_1_normal5.off-target_coverage.cnn
# WES_IL_T_1-tumor5
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_T_1.bwa.dedup.bam Exome_targets.bed -o WES_IL_T_1_tumor5.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_T_1.bwa.dedup.bam Exome_off-targets.bed -o WES_IL_T_1_tumor5.off-target_coverage.cnn
# WES_IL_N_2-normal6
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_N_2.bwa.dedup.bam Exome_targets.bed -o WES_IL_N_2_normal6.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_N_2.bwa.dedup.bam Exome_off-targets.bed -o WES_IL_N_2_normal6.off-target_coverage.cnn
# WES_IL_T_2-tumor6
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_T_2.bwa.dedup.bam Exome_targets.bed -o WES_IL_T_2_tumor6.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_T_2.bwa.dedup.bam Exome_off-targets.bed -o WES_IL_T_2_tumor6.off-target_coverage.cnn
# WES_IL_N_3-normal7
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_N_3.bwa.dedup.bam Exome_targets.bed -o WES_IL_N_3_normal7.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_N_3.bwa.dedup.bam Exome_off-targets.bed -o WES_IL_N_3_normal7.off-target_coverage.cnn
# WES_IL_T_3-tumor7
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_T_3.bwa.dedup.bam Exome_targets.bed -o WES_IL_T_3_tumor7.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_T_3.bwa.dedup.bam Exome_off-targets.bed -o WES_IL_T_3_tumor7.off-target_coverage.cnn
# WES_LL_N_1-normal8
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_LL_N_1.bwa.dedup.bam Exome_targets.bed -o WES_LL_N_1_normal8.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_LL_N_1.bwa.dedup.bam Exome_off-targets.bed -o WES_LL_N_1_normal8.off-target_coverage.cnn
# WES_LL_T_1-tumor8
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_LL_T_1.bwa.dedup.bam Exome_targets.bed -o WES_LL_T_1_tumor8.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_LL_T_1.bwa.dedup.bam Exome_off-targets.bed -o WES_LL_T_1_tumor8.off-target_coverage.cnn
# WES_NC_N_1-normal9
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NC_N_1.bwa.dedup.bam Exome_targets.bed -o WES_NC_N_1_normal9.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NC_N_1.bwa.dedup.bam Exome_off-targets.bed -o WES_NC_N_1_normal9.off-target_coverage.cnn
# WES_NC_T_1-tumor9
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NC_T_1.bwa.dedup.bam Exome_targets.bed -o WES_NC_T_1_tumor9.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NC_T_1.bwa.dedup.bam Exome_off-targets.bed -o WES_NC_T_1_tumor9.off-target_coverage.cnn
# WES_NV_N_1-normal10
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_N_1.bwa.dedup.bam Exome_targets.bed -o WES_NV_N_1_normal10.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_N_1.bwa.dedup.bam Exome_off-targets.bed -o WES_NV_N_1_normal10.off-target_coverage.cnn
# WES_NV_T_1-tumor10
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_T_1.bwa.dedup.bam Exome_targets.bed -o WES_NV_T_1_tumor10.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_T_1.bwa.dedup.bam Exome_off-targets.bed -o WES_NV_T_1_tumor10.off-target_coverage.cnn
# WES_NV_N_2-normal11
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_N_2.bwa.dedup.bam Exome_targets.bed -o WES_NV_N_2_normal11.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_N_2.bwa.dedup.bam Exome_off-targets.bed -o WES_NV_N_2_normal11.off-target_coverage.cnn
# WES_NV_T_2-tumor11
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_T_2.bwa.dedup.bam Exome_targets.bed -o WES_NV_T_2_tumor11.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_T_2.bwa.dedup.bam Exome_off-targets.bed -o WES_NV_T_2_tumor11.off-target_coverage.cnn
# WES_NV_N_3-normal12
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_N_3.bwa.dedup.bam Exome_targets.bed -o WES_NV_N_3_normal12.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_N_3.bwa.dedup.bam Exome_off-targets.bed -o WES_NV_N_3_normal12.off-target_coverage.cnn
# WES_NV_T_3-tumor12
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_T_3.bwa.dedup.bam Exome_targets.bed -o WES_NV_T_3_tumor12.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_T_3.bwa.dedup.bam Exome_off-targets.bed -o WES_NV_T_3_tumor12.target_coverage.cnn
```

```markup
#!/bin/bash
# run_coverage.sh
# 此脚本用于计算 12 对 WES 样本（正常和肿瘤）的目标区域和非目标区域覆盖度。
# 请确认 Exome_targets.bed 和 Exome_off-targets.bed 文件位于当前目录下（或调整路径）。

# 计算第1对样本: WES_EA_N_1 (normal1) 和 WES_EA_T_1 (tumor1)
echo "Processing WES_EA_N_1 (normal1)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_N_1.bwa.dedup.bam Exome_targets.bed -o WES_EA_N_1_normal1.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_N_1.bwa.dedup.bam Exome_off-targets.bed -o WES_EA_N_1_normal1.off-target_coverage.cnn

echo "Processing WES_EA_T_1 (tumor1)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_T_1.bwa.dedup.bam Exome_targets.bed -o WES_EA_T_1_tumor1.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_T_1.bwa.dedup.bam Exome_off-targets.bed -o WES_EA_T_1_tumor1.off-target_coverage.cnn

# 第2对样本: WES_FD_N_1 (normal2) 和 WES_FD_T_1 (tumor2)
echo "Processing WES_FD_N_1 (normal2)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_N_1.bwa.dedup.bam Exome_targets.bed -o WES_FD_N_1_normal2.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_N_1.bwa.dedup.bam Exome_off-targets.bed -o WES_FD_N_1_normal2.off-target_coverage.cnn

echo "Processing WES_FD_T_1 (tumor2)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_T_1.bwa.dedup.bam Exome_targets.bed -o WES_FD_T_1_tumor2.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_T_1.bwa.dedup.bam Exome_off-targets.bed -o WES_FD_T_1_tumor2.off-target_coverage.cnn

# 第3对样本: WES_FD_N_2 (normal3) 和 WES_FD_T_2 (tumor3)
echo "Processing WES_FD_N_2 (normal3)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_N_2.bwa.dedup.bam Exome_targets.bed -o WES_FD_N_2_normal3.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_N_2.bwa.dedup.bam Exome_off-targets.bed -o WES_FD_N_2_normal3.off-target_coverage.cnn

echo "Processing WES_FD_T_2 (tumor3)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_T_2.bwa.dedup.bam Exome_targets.bed -o WES_FD_T_2_tumor3.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_T_2.bwa.dedup.bam Exome_off-targets.bed -o WES_FD_T_2_tumor3.off-target_coverage.cnn

# 第4对样本: WES_FD_N_3 (normal4) 和 WES_FD_T_3 (tumor4)
echo "Processing WES_FD_N_3 (normal4)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_N_3.bwa.dedup.bam Exome_targets.bed -o WES_FD_N_3_normal4.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_N_3.bwa.dedup.bam Exome_off-targets.bed -o WES_FD_N_3_normal4.off-target_coverage.cnn

echo "Processing WES_FD_T_3 (tumor4)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_T_3.bwa.dedup.bam Exome_targets.bed -o WES_FD_T_3_tumor4.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_T_3.bwa.dedup.bam Exome_off-targets.bed -o WES_FD_T_3_tumor4.off-target_coverage.cnn

# 第5对样本: WES_IL_N_1 (normal5) 和 WES_IL_T_1 (tumor5)
echo "Processing WES_IL_N_1 (normal5)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_N_1.bwa.dedup.bam Exome_targets.bed -o WES_IL_N_1_normal5.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_N_1.bwa.dedup.bam Exome_off-targets.bed -o WES_IL_N_1_normal5.off-target_coverage.cnn

echo "Processing WES_IL_T_1 (tumor5)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_T_1.bwa.dedup.bam Exome_targets.bed -o WES_IL_T_1_tumor5.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_T_1.bwa.dedup.bam Exome_off-targets.bed -o WES_IL_T_1_tumor5.off-target_coverage.cnn

# 第6对样本: WES_IL_N_2 (normal6) 和 WES_IL_T_2 (tumor6)
echo "Processing WES_IL_N_2 (normal6)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_N_2.bwa.dedup.bam Exome_targets.bed -o WES_IL_N_2_normal6.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_N_2.bwa.dedup.bam Exome_off-targets.bed -o WES_IL_N_2_normal6.off-target_coverage.cnn

echo "Processing WES_IL_T_2 (tumor6)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_T_2.bwa.dedup.bam Exome_targets.bed -o WES_IL_T_2_tumor6.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_T_2.bwa.dedup.bam Exome_off-targets.bed -o WES_IL_T_2_tumor6.off-target_coverage.cnn

# 第7对样本: WES_IL_N_3 (normal7) 和 WES_IL_T_3 (tumor7)
echo "Processing WES_IL_N_3 (normal7)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_N_3.bwa.dedup.bam Exome_targets.bed -o WES_IL_N_3_normal7.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_N_3.bwa.dedup.bam Exome_off-targets.bed -o WES_IL_N_3_normal7.off-target_coverage.cnn

echo "Processing WES_IL_T_3 (tumor7)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_T_3.bwa.dedup.bam Exome_targets.bed -o WES_IL_T_3_tumor7.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_T_3.bwa.dedup.bam Exome_off-targets.bed -o WES_IL_T_3_tumor7.off-target_coverage.cnn

# 第8对样本: WES_LL_N_1 (normal8) 和 WES_LL_T_1 (tumor8)
echo "Processing WES_LL_N_1 (normal8)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_LL_N_1.bwa.dedup.bam Exome_targets.bed -o WES_LL_N_1_normal8.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_LL_N_1.bwa.dedup.bam Exome_off-targets.bed -o WES_LL_N_1_normal8.off-target_coverage.cnn

echo "Processing WES_LL_T_1 (tumor8)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_LL_T_1.bwa.dedup.bam Exome_targets.bed -o WES_LL_T_1_tumor8.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_LL_T_1.bwa.dedup.bam Exome_off-targets.bed -o WES_LL_T_1_tumor8.off-target_coverage.cnn

# 第9对样本: WES_NC_N_1 (normal9) 和 WES_NC_T_1 (tumor9)
echo "Processing WES_NC_N_1 (normal9)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NC_N_1.bwa.dedup.bam Exome_targets.bed -o WES_NC_N_1_normal9.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NC_N_1.bwa.dedup.bam Exome_off-targets.bed -o WES_NC_N_1_normal9.off-target_coverage.cnn

echo "Processing WES_NC_T_1 (tumor9)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NC_T_1.bwa.dedup.bam Exome_targets.bed -o WES_NC_T_1_tumor9.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NC_T_1.bwa.dedup.bam Exome_off-targets.bed -o WES_NC_T_1_tumor9.off-target_coverage.cnn

# 第10对样本: WES_NV_N_1 (normal10) 和 WES_NV_T_1 (tumor10)
echo "Processing WES_NV_N_1 (normal10)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_N_1.bwa.dedup.bam Exome_targets.bed -o WES_NV_N_1_normal10.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_N_1.bwa.dedup.bam Exome_off-targets.bed -o WES_NV_N_1_normal10.off-target_coverage.cnn

echo "Processing WES_NV_T_1 (tumor10)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_T_1.bwa.dedup.bam Exome_targets.bed -o WES_NV_T_1_tumor10.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_T_1.bwa.dedup.bam Exome_off-targets.bed -o WES_NV_T_1_tumor10.off-target_coverage.cnn

# 第11对样本: WES_NV_N_2 (normal11) 和 WES_NV_T_2 (tumor11)
echo "Processing WES_NV_N_2 (normal11)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_N_2.bwa.dedup.bam Exome_targets.bed -o WES_NV_N_2_normal11.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_N_2.bwa.dedup.bam Exome_off-targets.bed -o WES_NV_N_2_normal11.off-target_coverage.cnn

echo "Processing WES_NV_T_2 (tumor11)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_T_2.bwa.dedup.bam Exome_targets.bed -o WES_NV_T_2_tumor11.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_T_2.bwa.dedup.bam Exome_off-targets.bed -o WES_NV_T_2_tumor11.off-target_coverage.cnn

# 第12对样本: WES_NV_N_3 (normal12) 和 WES_NV_T_3 (tumor12)
echo "Processing WES_NV_N_3 (normal12)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_N_3.bwa.dedup.bam Exome_targets.bed -o WES_NV_N_3_normal12.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_N_3.bwa.dedup.bam Exome_off-targets.bed -o WES_NV_N_3_normal12.off-target_coverage.cnn

echo "Processing WES_NV_T_3 (tumor12)..."
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_T_3.bwa.dedup.bam Exome_targets.bed -o WES_NV_T_3_tumor12.target_coverage.cnn
cnvkit.py coverage /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_T_3.bwa.dedup.bam Exome_off-targets.bed -o WES_NV_T_3_tumor12.off-target_coverage.cnn

echo "Coverage calculation completed for all samples."

```

Step3.步骤3：构建参考 (Create Reference)

```markup
cnvkit.py reference /data/liuyuxin/Project-7-tools/cnvkit/cnvkit_HCC1395_WES_output/*_normal*.target_coverage.cnn \
  /data/liuyuxin/Project-7-tools/cnvkit/cnvkit_HCC1395_WES_output/*_normal*.off-target_coverage.cnn \
-f /data/share/1000GP_data/technology/reference/GRCh38_full_analysis_set_plus_decoy_hla.fa \
-o HCC1395_WES_normal_reference.cnn
```

失败：

Number of target and antitarget files: 24, 0
Sample sex not provided; inferring from samples.
Relative log2 coverage of chrX=-0.925, chrY=-23.8 (maleness=16.5 x 0.891 = 14.7) --> assuming male
Relative log2 coverage of chrX=-0.884, chrY=-23.9 (maleness=12 x 0.897 = 10.8) --> assuming male
Relative log2 coverage of chrX=-0.888, chrY=-23.9 (maleness=12.6 x 0.897 = 11.3) --> assuming male
Relative log2 coverage of chrX=-0.92, chrY=-23.9 (maleness=13.9 x 0.899 = 12.5) --> assuming male
Relative log2 coverage of chrX=-0.862, chrY=-25.1 (maleness=9.05 x 0.891 = 8.07) --> assuming male
Relative log2 coverage of chrX=-0.798, chrY=-26.1 (maleness=7.61 x 0.908 = 6.91) --> assuming male
Relative log2 coverage of chrX=-0.766, chrY=-25.1 (maleness=6.65 x 0.902 = 6) --> assuming male
Relative log2 coverage of chrX=-0.965, chrY=-24.3 (maleness=12.1 x 0.902 = 11) --> assuming male
Relative log2 coverage of chrX=-0.861, chrY=-24.7 (maleness=8.63 x 0.902 = 7.78) --> assuming male
Relative log2 coverage of chrX=-0.815, chrY=-25.6 (maleness=5.89 x 0.914 = 5.38) --> assuming male
Relative log2 coverage of chrX=-0.818, chrY=-25.5 (maleness=5.54 x 0.908 = 5.03) --> assuming male
Relative log2 coverage of chrX=-0.813, chrY=-20.7 (maleness=5.88 x 0.908 = 5.34) --> assuming male
Relative log2 coverage of chrX=-1.14, chrY=-9.86 (maleness=19.7 x 0.751 = 14.8) --> assuming male
Relative log2 coverage of chrX=-1.17, chrY=-10.9 (maleness=22.9 x 0.712 = 16.3) --> assuming male
Relative log2 coverage of chrX=-1.17, chrY=-10.8 (maleness=23.6 x 0.731 = 17.3) --> assuming male
Relative log2 coverage of chrX=-1.16, chrY=-10.6 (maleness=21.2 x 0.751 = 15.9) --> assuming male
Relative log2 coverage of chrX=-1.08, chrY=-9.94 (maleness=29.3 x 0.78 = 22.9) --> assuming male
Relative log2 coverage of chrX=-0.988, chrY=-8.28 (maleness=142 x 0.739 = 105) --> assuming male
Relative log2 coverage of chrX=-0.986, chrY=-8.53 (maleness=144 x 0.78 = 113) --> assuming male
Relative log2 coverage of chrX=-1.14, chrY=-10.8 (maleness=47.1 x 0.751 = 35.4) --> assuming male
Relative log2 coverage of chrX=-1.08, chrY=-10.8 (maleness=26 x 0.731 = 19) --> assuming male
Relative log2 coverage of chrX=-1.05, chrY=-9.07 (maleness=45.8 x 0.712 = 32.6) --> assuming male
Relative log2 coverage of chrX=-1.06, chrY=-9.16 (maleness=54.3 x 0.712 = 38.6) --> assuming male
Relative log2 coverage of chrX=-1.07, chrY=-8.88 (maleness=39.7 x 0.731 = 29) --> assuming male
Loading /data/liuyuxin/Project-7-tools/cnvkit/cnvkit_HCC1395_WES_output/WES_EA_N_1_normal1.off-target_coverage.cnn
Calculating GC and RepeatMasker content in /data/share/1000GP_data/technology/reference/GRCh38_full_analysis_set_plus_decoy_hla.fa ...
Extracting sequences from chromosome chr1
Extracting sequences from chromosome chr2
Extracting sequences from chromosome chr3
Extracting sequences from chromosome chr4
Extracting sequences from chromosome chr5
Extracting sequences from chromosome chr6
Extracting sequences from chromosome chr7
Extracting sequences from chromosome chr8
Extracting sequences from chromosome chr9
Extracting sequences from chromosome chr10
Extracting sequences from chromosome chr11
Extracting sequences from chromosome chr12
Extracting sequences from chromosome chr13
Extracting sequences from chromosome chr14
Extracting sequences from chromosome chr15
Extracting sequences from chromosome chr16
Extracting sequences from chromosome chr17
Extracting sequences from chromosome chr18
Extracting sequences from chromosome chr19
Extracting sequences from chromosome chr20
Extracting sequences from chromosome chr21
Extracting sequences from chromosome chr22
Extracting sequences from chromosome chrX
Extracting sequences from chromosome chrY
Correcting for GC bias for WES_EA_N_1_normal1.off-target_coverage...
Correcting for density bias for WES_EA_N_1_normal1.off-target_coverage...
Loading /data/liuyuxin/Project-7-tools/cnvkit/cnvkit_HCC1395_WES_output/WES_EA_N_1_normal1.target_coverage.cnn
Traceback (most recent call last):
File "/data/liuyuxin/miniconda3/envs/cnvkit/bin/cnvkit.py", line 10, in&#x20;
sys.exit(main())
File "/data/liuyuxin/miniconda3/envs/cnvkit/lib/python3.9/site-packages/cnvlib/cnvkit.py", line 10, in main
args.func(args)
File "/data/liuyuxin/miniconda3/envs/cnvkit/lib/python3.9/site-packages/cnvlib/commands.py", line 821, in _cmd_reference
ref_probes = reference.do_reference(
File "/data/liuyuxin/miniconda3/envs/cnvkit/lib/python3.9/site-packages/cnvlib/reference.py", line 106, in do_reference
ref_probes = combine_probes(
File "/data/liuyuxin/miniconda3/envs/cnvkit/lib/python3.9/site-packages/cnvlib/reference.py", line 179, in combine_probes
ref_df, all_logr, all_depths = load_sample_block(
File "/data/liuyuxin/miniconda3/envs/cnvkit/lib/python3.9/site-packages/cnvlib/reference.py", line 328, in load_sample_block
raise RuntimeError(
RuntimeError: /data/liuyuxin/Project-7-tools/cnvkit/cnvkit_HCC1395_WES_output/WES_EA_N_1_normal1.target_coverage.cnn bins do not match those in /data/liuyuxin/Project-7-tools/cnvkit/cnvkit_HCC1395_WES_output/WES_EA_N_1_normal1.off-target_coverage.cnn

## 重新运行：

```markup
cnvkit.py batch \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Tumor_1.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_Tumor_1.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_Tumor_2.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_Tumor_3.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_Tumor_1.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_Tumor_2.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_Tumor_3.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_LL_Tumor_1.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NC_Tumor_1.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_Tumor_1.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_Tumor_2.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_Tumor_3.bwa.dedup.bam \
 --normal \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Normal_1.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_Normal_1.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_Normal_2.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_Normal_3.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_Normal_1.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_Normal_2.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_Normal_3.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_LL_Normal_1.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NC_Normal_1.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_Normal_1.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_Normal_2.bwa.dedup.bam \
 /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_Normal_3.bwa.dedup.bam \
--targets /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_modify.bed \
--fasta /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/GRCh38/GRCh38.d1.vd1.fa \
--access /data/liuyuxin/Project-7-tools/cnvkit/cnvkit-output/cnvkit-master/data/access-10kb.hg38.bed \
--output-reference /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/HCC1395_WES_reference.cnn \
--output-dir /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output \
--diagram --scatter \
-p 16
```

输出：

```markup
CNVkit 0.9.12
Detected file format: bed
Splitting large targets
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/S07604624_modify.target.bed with 397288 regions
Detected file format: bed
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/S07604624_modify.antitarget.bed with 41262 regions
Building a copy number reference from normal samples...
Processing reads in WES_FD_Normal_1.bwa.dedup.bam
Processing reads in WES_FD_Normal_1.bwa.dedup.bam
Processing reads in WES_EA_Normal_1.bwa.dedup.bam
Processing reads in WES_EA_Normal_1.bwa.dedup.bam
Processing reads in WES_FD_Normal_3.bwa.dedup.bam
Processing reads in WES_FD_Normal_3.bwa.dedup.bam
Processing reads in WES_LL_Normal_1.bwa.dedup.bam
Processing reads in WES_LL_Normal_1.bwa.dedup.bam
Processing reads in WES_IL_Normal_2.bwa.dedup.bam
Processing reads in WES_IL_Normal_2.bwa.dedup.bam
Processing reads in WES_FD_Normal_2.bwa.dedup.bam
Processing reads in WES_FD_Normal_2.bwa.dedup.bam
Processing reads in WES_IL_Normal_3.bwa.dedup.bam
Processing reads in WES_IL_Normal_3.bwa.dedup.bam
Processing reads in WES_IL_Normal_1.bwa.dedup.bam
Processing reads in WES_IL_Normal_1.bwa.dedup.bam
Time: 45.929 seconds (66230 reads/sec, 898 bins/sec)
Summary: #bins=41262, #reads=3041896, mean=73.7215, min=0.0, max=7715.548638132295
Percent reads in regions: 4.377 (of 69491768 mapped)
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/WES_LL_Normal_1.bwa.dedup.antitargetcoverage.cnn with 41262 regions
Processing reads in WES_NC_Normal_1.bwa.dedup.bam
Time: 68.467 seconds (129529 reads/sec, 603 bins/sec)
Summary: #bins=41262, #reads=8868458, mean=214.9304, min=0.0, max=6835.758241758242
Percent reads in regions: 12.964 (of 68410607 mapped)
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/WES_FD_Normal_1.bwa.dedup.antitargetcoverage.cnn with 41262 regions
Processing reads in WES_NC_Normal_1.bwa.dedup.bam
Time: 76.790 seconds (142224 reads/sec, 537 bins/sec)
Summary: #bins=41262, #reads=10921377, mean=264.6837, min=0.0, max=7801.827067669173
Percent reads in regions: 13.610 (of 80247670 mapped)
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/WES_FD_Normal_3.bwa.dedup.antitargetcoverage.cnn with 41262 regions
Processing reads in WES_NV_Normal_1.bwa.dedup.bam
Time: 77.377 seconds (129931 reads/sec, 533 bins/sec)
Summary: #bins=41262, #reads=10053680, mean=243.6547, min=0.0, max=7082.851063829788
Percent reads in regions: 12.889 (of 78002112 mapped)
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/WES_FD_Normal_2.bwa.dedup.antitargetcoverage.cnn with 41262 regions
Processing reads in WES_NV_Normal_1.bwa.dedup.bam
Time: 111.328 seconds (84301 reads/sec, 371 bins/sec)
Summary: #bins=41262, #reads=9385034, mean=227.4498, min=0.0, max=15786.388888888889
Percent reads in regions: 5.335 (of 175913325 mapped)
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/WES_EA_Normal_1.bwa.dedup.antitargetcoverage.cnn with 41262 regions
Processing reads in WES_NV_Normal_2.bwa.dedup.bam
Time: 141.782 seconds (89514 reads/sec, 291 bins/sec)
Summary: #bins=41262, #reads=12691395, mean=307.5807, min=0.0, max=18875.07894736842
Percent reads in regions: 4.551 (of 278847976 mapped)
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/WES_IL_Normal_1.bwa.dedup.antitargetcoverage.cnn with 41262 regions
Processing reads in WES_NV_Normal_2.bwa.dedup.bam
Time: 119.153 seconds (187093 reads/sec, 346 bins/sec)
Summary: #bins=41262, #reads=22292683, mean=540.2715, min=0.0, max=10847.330434782609
Percent reads in regions: 19.693 (of 113201214 mapped)
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/WES_NC_Normal_1.bwa.dedup.antitargetcoverage.cnn with 41262 regions
Processing reads in WES_NV_Normal_3.bwa.dedup.bam
Processing reads in WES_NV_Normal_3.bwa.dedup.bam
Time: 294.548 seconds (134400 reads/sec, 140 bins/sec)
Summary: #bins=41262, #reads=39587266, mean=959.4122, min=0.0, max=30888.96
Percent reads in regions: 10.027 (of 394790745 mapped)
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/WES_NV_Normal_1.bwa.dedup.antitargetcoverage.cnn with 41262 regions
Time: 283.411 seconds (127022 reads/sec, 146 bins/sec)
Summary: #bins=41262, #reads=35999455, mean=872.4603, min=0.0, max=32157.392
Percent reads in regions: 8.686 (of 414468687 mapped)
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/WES_NV_Normal_2.bwa.dedup.antitargetcoverage.cnn with 41262 regions
Time: 429.094 seconds (169857 reads/sec, 96 bins/sec)
Summary: #bins=41262, #reads=72884741, mean=1766.3890, min=0.0, max=90294.0366972477
Percent reads in regions: 10.418 (of 699576848 mapped)
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/WES_IL_Normal_2.bwa.dedup.antitargetcoverage.cnn with 41262 regions
Time: 459.781 seconds (342157 reads/sec, 90 bins/sec)
Summary: #bins=41262, #reads=157317236, mean=3812.6421, min=0.0, max=232775.47872340426
Percent reads in regions: 41.221 (of 381642309 mapped)
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/WES_IL_Normal_3.bwa.dedup.antitargetcoverage.cnn with 41262 regions
Time: 262.941 seconds (133406 reads/sec, 157 bins/sec)
Summary: #bins=41262, #reads=35078022, mean=850.1290, min=0.0, max=29843.096
Percent reads in regions: 9.389 (of 373602043 mapped)
Wrote /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/WES_NV_Normal_3.bwa.dedup.antitargetcoverage.cnn with 41262 regions

```

失败，生成的/data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/S07604624_modify.target.bed中有chrUn_KI270742v1	32126	32308	-
chrUn_KI270742v1	43987	44107	-
chrUn_KI270742v1	45162	45461	-
chrUn_KI270742v1	68283	68403	-
chrUn_KI270742v1	92624	92744	-
chr1_KI270706v1_random	45944	46086	-
chr1_KI270706v1_random	69450	69570	-
chr1_KI270766v1_alt	1658	1778	-
chr1_KI270766v1_alt	3369	3526	-
chr1_KI270766v1_alt	18548	18758	-
chr1_KI270766v1_alt	19205	19533	-
chr1_KI270766v1_alt	19533	19862	-
chr1_KI270766v1_alt	166779	167108	-
chr1_KI270766v1_alt	167108	167438	-
chr1_KI270766v1_alt	168238	168566	-
chr1_KI270766v1_alt	168566	168895	-
chr4_GL000008v2_random	7367	7491	-
chr4_GL000008v2_random	131811	131954	-
chr4_GL000008v2_random	155305	155425	-
chr4_GL000008v2_random	177447	177567	-
chr4_GL000008v2_random	191480	191600	-
chr7_KI270803v1_alt	364474	364832	-
chr7_KI270803v1_alt	368020	368381	-
chr7_KI270803v1_alt	372831	372953	-
chr7_KI270803v1_alt	373116	373241	-
chr7_KI270803v1_alt	375589	375818	-
chr7_KI270803v1_alt	375818	376047	-
chr7_KI270803v1_alt	383382	383589	-
chr7_KI270803v1_alt	383737	383995	-
chr7_KI270803v1_alt	383999	384119	-
chr7_KI270803v1_alt	391289	391627	-
chr7_KI270803v1_alt	404403	404617	-
chr7_KI270803v1_alt	408444	408651	-
chr7_KI270803v1_alt	408807	409057	-
chr7_KI270803v1_alt	417571	417879	-
chr7_KI270803v1_alt	434497	434620	-
chr7_KI270803v1_alt	434666	434796	-
chr7_KI270803v1_alt	434885	435008	-
chr7_KI270803v1_alt	438806	438935	-
chr7_KI270803v1_alt	439048	439213	-
chr7_KI270803v1_alt	446755	446876	-
chr7_KI270803v1_alt	447040	447161	-
chr7_KI270803v1_alt	453173	453294	-
chr7_KI270803v1_alt	453465	453586	-
chr7_KI270803v1_alt	466173	466338	-
chr7_KI270803v1_alt	466428	466602	-
chr7_KI270803v1_alt	471504	471630	-
chr7_KI270803v1_alt	471692	471813	-
chr7_KI270803v1_alt	475764	475884	-
chr7_KI270803v1_alt	476056	476242	-
chr7_KI270803v1_alt	483650	483858	-
chr7_KI270803v1_alt	483932	484128	-
chr7_KI270803v1_alt	491017	491138	-
chr7_KI270803v1_alt	491262	491383	-
chr7_KI270803v1_alt	495150	495280	-
chr7_KI270803v1_alt	495455	495705	-
chr7_KI270803v1_alt	503707	503874	-
chr7_KI270803v1_alt	503962	504137	-
chr7_KI270803v1_alt	511035	511161	-
chr7_KI270803v1_alt	511231	511396	-
chr7_KI270803v1_alt	515699	515820	-
chr7_KI270803v1_alt	515891	516021	-
chr7_KI270803v1_alt	773426	773562	-
chr7_KI270803v1_alt	774701	774822	-
chr7_KI270803v1_alt	775937	776063	-
chr7_KI270803v1_alt	776477	776751	-
chr7_KI270803v1_alt	776911	777031	-
chr7_KI270803v1_alt	777036	777156	-
chr14_GL000009v2_random	22080	22262	-
chr14_GL000009v2_random	33926	34046	-
chr14_GL000009v2_random	35102	35322	-
chr14_GL000009v2_random	58161	58281	-
chr14_GL000009v2_random	81300	81442	-
chr14_GL000009v2_random	122549	122669	-
chr15_KI270850v1_alt	8528	8893	-
chr15_KI270850v1_alt	11035	11155	-
chr15_KI270850v1_alt	11160	11280	-
chr15_KI270850v1_alt	26294	26474	-
chr15_KI270850v1_alt	37747	37867	-
chr15_KI270850v1_alt	52750	52917	-
chr15_KI270850v1_alt	53133	53337	-
chr15_KI270850v1_alt	54031	54154	-
chr15_KI270850v1_alt	54209	54427	-
chr15_KI270850v1_alt	90276	90396	-
chr15_KI270850v1_alt	90665	90870	-
chr15_KI270850v1_alt	93255	93375	-
chr15_KI270850v1_alt	93425	93632	-
chr15_KI270850v1_alt	94933	95155	-
chr15_KI270850v1_alt	98418	98538	-
chr15_KI270850v1_alt	100493	100613	-
chr15_KI270850v1_alt	104204	104480	-
chr15_KI270850v1_alt	104480	104757	-
chr15_KI270850v1_alt	104757	105033	-
chr15_KI270850v1_alt	105033	105310	-
chr15_KI270850v1_alt	105310	105587	-
chr15_KI270850v1_alt	106725	106845	-
chr15_KI270850v1_alt	108454	108656	-
chr15_KI270850v1_alt	113410	113530	-
chr15_KI270850v1_alt	121379	121499	-
chr15_KI270850v1_alt	124735	124942	-
chr15_KI270850v1_alt	124992	125112	-
chr15_KI270850v1_alt	125125	125245	-
chr15_KI270850v1_alt	131711	131878	-
chr15_KI270850v1_alt	132094	132298	-
chr15_KI270850v1_alt	132992	133115	-
chr15_KI270850v1_alt	133170	133388	-
chr15_KI270850v1_alt	133617	133941	-
chr15_KI270850v1_alt	134265	134529	-
chr15_KI270850v1_alt	134822	135013	-
chr15_KI270850v1_alt	135555	135761	-
chr15_KI270850v1_alt	135891	136088	-
chr15_KI270850v1_alt	136602	136795	-
chr15_KI270850v1_alt	138297	138483	-
chr17_KI270909v1_alt	237465	237665	-
chr17_KI270909v1_alt	238015	238136	-
chr17_KI270909v1_alt	250996	251196	-
chr17_KI270909v1_alt	251585	251715	-
chr17_KI270909v1_alt	252074	252468	-
chr17_KI270909v1_alt	266862	266986	-
chr17_KI270909v1_alt	267145	267265	-
chr17_KI270909v1_alt	267558	267850	-
chr17_KI270909v1_alt	268047	268303	-
chr19_KI270938v1_alt	273131	273298	-
chr19_KI270938v1_alt	273559	273679	-
chr19_KI270938v1_alt	273684	273850	-
chr19_KI270938v1_alt	274141	274357	-
chr19_KI270938v1_alt	274598	274808	-
chr19_KI270938v1_alt	274992	275113	-
chr19_KI270938v1_alt	275311	275433	-
chr19_KI270938v1_alt	275556	275778	-
chr19_KI270938v1_alt	275818	275946	-
chr22_KI270879v1_alt	237661	237791	-
chr22_KI270879v1_alt	239817	240026	-
chr22_KI270879v1_alt	267366	267491	-
chr22_KI270879v1_alt	267701	267831	-
chr22_KI270879v1_alt	267921	268051	-
chr22_KI270879v1_alt	270300	270497	-
chr22_KI270879v1_alt	270536	270744	-
chr22_KI270879v1_alt	270966	271149	-
chr22_KI270879v1_alt	271916	272043	-
chr22_KI270879v1_alt	273383	273745	-
chr22_KI270879v1_alt	275710	275974	-
chr22_KI270879v1_alt	276877	277179	-
chr22_KI270879v1_alt	278020	278141	-
chr22_KI270879v1_alt	278247	278367	-
chr22_KI270879v1_alt	278370	278521	-
chr22_KI270879v1_alt	290405	290525	-

解决办法：

发现问题出现/data/liuyuxin/Project-7-tools/cnvkit/cnvkit-0.9.12/data/refFlat_hg38.txt上，删除refFlat_hg38.txt里的非标准染色体

```markup
awk '$3 ~ /^chr([0-9]+|X|Y)$/' /data/liuyuxin/Project-7-tools/cnvkit/cnvkit-0.9.12/data/refFlat_hg38.txt > /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/refFlat_hg38.standard.txt
```

验证

```markup
awk '{print $3}' refFlat_hg38.standard.txt | sort | uniq

#输出
(cnvkit) liuyuxin@A3:/data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output$ awk '{print $3}' refFlat_hg38.standard.txt | sort | uniq
chr1
chr10
chr11
chr12
chr13
chr14
chr15
chr16
chr17
chr18
chr19
chr2
chr20
chr21
chr22
chr3
chr4
chr5
chr6
chr7
chr8
chr9
chrX
chrY
成功
```

重新运行：

```markup
cnvkit.py batch \
  /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/*Tumor*.bam \
  --normal /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/*Normal*.bam \
  --targets /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_modify.bed \
  --annotate /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/refFlat_hg38.standard.txt \
  --fasta /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/GRCh38/GRCh38.d1.vd1.fa \
  --access /data/liuyuxin/Project-7-tools/cnvkit/cnvkit-0.9.12/data/access-10kb.hg38.bed \
  --output-reference /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/HCC1395_WES_reference.cnn \
  --output-dir /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output \
  --diagram \
  --scatter \
  -p 20

```

仍然出现原问题

猜测为access文件的问题：

不用data里自带的，自己生成：

```markup
cnvkit.py access GRCh38.d1.vd1.fa \
    -o access.hg38.bed
# 清理非标准染色体
grep -E "^chr([0-9]+|X|Y)\b" access.hg38.bed > access.standard.hg38.bed
```

运行

```markup
cnvkit.py batch \
  /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/*Tumor*.bam \
  --normal /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/*Normal*.bam \
  --targets /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_modify.bed \
  --annotate /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/refFlat_hg38.standard.txt \
  --fasta /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/GRCh38/GRCh38.d1.vd1.fa \
  --access /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/access.standard.hg38.bed \
  --output-reference /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/HCC1395_WES_reference.cnn \
  --output-dir /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output \
  --diagram \
  --scatter \
  -p 20
```

失败，不带注释信息尝试：

```markup
cnvkit.py batch \
  /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/*Tumor*.bam \
  --normal /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/*Normal*.bam \
  --targets /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_modify.bed \
  --fasta /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/GRCh38/GRCh38.d1.vd1.fa \
  --access /data/liuyuxin/Project-7-tools/cnvkit/cnvkit-0.9.12/data/access-10kb.hg38.bed \
  --output-reference /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/HCC1395_WES_reference.cnn \
  --output-dir /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output \
  --diagram \
  --scatter \
  -p 20
```

失败，生成的on-target的bed文件中仍有非标准染色体
