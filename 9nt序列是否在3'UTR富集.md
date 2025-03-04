# 准备文件
### 全基因组长度并统计次数
h38.gtf是已经筛选过3'UTR的基因注释文件
``` 
##统计该序列在基因组(man_slect)中出现的次数
grep 'tag "MANE_Select"' Homo_sapiens.GRCh38.113.gtf > h38_man.gtf （公共）
bedtools intersect -a h38_man.gtf -b locate_1.bed -wa -wb > sum
wc -l sum #39669次
##筛选3utr mane_select标签的基因gtf文件（公共）
grep 'tag "MANE_Select"' h38.gtf > h38_man_3utr.gtf
```
```
##
awk -F'\t' '$3 == "transcript" {print}' h38_man.gtf > h38_man_transcript.gtf
```

```
##Rstudio  获取染色体长度（公共）
#安装包
if (!require("BiocManager", quietly = TRUE))
  install.packages("BiocManager")
BiocManager::install("Biostrings")
library(Biostrings) 
#读取FASTA文件中的基因组序列
genome <- readDNAStringSet("D:/新桌面/zhaolab/生信/元件匹配_12.19/ATTTTGCTT/R数据文件/Homo_sapiens.GRCh38.dna.primary_assembly.fa") 
#筛选出主要的常染色体和性染色体
main_chromosomes <- grep("^(\\d+|X|Y)", names(genome), value = TRUE)
genome_main <- genome[main_chromosomes]
#获取每个染色体的长度 
chromosome_lengths <- width(genome_main) 
names(chromosome_lengths) <- c("1","10","11","12","13","14","15","16","17","18","19",
                               "2","20","21","22","3","4","5","6","7","8","9","X","Y")
#查看染色体信息（例如，chromosome名和长度） 
chromosome_lengths
save(chromosome_lengths,file = "D:/新桌面/zhaolab/生信/元件匹配_12.19/ATTTTGCTT/R数据文件/chromosome_lengths.RData")
```
 ### 3'UTR基因组位置文件
 ```
##Rstudio 获取3utr 标签mane_select的位置信息（公共）
utr_3_prime_regions <- read.table("D:/新桌面/h38_man_3utr.gtf", header = FALSE,sep = "\t")
library(dplyr)
utr_3_prime_regions <- utr_3_prime_regions %>% 
  rename(chromosome = V1, start = V4, end = V5, strand = V7)
utr_3_prime_regions <- utr_3_prime_regions %>% select(chromosome, start, end, strand)
save(utr_3_prime_regions,file = "D:/新桌面/zhaolab/生信/元件匹配_12.19/ATTTTGCTT/R数据文件/utr_3_prime_regions.RData")
```
### 随机模拟进行富集分析（Rstudio）
```
##模拟9nt序列的抽取并统计匹配
set.seed(123)  #设置随机种子
n_simulations <- 10000  #模拟次数
sequences_per_simulation <- 39669 #每次模拟抽取的序列数(可变)
sequence_length <- 9  #序列长度是9nt
genome_length <- sum(chromosome_lengths)  #假设基因组的长度

# 记录每次模拟中位于3'UTR的位点数量
utr_sense_count <- numeric(n_simulations)

# 创建一个数据框，保存每个染色体的长度
chromosome_data <- data.frame(
  chromosome = names(chromosome_lengths),
  length = chromosome_lengths)

##is_in_utr函数检查位点
# 检查一个位置是否在任意3' UTR区域内
is_in_utr <- function(position, utr_regions) {
  any(position >= utr_regions$start & position <= utr_regions$end)
}

# 模拟过程
for (i in 1:n_simulations) {
  # 随机生成39669个9nt序列的染色体和起始位置
  random_chromosomes <- sample(chromosome_data$chromosome, sequences_per_simulation, replace = TRUE)
  random_positions <- numeric(sequences_per_simulation)
  
  for (j in 1:sequences_per_simulation) {
    chrom <- random_chromosomes[j]
    chrom <- as.character(chrom)
    chrom_length <- chromosome_data$length[chromosome_data$chromosome == chrom]
    
    # 在指定染色体范围内随机选择9nt序列的起始位置
    random_positions[j] <- sample(1:(chrom_length - sequence_length + 1), 1)
  }
  # 统计有多少个9nt序列位于任意3' UTR区域
  utr_count <- sum(sapply(random_positions, is_in_utr,  utr_regions = utr_3_prime_regions[utr_3_prime_regions$chromosome == chrom,]))
  # 保存每次模拟的结果
  utr_sense_count[i] <- utr_count
}

#转换文件内容格式
count <- utr_sense_count %>% 
         sort() %>%
         table()
count <- as.data.frame(count)
names(count) <- c("3_utr","Frequency")
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTMxMTE1Mjk5MiwtMTAyMjA5MzE3MSwxMz
U2MDk5MTY3LDc2Nzg4MjQxNCwtOTI3MTUyMzMzLC0xMDk1MjE4
NDEzLC0xOTcwODkxOTg1LC0xNjg4MDg3MTMxLC0xODY2NTYxMz
csMTg1MDUwNjEyMywtNjYwMDczMTk2LDg2ODgwNjQzNSwyMDc4
MjkwNDgwLC00NDUzNjkzNDksLTE0NjA0NjQ4NTddfQ==
-->