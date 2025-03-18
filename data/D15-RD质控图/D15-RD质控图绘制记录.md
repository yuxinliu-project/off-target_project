#### 绘制RD图实验记录

#### 目的：检测2692个cram的覆盖度，筛选出不合格的cram

#### 时间：2024/12/20

#### 服务器：A3

#### 所用软件：gatk4.2.0.0

##### Step1.使用gatk4.2.0.0检测覆盖度

1. 分为36组，前35组75个cram，第36组67个cram

   list文件位置：/data/liuyuxin/1000_project_output/2692_cram_36_list

   gatk输出目录：/data/liuyuxin/1000_project_output/group*_depth

   创建脚本：groups_36.sh（gpt生成）

   1. 赋予权限

      ```markup
      chmod +x groups_36.sh
      ```

   2. 写入脚本

      ```markup
      #!/bin/bash

      # 定义输入文件路径
      input_file="list_sorted.txt"

      # 定义输出目录
      output_dir="/data/liuyuxin/1000_project_output/2692_cram_36_list"

      # 确保输出目录存在
      mkdir -p "$output_dir"

      # 总文件数
      total_files=$(wc -l < "$input_file")

      # 每组的文件数
      group_size=75

      # 总组数
      total_groups=36

      # 当前文件计数
      current_file=0

      # 循环分组
      for i in $(seq 1 $total_groups); do
          if [[ $i -eq $total_groups ]]; then
              # 最后一组
              group_files=$((total_files - current_file))
          else
              # 普通组
              group_files=$group_size
          fi

          # 输出当前组的文件到对应的列表
          start=$((current_file + 1))
          end=$((current_file + group_files))
          sed -n "${start},${end}p" "$input_file" > "$output_dir/group_$i.list"

          # 更新当前文件计数
          current_file=$((current_file + group_files))
      done

      echo "文件分组完成！分组输出到 $output_dir"
      ```

   3. 运行脚本：

      ```markup
      ./groups_36.sh
      ```

      成功分组

2. 创建36个文件夹以供输出分组：创建create_folders.sh

   1. 赋予权限

      ```markup
      chmod +x create_36_folders.sh
      ```

   2. 写入脚本：

      ```markup
      #!/bin/bash

      # 定义输出目录
      output_dir="/data/liuyuxin/1000_project_output"

      # 创建文件夹
      for i in $(seq 1 36); do
          mkdir -p "$output_dir/group_${i}_depth"
      done

      echo "36个文件夹已创建，路径为：$output_dir"
      ```

   3. 运行脚本：

      ```markup
      ./create_36_folders.sh
      ```

      成功创建文件夹

3. gatk4.6.0.0需Java环境，Java11已下载至data/liuyuxin

   配置环境：

   ```R
   export JAVA_HOME=/data/liuyuxin/java/jdk-11.0.2
   export PATH=$JAVA_HOME/bin:$PATH
   #验证是否成功
   java -version
   #存放至环境变量
   echo 'export JAVA_HOME=/data/liuyuxin/java/jdk-11.0.2' >> ~/.bashrc
   echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
   ```

   存放至环境变量失败，每次都要重新运行

4. 运行gatk创建字典文件：

   ```R
   cd Softwares/gatk-4.2.0.0
   python3 gatk CreateSequenceDictionary
   -R /data/liuyuxin/Project_Data/hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa
   -O /data/liuyuxin/Project_Data/hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.dict
   ```

   成功

5. 运行gatk检测覆盖度：36组，10线程运行

   ```markup
   python3 gatk --java-options "-Xmx8G -XX:ParallelGCThreads=10" DepthOfCoverage \
   -R /data/liuyuxin/Project_Data/hg38_reference/GRCh38_full_analysis_set_plus_decoy_hla.fa \
   -I /data/liuyuxin/1000_project_output/2692_cram_36_list/group_1.list \
   -O /data/liuyuxin/1000_project_output/group_1_depth/group_1_depth_gatk_output \
   -L /data/liuyuxin/Project_Data/hg38_bed/merge_Exome.v1.bed
   ```

   成功运行全部36组，运行时间：20/17:00-22/9:44，1900-3200 分钟，31-53小时

##### Step2.合并36组depth

1. 使用xhmm的mergeGATKdepths工具36组depth

   ```markup
   ./xhmm \
   --mergeGATKdepths \
   -o ./2692_cram_mergedepths.RD.txt \
   --GATKdepths /data/liuyuxin/1000_project_output/group_1_depth/group_1_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_2_depth/group_2_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_3_depth/group_3_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_4_depth/group_4_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_5_depth/group_5_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_6_depth/group_6_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_7_depth/group_7_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_8_depth/group_8_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_9_depth/group_9_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_10_depth/group_10_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_11_depth/group_11_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_12_depth/group_12_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_13_depth/group_13_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_14_depth/group_14_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_15_depth/group_15_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_16_depth/group_16_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_17_depth/group_17_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_18_depth/group_18_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_19_depth/group_19_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_20_depth/group_20_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_21_depth/group_21_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_22_depth/group_22_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_23_depth/group_23_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_24_depth/group_24_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_25_depth/group_25_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_26_depth/group_26_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_27_depth/group_27_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_28_depth/group_28_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_29_depth/group_29_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_30_depth/group_30_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_31_depth/group_31_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_32_depth/group_32_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_33_depth/group_33_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_34_depth/group_34_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_35_depth/group_35_DepthOfCoverage_noNaN.sample_interval_summary \
   --GATKdepths /data/liuyuxin/1000_project_output/group_36_depth/group_36_DepthOfCoverage_noNaN.sample_interval_summary 
   ```

   ==失败：== 生成的.sample_interval_summary文件只有1行1列，xhmm识别失败

