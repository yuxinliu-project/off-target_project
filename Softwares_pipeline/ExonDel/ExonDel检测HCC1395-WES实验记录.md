#### ExonDel检测HCC1395-WES实验记录

1. 准备：

   1. 准备软件包：Github下载Code→ExonDel-master.zip，存放至/data/share/liuyuxin_tanrenjie/Softwares

      ```markup
      unzip /data/share/liuyuxin_tanrenjie/Softwares/ExonDel-master.zip
      cd /data/share/liuyuxin_tanrenjie/Softwares/ExonDel-master
      ```

   2. 安装依赖环境

      ```markup
      #启动环境变量
      source ~/.bashrc

      #检查
      bash test.modules
      #output：
      ok   File::Basename
       ok   File::Copy 
       ok   FindBin
       ok   Getopt::Long
       fail HTML::Template
       fail Report::Generate
       ok   threads
       ok   threads::shared

      #安装：
      bash install.modules HTML::Template
      bash install.modules Report::Generate#失败
      ```

   3. Report::Generate解决

      ```markup
      find . -name "Generate.pm"
      #out：./Report/Generate.pm

      #修改 ExonDel.pl 文件，让 Perl 找到这个模块
      nano ExonDel.pl
      #在文件前几行，use strict; 之后加一行：use lib './Report'; 
      ```

   4. 准备ExonDel 所需的 RefSeq GTF格式注释表

      OFF-PEAK/data中自带hg38_refseqGenes.csv，改名成hg38_refseqGenes.gtf即可，存放至/data/share/liuyuxin_tanrenjie/HCC1395_data/hg38_ncbiRefSeq.gtf

   5. 生成 BAM 列表：

      ```markup
      cd /data/share/liuyuxin_tanrenjie/HCC1395_data/WES
      ls *.bam | awk '{print "/data/share/liuyuxin_tanrenjie/HCC1395_data/WES/"$0"\t"substr($0, 1, length($0)-4)}' > bam_list.txt
      ```

   6. ExonDel.cfg 文件设置

      创建/data/share/liuyuxin_tanrenjie/HCC1395_data/WES/ExonDel.cfg

      ```markup
      cat <<EOF > /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/ExonDel.cfg
      [perl]
      bedfile=/data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_delete.bed
      refseq=/data/share/liuyuxin_tanrenjie/HCC1395_data/hg38_ncbiRefSeq.gtf
      reffa=/data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/GRCh38/GRCh38.d1.vd1.fa
      samtoolsBin=samtools
      RBin=/usr/bin/R
      exon_bp_cover_threshold=0.1
      overall_exon_count_threshold=1
      logFileName=exondel.log
      adjustGC=FALSE
      minWinLength=1
      maxWinLength=9
      minExonNum=3
      EOF
      ```

2. 检测：

   ```markup
   perl /data/share/liuyuxin_tanrenjie/Softwares/ExonDel-master/ExonDel.pl \
    -i /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/bam_list.txt \
    -c /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/ExonDel.cfg \
    -o /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/ExonDel_HCC1395-WES_output \
    -t 8
   ```

   **==失败==**，out:

   ```markup
   [Thu Apr 10 00:47:03 2025] Thread 1 finished
   [Thu Apr 10 00:47:12 2025] Thread 2 finished
   [Thu Apr 10 00:47:40 2025] Thread 3 finished
   [Thu Apr 10 00:47:40 2025] Thread 4 finished
   [Thu Apr 10 00:47:40 2025] Thread 5 finished
   [Thu Apr 10 00:47:40 2025] Thread 6 finished
   [Thu Apr 10 00:47:40 2025] Thread 7 finished
   [Thu Apr 10 00:47:40 2025] Thread 8 finished
   [Thu Apr 10 00:47:40 2025] Finish bam file
   [Thu Apr 10 00:47:40 2025] Analyzing Exon Deletion
   [Thu Apr 10 01:04:31 2025] Something wrong in running R. Please check the ExonDel.rLog file!
   [Thu Apr 10 01:04:31 2025] ExonDel.rLog content:
   [1] "Loading /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/ExonDel_HCC1395-WES_output/genesPassQCwithGC.bed to R"
   [1] "Loading /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/ExonDel_HCC1395-WES_output/genesPassQCwithGC.bed.depth to R"
   [1] "Quantile NA and NA of all reads for each sample will be used as cutoff in exon deletion detection"
   Error in if (max(rawData, na.rm = T) <= cutoff2) { :
   missing value where TRUE/FALSE needed
   Calls: sapply -> lapply -> FUN -> sigNumber
   Execution halted
   can't find /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/ExonDel_HCC1395-WES_output/exonDelsCutoffs.csv
   ```

