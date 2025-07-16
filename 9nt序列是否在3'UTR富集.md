# 准备文件
## 获取全基因组长度（Rstudio ）
```
##获取染色体长度（公共）
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
## 3'UTR基因组位置文件（Rstudio ）
 ```
##获取3utr 标签mane_select的位置信息（公共）
utr_3_prime_regions <- read.table("D:/新桌面/h38_man_3utr.gtf", header = FALSE,sep = "\t")
library(dplyr)
utr_3_prime_regions <- utr_3_prime_regions %>% 
  rename(chromosome = V1, start = V4, end = V5, strand = V7) %>% 
  select(chromosome, start, end, strand)
save(utr_3_prime_regions,file = "D:/新桌面/zhaolab/生信/元件匹配_12.19/ATTTTGCTT/R数据文件/utr_3_prime_regions.RData")
```

## 3'UTR正义链或反义链基因组位置（Rstudio）
```
##正链
utr_3_prime_regions_c <- utr_3_prime_regions[utr_3_prime_regions$strand == "+",]
save(utr_3_prime_regions_c,file = "utr_3_prime_regions_c.RData")
##反链
utr_3_prime_regions_n <- utr_3_prime_regions[utr_3_prime_regions$strand == "-",]
save(utr_3_prime_regions_n,file = "utr_3_prime_regions_n.RData")
```
## 主要转录本基因组正义链或反义链位置范围（Rstudio）
```
#建立基因组位置信息数据框
h38_man_transcript <- read.table("h38_man_transcript.gtf",header = FALSE, sep="\t")
#改列名，筛选特定列
library(dplyr)
h38_man_transcript <- h38_man_transcript %>%
     rename(chromosome =V1,start =V4,end=V5,strand =V7) %>%
     select(chromosome, start, end, strand)
#筛选正/反义链
h38_man_transcript_n <- h38_man_transcript[h38_man_transcript$strand =="-",]
save(h38_man_transcript_n,file="h38_man_transcript_n.RData")
```
## 随机模拟进行motif在3UTR正链负链上富集分析（Rstudio）
```
##模拟9nt序列的抽取并统计匹配
set.seed(123)  #设置随机种子
n_simulations <- 10000  #模拟次数
sequences_per_simulation <- 36205 #每次模拟抽取的序列数(可变)
sequence_length <- 9  #序列长度是9nt
#### genome_length <- sum(chromosome_lengths)  #基因组的长度 ####
# 预先计算每个染色体的有效长度
effective_lengths <- chromosome_lengths - sequence_length +1
#计算3utr的有效end
library("dplyr")
utr_3_prime_regions_c <- utr_3_prime_regions_c %>% mutate(effective_end = end-sequence_length+1)
utr_3_prime_regions_n <- utr_3_prime_regions_n %>% mutate(effective_end = end-sequence_length+1)

# 记录每次模拟中位于3'UTR的位点数量
utr_sense_count <- numeric(n_simulations)
utr_antisense_count <- numeric(n_simulations)

# 创建一个数据框，保存每个染色体的长度
chromosome_data <- data.frame(
  chromosome = names(chromosome_lengths),
  length = chromosome_lengths)

