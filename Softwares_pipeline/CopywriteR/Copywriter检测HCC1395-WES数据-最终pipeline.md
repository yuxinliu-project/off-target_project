#### Copywriter检测HCC1395-WES数据-最终pipeline

根据SavvyCNV基准测试重新运行

```markup
source ~/.bashrc 
conda activate copywriter
```

1. 更换BiocParallel包版本：

   ```markup
   remove.packages("BiocParallel")
   install.packages(c("foreach", "BatchJobs", "BBmisc"))
   install.packages("/data/share/liuyuxin_tanrenjie/Softwares/Required-Softwares-Libraries/BiocParallel_0.6.1.tar.gz", 
                    repos = NULL, 
                    type = "source")
   ```

2. 检测：

   ```R
   library(CopyhelpeR) 
   library(CopywriteR) 
   bam_path <- "/data/share/liuyuxin_tanrenjie/HCC1395_data/WES"
   output_path <- "/data/share/liuyuxin_tanrenjie/Softwares_HCC1395-WES_output/CopywriteR_HCC1395-WES_output"
   capture_bed <- "/data/share/liuyuxin_tanrenjie/HCC1395_data/reference_genome/Exome_Target_bed/S07604624_delete.bed"
   preCopywriteR(output.folder = output_path, bin.size = 100000, ref.genome = "hg38")
   reference.folder = file.path(output_path, "hg38_100kb")
   load(file = file.path(output_path, "hg38_100kb", "blacklist.rda"))
   load(file = file.path(output_path, "hg38_100kb", "GC_mappability.rda"))

   tumor_samples <- c("WES_EA_Tumor_1",
                      "WES_FD_Tumor_1", "WES_FD_Tumor_2", "WES_FD_Tumor_3",
                      "WES_IL_Tumor_1", "WES_IL_Tumor_2", "WES_IL_Tumor_3",
                      "WES_LL_Tumor_1",
                      "WES_NC_Tumor_1",
                      "WES_NV_Tumor_1", "WES_NV_Tumor_2", "WES_NV_Tumor_3")

   normal_samples <- c("WES_EA_Normal_1",
                       "WES_FD_Normal_1", "WES_FD_Normal_2", "WES_FD_Normal_3",
                       "WES_IL_Normal_1", "WES_IL_Normal_2", "WES_IL_Normal_3",
                       "WES_LL_Normal_1",
                       "WES_NC_Normal_1",
                       "WES_NV_Normal_1", "WES_NV_Normal_2", "WES_NV_Normal_3")
   sample.control <- data.frame(
     samples = file.path(bam_path, paste0(tumor_samples, ".bwa.dedup.bam")),
     controls = file.path(bam_path, paste0(normal_samples, ".bwa.dedup.bam"))
   )
   bp.param <- SnowParam(workers = 12, type = "SOCK")
   CopywriteR(
     sample.control = sample.control, 
     destination.folder = output_path, 
     reference.folder = file.path(output_path, "hg38_100kb"),
     bp.param = bp.param
   )
   ```

   指令运行成功，BiocParallel包没问题了，但是chr格式没统一，报错了：

   out：

   ```markup
   > CopywriteR(
     sample.control = sample.control, 
     destination.folder = output_path, 
     reference.folder = file.path(output_path, "hg38_100kb"),
     bp.param = bp.param
   )
   The following samples will be analyzed: 
   sample: WES_EA_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_EA_Normal_1.bwa.dedup.bam
   sample: WES_FD_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_FD_Normal_1.bwa.dedup.bam
   sample: WES_FD_Tumor_2.bwa.dedup.bam ; 	 matching control: WES_FD_Normal_2.bwa.dedup.bam
   sample: WES_FD_Tumor_3.bwa.dedup.bam ; 	 matching control: WES_FD_Normal_3.bwa.dedup.bam
   sample: WES_IL_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_IL_Normal_1.bwa.dedup.bam
   sample: WES_IL_Tumor_2.bwa.dedup.bam ; 	 matching control: WES_IL_Normal_2.bwa.dedup.bam
   sample: WES_IL_Tumor_3.bwa.dedup.bam ; 	 matching control: WES_IL_Normal_3.bwa.dedup.bam
   sample: WES_LL_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_LL_Normal_1.bwa.dedup.bam
   sample: WES_NC_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_NC_Normal_1.bwa.dedup.bam
   sample: WES_NV_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_NV_Normal_1.bwa.dedup.bam
   sample: WES_NV_Tumor_2.bwa.dedup.bam ; 	 matching control: WES_NV_Normal_2.bwa.dedup.bam
   sample: WES_NV_Tumor_3.bwa.dedup.bam ; 	 matching control: WES_NV_Normal_3.bwa.dedup.bam
   The bin size for this analysis is 100000 
   The capture region file is not specified 
   This analysis will be run on 12 cpus 
   Error: 24 errors; first error:
     Error in .io_bam(.count_bamfile, file, param = param): seqlevels(param) not in BAM header:
       seqlevels: ‘1’, ‘2’, ‘3’, ‘4’, ‘5’, ‘6’, ‘7’, ‘8’, ‘9’, ‘10’, ‘11’, ‘12’, ‘13’, ‘14’, ‘15’, ‘16’, ‘17’, ‘18’, ‘19’, ‘20’, ‘21’, ‘22’, ‘X’, ‘Y’
       file: WES_EA_Tumor_1.bwa.dedup_properreads.bam
       index: WES_EA_Tumor_1.bwa.dedup_properreads.bam

   For more information, use bplasterror(). To resume calculation, re-call
     the function and set the argument 'BPRESUME' to TRUE or wrap the
     previous call in bpresume().

   First traceback:
     32: local({
             master <- "localhost"
             port <- ""
             snowlib <- Sys.getenv("R_SNOW_LIB")
             outfile <- Sys.getenv("R_SNOW_OUTFILE")
             args <- commandArgs()
             pos <- match("--args", args)
             args <- args[-(1:pos)]
             for (a in args) {
                 pos <- regexpr("=", a)
   ```

