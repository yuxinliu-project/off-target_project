###### 原始路径重新分步执行，具体详见：cnvkit检测HCC1395数据实验记录



##### 一. 准备阶段

进入工作目录

```markup
cd /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output
```

1. 准备Target文件 (拆分并注释基因名)

   ```markup
   cnvkit.py target \
     /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_modify.bed \
     --annotate /data/liuyuxin/Project-7-tools/cnvkit/cnvkit-0.9.12/data/refFlat_hg38.txt \
     --split \
     -o S07604624_modify.target.bed

   # 输出
   Detected file format: bed
   Splitting large targets
   Applying annotations as target names
   Detected file format: refflat
   Wrote S07604624_modify.target.bed with 397288 regions
   ```

2. 准备Antitarget文件

   ```markup
   cnvkit.py antitarget \
     S07604624_modify.target.bed \
     --access /data/liuyuxin/Project-7-tools/cnvkit/cnvkit-0.9.12/data/access-10kb.hg38.bed \
     -o S07604624_modify.antitarget.bed
     
   # 输出
   Detected file format: bed
   Detected file format: bed
   Wrote S07604624_modify.antitarget.bed with 41262 regions
   ```

3. 检查并去除非标准染色体 (区分第一次尝试的重要步骤)

   ```markup
   # 过滤Target文件的非标准染色体
   grep -E "^chr([0-9]+|X|Y)\b" S07604624_modify.target.bed > S07604624_modify.target.standard.bed

   # 过滤Antitarget文件的非标准染色体
   grep -E "^chr([0-9]+|X|Y)\b" S07604624_modify.antitarget.bed > S07604624_modify.antitarget.standard.bed
   ```

##### 二. 为每个Normal样本分别计算Coverage

建个脚本