2. 处理.sample_interval_summary：以group1为例

   ```Shell
   awk '{gsub(",", "\t"); print}' group_1_depth_gatk_output.sample_interval_summary > group_1_DepthOfCoverage.sample_interval_summary
   ```

3. 合并仍然失败，因为里面包含NaN，删除有NaN数据的所有行（保险起见awk制作新文件）

   ```Shell
   awk '!/NaN/' group_1_DepthOfCoverage.sample_interval_summary > group_1_DepthOfCoverage_noNaN.sample_interval_summary
   ```

4. 重新使用xhmm的mergeGATKdepths工具合并36组depths

   ==成功==

##### Step3.绘制RD、D15图

1. 用RStudio计算所有样本的D15

   ```R
   # 加载数据
   coverage_data <- read.table("2692_cram_mergedepths.RD.txt", header = TRUE, sep = "\t", check.names = FALSE)

   # 总区域数
   total_regions <- ncol(coverage_data) - 1  # 除去样本名列

   # 计算 D15
   D15_values <- apply(coverage_data[, -1], 1, function(x) {
       sum(x >= 15) / total_regions * 100
   })

   # 创建 D15 数据框
   D15_data <- data.frame(Sample = coverage_data$GATK._mean_cvg, D15_Percentage = D15_values)

   # 保存到文件
   write.table(D15_data, "D15_coverage_quality.txt", sep = "\t", row.names = FALSE, quote = FALSE)

   ```

2. 在RStudio中绘图

   ```markup
   # 加载必要的包
   library(ggplot2)

   # 1. 读取原始覆盖度文件
   coverage_data <- read.table("2692_cram_mergedepths.RD.txt", header = TRUE, sep = "\t", check.names = FALSE)

   # 2. 计算每个样本的平均覆盖度（RD）
   # 去掉第一列（样本名）后计算均值
   mean_coverage <- rowMeans(coverage_data[, -1])

   # 3. 加载 D15 数据
   d15_data <- read.table("D15_coverage_quality.txt", header = TRUE, sep = "\t")

   # 4. 合并 RD 和 D15 数据
   # 添加样本列名以便匹配
   coverage_data$Sample <- coverage_data$GATK._mean_cvg  # 样本名列
   plot_data <- merge(
     data.frame(Sample = coverage_data$Sample, Mean_Coverage = mean_coverage),
     d15_data,
     by = "Sample"
   )

   # 检查合并结果
   print(head(plot_data))

   # 5. 绘制散点图
   ggplot(plot_data, aes(x = Mean_Coverage, y = D15_Percentage)) +
     geom_point(color = "blue", size = 2) +  # 绘制散点
     theme_minimal() +                      # 应用简洁主题
     labs(
       title = "Mean Coverage (RD) vs Percentage Above 15X (D15)",
       x = "Mean Coverage (RD)",
       y = "Percentage Above 15X (D15)"
     ) +
     theme(
       plot.title = element_text(hjust = 0.5, size = 16),
       axis.title.x = element_text(size = 14),
       axis.title.y = element_text(size = 14)
     )

   ```

   ==成功生成：== RD_D15.pdf

   ==sample名字未标出，改成交互式图像：==

   ```markup
   library(plotly)

   # 使用 plotly 绘制交互式散点图
   plot <- plot_ly(
     data = plot_data,                          # 数据框
     x = ~Mean_Coverage,                        # 横坐标为 Mean_Coverage
     y = ~D15_Percentage,                       # 纵坐标为 D15_Percentage
     text = ~Sample,                            # 鼠标悬停显示样本名称
     type = "scatter",                          # 散点图
     mode = "markers",                          # 绘制点
     marker = list(color = 'blue', size = 5)    # 设置点的颜色和大小
   )

   # 添加标题和坐标轴标签
   plot <- plot %>%
     layout(
       title = "Mean Coverage (RD) vs Percentage Above 15X (D15)",
       xaxis = list(title = "Mean Coverage (RD)"),
       yaxis = list(title = "Percentage Above 15X (D15)")
     )

   # 显示图形
   plot

   #输出交互式图像
   library(htmlwidgets)
   saveWidget(plot, "RD_D15_plot.html", selfcontained = TRUE)

   ```

   ==输出：== RD_D15_plot.html

   成功，将RD/D15两幅图输出存放至/data/liuyuxin/1000_project_output/xhmm_merge_2692_depth中

