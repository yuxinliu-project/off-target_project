#### SavvyCNV检测HCC1395-WES-pipeline

SavvyCNV为SavvySuite其中一部分，官方pipeline：https://github.com/rdemolgen/SavvySuite

SavvySuite模块：

| 模块                   | 作用                                                             |
| :------------------- | :------------------------------------------------------------- |
| CoverageBinner&#x20; | 将 BAM/CRAM 文件转换为基因组覆盖率统计文件                                     |
| SelectControlSamples | 选择要处理的样本子集 —— 注意：此步骤仅适用于当你要对单个样本检测CNVs，但拥有超过200个其他样本进行比较时      |
| SavvyCNV&#x20;       | 进行降噪，检测单个Sample的CNV                                            |
| SavvyCNVJointCaller  | 联合检测：希望将多个个体（例如同一家族的不同成员）的 CNV 检测为一个 CNV 检测适用。将 CNVs 共同检测提高准确性 |

1. 准备

   1. 下载SavvySuite-1.0.tar 放至/data/share/liuyuxin_tanrenjie/Softwares中

   2. 下载gatk-4.2.0.zip 放至/data/share/liuyuxin_tanrenjie/Softwares/Required-Softwares-Libraries

   ```R
   # 解压gatk
   unzip /data/share/liuyuxin_tanrenjie/Softwares/Required-Softwares-Libraries/gatk-4.2.0.0.zip

   # 解压SavvySuite
   tar -xzvf /data/share/liuyuxin_tanrenjie/Softwares/SavvySuite-1.0.tar.gz

   # 设置环境变量（需要java，java11已设置完环境变量）
   export GATK_HOME=/data/share/liuyuxin_tanrenjie/Softwares/Required-Softwares-Libraries/gatk-4.2.0.0
   export SAVVY_SUITE_HOME=/data/share/liuyuxin_tanrenjie/Softwares/SavvySuite-1.0
   # 将 GATK 的本地 Jar 和 SavvySuite 目录添加到 CLASSPATH 中
   export CLASSPATH=$GATK_HOME/gatk-package-4.2.0.0-local.jar:$SAVVY_SUITE_HOME:$CLASSPATH）
   ```

2. 运行 CoverageBinner

   ```R
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Tumor_1.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_EA_Tumor_1.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Normal_1.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_EA_Normal_1.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_Tumor_1.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_FD_Tumor_1.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_Tumor_2.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_FD_Tumor_2.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_Tumor_3.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_FD_Tumor_3.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_Normal_1.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_FD_Normal_1.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_Normal_2.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_FD_Normal_2.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_FD_Normal_3.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_FD_Normal_3.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_Tumor_1.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_IL_Tumor_1.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_Tumor_2.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_IL_Tumor_2.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_Tumor_3.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_IL_Tumor_3.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_Normal_1.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_IL_Normal_1.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_Normal_2.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_IL_Normal_2.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_IL_Normal_3.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_IL_Normal_3.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_LL_Tumor_1.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_LL_Tumor_1.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_LL_Normal_1.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_LL_Normal_1.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NC_Tumor_1.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_NC_Tumor_1.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NC_Normal_1.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_NC_Normal_1.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_Tumor_1.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_NV_Tumor_1.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_Tumor_2.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_NV_Tumor_2.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_Tumor_3.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_NV_Tumor_3.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_Normal_1.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_NV_Normal_1.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_Normal_2.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_NV_Normal_2.coverageBinner
   java -Xmx1g CoverageBinner /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_NV_Normal_3.bwa.dedup.bam > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_NV_Normal_3.coverageBinner
   ```

   out：/data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_EA_Normal_1.coverageBinner

   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/WES_EA_Tumor_1.coverageBinner

3. 使用 SavvyCNV 进行 CNV 检测

   ```markup
   java -Xmx30g SavvyCNV -d 200000 \
     -case /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/*Tumor*.coverageBinner \
     -control /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/*Normal*.coverageBinner \
     -data -g -a -headers \
     > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/Result/cnv_list.csv \
     2> /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/Result/log_messages.txt

   ```

   **==失败：==** 无论怎么修改命令顺序都提示无法读取case，control指令，并把control指令识别成了文件名



