# 准备文件
## 全基因组长度并统计次数
h38.gtf是已经筛选过3'UTR的基因注释文件
``` 
##统计该序列在基因组(man_slect、transcript)中出现的次数
grep 'tag "MANE_Select"' Homo_sapiens.GRCh38.113.gtf > h38_man.gtf （公共）
awk -F'\t' '$3 == "transcript" {print}' h38_man.gtf > h38_man_transcript.gtf （公共）
bedtools intersect -a h38_man_transcript.gtf -b locate_1.bed -wa -wb > sum
#所处链相同说明该原件存在mRNA上
awk -F'\t' -vOFS='\t' '$7==$15' sum > sum_trans
wc -l sum_trans #21034次
#改名
mv sum_trans sum_trans_21034

##筛选3utr mane_select标签的基因gtf文件（公共）
grep 'tag "MANE_Select"' h38.gtf > h38_man_3utr.gtf
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
##反义链
utr_3_prime_regions_n <- utr_3_prime_regions[utr_3_prime_regions$strand == "-",]
save(utr_3_prime_regions,file = "utr_3_prime_regions_n.RData")
```
## 基因组正义链或反义链位置范围（Rstudio）
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
## 随机模拟进行富集分析（Rstudio）
```
##模拟9nt序列的抽取并统计匹配
set.seed(123)  #设置随机种子
n_simulations <- 10000  #模拟次数
sequences_per_simulation <- 21034 #每次模拟抽取的序列数(可变)
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
  # 生成随机染色体
  random_chromosomes <- sample(chromosome_data$chromosome, sequences_per_simulation, replace = TRUE)
  # 计算每个染色体的长度并生成随机起始位置
  chrom_lengths <- chromosome_data$length[match(random_chromosomes, chromosome_data$chromosome)]
  effective_lengths <- chrom_lengths - sequence_length +1 
  random_positions <- sapply(effective_lengths,function(len) sample(1:len,1))
  # 统计有多少个9nt序列位于任意3' UTR区域
  utr_count <- sum(mapply( function(chrom, pos) is_in_utr(pos, utr_regions = utr_3_prime_regions[utr_3_prime_regions$chromosome == chrom,]), random_chromosomes, random_positions ))
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
eyJoaXN0b3J5IjpbMTY1MTY5NzgyNiwzNzY2ODEyMjIsNzExNT
E4ODAxLDE5MTk0Mjg0MiwxMDUxNjYyMTMwLC0yNzg1NDE2OTAs
LTEyMTU4MDc3NzgsLTIxMTUyNzk0NDYsLTg2ODMzMzQ2MywxNj
E1NzQwMzU0LDE0MTcyMTg5OTUsMTgwNjY4NTUzMSwzNjczMTUy
NDcsLTEwMjIwOTMxNzEsMTM1NjA5OTE2Nyw3Njc4ODI0MTQsLT
kyNzE1MjMzMywtMTA5NTIxODQxMywtMTk3MDg5MTk4NSwtMTY4
ODA4NzEzMV19
-->