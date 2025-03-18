###### OFF-PEAK使用记录

###### 目的：按照文献方法使用OFF-PEAK以便更理解文献

###### 时间：2024/11/11

###### 下载地址：https://github.com/mquinodo/OFF-PEAK

###### 所需其他工具：BEDTools、mosdepth、R

###### 服务器：A3

###### 位置：/data/liuyuxin/OFF-PEAK-main

### Step1. 处理目标区域（外显子bed文件）

1. 所用文件为xhmm测试时排序、融合完的hg38版本的bed文件（详见xhmm2）

2. XShell：

   由于Share文件夹中的bam文件命名方式不带chr，所以需统一格式

   1. 将hg19_bed/20130108.exome.targets.bed文件手动用excle删除chr

   2. 文件夹hg19_referance：

      ```Shell
      sed 's/^>chr/>/' hg19.fa > hg19_modified.fa
      ```

3. XShell：

```Shell
cd OFF-PEAK-main
```

```markup
bash 01_targets-processing.sh
--genome hg19
--targets hg19_bed/20130108.exome.targets.bed
--name 11_11_hg19.bed_target-panel
--ref hg19_reference/hg19_modified.fa
--minOntarget 100
--maxOntarget 300
--minOfftarget 1
--maxOfftarget 50000
--paddingOfftarget 300
--nochr
```

输出文件：data/merge_Exome.v1.target-panel.bed

4. 官网code and options

   **code：**

   Target processing

   The main script 01_targets-processing.sh takes as input a BED file which contains the target regions of the sequencing data as well as the reference genome (FASTA format). It is outputing a BED file containing processed on-target and off-target regions. It is called with bash and its computation time for an exome BED file is few minutes:

   ```markup
   bash 01_targets-processing.sh
     --genome [hg19|hg38]
     --targets target.bed
     --name target-panel
     --ref ref_genome.fa
     [other options]
   ```

The output file (target-panel.bed) will be placed in the data folder.

**options：**

###### Required arguments

| Option    | Value       | Description                                                                   |
| :-------- | :---------- | :---------------------------------------------------------------------------- |
| --targets | STRING      | BED file containing the target regions of the exome or targeted sequencing    |
| --genome  | [hg19/hg38] | Genome build used                                                             |
| --name    | STRING      | Name of the output file (without extension)                                   |
| --ref     | STRING      | Reference genome in FASTA format (same as used to map the reads in BAM files) |

#### Optional arguments

| Option             | Default | Value               | Description                                                                                                                |
| :----------------- | :------ | :------------------ | :------------------------------------------------------------------------------------------------------------------------- |
| --minOntarget      | 100     | 1 - Inf             | Minimal size of on-targets. Smaller on-target regions will be extended on each side to reach this value.                   |
| --maxOntarget      | 300     | &#x3e; minOntarget  | Maximal size of on-targets. Larger on-target regions will be splitted into regions of equal size.                          |
| --minOfftarget     | 1       | 1 - Inf             | Minimal size of off-targets. Smaller on-target regions will be discarded.                                                  |
| --maxOfftarget     | 50000   | &#x3e; minOfftarget | Maximal size of off-targets. Larger on-target regions will be splitted into regions of equal size.                         |
| --paddingOfftarget | 300     | 0 - Inf             | Padding around on-target regions to avoid targeted reads to be counted in off-target regions.                              |
| --nochr            | ---     | ---                 | Use if your BED file and your BAM files have the chromosome notation without "chr" like 1, 2,... instead of chr1, chr2,... |

### Step2. 计算样本的覆盖度

1. 所用文件：A3服务器中的 /data/share/1000GP_data/1kgp_exome_bam

2. 制作list文件sequence_bam.txt（30sample）（至少15个，30-50为最佳）

3. XShell：

   ```markup
   bash 02_coverage-count.sh
   --listBAM sequence_bam.txt
   --mosdepth mosdepth-master/mosdepth
   --work 11_11_hg19_output_directory
   --targetsBED data/11_11_hg19.bed_target-panel.bed
   --nochr
   ```

4. 官网code and options

**code：**

Coverage computation

The main script 02_coverage-count.sh takes as input a text file listing the sample BAM files (.bai index files are needed in the same folder, they can be generated with samtools index command) and IDs as well as the processed BED file from step 1. It is outputing a file containing the coverage for each sample and for each target. It is called with bash and its computation time is 2-3 minutes per sample for WES:

