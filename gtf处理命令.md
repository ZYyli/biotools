### 提出单列
```
awk -F'\t' '{print $2}' xx.gtf > xx.gtf
```
### 删除多列
```
awk -F'\t' '{$2=$6=$8="";print}' xx.gtf > xx.gtf
``` 
-F ：分隔符为制表符
'{$2=$6=$8="";print}' ：删除第2、6、8列
###  筛选3‘UTR的基因注释信息
```
 grep 'three_prime_utr' Homo_sapiens.GRCh38.113.gtf > h38.gtf
```
### 根据某段序列，从基因组fa文件筛选对应的位置
```
seqkit locate --bed -p ATTTTGCTT Homo_sapiens.GRCh38.dna.toplevel.fa > locate_1.bed
seqkit locate --bed -p AAGCAAAAT Homo_sapiens.GRCh38.dna.toplevel.fa > locate_2.bed
```
 -d, --degenerate 包含简并碱基模式和motif
  --gtf 输出为GTF格式
  -i, --ignore-case 忽视字母大小写
  -m, --max-mismatch int 通过序列匹配时允许的最大错配
  -G, --non-greedy 非贪婪模式，更快但是可能错过与其他重叠的motif
  -P, --only-positive-strand 只搜索正链
  -f, --pattern-file 模式或motif文件（fasta格式）
  -p, --pattern strings 搜索pattern或motif
### 筛选+链ATTTTGCTT ，-链AAGCAAAAT
```
grep '+' locate_1.bed > locate_11.bed
grep '-' locate_2.bed > locate_11.bed
```
### 合并两个bed文件并排序
先按照染色体进行升序排列，然后按照起始位置再进行升序排列
```
cat locate_11.bed locate_22.bed | bedtools sort -i > locate.bed
```
#### 交集——序列在3’UTR中的区域位置及数量
[【bioinfo】bedtools之intersect命令参数_bedtools intersect-CSDN博客](https://blog.csdn.net/sinat_32872729/article/details/126541494)
###### 可以对两个基因组特征进行overlap，找到两者重合的区域。比如求两个peaks的交集，或者看很多位点信息在没在peaks或其他区域中
```
bedtools intersect -a h38.gtf -b locate.bed -wa -wb   > merge_1
##所处链相同说明该原件存在mRNA上
awk -F '\t' '$7==$15' merge_1 > merge_3
##删除多余列和值
awk -F'\t' '{$2=$6=$8=$14="";print}' merge_3 > merge_4
awk -F'\t' -vOFS='\t' '{
    split($6, attributes, ";");attributes = "";
    for (i = 1; i <= length(attributes); i++) {
         if (a[i] ~ /^[ \t]+gene_id/ && a[i] ~ /^[ \t]+transcript_id/ && a[i] ~ /^[ \t]+gene_name/ && a[i] ~ /^[ \t]+transcript_name/) {
            new_attributes = new_attributes (i > 1 ? "; " : "") attributes[i];
         } 
    } $6 = new_attributes ; print
}' merge_4 > merge_5

awk -F'\t' -vOFS='\t' '{
    split($6, a, ";"); has_gene_id = 0; has_transcript_id = 0; has_gene_name = 0; has_transcript_name = 0;
    for (i in a) { 
         if (a[i] ~ /^[ \t]+gene_id/) {
            has_gene_id = 1;
         } else if (a[i] ~ /^[ \t]+transcript_id/) {
                    has_transcript_id = 1;
         } else if (a[i] ~ /^[ \t]+gene_name/) {
                    has_gene_name = 1;
         } else if (a[i] ~ /^[ \t]+transcript_name/) {
                    has_transcript_name = 1;
         }
    } 
    if (has_gene_id && has_transcript_id && has_gene_name && has_transcript_name) { print;
    }
}' merge_4 > merge_5
 
```
-wa -wb ：输出overlap的区域所在-a和-b中的原内容
###### 分别获得正负链的原件出现次数
```
#正链
awk -F '\t' '$7=="+"' h38.gtf > h38_1.gtf
bedtools intersect -a h38_1.gtf -b locate_11.bed -c > merge_21
##筛选出现次数大于0的（经过试验最多3次）
awk -F'\t' '$10 > 0  {print}' merge_21 > merge_211
#负链
awk -F '\t' '$7=="-"' h38.gtf > h38_2.gtf
bedtools intersect -a h38_2.gtf -b locate_22.bed -c > merge_22
##筛选出现次数大于0的（经过试验最多3次）
awk -F'\t' '$10 > 0  {print}' merge_22 > merge_222
```
-c：包含着染色体位置的两个文件，分别记为A文件和B文件。对于A文件中染色体位置，输出在A文件中染色体位置中有多少B文件染色体位置与之有overlap。
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTI2MTkzMjQyOSw1NjAzMzIwNjksLTExMj
UyMjM1NzUsMjA4NzEyNjQ0NSwxNDc0NzU4OTU3LC0xNzc3MjA1
NzU2LDIwMDczNTMxNiwtMTYzMDg2OTIyOCw3OTY3OTMxODIsLT
M4NjQ5ODQ5MCwtMjE4NTM0NTA0LC0xODEzOTg5OTkyLC0xNTI5
MDYzMjkwLC0xMjg0OTM5Mzg0LC0xNzQzMDU5MDI3LDE1Mjk0NT
gxMTksLTE3MDExMjM2MTcsLTEzMzcwNTEyMDEsMTk0NjkwOTg2
OCw5NTk3NDEzNDhdfQ==
-->