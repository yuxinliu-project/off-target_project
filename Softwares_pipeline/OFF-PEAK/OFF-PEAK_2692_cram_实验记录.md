#### OFF-PEAK检测2692个cram程序设计：

###### 1. 构建脚本自动创建18组list：

```markup
#!/bin/bash
files=($(ls /data/share/1000GP_data/1kgp_exome_cram/*.cram))

total_files=${#files[@]}

group_size=150
last_group_size=142

for i in $(seq 1 18); do
    touch "group_$i.txt"
done

for i in $(seq 0 $((total_files - 1))); do
    group_index=$((i / group_size))  
    if [ $group_index -lt 17 ]; then
        group_number=$((group_index + 1))
    else
        group_number=18
    fi

    echo "${files[$i]}" >> "group_${group_number}.txt"
done

```

成功创建group1-18

###### 2. 设计程序：

1. Step1.

   ```markup
   bash 01_targets-processing.sh \
   --genome hg38 \
   --targets hg38_bed/output_1000G_Exome.v1.bed \
   --name 2692_cram_target-panel \
   --ref hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa \
   --minOntarget 100 \
   --maxOntarget 300 \
   --minOfftarget 1 \
   --maxOfftarget 50000 \
   --paddingOfftarget 300
   ```

2. Step2.（18组）

   ```markup
   cd Softwares/OFF-PEAK-main/ 
   bash 02_cram_coverage-count.sh \
   --listCRAM 2692_cram_18_list/group_18.txt \
   --mosdepth mosdepth-master/mosdepth \
   --work 2692_cram_output_directory/group_18_output_directory \
   --targetsBED data/2692_cram_target-panel.bed \
   --reference hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa
   ```

3. Step3.

   进入conda创建的r环境（4.4）

   ```markup
   source ~/.bashrc
   conda activate r44
   ```

   运行：

   ```markup
   Rscript 03_OFF-PEAK.R \
   --output 2692_cram_Step3_output_directory/group_18_Step3_output_directory \
   --data 2692_cram_output_directory/group_18_output_directory/ALL.target.tsv \
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

成功