1. 手动创建generate_coverage.sh

   内容：

   ```markup
   #!/bin/bash

   # 定义输入文件路径
   BAM_DIR="/data/share/liuyuxin_tanrenjie/HCC1395_data/WES"
   TARGET_BED="S07604624_modify.target.standard.bed"
   ANTITARGET_BED="S07604624_modify.antitarget.standard.bed"

   # 样本列表
   SAMPLES=(
     "WES_EA_Normal_1"
     "WES_FD_Normal_1"
     "WES_FD_Normal_2"
     "WES_FD_Normal_3"
     "WES_IL_Normal_1"
     "WES_IL_Normal_2"
     "WES_IL_Normal_3"
     "WES_LL_Normal_1"
     "WES_NC_Normal_1"
     "WES_NV_Normal_1"
     "WES_NV_Normal_2"
     "WES_NV_Normal_3"
   )

   # 逐个处理样本，生成coverage文件
   for SAMPLE in "${SAMPLES[@]}"
   do
     echo "Processing ${SAMPLE} target coverage..."
     cnvkit.py coverage \
       ${BAM_DIR}/${SAMPLE}.bwa.dedup.bam \
       ${TARGET_BED} \
       -o ${SAMPLE}.targetcoverage.cnn

     echo "Processing ${SAMPLE} antitarget coverage..."
     cnvkit.py coverage \
       ${BAM_DIR}/${SAMPLE}.bwa.dedup.bam \
       ${ANTITARGET_BED} \
       -o ${SAMPLE}.antitargetcoverage.cnn
   done

   echo "All coverage files generated successfully."
   ```

   2. 赋予权限

   ```markup
   chmod +x generate_coverage.sh
   ```

   3. 执行脚本

   ```markup
   ./generate_coverage.sh
   ```

   成功：

   ```markup
   # out
   Processing WES_EA_Normal_1 target coverage...
   Processing reads in WES_EA_Normal_1.bwa.dedup.bam
   Time: 654.575 seconds (131719 reads/sec, 607 bins/sec)
   Summary: #bins=397141, #reads=86220257, mean=217.1024, min=0.0, max=9386.42857142857
   Percent reads in regions: 49.013 (of 175913325 mapped)
   Wrote WES_EA_Normal_1.targetcoverage.cnn with 397141 regions
   Processing WES_EA_Normal_1 antitarget coverage...
   Processing reads in WES_EA_Normal_1.bwa.dedup.bam
   Time: 62.339 seconds (150548 reads/sec, 662 bins/sec)
   Summary: #bins=41262, #reads=9385034, mean=227.4498, min=0.0, max=15786.388888888889
   Percent reads in regions: 5.335 (of 175913325 mapped)
   Wrote WES_EA_Normal_1.antitargetcoverage.cnn with 41262 regions
   Processing WES_FD_Normal_1 target coverage...
   Processing reads in WES_FD_Normal_1.bwa.dedup.bam
   Time: 246.323 seconds (130383 reads/sec, 1612 bins/sec)
   Summary: #bins=397141, #reads=32116387, mean=80.8690, min=0.0, max=3367.5824175824177
   Percent reads in regions: 46.947 (of 68410607 mapped)
   Wrote WES_FD_Normal_1.targetcoverage.cnn with 397141 regions
   Processing WES_FD_Normal_1 antitarget coverage...
   Processing reads in WES_FD_Normal_1.bwa.dedup.bam
   Time: 52.283 seconds (169624 reads/sec, 789 bins/sec)
   Summary: #bins=41262, #reads=8868458, mean=214.9304, min=0.0, max=6835.758241758242
   Percent reads in regions: 12.964 (of 68410607 mapped)
   Wrote WES_FD_Normal_1.antitargetcoverage.cnn with 41262 regions
   Processing WES_FD_Normal_2 target coverage...
   Processing reads in WES_FD_Normal_2.bwa.dedup.bam
   Time: 277.082 seconds (126531 reads/sec, 1433 bins/sec)
   Summary: #bins=397141, #reads=35059294, mean=88.2792, min=0.0, max=3706.368794326241
   Percent reads in regions: 44.947 (of 78002112 mapped)
   Wrote WES_FD_Normal_2.targetcoverage.cnn with 397141 regions
   Processing WES_FD_Normal_2 antitarget coverage...
   Processing reads in WES_FD_Normal_2.bwa.dedup.bam
   Time: 60.278 seconds (166788 reads/sec, 685 bins/sec)
   Summary: #bins=41262, #reads=10053680, mean=243.6547, min=0.0, max=7082.851063829788
   Percent reads in regions: 12.889 (of 78002112 mapped)
   Wrote WES_FD_Normal_2.antitargetcoverage.cnn with 41262 regions
   Processing WES_FD_Normal_3 target coverage...
   Processing reads in WES_FD_Normal_3.bwa.dedup.bam
   Time: 280.982 seconds (131324 reads/sec, 1413 bins/sec)
   Summary: #bins=397141, #reads=36899706, mean=92.9134, min=0.0, max=4013.9548872180453
   Percent reads in regions: 45.982 (of 80247670 mapped)
   Wrote WES_FD_Normal_3.targetcoverage.cnn with 397141 regions
   Processing WES_FD_Normal_3 antitarget coverage...
   Processing reads in WES_FD_Normal_3.bwa.dedup.bam
   Time: 61.349 seconds (178021 reads/sec, 673 bins/sec)
   Summary: #bins=41262, #reads=10921377, mean=264.6837, min=0.0, max=7801.827067669173
   Percent reads in regions: 13.610 (of 80247670 mapped)
   Wrote WES_FD_Normal_3.antitargetcoverage.cnn with 41262 regions
   Processing WES_IL_Normal_1 target coverage...
   Processing reads in WES_IL_Normal_1.bwa.dedup.bam
   Time: 872.260 seconds (131865 reads/sec, 455 bins/sec)
   Summary: #bins=397141, #reads=115020697, mean=289.6218, min=0.0, max=14876.280701754386
   Percent reads in regions: 41.249 (of 278847976 mapped)
   Wrote WES_IL_Normal_1.targetcoverage.cnn with 397141 regions
   Processing WES_IL_Normal_1 antitarget coverage...
   Processing reads in WES_IL_Normal_1.bwa.dedup.bam
   Time: 81.316 seconds (156075 reads/sec, 507 bins/sec)
   Summary: #bins=41262, #reads=12691395, mean=307.5807, min=0.0, max=18875.07894736842
   Percent reads in regions: 4.551 (of 278847976 mapped)
   Wrote WES_IL_Normal_1.antitargetcoverage.cnn with 41262 regions
   Processing WES_IL_Normal_2 target coverage...
   Processing reads in WES_IL_Normal_2.bwa.dedup.bam
   Time: 2196.783 seconds (156420 reads/sec, 181 bins/sec)
   Summary: #bins=397141, #reads=343620478, mean=865.2355, min=0.0, max=32761.963302752294
   Percent reads in regions: 49.118 (of 699576848 mapped)
   Wrote WES_IL_Normal_2.targetcoverage.cnn with 397141 regions
   Processing WES_IL_Normal_2 antitarget coverage...
   Processing reads in WES_IL_Normal_2.bwa.dedup.bam
   Time: 306.223 seconds (238012 reads/sec, 135 bins/sec)
   Summary: #bins=41262, #reads=72884741, mean=1766.3890, min=0.0, max=90294.0366972477
   Percent reads in regions: 10.418 (of 699576848 mapped)
   Wrote WES_IL_Normal_2.antitargetcoverage.cnn with 41262 regions
   Processing WES_IL_Normal_3 target coverage...
   Processing reads in WES_IL_Normal_3.bwa.dedup.bam
   Time: 801.959 seconds (176678 reads/sec, 495 bins/sec)
   Summary: #bins=397141, #reads=141688320, mean=356.7708, min=0.0, max=12919.191489361701
   Percent reads in regions: 37.126 (of 381642309 mapped)
   Wrote WES_IL_Normal_3.targetcoverage.cnn with 397141 regions
   Processing WES_IL_Normal_3 antitarget coverage...
   Processing reads in WES_IL_Normal_3.bwa.dedup.bam
   Time: 426.920 seconds (368494 reads/sec, 97 bins/sec)
   Summary: #bins=41262, #reads=157317236, mean=3812.6421, min=0.0, max=232775.47872340426
   Percent reads in regions: 41.221 (of 381642309 mapped)
   Wrote WES_IL_Normal_3.antitargetcoverage.cnn with 41262 regions
   Processing WES_LL_Normal_1 target coverage...
   Processing reads in WES_LL_Normal_1.bwa.dedup.bam
   Time: 300.396 seconds (135806 reads/sec, 1322 bins/sec)
   Summary: #bins=397141, #reads=40795594, mean=102.7232, min=0.0, max=4959.011673151751
   Percent reads in regions: 58.706 (of 69491768 mapped)
   Wrote WES_LL_Normal_1.targetcoverage.cnn with 397141 regions
   Processing WES_LL_Normal_1 antitarget coverage...
   Processing reads in WES_LL_Normal_1.bwa.dedup.bam
   Time: 26.752 seconds (113707 reads/sec, 1542 bins/sec)
   Summary: #bins=41262, #reads=3041896, mean=73.7215, min=0.0, max=7715.548638132295
   Percent reads in regions: 4.377 (of 69491768 mapped)
   Wrote WES_LL_Normal_1.antitargetcoverage.cnn with 41262 regions
   Processing WES_NC_Normal_1 target coverage...
   Processing reads in WES_NC_Normal_1.bwa.dedup.bam
   Time: 400.425 seconds (157457 reads/sec, 992 bins/sec)
   Summary: #bins=397141, #reads=63049662, mean=158.7589, min=0.0, max=6411.034782608695
   Percent reads in regions: 55.697 (of 113201214 mapped)
   Wrote WES_NC_Normal_1.targetcoverage.cnn with 397141 regions
   Processing WES_NC_Normal_1 antitarget coverage...
   Processing reads in WES_NC_Normal_1.bwa.dedup.bam
   Time: 112.419 seconds (198299 reads/sec, 367 bins/sec)
   Summary: #bins=41262, #reads=22292683, mean=540.2715, min=0.0, max=10847.330434782609
   Percent reads in regions: 19.693 (of 113201214 mapped)
   Wrote WES_NC_Normal_1.antitargetcoverage.cnn with 41262 regions
   Processing WES_NV_Normal_1 target coverage...
   Processing reads in WES_NV_Normal_1.bwa.dedup.bam
   Time: 1520.159 seconds (115402 reads/sec, 261 bins/sec)
   Summary: #bins=397141, #reads=175428858, mean=441.7294, min=0.0, max=11842.432
   Percent reads in regions: 44.436 (of 394790745 mapped)
   Wrote WES_NV_Normal_1.targetcoverage.cnn with 397141 regions
   Processing WES_NV_Normal_1 antitarget coverage...
   Processing reads in WES_NV_Normal_1.bwa.dedup.bam
   Time: 218.550 seconds (181136 reads/sec, 189 bins/sec)
   Summary: #bins=41262, #reads=39587266, mean=959.4122, min=0.0, max=30888.96
   Percent reads in regions: 10.027 (of 394790745 mapped)
   Wrote WES_NV_Normal_1.antitargetcoverage.cnn with 41262 regions
   Processing WES_NV_Normal_2 target coverage...
   Processing reads in WES_NV_Normal_2.bwa.dedup.bam
   Time: 1648.065 seconds (114123 reads/sec, 241 bins/sec)
   Summary: #bins=397141, #reads=188081772, mean=473.5894, min=0.0, max=12254.888
   Percent reads in regions: 45.379 (of 414468687 mapped)
   Wrote WES_NV_Normal_2.targetcoverage.cnn with 397141 regions
   Processing WES_NV_Normal_2 antitarget coverage...
   Processing reads in WES_NV_Normal_2.bwa.dedup.bam
   Time: 209.180 seconds (172098 reads/sec, 197 bins/sec)
   Summary: #bins=41262, #reads=35999455, mean=872.4603, min=0.0, max=32157.392
   Percent reads in regions: 8.686 (of 414468687 mapped)
   Wrote WES_NV_Normal_2.antitargetcoverage.cnn with 41262 regions
   Processing WES_NV_Normal_3 target coverage...
   Processing reads in WES_NV_Normal_3.bwa.dedup.bam
   Time: 1473.769 seconds (117569 reads/sec, 269 bins/sec)
   Summary: #bins=397141, #reads=173268986, mean=436.2909, min=0.0, max=11964.384
   Percent reads in regions: 46.378 (of 373602043 mapped)
   Wrote WES_NV_Normal_3.targetcoverage.cnn with 397141 regions
   Processing WES_NV_Normal_3 antitarget coverage...
   Processing reads in WES_NV_Normal_3.bwa.dedup.bam
   Time: 196.953 seconds (178103 reads/sec, 210 bins/sec)
   Summary: #bins=41262, #reads=35078022, mean=850.1290, min=0.0, max=29843.096
   Percent reads in regions: 9.389 (of 373602043 mapped)
   Wrote WES_NV_Normal_3.antitargetcoverage.cnn with 41262 regions
   All coverage files generated successfully.
   ```

