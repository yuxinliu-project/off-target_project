#### 时间：2024/11/21&#x20;

#### 目的：修改OFF-PEAK的Step2中02_coverage-count.sh以支持计算cram文件覆盖率

#### 服务器：A3

##### Step1：尝试mosdepth，单独跑通：

```markup
$mosdepth  -n -t 2 -Q 20 -b $work/targetsBED.bed $work/$pat $bam命令（102行）
```

1. 准备：

   1. cram文件：

      /data/share/1000GP_data/1kgp_exome_cram/HG00096.alt_bwamem_GRCh38DH.20150826.GBR.exome.cram

   2. bed文件：

      /data/liuyuxin/Softwares/hg38_bed/output_1000G_Exome.v1.bed

   3. reference文件：

      /data/liuyuxin/Softwares/hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa

2. 设计代码：

   ```markup
   mosdepth -n -t 2 -Q 20 -b ../hg38_bed/output_1000G_Exome.v1.bed /data/liuyuxin/Softwares/mosdepth-master/cram_test_output/test_output_prefix ../../../share/1000GP_data/1kgp_exome_cram/HG00096.alt_bwamem_GRCh38DH.20150826.GBR.exome.cram
   ```

3. 设置reference环境变量以方便调用：

   ```markup
   export REF_PATH=/data/liuyuxin/Softwares/hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa
   ```

4. 将mosdepth添加到PATH环境变量中：

   ```markup
   export PATH=/data/liuyuxin/Softwares/mosdepth-master:$PATH
   ```

5. 开始运行

   ###### 第一次尝试：

   ```markup
   cd Softwares/
   cd mosdepth-master/
   ```

   ==运行失败==：bash: /data/liuyuxin/Softwares/mosdepth-master/mosdepth: Permission denied

   ==判断==：mosdepth未授予完整的读写权限

   ==解决方法==：

   1. 查看mosdepth权限：

      ```markup
      ls -l /data/liuyuxin/Softwares/mosdepth-master/mosdepth
      ```

      输出：-rw-rw-r-- 1 liuyuxin liuyuxin 19231544 11月 21 15:37 /data/liuyuxin/Softwares/mosdepth-master/mosdepth

      确定为mosdepth权限不够

   2. 赋予权限：

      ```markup
      chmod +x /data/liuyuxin/Softwares/mosdepth-master/mosdepth
      ```

   3. 再次查看权限：

      ```markup
      ls -l /data/liuyuxin/Softwares/mosdepth-master/mosdepth
      ```

      输出：-rwxrwxr-x 1 liuyuxin liuyuxin 19231544 11月 21 15:37 /data/liuyuxin/Softwares/mosdepth-master/mosdepth

      成功赋予权限

   4. 根据GPT提示再赋予/data/liuyuxin/Softwares/mosdepth-master/cram_test_output文件夹读写权限：

      ```markup
      chmod +w /data/liuyuxin/Softwares/mosdepth-master/cram_test_output
      ```

   ###### 第二尝试：成功

   输出文件：

   | A3：Softwares/mosdepth-master/cram_test_output/test_output_prefix |
   | :--------------------------------------------------------------- |
   | test_output_prefix.mosdepth.global.dist.txt                      |
   | test_output_prefix.mosdepth.region.dist.txt                      |
   | test_output_prefix.mosdepth.summary.txt                          |
   | test_output_prefix.regions.bed.gz                                |
   | test_output_prefix.regions.bed.gz.csi                            |

==结论：== mosdepth完全可以对CRAM文件计算覆盖度

##### Step2：尝试修改：02_coverage-count.sh

1. 思路：

   1. 根据mosdepth使用经验来看，处理CRAM文件需要reference文件，所以需要在脚本中添加调用reference的代码

   2. 将BAM替换为CRAM