#### 发现下载的GitHub中的Releases中的SavvySuite-1.0并非最新版，Code中的SavvySuite-master才是最新版，重新下载并检测

1. 准备：

   下载SavvySuite-master.zip存放至/data/share/liuyuxin_tanrenjie/Softwares

   ```markup
   #解压
   unzip /data/share/liuyuxin_tanrenjie/Softwares/SavvySuite-master.zip

   #配置环境变量
   export CLASSPATH=/data/share/liuyuxin_tanrenjie/Softwares/Required-Softwares-Libraries/gatk-4.2.0.0/gatk-package-4.2.0.0-local.jar:/data/share/liuyuxin_tanrenjie/Softwares/SavvySuite-master

   #编译：
   javac *.java

   #进入环境变量
   source ~/.bashrc
   ```

2. 运行 CoverageBinner

   写个脚本：/data/share/liuyuxin_tanrenjie/Softwares/SavvySuite-master/run_coveragebinner.sh

   ```markup
   #!/bin/bash

   # 设置输入输出路径前缀
   INPUT_DIR="/data/share/liuyuxin_tanrenjie/HCC1395_data/WES"
   OUTPUT_DIR="/data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner"

   # 所有样本文件名（不带路径）
   samples=(
   "WES_EA_Tumor_1"
   "WES_EA_Normal_1"
   "WES_FD_Tumor_1"
   "WES_FD_Tumor_2"
   "WES_FD_Tumor_3"
   "WES_FD_Normal_1"
   "WES_FD_Normal_2"
   "WES_FD_Normal_3"
   "WES_IL_Tumor_1"
   "WES_IL_Tumor_2"
   "WES_IL_Tumor_3"
   "WES_IL_Normal_1"
   "WES_IL_Normal_2"
   "WES_IL_Normal_3"
   "WES_LL_Tumor_1"
   "WES_LL_Normal_1"
   "WES_NC_Tumor_1"
   "WES_NC_Normal_1"
   "WES_NV_Tumor_1"
   "WES_NV_Tumor_2"
   "WES_NV_Tumor_3"
   "WES_NV_Normal_1"
   "WES_NV_Normal_2"
   "WES_NV_Normal_3"
   )

   # 逐个运行 CoverageBinner
   for sample in "${samples[@]}"
   do
       echo "正在处理 $sample ..."
       java -Xmx1g CoverageBinner "${INPUT_DIR}/${sample}.bwa.dedup.bam" > "${OUTPUT_DIR}/${sample}.coverageBinner"
   done

   echo "所有样本处理完成！"
   ```

   赋予权限并执行：

   ```markup
   chmod +x run_coveragebinner.sh
   ./run_coveragebinner.sh
   ```

   运行成功，存放至：/data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner

3. SavvyCNV

   ```markup
   java -Xmx30g SavvyCNV \
   -d 200000 \
   -headers \
   -data \
   -case \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_EA_Tumor_1.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_FD_Tumor_1.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_FD_Tumor_2.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_FD_Tumor_3.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_IL_Tumor_1.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_IL_Tumor_2.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_IL_Tumor_3.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_LL_Tumor_1.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_NC_Tumor_1.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_NV_Tumor_1.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_NV_Tumor_2.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_NV_Tumor_3.coverageBinner \
   -control \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_EA_Normal_1.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_FD_Normal_1.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_FD_Normal_2.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_FD_Normal_3.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_IL_Normal_1.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_IL_Normal_2.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_IL_Normal_3.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_LL_Normal_1.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_NC_Normal_1.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_NV_Normal_1.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_NV_Normal_2.coverageBinner \
   /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/CoverageBinner/WES_NV_Normal_3.coverageBinner \
   > /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/Result/cnv_list_ontarget.csv \
   2> /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/Result/log_ontarget.txt
   ```

成功运行，Result保存至/data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/SavvySuite_HCC1395-WES_output/Result













































