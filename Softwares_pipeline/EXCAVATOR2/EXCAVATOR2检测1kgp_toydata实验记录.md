### EXCAVATOR2使用记录

#### 日期：2025-01-13

##### 工具：Xshell

##### 测试数据：/data/share/1000GP_data/1kgp_exome_bam里10个bam文件

##### 服务器：A3

##### 文件夹：/data/liuyuxin/Softwares/Excavator2



###### Step1.准备samtools、bedtools至环境中

1. samtools

```markup
echo 'export PATH=/data/liuyuxin/Softwares/samtools-1.21:$PATH' >> ~/.bashrc
#测试
which samtools
samtools --version

```

2. bedtools

```markup
cd /data/liuyuxin/Softwares
wget https://github.com/arq5x/bedtools2/releases/download/v2.30.0/bedtools-2.30.0.tar.gz
tar -xzvf bedtools-2.30.0.tar.gz
cd bedtools2    # 或 bedtools-2.30.0
make
mkdir -p /data/liuyuxin/Softwares/bedtools/bin
cp bin/bedtools /data/liuyuxin/Softwares/bedtools/bin
echo 'export PATH=/data/liuyuxin/Softwares/bedtools/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
#测试
which bedtools
bedtools --version
```



##### 该软件分成了3个模块，对应3个脚本，具体操作步骤如下

##### 1. TargetPerla.pl

提供一个捕获区域的bed文件，计算in-target和off-target区域的GC含量，mappability值，用于后续的归一化操作

```R
perl TargetPerla.pl \
SourceTarget.txt \  #记录了基因组对应的bw文件和fasta文件的路径，文件夹里自带，使用时更改：/data/liuyuxin/Softwares/Excavator2/EXCAVATOR2_Package_v1.1.2/data/GCA_000001405.15_GRCh38.bw /data/liuyuxin/Project_Data/hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa；空格分隔的两列，第一列为bw文件的路径，是软件自带的，位于软件的安装目录，用于计算基因组不同区域的mappability，第二列为fasta文件的路径，用于计算不同区域的GC含量。
/data/liuyuxin/Project_Data/hg38_bed/merge_Exome.v1.bed \  #捕获区域的bed文件
text_10cram_Target_w50000 \  #输出结果的前缀
50000 \ #窗口的固定长度（off-target划分为多个50000bp（50kb）
hg38 #指定基因组的版本
```

==失败：== 运行 TargetPerla.pl 脚本时遇到的主要问题是缺少 libpng12.so.0 共享库。这导致 bigWigAverageOverBed 工具无法正常运行，从而阻碍了 TargetPerla.pl 脚本的执行

&#x20;==解决：== 安装 libpng12.so.0

```Shell
mkdir -p /data/liuyuxin/Softwares/libpng12
cd /data/liuyuxin/Softwares/libpng12

#官网下载：https://sourceforge.net/projects/libpng/files/libpng12/older-releases/1.2.50/libpng-1.2.50.tar.gz
tar -zxvf libpng-1.2.50.tar.gz #解压
cd libpng-1.2.50
./configure --prefix=/data/liuyuxin/Softwares/libpng12 #编译
make
make install
echo 'export LD_LIBRARY_PATH=/data/liuyuxin/Softwares/libpng12/lib:$LD_LIBRARY_PATH' >> ~/.bashrc  #配置环境变量
source ~/.bashrc
ldd /data/liuyuxin/Softwares/Excavator2/EXCAVATOR2_Package_v1.1.2/lib/OtherLibrary/bigWigAverageOverBed | grep libpng12  #验证安装
```



==重新尝试：==&#x20;

```Shell
chmod +x /data/liuyuxin/Softwares/Excavator2/EXCAVATOR2_Package_v1.1.2/lib/OtherLibrary/bigWigAverageOverBed  #赋予执行权限
#重新运行
perl TargetPerla.pl \
SourceTarget.txt \
/data/liuyuxin/Project_Data/hg38_bed/merge_Exome.v1.bed \
text_10cram_Target_w50000 \
50000 \
hg38

```

成功

##### 2. EXCAVATORDataPrepare.pl

计算测序深度，进行归一化处理

```markup
perl EXCAVATORDataPrepare.pl \
10cram_test_list.txt \  #样本对应的cram文件，输出结果的路径，样本名称信息：/data/share/1000GP_data/1kgp_exome_cram/HG00096.alt_bwamem_GRCh38DH.20150826.GBR.exome.cram HG00096 /data/liuyuxin/Softwares/Excavator2/output/HG00096 HG00096；空格分隔
--processors 30 \  #指定并行的线程数
--target text_10cram_Target_w50000 \  #参数指定第一步生成的target的名称
--assembly hg38
```



###### 出现问题，重新从安装开始

```markup
# 1.解压软件包
tar -xvzf EXCAVATOR2_Package_v1.1.2.tgz

# 2.编译 Fortran 子程序
cd /path/to/EXCAVATOR2/lib/F77
R CMD SHLIB F4R.f
R CMD SHLIB FastJointSLMLibraryI.f

# 3.设置环境变量
export PATH=$PATH:/path/to/EXCAVATOR2

```

###### Step1. TargetPerla.pl

```markup
perl TargetPerla.pl \
SourceTarget.txt \
/data/liuyuxin/Project_Data/hg19_bed/20130108.exome.targets.updated.bed \
text_10bam_Target_w50000 \
50000 \
hg19
```

==成功==&#x20;

###### Step2. EXCAVATORDataPrepare.pl

```markup
perl EXCAVATORDataPrepare.pl \
10bam_test.list.txt \
--processors 30 \
--target text_10bam_Target_w50000 \
--assembly hg19
```

###### bed、fa、bam格式未统一，统一后再重新运行Step1

```markup
perl TargetPerla.pl \
SourceTarget.txt \
/data/liuyuxin/Project_Data/hg19_bed/20130108.exome.targets.updated.bed \
text_10bam_Target_w50000 \
50000 \
hg19
```

###### Step2. EXCAVATORDataPrepare.pl

```markup
perl EXCAVATORDataPrepare.pl \
10bam_test.list.txt \
--processors 30 \
--target text_10bam_Target_w50000 \
--assembly hg19
```

==失败：缺少R包==

安装所需R包：

```markup
install.packages("Hmisc")
install.packages("bitops")
install.packages("caTools")
install.packages("ggplot2")
install.packages("data.table")
install.packages("dplyr")

```

重新运行Step2：

```markup
perl EXCAVATORDataPrepare.pl \
10bam_test.list.txt \
--processors 30 \
--target text_10bam_Target_w50000 \
--assembly hg19
```

成功

##### 3. EXCAVATORDataAnalysis.pl

执行HSLM segmentation算法和FastCall算法，进行CNV分析

选择NA06985作为控制样本

选择将所有的实验样本混合与对照样本进行比较的pooling模式

```markup
perl EXCAVATORDataAnalysis.pl \
ExperimentalFileAnalysis.txt \
--processors 30  \
--target text_10bam_Target_w50000 \
--assembly hg19 \
--output Results_MyProject_w50K \
--mode pooling
```



运行成功，生成文件存放至/data/liuyuxin/Softwares/Excavator2/EXCAVATOR2_Package_v1.1.2/Results_10bam_test_w50K中





























