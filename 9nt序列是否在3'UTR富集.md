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
eyJoaXN0b3J5IjpbOTY3MDc4MTIxLDg2ODgwNjQzNSwyMDc4Mj
kwNDgwLC00NDUzNjkzNDksLTE0NjA0NjQ4NTddfQ==
-->