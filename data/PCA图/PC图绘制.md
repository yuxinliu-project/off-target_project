### PC图绘制

使用本地RSTudio

使用R进行PCA

1. 安装和加载必要的包

   ```markup
   install.packages(c("ggplot2", "scales"))
   library(ggplot2)
   ```

2. 读取和预处理数据

   ```markup
   # 读取数据
   # 假设文件是制表符分隔的，如果是逗号分隔的，修改sep参数为','
   cnv_data <- read.table("2692_cram_mergedepths.RD.txt", header=TRUE, sep="\t", row.names=1)

   # 查看数据基本信息
   dim(cnv_data)
   head(cnv_data)

   # 检查缺失值
   sum(is.na(cnv_data))

   # 如果有缺失值，可以选择删除包含缺失值的样本
   cnv_data <- na.omit(cnv_data)

   # 标准化数据
   scaled_cnv <- scale(cnv_data, center = TRUE, scale = TRUE)

   # 执行PCA
   pca_result <- prcomp(scaled_cnv, center = FALSE, scale. = FALSE)  # 已经标准化

   # 查看PCA结果
   summary(pca_result)

   ```

   保存result：

   ```markup
   # 保存标准差 (sdev)
   write.csv(pca_result$sdev, file = "pca_sdev.csv", row.names = FALSE)

   # 保存主成分的载荷矩阵 (rotation)
   write.csv(pca_result$rotation, file = "pca_rotation.csv")

   # 保存主成分得分矩阵 (x)
   write.csv(pca_result$x, file = "pca_scores.csv")

   ```

3. 绘制PC图

   ```markup
   # 提取前两个主成分
   pca_df <- data.frame(PC1 = pca_result$x[,1],
                        PC2 = pca_result$x[,2],
                        Sample = rownames(cnv_data))

   # 绘制PC图
   ggplot(pca_df, aes(x = PC1, y = PC2, label = Sample)) +
     geom_point(color = 'blue', size = 2) +
     geom_text(vjust = 1.5, size = 3) +
     labs(title = "PCA of CNV Coverage Data",
          x = paste0("PC1 (", round(summary(pca_result)$importance[2,1]*100, 2), "%)"),
          y = paste0("PC2 (", round(summary(pca_result)$importance[2,2]*100, 2), "%)")) +
     theme_minimal()

   ```

4. 交互图

   ```markup
   # 加载必要的包
   library(plotly)

   # 创建交互式 PCA 图
   plotly_pca <- plot_ly(
     data = pca_df,
     x = ~PC1,
     y = ~PC2,
     type = 'scatter',
     mode = 'markers',
     text = ~paste("Sample:", Sample, "<br>PC1:", round(PC1, 2), "<br>PC2:", round(PC2, 2)),
     hoverinfo = 'text',
     marker = list(size = 10, color = 'blue')
   ) %>%
     layout(
       title = "Interactive PCA Plot of CNV Coverage Data",
       xaxis = list(title = paste0("PC1 (", round(summary(pca_result)$importance[2, 1]*100, 2), "%)")),
       yaxis = list(title = paste0("PC2 (", round(summary(pca_result)$importance[2, 2]*100, 2), "%)")),
       showlegend = FALSE
     )

   # 显示交互式图表
   plotly_pca

   ```

