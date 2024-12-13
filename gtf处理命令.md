## 删除列
#### 删除单列
```
awk -F'\t' '{print $2}' xx.gtf > xx.gtf
```
#### 删除多列
```
awk -F'\t' '{$2=$6=$8="";print}' xx.gtf > xx.gtf
``` 
-F ：分隔符为制表符
'{$2=$6=$8="";print}' ：删除第2、6、8列
####  筛选3‘UTR的基因注释信息
```
 grep 'three_prime_utr' n1.gtf > n2.gtf
```
#### 根据某段序列，从基因组fa文件筛选对应的位置
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
#### 筛选+链ATTTTGCTT ，-链AAGCAAAAT
```
grep '+' locate_1.bed > locate_11.bed
grep '-' locate_2.bed > locate_11.bed
```
#### 合并两个bed文件并排序
###### 先按照染色体进行升序排列，然后按照起始位置再进行升序排列
```
cat locate_11.bed locate_11.bed | bedtools sort -i > locate.bed
```
#### 
```
 bedtools intersect -a n2.gtf -b locate_11.bed -wa -wb > merge
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5OTExMDc5MDUsMTMwMTgzMDQ2OSwtMT
k0NTg4ODE5NywzODg2NDk0MzksNDY3MzUwOTk3LDEzNTcwODg3
MzUsMTg1NjgwNTU1NCwxMjQyMTczNDQ2LDExMTA1MTQ5ODgsLT
ExMjAyOTA3MzEsLTk1MTU0MTE5NV19
-->