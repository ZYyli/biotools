### 删除单列
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
###### 可以对两个基因组特征进行overlap，找到两者重合的区域。比如求两个peaks的交集，或者看很多位点信息在没在peaks或其他区域中
```
bedtools intersect -a h38.gtf -b locate.bed -wa -wb   > merge_1
##所处链相同说明该原件存在mRNA上
awk -F '\t' '$7==$15' merge_1 > merge_3
```
-wa -wb ：输出overlap的区域所在-a和-b中的原内容
###### 分别获得正负链的原件出现次数
```
awk -F '\t' '$7=="+"' h38.gtf > h38_1.gtf
awk -F '\t' '$7=="-"' h38.gtf > h38_2.gtf
bedtools intersect -a h38_1.gtf -b locate_11.bed -c > merge_21
bedtools intersect -a h38_2.gtf -b locate_22.bed -c > merge_22
```
-c：包含着染色体位置的两个文件，分别记为A文件和B文件。对于A文件中染色体位置，输出在A文件中染色体位置中有多少B文件染色体位置与之有overlap.
```ruby
$ cat A.bed 
chr1 10 20 
chr1 30 40 
$ cat B.bed
chr1 15 20
chr1 18 25
$ bedtools intersect -a A.bed -b B.bed -c
chr1 10 20 2
chr1 30 40 0
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTUyOTQ1ODExOSwtMTcwMTEyMzYxNywtMT
MzNzA1MTIwMSwxOTQ2OTA5ODY4LDk1OTc0MTM0OCwxNjYyMzc3
OTM0LC0xNTA5MjYzMTgzLDE0Mzc3MzYyODcsLTE4MzgwOTI1MD
gsMTMwMTgzMDQ2OSwtMTk0NTg4ODE5NywzODg2NDk0MzksNDY3
MzUwOTk3LDEzNTcwODg3MzUsMTg1NjgwNTU1NCwxMjQyMTczND
Q2LDExMTA1MTQ5ODgsLTExMjAyOTA3MzEsLTk1MTU0MTE5NV19

-->