# 模拟过程
for (i in 1:n_simulations) {
  # 生成随机染色体和随机起始位置、随机正反链
  random_chromosomes <- sample(chromosome_data$chromosome, sequences_per_simulation, replace = TRUE)
  random_positions <- sample(effective_lengths[match(random_chromosomes,chromosome_data$chromosome)],sequences_per_simulation, replace = TRUE)
  random_strand <- sample(c("+","-"),sequences_per_simulation,replace=TRUE)
  # 统计有多少个序列元件位于3'UTR区域
  utr_count <- 0
  utr_anticount <- 0
  for (j in 1:sequences_per_simulation) { 
     chrom <- random_chromosomes[j] 
     pos <- random_positions[j] 
     str <- random_strand[j]
     utr_regions_c <- utr_3_prime_regions_c[utr_3_prime_regions_c$chromosome == chrom,] 
     utr_regions_n <- utr_3_prime_regions_n[utr_3_prime_regions_n$chromosome == chrom,]
     #正义链
     if (any(pos >= utr_regions_c$start & pos <= utr_regions_c$effective_end & str == utr_regions$strand)) {
       utr_count <- utr_count + 1 } 
      #反义链
     if (any(pos >= utr_regions_n$start & pos <= utr_regions_n$effective_end & str == utr_regions$strand)) {
       utr_anticount <- utr_anticount + 1 } }
  # 保存每次模拟的结果
  utr_sense_count[i] <- utr_count
  utr_antisense_count[i] <- utr_anticount
}
save(utr_sense_count,file="utr_sense_count.RData")
save(utr_antisense_count,file="utr_antisense_count.RData")

library(foreach) 
library(doParallel)
# 注册并行后端 
registerDoParallel(cores = detectCores() - 1) 
# 使用foreach循环进行并行模拟 
utr_sense_count <- foreach(i = 1:n_simulations, .combine = c) %dopar% { 
   random_chromosomes <- sample(chromosome_data$chromosome, sequences_per_simulation, replace = TRUE) 
   random_positions <-sample(effective_lengths[match(random_chromosomes,chromosome_data$chromosome)],sequences_per_simulation, replace = TRUE) 
    random_strand <- sample(c("+","-"),sequences_per_simulation,replace=TRUE)
   utr_count <- 0 
   for (j in 1:sequences_per_simulation) { 
        chrom <- random_chromosomes[j] 
        pos <- random_positions[j] 
        str <- random_strand[j]
        utr_regions <- utr_3_prime_regions[utr_3_prime_regions$chromosome == chrom,] 
        if (any(pos > utr_regions$start & pos < utr_regions$effective_end & str == utr_regions$strand)) { 
            utr_count <- utr_count + 1 } } 
   return(utr_count) }


#转换文件内容格式
count <- utr_sense_count %>% 
         sort() %>%
         table()
count <- as.data.frame(count)
names(count) <- c("3_utr","Frequency")
```

## 随机模拟进行motif在主要转录本正链负链上富集分析（Rstudio）
```
##模拟9nt序列的抽取并统计匹配
set.seed(123)  #设置随机种子
n_simulations <- 10000  #模拟次数
sequences_per_simulation <- 36205 #每次模拟抽取的序列数(可变)
sequence_length <- 9  #序列长度是9nt
#### genome_length <- sum(chromosome_lengths)  #基因组的长度 ####
# 预先计算每个染色体的有效长度
effective_lengths <- chromosome_lengths - sequence_length +1
#计算3utr的有效end
library("dplyr")
utr_3_prime_regions_c <- utr_3_prime_regions_c %>% mutate(effective_end = end-sequence_length+1)
utr_3_prime_regions_n <- utr_3_prime_regions_n %>% mutate(effective_end = end-sequence_length+1)

# 记录每次模拟中位于3'UTR的位点数量
utr_sense_count <- numeric(n_simulations)
utr_antisense_count <- numeric(n_simulations)

# 创建一个数据框，保存每个染色体的长度
chromosome_data <- data.frame(
  chromosome = names(chromosome_lengths),
  length = chromosome_lengths)

