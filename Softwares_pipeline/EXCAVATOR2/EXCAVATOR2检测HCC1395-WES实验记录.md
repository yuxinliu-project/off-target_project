### EXCAVATOR2检测HCC1395-pipeline

#### 日期：2025-03-31

###### 1. 准备

1. 查看samtools、bedtools是否已安装

   ```markup
   samtools --version
   # output：samtools 1.13
   bedtools --version
   # output：bedtools v2.30.0
   ```

2. 准备EXCAVATOR2

   ```markup
   cd /data/share/liuyuxin_tanrenjie/Softwares
   tar -xzvf /data/share/liuyuxin_tanrenjie/Softwares/EXCAVATOR2_Package_v1.1.2.tgz
   cd /data/share/liuyuxin_tanrenjie/Softwares/EXCAVATOR2_Package_v1.1.2/lib/F77

   # 编译
   R CMD SHLIB F4R.f
   R CMD SHLIB FastJointSLMLibraryI.f
   ```

3. 安装libpng

   官网下载：https://sourceforge.net/projects/libpng/files/libpng12/1.2.59/

   放至：/data/share/liuyuxin_tanrenjie/Softwares/libpng-1.2.59.tar.gz

   ```markup
   cd /data/share/liuyuxin_tanrenjie/Softwares/Required-Softwares-Libraries
   tar -zxvf libpng-1.2.59.tar.gz
   cd libpng-1.2.59
   ./configure --prefix=/data/share/liuyuxin_tanrenjie/Softwares/Required-Softwares-Libraries

   # 编译：
   make
   make install

   # 设置环境变量：
   export LD_LIBRARY_PATH=/data/share/liuyuxin_tanrenjie/Softwares/Required-Softwares-Libraries/lib:$LD_LIBRARY_PATH

   # 验证安装：
   ls /data/share/liuyuxin_tanrenjie/Softwares/Required-Softwares-Libraries/lib
   ```

###### 2. 3个模块，对应3个脚本

1. TargetPerla.pl

   准备：

   更改：/data/share/liuyuxin_tanrenjie/Softwares/EXCAVATOR2_Package_v1.1.2/SourceTarget.txt内容

   将：/.../ucsc.hg19.bw /.../human.fasta

   更改为：/data/share/liuyuxin_tanrenjie/Softwares/EXCAVATOR2_Package_v1.1.2/data/GCA_000001405.15_GRCh38.bw /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/GRCh38/GRCh38.d1.vd1.fa

   ```markup
   perl TargetPerla.pl \
   SourceTarget.txt \
   /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_delete.bed \
   Step1-HCC1395-WES \
   50000 \
   hg38
   ```

   output：/data/share/liuyuxin_tanrenjie/Softwares/EXCAVATOR2_Package_v1.1.2/data/targets/hg38/Step1-HCC1395-WES

2. EXCAVATORDataPrepare.pl

   准备：创建HCC1395-WES_list.txt

   内容：/data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Normal_1.bwa.dedup.bam /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/EXCAVATOR2_HCC1395-WES_output/Step2_RC_output/EA_Normal_1 EA_Normal_1

   ```markup
   perl EXCAVATORDataPrepare.pl \
   HCC1395-WES_list.txt \
   --processors 30 \
   --target Step1-HCC1395-WES \
   --assembly hg38
   ```

   output：/data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/EXCAVATOR2_HCC1395-WES_output/Step2_RC_output

3. EXCAVATORDataAnalysis.pl

   准备：创建：HCC1395-WES_EXCAVATORDataAnalysis.txt

   T1 /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/EXCAVATOR2_HCC1395-WES_output/Step2_RC_output/EA_Tumor_1 EA_Tumor_1 C1 /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/EXCAVATOR2_HCC1395-WES_output/Step2_RC_output/EA_Normal_1 EA_Normal_1

   ```markup
   perl EXCAVATORDataAnalysis.pl \
   HCC1395-WES_EXCAVATORDataAnalysis.txt \
   --processors 8 \
   --target Step1-HCC1395-WES \
   --assembly hg38 \
   --output Result_HCC1395-WES \
   --mode paired
   ```

##### 失败：

Error in JointSegIn(DataSeq1, muk, mi, smu, sepsilon, Pos1, omega, eta,  :
NA/NaN/Inf in foreign function call (arg 3)
Execution halted
Error in JointSegIn(DataSeq1, muk, mi, smu, sepsilon, Pos1, omega, eta,  :
NA/NaN/Inf in foreign function call (arg 3)

分析：

1. 参考基因组问题

   尝试：

   1. 更换为/data/share/1000GP_data/technology/reference/GRCh38_full_analysis_set_plus_decoy_hla.fa 仍然失败

   2. 检测 bam，无论是share文件夹里的1kgp的参考基因组还是HCC1395的参考基因组都检测成功

      排除FASTA文件问题

2. bed文件问题

   尝试：

   1. 检查 .bed 文件中的无效坐标（如 start > end）

      ```markup
      awk '$2 >= $3 {print}' /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_delete.bed
      ```

   2. 检查非Linux符号

      ```markup
      cat -v /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_delete.bed | head
      ```

   都没问题

   目前怀疑是bed文件修改出来的问题











