2. 按行顺序修改

   1. ==添加noreference函数：==&#x20;

      类比5-9行：

      ```markup
      nomosdepth() { echo "## ERROR: Mosdepth not found (--mosdepth). Exit." 1>&2; exit 1; }
      nowork() { echo "## ERROR: You need to provide the working directory through --work option. Exit." 1>&2; exit 1; }
      nolistBAM() { echo "## ERROR: You need to provide the text file containing the list of BAM files and samples ID through --listBAM option. Exit." 1>&2; exit 1; }
      notargetsBED() { echo "## ERROR: You need to provide the BED file containing the on-targets and off-targets through --targetsBED option. Exit." 1>&2; exit 1; }
      errorlistBAM() { echo "## ERROR: The list of BAM files and samples ID provided through --listBAM should have 2 columns for BAM files and sample IDs and be tab delimited. Exit." 1>&2; exit 1; }
      ```

      将noreference()函数添加至第10行，在--reference选项输出错误信息时调用，提示错误原因：

      ```markup
      noreference() { echo "## ERROR: You need to provide the reference genome file through --reference option. Exit." 1>&2; exit 1; }
      ```

   2. ==同步将BAM修改为CRAM格式：==&#x20;

      第7行：

      ```markup
      nolistBAM() { echo "## ERROR: You need to provide the text file containing the list of BAM files and samples ID through --listBAM option. Exit." 1>&2; exit 1; }
      ```

      修改为：

      ```markup
      nolistCRAM() { echo "## ERROR: You need to provide the text file containing the list of CRAM files and samples ID through --listBAM option. Exit." 1>&2; exit 1; }
      ```

      第9行：

      ```markup
      errorlistBAM() { echo "## ERROR: The list of BAM files and samples ID provided through --listBAM should have 2 columns for BAM files and sample IDs and be tab delimited. Exit." 1>&2; exit 1; }
      ```

      修改为：

      ```markup
      errorlistCRAM() { echo "## ERROR: The list of CRAM files and samples ID provided through --listCRAM should have 2 columns for CRAM files and sample IDs and be tab delimited. Exit." 1>&2; exit 1; }
      ```

   3. ==在循环中加入reference索引：==&#x20;

      循环：12-36行：

      ```markup
      while getopts ":-:" o; do
          case "${o}" in
          	-)  
              	case $OPTARG in
                      work)
                          work="${!OPTIND}"; OPTIND=$(( $OPTIND + 1 ))
                          ;;
                      listCRAM)
                          listCRAM="${!OPTIND}"; OPTIND=$(( $OPTIND + 1 ))
                          ;;
                      mosdepth)
                          mosdepth="${!OPTIND}"; OPTIND=$(( $OPTIND + 1 ))
                          ;;
                      targetsBED)
                          targetsBED="${!OPTIND}"; OPTIND=$(( $OPTIND + 1 ))
                          ;;
      				*)
                  		usage
                  	;;
               esac ;;        
              *)
                  usage
                  ;;
          esac
      done
      ```

      加至28-30行：

      ```markup
      reference)
          reference="${!OPTIND}"; OPTIND=$((OPTIND + 1))
          ;;
      ```

   4. ==同步将其中BAM修改为CRAM格式：==&#x20;

      19-20行：

      ```
                      listBAM)
                          listBAM="${!OPTIND}"; OPTIND=$(( $OPTIND + 1 ))
      ```

      修改为：

      ```
                      listCRAM)
                          listCRAM="${!OPTIND}"; OPTIND=$(( $OPTIND + 1 ))
      ```

   5. ==添加if条件判断来判断reference是否正确：==（错误时调用noreference函数）

      参考42-53行：

      ```markup
      if [ -z "${work}" ]; then
          nowork
      fi
      if [ -z "${listBAM}" ]; then
          nolistBAM
      fi
      if [ -z "${mosdepth}" ]; then
          nomosdepth
      fi
      if [ -z "${targetsBED}" ]; then
          notargetsBED
      fi
      ```

      添加至54-56行：

      ```markup
      if [ -z "${reference}" ]; then
          noreference
      fi
      ```

   6. ==同步将其中BAM修改为CRAM格式：==&#x20;

      45-46行：

      ```markup
      if [ -z "${listBAM}" ]; then
          nolistBAM
      ```

      修改为：

      ```markup
      if [ -z "${listCRAM}" ]; then
          nolistCRAM
      ```

   7. sed工具处理listBAM命令改为CRAM：

      58行：

      ```markup
      sed '/^$/d' $listBAM > $listBAM.temp
      ```

      修改为：

      ```markup
      sed '/^$/d' $listCRAM > $listCRAM.temp
      ```

   8. ==awk工具检查输入文件是否缺少样本ID命令中的BAM修改为CRAM：==

      64-66行：

      ```markup
      if [[ `awk -F"\t" 'NF==1' $listBAM` ]]; then
          echo "   Only one field detected. ID will be deduced from the BAM file name"
      fi
      ```

      修改为：

      ```markup
      if [[ `awk -F"\t" 'NF==1' $listCRAM` ]]; then
          echo "   Only one field detected. ID will be deduced from the CRAM file name"
      fi
      ```

   9. ==在69-87行逐行读取$listBAM.temp 文件命令中加入检查CRAM 文件及其对应的 CRAI 索引文件是否存在命令，以及同步修改其中的BAM命令为CRAM命令：==&#x20;

      69-87行：

      ```markup
      while read file
      do
      	a=$(echo $file | awk -F" " '{print NF}')
      	if [[ "$a" == 1 ]]; then
      		pat=$(echo $file | cut -f1 -d" " | awk -F"\t" '{n=split($1,a,"/"); split(a[n],b,".bam"); print b[1]}')
      		bam=$(echo $file | cut -f1 -d" ")
      	fi
      	if [ $a -gt 1 ]; then
      		pat=$(echo $file | cut -f2 -d" ")
      		bam=$(echo $file | cut -f1 -d" ")
      	fi
      	if [ ! -f $bam ]; then
      		rm $listBAM.temp
      		echo "## ERROR: BAM file $bam not found. Exit." 1>&2; exit 1;
      	fi
      done < $listBAM.temp

      rm -f $work/list.txt
      touch $work/list.txt
      ```

      修改为：

      ```markup
      while read file
      do
      	a=$(echo $file | awk -F" " '{print NF}')
      	if [[ "$a" == 1 ]]; then
      		pat=$(echo $file | cut -f1 -d" " | awk -F"\t" '{n=split($1,a,"/"); split(a[n],b,".cram"); print b[1]}')
      		cram=$(echo $file | cut -f1 -d" ")
      	fi
      	if [ $a -gt 1 ]; then
      		pat=$(echo $file | cut -f2 -d" ")
      		cram=$(echo $file | cut -f1 -d" ")
      	fi
      	if [ ! -f $cram ]; then
      		rm $listCRAM.temp
      		echo "## ERROR: CRAM file $bam not found. Exit." 1>&2; exit 1;
      	fi
      	if [ ! -f $cram.crai ]; then
              rm $listBAM.temp
              echo "## ERROR: CRAI index file for $cram not found. Exit." 1>&2; exit 1;
          fi
      done < $listCRAM.temp

      rm -f $work/list.txt
      touch $work/list.txt
      ```

   10. ==mosdepth计算覆盖度代码修改为使用CRAM格式以及增加调用reference指令：==&#x20;

       93-121行：

       ```markup
       while read file
       do
       	a=$(echo $file | awk -F" " '{print NF}')
       	if [[ "$a" == 1 ]]; then
       		pat=$(echo $file | cut -f1 -d" " | awk -F"\t" '{n=split($1,a,"/"); split(a[n],b,".bam"); print b[1]}')
       		bam=$(echo $file | cut -f1 -d" ")
       		echo $pat
       	fi
       	if [ $a -gt 1 ]; then
       		echo "more than 1 field"
       		pat=$(echo $file | cut -f2 -d" ")
       		bam=$(echo $file | cut -f1 -d" ")
       	fi
       	echo $pat >> $work/list.txt
       	if [ ! -f $work/coverage/$pat.cov.tsv ] || [ ! -s $work/coverage/$pat.cov.tsv ]; then
       		echo "   Processing sample $pat"
       		# downloaded here: https://github.com/brentp/mosdepth/releases/download/v0.3.2/mosdepth
       		# -n = no-per-base; -t = threads; -Q mapq; precision is for rounding of numbers
       		# gives the average per base coverage
       		export MOSDEPTH_PRECISION=10
       		$mosdepth  -n -t 2 -Q 20 -b $work/targetsBED.bed $work/$pat $bam
       		zcat $work/$pat.regions.bed.gz > $work/$pat.regions.bed
       		# remultiply the per-base average by the size to have the total coverage
       		mkdir -p $work/coverage
       		awk -F"\t" '{print $1 "\t" $2 "\t" $3 "\t" $4 "\t" $5*($3-$2)}' $work/$pat.regions.bed > $work/coverage/$pat.cov.tsv
       		# cleaning
       		rm $work/$pat.regions.bed.gz $work/$pat.regions.bed $work/$pat.mosdepth* $work/$pat.regions.bed.gz.csi
       	fi
       done < $listBAM.temp
       ```

       修改为：

       ```markup
       while read file
       do
       	a=$(echo $file | awk -F" " '{print NF}')
       	if [[ "$a" == 1 ]]; then
       		pat=$(echo $file | cut -f1 -d" " | awk -F"\t" '{n=split($1,a,"/"); split(a[n],b,".cram"); print b[1]}')
       		cram=$(echo $file | cut -f1 -d" ")
       		echo $pat
       	fi
       	if [ $a -gt 1 ]; then
       		echo "more than 1 field"
       		pat=$(echo $file | cut -f2 -d" ")
       		cram=$(echo $file | cut -f1 -d" ")
       	fi
       	echo $pat >> $work/list.txt
       	if [ ! -f $work/coverage/$pat.cov.tsv ] || [ ! -s $work/coverage/$pat.cov.tsv ]; then
       		echo "   Processing sample $pat"
       		# downloaded here: https://github.com/brentp/mosdepth/releases/download/v0.3.2/mosdepth
       		# -n = no-per-base; -t = threads; -Q mapq; precision is for rounding of numbers
       		# gives the average per base coverage
       		export MOSDEPTH_PRECISION=10
       		$mosdepth --reference $reference -n -t 2 -Q 20 -b $work/targetsBED.bed $work/$pat $cram
       		zcat $work/$pat.regions.bed.gz > $work/$pat.regions.bed
       		# remultiply the per-base average by the size to have the total coverage
       		mkdir -p $work/coverage
       		awk -F"\t" '{print $1 "\t" $2 "\t" $3 "\t" $4 "\t" $5*($3-$2)}' $work/$pat.regions.bed > $work/coverage/$pat.cov.tsv
       		# cleaning
       		rm $work/$pat.regions.bed.gz $work/$pat.regions.bed $work/$pat.mosdepth* $work/$pat.regions.bed.gz.csi
       	fi
       done < $listCRAM.temp
       ```

   11. ==删除中间文件命令中的bam修改为cram：==&#x20;

       123行：

       ```markup
       rm $work/targetsBED.bed $listBAM.temp
       ```

       修改为：

       ```markup
       rm $work/targetsBED.bed $listCRAM.temp
       ```

   保存为02_cram_coverage-count.sh以作区分