# 模拟过程
for (i in 1:n_simulations) {
  # 生成随机染色体和随机起始位置、随机正反链
  random_chromosomes <- sample(chromosome_data$chromosome, sequences_per_simulation, replace = TRUE)
  random_positions <- sample(effective_lengths[match(random_chromosomes,chromosome_data$chromosome)],sequences_per_simulation, replace = TRUE)
  random_strand <- sample(c("+","-"),sequences_per_simulation,replace=TRUE)
  # 统计有多少个序列元件位于3'UTR区域
  utr_count <- 0
  utr_anticount <- 0
  for (j in 1:sequences_per_simulation) { 
     chrom <- random_chromosomes[j] 
     pos <- random_positions[j] 
     str <- random_strand[j]
     utr_regions_c <- utr_3_prime_regions_c[utr_3_prime_regions_c$chromosome == chrom,] 
     utr_regions_n <- utr_3_prime_regions_n[utr_3_prime_regions_n$chromosome == chrom,]
     #正义链
     if (any(pos >= utr_regions_c$start & pos <= utr_regions_c$effective_end & str == utr_regions$strand)) {
       utr_count <- utr_count + 1 } 
      #反义链
     if (any(pos >= utr_regions_n$start & pos <= utr_regions_n$effective_end & str == utr_regions$strand)) {
       utr_anticount <- utr_anticount + 1 } }
  # 保存每次模拟的结果
  utr_sense_count[i] <- utr_count
  utr_antisense_count[i] <- utr_anticount
}
save(utr_sense_count,file="utr_sense_count.RData")
save(utr_antisense_count,file="utr_antisense_count.RData")

library(foreach) 
library(doParallel)
# 注册并行后端 
registerDoParallel(cores = detectCores() - 1) 
# 使用foreach循环进行并行模拟 
utr_sense_count <- foreach(i = 1:n_simulations, .combine = c) %dopar% { 
   random_chromosomes <- sample(chromosome_data$chromosome, sequences_per_simulation, replace = TRUE) 
   random_positions <-sample(effective_lengths[match(random_chromosomes,chromosome_data$chromosome)],sequences_per_simulation, replace = TRUE) 
    random_strand <- sample(c("+","-"),sequences_per_simulation,replace=TRUE)
   utr_count <- 0 
   for (j in 1:sequences_per_simulation) { 
        chrom <- random_chromosomes[j] 
        pos <- random_positions[j] 
        str <- random_strand[j]
        utr_regions <- utr_3_prime_regions[utr_3_prime_regions$chromosome == chrom,] 
        if (any(pos > utr_regions$start & pos < utr_regions$effective_end & str == utr_regions$strand)) { 
            utr_count <- utr_count + 1 } } 
   return(utr_count) }


#转换文件内容格式
count <- utr_sense_count %>% 
         sort() %>%
         table()
count <- as.data.frame(count)
names(count) <- c("3_utr","Frequency")
```

## 检验数据是否符合正态分布
##### Anderson-Darling 检验（适用于大样本）
```
##R
install.packages("nortest")
library("nortest")
ad.test(utr_sense_count)
##p-value > 0.05，数据服从正态分布；p-value ≤ 0.05，数据不服从正态分布。
```
#### Kolmogorov-Smirnov (K-S) 检验
```
ks.test(utr_sense_count, "pnorm", mean = mean(utr_sense_count), sd = sd(utr_sense_count))
```
#### Shapiro-Wilk 正态性检验
```
shapiro.test(utr_sense_count)
##如果p-value > 0.05，则数据近似服从正态分布。
如果p-value ≤ 0.05，拒绝原假设，说明数据不服从正态分布。
注意：Shapiro-Wilk 适用于n ≤ 5000的数据集，对于更大数据集，使用 Kolmogorov-Smirnov 或 Anderson-Darling。
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTkxMzY2OTQ4NCwtNzg0NjA0ODcyLC0xMj
AxNzY1MjI2LDEwMTQzODc2NDYsLTkzMTQ4NzUyMiwyMDM3ODkx
NDI1LDUwNTg1OTY5MCwtMTk0MTE4NTQwMiwxNTI4NjkxMDY4LC
0xNTc4MzkyNTY2LC0xOTc0NTAwOTU2LC0xNTExNzEwMjI2LDQ5
NjI3NTk4OSwxMzYzNjIwOTY2LDU3MTUxMTgyMyw1NzY0MTA5OT
IsOTU5MjAxNDg2LDI4MTY4Njg1OCwtNDU0MDkwMTAsLTM0OTU0
MzQ4Nl19
-->