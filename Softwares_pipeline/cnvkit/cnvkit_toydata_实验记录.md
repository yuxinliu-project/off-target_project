#### CNVkit实验记录

###### 目的：检测2692个cram的CNV，为论文准备好数据

###### 时间：2024/11/24

###### Github: https://github.com/etal/cnvkit

###### 所用服务器：A3

###### 所选环境：未用conda，正常使用

###### 工具位置：/data/liuyuxin/.local/lib/python3.10/site-packages

###### 输出文件位置：/data/liuyuxin/Softwares/cnvkit-output/11.27_cram_50_output_directory

1. 安装依赖库：

   ```markup
   pip install numpy scipy matplotlib pandas pysam
   ```

2. 安装cnvkit

   ```markup
   pip install cnvkit
   ```

3. 添加到环境变量中：

   ```markup
   export PATH=$PATH:/data/liuyuxin/.local/bin
   ```

4. 打开cnvkit-output文件夹（全部生成至此，方便查看）：

   ```markup
   cd Softwares
   cd cnvkit-output
   ```

5. 准备.cnn:

   ```markup
   cnvkit.py -f ../hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa -o reference.cnn
   ```

   失败：无reference.bam

   解决：使用平坦参考：选择target region，创建：antitarget region（off-target）：

   ```markup
   cnvkit.py antitarget ../hg38_bed/output_1000G_Exome.v1.bed -g ../hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa -o off-target.bed
   ```

   失败：理解错误，应先创建基因组可访问区域文件：

   ```markup
   cnvkit.py access ../hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa -o accessible.bed
   ```

   成功创建：accessible.bed

   创建antitarget region（off-target）：

   ```markup
   cnvkit.py antitarget ../hg38_bed/output_1000G_Exome.v1.bed -g accessible.bed -o off-target.bed
   ```

   成功生成：off-target.bed

   创建cnn：

   ```markup
   cnvkit.py reference -t output_1000G_Exome.v1.bed -a off-target.bed -f ../hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa -o hg38_reference.cnn
   ```

   失败：原因：NumPy2.0版本过高

   解决：降级NumPy 版本至1.24

   ```markup
   pip install numpy==1.24
   ```

   再次尝试：

   ```markup
   cnvkit.py reference -t output_1000G_Exome.v1.bed -a off-target.bed -f ../hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa -o hg38_reference.cnn
   ```

   成功生成：hg38_reference.cnn

6. 检测：

   ```markup
   cnvkit.py batch --list sequence_cram.txt -r hg38_reference.cnn -f ../hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa -o 11.27_cram_50_output_directory
   ```

   失败：

   改进：

   ```markup
   cnvkit.py batch --samples sequence_cram.txt -r hg38_reference.cnn -f ../hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa --output-dir 11.27_cram_50_output_directory
   ```

   失败：cnvkit无法处理list

   改进：

   ```markup
   cat sequence_cram.txt | xargs cnvkit.py batch -r hg38_reference.cnn -f ../hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa --output-dir 11.27_cram_50_output_directory
   ```

   失败：-r和-f重复

   改进：

   ```markup
   cat sequence_cram.txt | xargs cnvkit.py batch -r hg38_reference.cnn --output-dir 11.27_cram_50_output_directory
   ```

   失败：原因：Error in library("DNAcopy") : there is no package called ‘DNAcopy’

   解决方案：R环境中安装DNAcopy包：进入congdar44安装，比较稳定，可随意赋予权限，并设置国内源：

   ```markup
   source ~/.bashrc
   conda activate r44
   options(repos = c(CRAN = "https://mirrors.tuna.tsinghua.edu.cn/CRAN/"))
   options(BioC_mirror = "https://mirrors.tuna.tsinghua.edu.cn/bioconductor/")
   BiocManager::install("DNAcopy")
   ```

   成功安装DNAcopy

   再次尝试：

   ```markup
   cat sequence_cram.txt | xargs cnvkit.py batch -r hg38_reference.cnn --output-dir 11.27_cram_50_output_directory
   ```

   失败：改进（每个路径都是一个整体）

   ```markup
   cat sequence_cram.txt | xargs -d '\n' cnvkit.py batch -r hg38_reference.cnn --output-dir 11.27_cram_50_output_directory
   ```

   cram文件名称过长，改进：

   创建符号链接，将长路径映射到一个短路径：

   ```markup
   ln -s /data/share/1000GP_data/1kgp_exome_cram ~/cram_link
   ```

   再次尝试：使用find、head命令检测前50个cram

   ```markup
   find /data/share/1000GP_data/1kgp_exome_cram/ -name "*.cram" -not -path "*/failed_crams/*" -print0 | sort -z | head -z -n 50 | xargs -0 -I {} cnvkit.py batch {} -r hg38_reference.cnn --output-dir 11.27_cram_50_output_directory
   ```

   成功，输出文件：

   | /data/liuyuxin/Softwares/cnvkit-output/11.27_cram_50_output_directory |
   | :-------------------------------------------------------------------- |
   | HG00096.alt_bwamem_GRCh38DH.20150826.GBR.exome.antitargetcoverage.cnn |
   | HG00096.alt_bwamem_GRCh38DH.20150826.GBR.exome.bintest.cns            |
   | HG00096.alt_bwamem_GRCh38DH.20150826.GBR.exome.call.cns               |
   | HG00096.alt_bwamem_GRCh38DH.20150826.GBR.exome.cnr                    |
   | HG00096.alt_bwamem_GRCh38DH.20150826.GBR.exome.cns                    |
   | HG00096.alt_bwamem_GRCh38DH.20150826.GBR.exome.targetcoverage.cnn     |