##### 三. 构建Reference文件

```markup
cnvkit.py reference \
  *.targetcoverage.cnn *.antitargetcoverage.cnn \
  --fasta /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/GRCh38/GRCh38.d1.vd1.fa \
  -o HCC1395_WES_normal_reference.cnn
```

成功：

```markup
# out：
Number of target and antitarget files: 12, 12
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
Relative log2 coverage of chrX=-1.09, chrY=-11.1 (maleness=19.9 x 0.74 = 14.8) --> assuming male
Relative log2 coverage of chrX=-1.15, chrY=-12 (maleness=23.9 x 0.704 = 16.8) --> assuming male
Relative log2 coverage of chrX=-1.15, chrY=-11.8 (maleness=25.3 x 0.704 = 17.8) --> assuming male
Relative log2 coverage of chrX=-1.15, chrY=-11.8 (maleness=22.1 x 0.759 = 16.8) --> assuming male
Relative log2 coverage of chrX=-1.07, chrY=-11 (maleness=28.7 x 0.79 = 22.7) --> assuming male
Relative log2 coverage of chrX=-0.996, chrY=-9.13 (maleness=156 x 0.751 = 117) --> assuming male
Relative log2 coverage of chrX=-0.989, chrY=-8.98 (maleness=135 x 0.79 = 106) --> assuming male
Relative log2 coverage of chrX=-1.15, chrY=-11.8 (maleness=51 x 0.722 = 36.8) --> assuming male
Relative log2 coverage of chrX=-1.07, chrY=-11.9 (maleness=26.5 x 0.722 = 19.1) --> assuming male
Relative log2 coverage of chrX=-1.04, chrY=-9.95 (maleness=45 x 0.704 = 31.7) --> assuming male
Relative log2 coverage of chrX=-1.04, chrY=-10.2 (maleness=55.7 x 0.704 = 39.2) --> assuming male
Relative log2 coverage of chrX=-1.05, chrY=-9.88 (maleness=38.2 x 0.722 = 27.6) --> assuming male
Loading WES_EA_Normal_1.targetcoverage.cnn
Calculating GC and RepeatMasker content in /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/GRCh38/GRCh38.d1.vd1.fa ...
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
Correcting for GC bias for WES_EA_Normal_1...
Correcting for density bias for WES_EA_Normal_1...
Loading WES_FD_Normal_1.targetcoverage.cnn
Correcting for GC bias for WES_FD_Normal_1...
Correcting for density bias for WES_FD_Normal_1...
Loading WES_FD_Normal_2.targetcoverage.cnn
Correcting for GC bias for WES_FD_Normal_2...
Correcting for density bias for WES_FD_Normal_2...
Loading WES_FD_Normal_3.targetcoverage.cnn
Correcting for GC bias for WES_FD_Normal_3...
Correcting for density bias for WES_FD_Normal_3...
Loading WES_IL_Normal_1.targetcoverage.cnn
Correcting for GC bias for WES_IL_Normal_1...
Correcting for density bias for WES_IL_Normal_1...
Loading WES_IL_Normal_2.targetcoverage.cnn
Correcting for GC bias for WES_IL_Normal_2...
Correcting for density bias for WES_IL_Normal_2...
Loading WES_IL_Normal_3.targetcoverage.cnn
Correcting for GC bias for WES_IL_Normal_3...
Correcting for density bias for WES_IL_Normal_3...
Loading WES_LL_Normal_1.targetcoverage.cnn
Correcting for GC bias for WES_LL_Normal_1...
Correcting for density bias for WES_LL_Normal_1...
Loading WES_NC_Normal_1.targetcoverage.cnn
Correcting for GC bias for WES_NC_Normal_1...
Correcting for density bias for WES_NC_Normal_1...
Loading WES_NV_Normal_1.targetcoverage.cnn
Correcting for GC bias for WES_NV_Normal_1...
Correcting for density bias for WES_NV_Normal_1...
Loading WES_NV_Normal_2.targetcoverage.cnn
Correcting for GC bias for WES_NV_Normal_2...
Correcting for density bias for WES_NV_Normal_2...
Loading WES_NV_Normal_3.targetcoverage.cnn
Correcting for GC bias for WES_NV_Normal_3...
Correcting for density bias for WES_NV_Normal_3...
Loading WES_EA_Normal_1.antitargetcoverage.cnn
Calculating GC and RepeatMasker content in /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/GRCh38/GRCh38.d1.vd1.fa ...
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
Correcting for GC bias for WES_EA_Normal_1...
Correcting for RepeatMasker bias for WES_EA_Normal_1...
Loading WES_FD_Normal_1.antitargetcoverage.cnn
Correcting for GC bias for WES_FD_Normal_1...
Correcting for RepeatMasker bias for WES_FD_Normal_1...
Loading WES_FD_Normal_2.antitargetcoverage.cnn
Correcting for GC bias for WES_FD_Normal_2...
Correcting for RepeatMasker bias for WES_FD_Normal_2...
Loading WES_FD_Normal_3.antitargetcoverage.cnn
Correcting for GC bias for WES_FD_Normal_3...
Correcting for RepeatMasker bias for WES_FD_Normal_3...
Loading WES_IL_Normal_1.antitargetcoverage.cnn
Correcting for GC bias for WES_IL_Normal_1...
Correcting for RepeatMasker bias for WES_IL_Normal_1...
Loading WES_IL_Normal_2.antitargetcoverage.cnn
Correcting for GC bias for WES_IL_Normal_2...
Correcting for RepeatMasker bias for WES_IL_Normal_2...
Loading WES_IL_Normal_3.antitargetcoverage.cnn
Correcting for GC bias for WES_IL_Normal_3...
Correcting for RepeatMasker bias for WES_IL_Normal_3...
Loading WES_LL_Normal_1.antitargetcoverage.cnn
Correcting for GC bias for WES_LL_Normal_1...
Correcting for RepeatMasker bias for WES_LL_Normal_1...
Loading WES_NC_Normal_1.antitargetcoverage.cnn
Correcting for GC bias for WES_NC_Normal_1...
Correcting for RepeatMasker bias for WES_NC_Normal_1...
Loading WES_NV_Normal_1.antitargetcoverage.cnn
Correcting for GC bias for WES_NV_Normal_1...
Correcting for RepeatMasker bias for WES_NV_Normal_1...
Loading WES_NV_Normal_2.antitargetcoverage.cnn
Correcting for GC bias for WES_NV_Normal_2...
Correcting for RepeatMasker bias for WES_NV_Normal_2...
Loading WES_NV_Normal_3.antitargetcoverage.cnn
Correcting for GC bias for WES_NV_Normal_3...
Correcting for RepeatMasker bias for WES_NV_Normal_3...
Calculating average bin coverages
Calculating bin spreads
Targets: 33093 (8.333%) bins failed filters (log2 < -5.0, log2 > 5.0, spread > 1.0)
Antitargets: 564 (1.367%) bins failed filters
Wrote HCC1395_WES_normal_reference.cnn with 438403 regions
```