```
bash 02_coverage-count.sh
  --listBAM list_BAM.txt
  --mosdepth mosdepth-executable
  --work output_directory
  --targetsBED data/target-panel.bed
```

**options：**

###### Required arguments

| Option       | Value  | Description                                                                                                                                                                                                                                                                                  |
| :----------- | :----- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| --listBAM    | STRING | Text file containing two tab-delimited columns: BAM files (with absolute location, for example /user/Download/Patient1.bam) and IDs (for example Patient1). It can also have only one column with BAM files. In this case, IDs will be deduced from the name of BAM files (not recommended). |
| --mosdepth   | STRING | Mosdepth executable file that can be downloaded from the github page here: [Link]                                                                                                                                                                                                            |
| --work       | STRING | Output directory                                                                                                                                                                                                                                                                             |
| --targetsBED | STRING | BED file with processed targets produced in step 1                                                                                                                                                                                                                                           |

### Step3.CNV发现（注释CNV、图、统计数据）

1. XShell：

   1. 03_OFF-PEAK.R脚本需要使用optparse包，安装：

   ```Shell
   R -e "install.packages('optparse', repos='http://cran.r-project.org')"
   ```

   2. 03_OFF-PEAK.R 脚本需要加载 ExomeDepth 包，安装：

   ```markup
   R -e "install.packages('ExomeDepth', repos='http://cran.r-project.org')"
   ```

   失败，需要用Bioconductor安装：安装Bioconductor：

   ```Shell
   R -e 'if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager")'
   ```

   3. 安装：ExomeDepth：

   ```markup
   R -e "install.packages('ExomeDepth', repos='http://cran.r-project.org')"
   ```

```S
Rscript 03_OFF-PEAK.R 
--output 11_11_output_directory 
--data 11_11_hg19_output_directory/ALL.target.tsv 
--databasefile data/data-hg19.RData 
--mincor 0.9 
--minsignal 2500 
--maxvar -0.2 
--leaveoneout 1 
--downsample 20000 
--nbFake 500 
--stopPC 0.0001 
--minZ 4 
--minOfftarget 1000 
--chromosome-plots 
--genome-plots
--nb-plots 10
```

运行失败，原因：ExomeDepth不适配此版本的R和Bioconductor

解决：官网方法：

```Shell
R -e "url <- 'https://cran.r-project.org/src/contrib/Archive/ExomeDepth/ExomeDepth_1.1.16.tar.gz'; pkgFile <- 'ExomeDepth_1.1.16.tar.gz'; download.file(url = url, destfile = pkgFile); install.packages(c('Biostrings','IRanges','Rsamtools','GenomicRanges','aod','VGAM','GenomicAlignments','dplyr','magrittr')); install.packages(pkgs=pkgFile, type='source', repos=NULL); unlink(pkgFile)"
```

失败

###### 解决方法：

1. conda创建了R3.6、R4.0、R4.2、R4.4四个版本，分别命名为R36，R40，R42，R44

2. condaR36，R40过多工具包不适配，老版本大部分找不到

3. conda R42 只剩安装GenomicAlignments所需依赖包Matrix无法安装最新版本

4. 使用服务器R4.4失败：权限不够

5. ==condaR44==：解决

   ```R
   BiocManager::install("Biostrings") 
   BiocManager::install("IRanges") 
   BiocManager::install("Rsamtools") 
   BiocManager::install("GenomicRanges") 
   BiocManager::install("GenomicAlignments") 
   install.packages("aod") 
   install.packages("VGAM") 
   install.packages("dplyr") 
   install.packages("magrittr")
   ```

逐个安装以便逐个解决问题（根据gpt一步一步解决所需依赖包）

成功

各工具版本

```R
packageVersion("Biostrings")
[1] ‘2.74.0’
packageVersion("IRanges")
[1] ‘2.40.0’
packageVersion("Rsamtools")
[1] ‘2.22.0’
packageVersion("GenomicRanges")
[1] ‘1.58.0’
packageVersion("GenomicAlignments")
[1] ‘1.42.0’
packageVersion("aod")
[1] ‘1.3.3’
packageVersion("VGAM")
[1] ‘1.1.12’
packageVersion("dplyr")
[1] ‘1.1.4’
packageVersion("magrittr")
[1] ‘2.0.3’
packageVersion("ExomeDepth")
[1] ‘1.1.16’
```

###### 重新尝试运行Rscript 03_OFF-PEAK.R：