3. 排查问题：

   **==分析原因：==**

   1. 输入的样本深度太低（全是 0 或 NA）

   2. 某些样本在 .bed.depth 文件中没有 coverage 数据 ，导致合并后的 genesPassQCwithGC.bed.depth 文件出现了空列

   解决：

   1. 看 .depth 是否有异常值：

      ```markup
      grep -v -e "^#" /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/ExonDel_HCC1395-WES_output/genesPassQCwithGC.bed.depth | head -n 5
      #out：
      WES_EA_Normal_1.bwa.dedup	0	0	0	0	0	0	0	0	0	0	0	0	0   0
      WES_EA_Tumor_1.bwa.dedup	0	0	0	0	0	0	0	0	0	0	0	0	0   0
      WES_FD_Normal_1.bwa.dedup	0	0	0	0	0	0	0	0	0	0	0	0	0   0
      WES_FD_Normal_2.bwa.dedup	0	0	0	0	0	0	0	0	0	0	0	0	0   0
      WES_FD_Normal_3.bwa.dedup	0	0	0	0	0	0	0	0	0	0	0	0	0   0
      WES_FD_Tumor_1.bwa.dedup	0	0	0	0	0	0	0	0	0	0	0	0	0   0
      WES_FD_Tumor_2.bwa.dedup	0	0	0	0	0	0	0	0	0	0	0	0	0   0
      WES_FD_Tumor_3.bwa.dedup	0	0	0	0	0	0	0	0	0	0	0	0	0   0
      WES_IL_Normal_1.bwa.dedup	0	0	0	0	0	0	0	0	0	0	0	0	0   0
      WES_IL_Normal_2.bwa.dedup	0	0	0	0	0	0	0	0	0	0	0	0	0   
      ```

      结论：ExonDel 检测流程中生成的 genesPassQCwithGC.bed.depth 文件内容不正常，这一步本应统计每个样本在每个 exon 区间的测序深度，但这里全是 0，才会导致后续所有 quantile 为 NA，进而 R 报错

   2. 分析原因：

      1. bed 区域与 BAM 文件都为chr格式，非格式问题

      2. 检查 bed 区域与 BAM 是否重叠

         ```markup
         samtools depth -b /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_delete.bed \
          /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Normal_1.bwa.dedup.bam | head
         # out：
         chr1	12081	100
         chr1	12082	98
         chr1	12083	101
         chr1	12084	101
         chr1	12085	102
         chr1	12086	101
         chr1	12087	100
         chr1	12088	102
         chr1	12089	105
         chr1	12090	109
         ```

         匹配，没问题

      3. ExonDel 内部提取深度出了问题

         1. bed文件为标准3列，没问题

         2. ExonDel 在运行中 没有正确读取 BAM 文件

            1. 检查 BAM 列表文件

               ```markup
               head /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/bam_list.txt
               ```

               无windows字符，没问题

            2. 检查 BAM 文件是否可被 ExonDel 读取

               ```markup
               samtools view /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Normal_1.bwa.dedup.bam | head
               # out：
               B0P8DQ1:92:HV3LTBCXY:1:2213:11038:52050	177	chr1	10106	0	12S42M	chrX	153470871	0	TATCCCCTAACCCCCAACCCTAACCCTAACCCTAACCCTAACCCTAACCCTAAC	G<.AGGGGA<AA<A<.GGGGGGGGAAGA<AGGG<<GAGA<.GG<AAG.<G<.<<	MD:Z:42	RG:Z:WES_EA_N_1	NM:i:0	AS:i:42	XS:i:42
               B0P8DQ1:92:HV3LTBCXY:1:1111:12931:81050	129	chr1	10124	0	74M	chr20	
               ......
               ```

               没问题

            3. 手动尝试samtools模拟ExonDel可能在后台调用的命令

               ```markup
               /usr/bin/samtools depth -b /data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_delete.bed \
               /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/WES_EA_Normal_1.bwa.dedup.bam > test.depth
               ```

               查看：

               ```markup
               head /data/share/liuyuxin_tanrenjie/Softwares/test.depth
               #out：
               chr1	12081	100
               chr1	12082	98
               chr1	12083	101
               chr1	12084	101
               chr1	12085	102
               chr1	12086	101
               chr1	12087	100
               chr1	12088	102
               chr1	12089	105
               chr1	12090	109

               wc -l /data/share/liuyuxin_tanrenjie/Softwares/test.depth
               # 检查总行数：out：
               # 89898783 /data/share/liuyuxin_tanrenjie/Softwares/test.depth

               awk '$3==0' /data/share/liuyuxin_tanrenjie/Softwares/test.depth | wc -l
               # 检查深度是否为0：out：
               # 966（966行为0）

               awk '{sum+=$3} END {print "Average depth:", sum/NR}' /data/share/liuyuxin_tanrenjie/Softwares/test.depth
               # 检查平均深度：out：
               # Average depth: 120.836
               ```

               samtools能生成，没有问题

            4. 怀疑问题出在ExonDel没有正确调用samtools：

               修改配置文件ExonDel.cfg，将samtools换成绝对路径：

               ```markup
               nano /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/ExonDel.cfg
               # 修改samtoolsBin=samtools为samtoolsBin=/usr/bin/samtools
               # 尽量不用记事本修改，防止出现windows字符
               ```

4. **==重新运行：==**

   ```markup
   perl /data/share/liuyuxin_tanrenjie/Softwares/ExonDel-master/ExonDel.pl \
    -i /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/bam_list.txt \
    -c /data/share/liuyuxin_tanrenjie/HCC1395_data/WES/ExonDel.cfg \
    -o /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/ExonDel_HCC1395-WES_output \
    -t 8

   ```

   仍然失败

5. 分析原因：

   检查生成的文件：

   ```markup
   head /data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/ExonDel_HCC1395-WES_output/genesPassQCwithGC.bed
   # out：
   1	17368	17436	MIR6859-1	NR_106918.1	0.60
   1	30365	30503	MIR1302-2	NR_036051.1	0.30
   1	450739	451678	OR4F29	NM_001005221.2	0.46
   1	683909	689079	OR4F16	XM_017002411.1	0.36
   1	711764	711842	OR4F16	XM_017002411.1	0.33
   1	683909	689079	OR4F16	XM_024449992.1	0.36
   1	720031	720101	OR4F16	XM_024449992.1	0.41
   1	685715	686654	OR4F16	NM_001005277.1	0.46
   1	826205	827522	LINC00115	NR_024321.1	0.53
   1	925730	925800	SAMD11	NM_152486.4	0.73
   ```

   原因：生成的bed文件有问题，尝试修改脚本