##### Step3：尝试运行：02_cram_coverage-count.sh

1. 命令已修改为

   ```markup
   bash 02_cram_coverage-count.sh \
   --listCRAM sequence_cram.txt \
   --mosdepth mosdepth-master/mosdepth \
   --work 11_23_cram_50_output_directory \
   --targetsBED data/11_23_cram_50_target-panel.bed \
   --reference hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa
   ```

2. OFF-PEAK已被整理至Software文件夹中（A3服务器），进入OFF-PEAK文件夹中

   ```markup
   cd Software
   cd OFF-PEAK
   ```

3. 使用dos2unix具将 Windows 的换行符转换为 Unix 格式：

   ```markup
   dos2unix sequence_cram.txt
   ```

4. 运行OFF-PEAK的Step1：

   ```markup
   bash 01_targets-processing.sh \
   --genome hg38 \
   --targets hg38_bed/output_1000G_Exome.v1.bed \
   --name 11_23_cram_50_target-panel \
   --ref hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa \
   --minOntarget 100 \
   --maxOntarget 300 \
   --minOfftarget 1 \
   --maxOfftarget 50000 \
   --paddingOfftarget 300
   ```

5. 运行OFF-PEAK的Step2：

   ```markup
   bash 02_cram_coverage-count.sh \
   --listCRAM sequence_cram.txt \
   --mosdepth mosdepth-master/mosdepth \
   --work 11_23_cram_50_output_directory \
   --targetsBED data/11_23_cram_50_target-panel.bed \
   --reference hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa
   ```

   ==失败：==&#x20;

   ```markup
   Step 1: Using mosdepth to extract coverage for each on-target and off-target
   more than 1 field
      Processing sample HG00096
   Usage: mosdepth [options] <prefix> <BAM-or-CRAM>
   error parsing arguments
   gzip: 11_23_cram_50_output_directory/HG00096.regions.bed.gz: No such file or directory
   rm: cannot remove '11_23_cram_50_output_directory/HG00096.regions.bed.gz': No such file or directory
   rm: cannot remove '11_23_cram_50_output_directory/HG00096.mosdepth*': No such file or directory
   rm: cannot remove '11_23_cram_50_output_directory/HG00096.regions.bed.gz.csi': No such file or directory
   ```

   尝试改进：

   1. 进入mosdepth所在的mosdepth-master文件夹中以此命令尝试运行mosdepth：

      ```markup
      cd mosdepth-master
      ./mosdepth -n -t 2 -Q 20 -b ../hg38_bed/output_1000G_Exome.v1.bed 11_23_cram_50_output_directory/HG00096_output ../../../../share/1000GP_data/1kgp_exome_cram/HG00096.alt_bwamem_GRCh38DH.20150826.GBR.exome.cram
      ```

      成功生成：

      | 名称                                      | 大小     |
      | :-------------------------------------- | :----- |
      | HG00096_output.mosdepth.global.dist.txt | 227kb  |
      | HG00096_output.mosdepth.region.dist.txt | 414kb  |
      | HG00096_output.mosdepth.summary.txt     | 2kb    |
      | HG00096_output.regions.bed.gz           | 20.9mB |
      | HG00096_output.regions.bed.gz.csi       | 338kb  |

      经检查，生成的文件大小，计算的覆盖度都没问题

   2. ==推断：脚本里调用mosdepth的指令中仍有错误：== 大概率出在reference上，猜测为113行出问题：

   3. ==分析：== 综合分析，发现原因出在单独运行mosdepth时是将reference设置为环境变量直接调用，而脚本中则使用--reference调用

   4. 思路：删除113行--reference指令，在此步骤前添加设置reference为环境变量指令

      1. 113行由：

         ```markup
         $mosdepth --reference $reference -n -t 2 -Q 20 -b $work/targetsBED.bed $work/$pat $cram
         ```

         更改为：

         ```markup
         $mosdepth -n -t 2 -Q 20 -b $work/targetsBED.bed $work/$pat $cram
         ```

      2. 将reference设置为环境变量代码写入至脚本93行：

         ```markup
         export REF_PATH=$reference
         ```

6. ==再次尝试：==&#x20;

   ```markup
   bash 02_cram_coverage-count.sh \
   --listCRAM sequence_cram.txt \
   --mosdepth mosdepth-master/mosdepth \
   --work 11_23_cram_50_output_directory \
   --targetsBED data/11_23_cram_50_target-panel.bed \
   --reference hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa
   ```

   成功运行

##### Step4：尝试运行OFF-PEAK的Rscript 03_OFF-PEAK.R

1. 进入创建好的conda的R4.4环境（具体步骤记录在OFF-PEAK使用记录完整版中）：

   ```markup
   source ~/.bashrc
   conda activate r44
   ```

   （进入R环境中时记得更改镜像源：options(repos = c(CRAN = "https://mirrors.tuna.tsinghua.edu.cn/CRAN/"))
   options(BioC_mirror = "https://mirrors.tuna.tsinghua.edu.cn/bioconductor/")） ）

2. 运行：

   ```markup
   cd Softwares/OFF-PEAK-main/ \
   Rscript 03_OFF-PEAK.R \
   --output 11_23_cram_50_Step3_output_directory \
   --data 11_23_cram_50_output_directory/ALL.target.tsv \
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