```markup
Rscript 03_OFF-PEAK.R --output 11_11_output_directory --data 11_11_hg19_output_directory/ALL.target.tsv --databasefile data/data-hg19.RData --mincor 0.9 --minsignal 2500 --maxvar -0.2 --leaveoneout 1 --downsample 20000 --nbFake 500 --stopPC 0.0001 --minZ 4 --minOfftarget 1000 --chromosome-plots --genome-plots --nb-plots 10
```

成功，但是有29个warning：There were 29 warnings (use warnings() to see them)

内容："WARNING: less than 15 other samples with R2>0.9, taking 15 samples with highest correlation. This can lead to decreased performances."

输出文件全部位于11_11_output_directory文件夹中

6. 官网code and options

**code：**

CNV discovery

The main script 03_OFF-PEAK.R takes as input the coverage data per target computed in step 2 and outputs multiple files including annotated CNVs, CNV plots and statistics. It is called with Rscript and its computation time is ~1 hour per 10 samples:

```
Rscript 03_OFF-PEAK.R
  --output output_directory
  --data ALL.target.tsv
  --databasefile data/data-hg19.RData
  [other options]
```

**options：**

###### Required arguments

| Option         | Value  | Description                                                                                                          |
| :------------- | :----- | :------------------------------------------------------------------------------------------------------------------- |
| --output       | STRING | Output directory                                                                                                     |
| --data         | STRING | Output of step 2                                                                                                     |
| --databasefile | STRING | Absolute path to RData file containing various information found in data folder (data-hg19.RData or data-hg38.RData) |

###### Optional arguments

| Option             | Default | Value      | Description                                                                      |
| :----------------- | :------ | :--------- | :------------------------------------------------------------------------------- |
| --mincor           | 0.9     | 0 - 0.99   | Minimal correlation of control samples compared to analyzed one.                 |
| --minsignal        | 2500    | 1 - Inf    | Minimal signal for a target to be analyzed.                                      |
| --maxvar           | -0.2    | -1 - 1     | Maximal variance for a target to be analyzed.                                    |
| --leaveoneout      | 1       | 0 or 1     | If 1, leave-one-out PCA (LOO-PCA) will be used. If 0, standard PCA will be used. |
| --downsample       | 20000   | 100 - Inf  | Number of downsampled targets for optimization of PC removal.                    |
| --nbFake           | 500     | 10 - 10000 | Number of fake CNVs used for optimization of PC removal.                         |
| --stopPC           | 0.0001  | 0 - 0.1    | Stopping criteria for optimization of PC removal.                                |
| --minZ             | 4       | 2 - 10     | Minimal absolute Z-score for single target CNV processing.                       |
| --minOfftarget     | 1000    | 1 - Inf    | Minimal size of off-targets without exons.                                       |
| --chromosome-plots | -       | -          | If present, coverage plots for each chromosome will be done.                     |
| --genome-plots     | -       | -          | If present, genome-wide coverage plots will be done.                             |
| --nb-plots         | 10      | 0 - Inf    | Number of plots for CNVs per individual                                          |

### Step4. CNV绘图

官网code and options：

**code：**

In step 3 , the 20 best CNVs per sample will be automatically plotted. This script is used to further plot other CNVs. The main script 04_OFF-PEAK-plot.R takes as input intermediate data from step 3 in order to do additional plots. It is called with Rscript and its computation time is few seconds per CNV:

```
Rscript 04_OFF-PEAK-plot.R
  --ID ID
  --chr chr
  --begin begin
  --end end
  --batch output_directory
  --out output-directory
  --databasefile data-hg19.RData
  [other options]
```

**options：**

###### Required arguments

| Option         | Value  | Description                                                                                     |
| :------------- | :----- | :---------------------------------------------------------------------------------------------- |
| --ID           | STRING | Sample ID                                                                                       |
| --chr          | STRING | Chromosome                                                                                      |
| --begin        | number | Begin position of CNV                                                                           |
| --end          | number | End position of CNV                                                                             |
| --batch        | STRING | Output directory of step 3 (output_directory)                                                   |
| --out          | STRING | Output directory                                                                                |
| --databasefile | STRING | Absolute path to RData file containing various information (data-hg19.RData or data-hg38.RData) |

###### Optional arguments

| Option | Default | Value   | Description                                          |
| :----- | :------ | :------ | :--------------------------------------------------- |
| --side | 20      | 1 - 100 | Number of target plotted on each side of the region. |