3. 重新生成带chr版本的pre文件：

   ```markup
   preCopywriteR(output.folder = output_path, bin.size = 100000, ref.genome = "hg38", prefix = "chr")
   bp.param <- SnowParam(workers = 24, type = "SOCK")
   CopywriteR(
     sample.control = sample.control, 
     destination.folder = output_path, 
     reference.folder = file.path(output_path, "hg38_100kb_chr"),
     bp.param = bp.param
   )
   ```

   失败：out

   ```markup
   The following samples will be analyzed: 
   sample: WES_EA_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_EA_Normal_1.bwa.dedup.bam
   sample: WES_FD_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_FD_Normal_1.bwa.dedup.bam
   sample: WES_FD_Tumor_2.bwa.dedup.bam ; 	 matching control: WES_FD_Normal_2.bwa.dedup.bam
   sample: WES_FD_Tumor_3.bwa.dedup.bam ; 	 matching control: WES_FD_Normal_3.bwa.dedup.bam
   sample: WES_IL_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_IL_Normal_1.bwa.dedup.bam
   sample: WES_IL_Tumor_2.bwa.dedup.bam ; 	 matching control: WES_IL_Normal_2.bwa.dedup.bam
   sample: WES_IL_Tumor_3.bwa.dedup.bam ; 	 matching control: WES_IL_Normal_3.bwa.dedup.bam
   sample: WES_LL_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_LL_Normal_1.bwa.dedup.bam
   sample: WES_NC_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_NC_Normal_1.bwa.dedup.bam
   sample: WES_NV_Tumor_1.bwa.dedup.bam ; 	 matching control: WES_NV_Normal_1.bwa.dedup.bam
   sample: WES_NV_Tumor_2.bwa.dedup.bam ; 	 matching control: WES_NV_Normal_2.bwa.dedup.bam
   sample: WES_NV_Tumor_3.bwa.dedup.bam ; 	 matching control: WES_NV_Normal_3.bwa.dedup.bam
   The bin size for this analysis is 100000 
   The capture region file is not specified 
   This analysis will be run on 24 cpus 
   Error: 12 errors; first error:
     Error in if (nrow(peaks) > 0) {: argument is of length zero

   For more information, use bplasterror(). To resume calculation, re-call
     the function and set the argument 'BPRESUME' to TRUE or wrap the
     previous call in bpresume().

   First traceback:
     26: local({
             master <- "localhost"
             port <- ""
             snowlib <- Sys.getenv("R_SNOW_LIB")
             outfile <- Sys.getenv("R_SNOW_OUTFILE")
             args <- commandArgs()
             pos <- match("--args", args)
             args <- args[-(1:pos)]
             for (a in args) {
                 pos <- regexpr("=", a)
                 name <- substr(a, 1, pos - 1)
                 value <- substr(a, pos + 1, nchar(a))
                 switch(name, MASTER = master <- value, PORT = port <- value, 
                     SNOWLIB = snowlib <- value, OUT = outfile <- value)
             }
             if (!(snowlib %in% .libPaths())) 
                 .libPaths(c(snowlib, .libPaths()))
             library(methods)
             li
   ```

4. 推测失败原因为bin设置问题

