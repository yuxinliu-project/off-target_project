### R包绘制PCA图实验记录

##### 日期：2025-01-07

##### 工具：RSTudio

##### 数据：经gatk计算过的2692个cram的融合覆盖度矩阵文件2692_cram_mergedepths.RD.txt

##### 服务器：A3

##### 文件夹：/data/liuyuxin/PC_plot

###### Step1. 安装、加载所需R包

```markup
install.packages("plotly")
install.packages("readr")
library(plotly)
library(readr)
```

###### Step2. 读取sample/depth矩阵文件

```markup
# 读取文件
data <- read.delim("2692_cram_mergedepths.RD.txt", header = TRUE, sep = "\t", row.names = 1)
```

###### Step3. PCA

```markup
# 确保数据为数值型矩阵
data_matrix <- as.matrix(data)

# 执行PCA
pca_result <- prcomp(data_matrix, scale. = TRUE)

# 提取PCA得分
pca_scores <- as.data.frame(pca_result$x)

# 添加样本名作为一列
pca_scores$Sample <- rownames(pca_scores)

```

###### Step4. 绘制交互式PCA图

```markup
# 计算方差解释比例
explained_variance <- round(100 * (pca_result$sdev^2 / sum(pca_result$sdev^2)), 2)

# 绘制交互式PCA图
p <- plot_ly(
  data = pca_scores,
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
    xaxis = list(title = paste0("PC1 (", explained_variance[1], "% variance explained)")),
    yaxis = list(title = paste0("PC2 (", explained_variance[2], "% variance explained)"))
  )

# 显示图形
p
```

成功生成交互图，并保存中间文件至/data/liuyuxin/PC_plot：

| 文件名                           | 注释                  |
| :---------------------------- | :------------------ |
| pca_scores.csv                | 所有样本在各主成分上的得分矩阵     |
| pca_scores_with_row_names.csv | 同上，但包含样本名作为行名       |
| pca_loadings.csv              | 主成分载荷矩阵（各变量对主成分的贡献） |
| explained_variance.csv        | 各主成分解释的方差比例         |

