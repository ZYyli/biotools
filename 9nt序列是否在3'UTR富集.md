## 准备文件
##### 全基因组长度并统计次数
h38.gtf是已经筛选过3'UTR的基因注释文件
``` 
##统计该序列在基因组(man_slect)中出现的次数
grep 'tag "MANE_Select"' Homo_sapiens.GRCh38.113.gtf > h38_man.gtf
bedtools intersect -a h38_man.gtf -b locate_1.bed -wa -wb > sum
wc -l sum #39669次
grep 'tag "MANE_Select"' h38.gtf > h38_man_3utr.gtf
```

Rstudio
```
##获取染色体长度
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


utr_3_prime_regions <- read.table("D:/新桌面/h38_man_3utr.gtf", header = FALSE,sep = "\t")
library(dplyr)
utr_3_prime_regions <- utr_3_prime_regions %>% 
  rename(chromosome = V1, start = V4, end = V5, strand = V7)
utr_3_prime_regions <- utr_3_prime_regions %>% select(chromosome, start, end, strand)
save(utr_3_prime_regions,file = "D:/新桌面/zhaolab/生信/元件匹配_12.19/ATTTTGCTT/R数据文件/utr_3_prime_regions.RData")
```

##### 3'UTR基因组位置文件
h38.gtf是已经筛选过3'UTR的基因注释文件
```
###Rstudio
##读取文件
utr_3_prime_regions <- read.table("D:/新桌面/h38_man_3utr.gtf", header = FALSE,sep = "\t")
library(dplyr)
##修改列名并删除列名
utr_3_prime_regions <- utr_3_prime_regions %>% 
  rename(chromosome = V1, start = V4, end = V5, strand = V7) %>% 
  select(chromosome, start, end, strand)
save(utr_3_prime_regions,file = "D:/新桌面/zhaolab/生信/元件匹配_12.19/ATTTTGCTT/R数据文件/utr_3_prime_regions.RData")
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI3MDU3NjYzMCw3Njc4ODI0MTQsLTkyNz
E1MjMzMywtMTA5NTIxODQxMywtMTk3MDg5MTk4NSwtMTY4ODA4
NzEzMSwtMTg2NjU2MTM3LDE4NTA1MDYxMjMsLTY2MDA3MzE5Ni
w4Njg4MDY0MzUsMjA3ODI5MDQ4MCwtNDQ1MzY5MzQ5LC0xNDYw
NDY0ODU3XX0=
-->