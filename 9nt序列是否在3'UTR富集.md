## 准备文件
##### 3'UTR基因组位置文件
h38.gtf是已经筛选过3'UTR的基因注释文件
``` 
##统计该序列在基因组(man_slect)中出现的次数
grep 'tag "MANE_Select"' Homo_sapiens.GRCh38.113.gtf > h38_man.gtf
bedtools intersect -a h38_man.gtf -b locate_1.bed -wa -wb > sum
wc -l sum #39669次
grep 'tag "MANE_Select"' h38.gtf > h38_man_3utr.gtf

##获取染色体长度
library(Biostrings) 
#读取FASTA文件中的基因组序列
genome <- readDNAStringSet("genome.fasta") 
#筛选出主要的常染色体和性染色体
main_chromosomes <- grep("^(\\d+|X|Y)", names(genome), value = TRUE)
genome_main <- genome[main_chromosomes]
#获取每个染色体的长度 
chromosome_lengths <- width(genome_main) 
names(chromosome_lengths) <- c("chr1","chr10","chr11","chr12","chr13","chr14","chr15","chr16","chr17","chr18","chr19",
"chr2","chr20","chr21","chr22","chr3","chr4","chr5","chr6","chr7","chr8","chr9","chrX","chrY")
#查看染色体信息（例如，chromosome名和长度）
chromosome_lengths
save(chromosome_lengths,file = "D:/新桌面/zhaolab/生信/元件匹配_12.19/ATTTTGCTT/R数据文件/chromosome_lengths.RData")

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
eyJoaXN0b3J5IjpbLTE2ODgwODcxMzEsLTE4NjY1NjEzNywxOD
UwNTA2MTIzLC02NjAwNzMxOTYsODY4ODA2NDM1LDIwNzgyOTA0
ODAsLTQ0NTM2OTM0OSwtMTQ2MDQ2NDg1N119
-->