##### 四. Tumor样本分析

单个Sample示例（未选择此方法）

```markup
# 生成cnr文件
cnvkit.py coverage \
  /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Tumor_1.bwa.dedup.bam \
  S07604624_modify.target.standard.bed \
  -o WES_EA_Tumor_1.targetcoverage.cnn

cnvkit.py coverage \
  /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Tumor_1.bwa.dedup.bam \
  S07604624_modify.antitarget.bed \
  -o WES_EA_Tumor_1.antitargetcoverage.cnn

cnvkit.py fix \
  WES_EA_Tumor_1.targetcoverage.cnn \
  WES_EA_Tumor_1.antitargetcoverage.cnn \
  HCC1395_WES_normal_reference.cnn \
  -o WES_EA_Tumor_1.cnr

# 分段
cnvkit.py segment WES_EA_Tumor_1.cnr -o WES_EA_Tumor_1.cns

# 散点图
cnvkit.py scatter WES_EA_Tumor_1.cnr -s WES_EA_Tumor_1.cns -o WES_EA_Tumor_1-scatter.pdf

# 染色体图
cnvkit.py diagram WES_EA_Tumor_1.cnr -s WES_EA_Tumor_1.cns -o WES_EA_Tumor_1-diagram.pdf

```

并行计算

```markup
cnvkit.py batch /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/*Tumor*.bam \
  -r HCC1395_WES_normal_reference.cnn \
  --output-dir /data/share/liuyuxin_tanrenjie/CNVkit_HCC1395-WES_output/Tumor_output \
  --scatter --diagram \
  -p 50
```

成功，文件输出至/data/share/liuyuxin_tanrenjie/OFF-PEAK_HCC1395-WES_output
