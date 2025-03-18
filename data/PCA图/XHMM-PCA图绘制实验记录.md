### XHMM数据绘制PCA图实验记录

##### 日期：2025-01-06

##### 工具：Xhmm、RSTudio

##### 数据：经gatk计算过的2692个cram的融合覆盖度矩阵文件2692_cram_mergedepths.RD.txt

##### 服务器：A3

##### 文件夹：/data/liuyuxin/PC_plot/XHMM_PCA_plot



###### Step1. 使用XHMM运行PCA

1. 使用xhmm中心化、标准化

   ```markup
   ./xhmm \
   --matrix \
   -r ./2692_cram_mergedepths.RD.txt \
   --centerData \
   --centerType target \
   -o ./2692_cram_mergedepths.filtered_centered.RD.txt
   ```

   | 位置：/data/liuyuxin/PC_plot/XHMM_PCA_plot        | 注释                   |
   | :--------------------------------------------- | :------------------- |
   | 2692_cram_mergedepths.filtered_centered.RD.txt | 中心、标准化Sample/Depth矩阵 |

2. PCA

   ```markup
   ./xhmm \
   --PCA \
   -r ./2692_cram_mergedepths.filtered_centered.RD.txt \
   --PCAfiles ./2692_cram_mergedepths.RD_PCA
   ```

   生成：

   | 位置：/data/liuyuxin/PC_plot/XHMM_PCA_plot      | 注释      |
   | :------------------------------------------- | :------ |
   | 2692_cram_mergedepths.RD_PCA.PC.txt          | 主成分载荷矩阵 |
   | 2692_cram_mergedepths.RD_PCA.PC_LOADINGS.txt | 主成分得分矩阵 |
   | 2692_cram_mergedepths.RD_PCA.PC_SD.txt       | 主成分的标准差 |

###### Step2. RSTudio绘制交互式PCA图

1. RSTudio读取数据

```markup
# 读取主成分得分矩阵
pca_scores <- read.delim("2692_cram_mergedepths.RD_PCA.PC_LOADINGS.txt", header = TRUE, sep = "\t", row.names = 1)

# 读取主成分标准差
pca_sd <- read.delim("2692_cram_mergedepths.RD_PCA.PC_SD.txt", header = TRUE, sep = "\t")

```

2. &#x20;计算方差解释比例

```markup
# 计算每个主成分的方差
pca_variance <- (pca_sd$SD)^2

# 计算总方差
total_variance <- sum(pca_variance)

# 计算方差解释比例
variance_explained <- pca_variance / total_variance * 100

# 查看 PC1 和 PC2 的解释比例
cat("PC1 Variance Explained: ", round(variance_explained[1], 2), "%\n")
cat("PC2 Variance Explained: ", round(variance_explained[2], 2), "%\n")

```

3. 转换主成分得分矩阵

```markup
# 转置矩阵，使样本为行，主成分为列
pca_scores_transposed <- as.data.frame(t(pca_scores))

# 添加样本名列
pca_scores_transposed$Sample <- rownames(pca_scores_transposed)

```

4. 绘制交互式 PC1-PC2 图

```markup
library(plotly)

# 绘制交互式散点图
p <- plot_ly(
  data = pca_scores_transposed,
  x = ~PC1,
  y = ~PC2,
  text = ~paste("Sample:", Sample, "<br>PC1:", round(PC1, 2), "<br>PC2:", round(PC2, 2)),
  hoverinfo = "text",
  mode = "markers",
  marker = list(size = 5, color = 'blue', opacity = 0.7)
)

# 添加标题和轴标签
p <- p %>%
  layout(
    title = "PC1 - PC2",
    xaxis = list(title = paste0("PC1 (", round(variance_explained[1], 2), "% Variance Explained)")),
    yaxis = list(title = paste0("PC2 (", round(variance_explained[2], 2), "% Variance Explained)"))
  )

# 显示交互式图
p

```

