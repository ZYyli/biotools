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
### 合并两个bed文件
(排序bedtools sort -i)
```
cat locate_11.bed locate_22.bed > locate.bed
```
#### 交集——序列在3’UTR中的区域位置及数量
[【bioinfo】bedtools之intersect命令参数_bedtools intersect-CSDN博客](https://blog.csdn.net/sinat_32872729/article/details/126541494)
###### 可以对两个基因组特征进行overlap，找到两者重合的区域。比如求两个peaks的交集，或者看很多位点信息在没在peaks或其他区域中
```
bedtools intersect -a h38.gtf -b locate.bed -wa -wb   > merge_1
##所处链相同说明该原件存在mRNA上
awk -F'\t' -vOFS='\t' '$7==$15' merge_1 > merge_3
##删除多余列和值
awk -F'\t' -vOFS='\t' '{$2=$6=$8=$14="";print}' merge_3 > merge_4
awk -F'\t' -vOFS='\t' '{
    # 初始化一个新的第九列字符串
    new_col9 = "";
    # 分割第九列的键值对
    n = split($9, attributes, ";");
    for (i = 1; i <= n; i++) {
        # 去除前后空格
        gsub(/^[ \t]+|[ \t]+$/, "", attributes[i]);
        # 检查键值对是否以特定字符串开头
        if (attributes[i] ~ /^gene_id/ || attributes[i] ~ /^transcript_id/ || attributes[i] ~ /^gene_name/ || attributes[i] ~ /^transcript_name/) {
            # 如果是，则将其添加到新的第九列字符串中
            new_col9 = new_col9 (new_col9 ? "; " : "") attributes[i];
        }
    }
    # 替换原来的第九列
    $9 = new_col9;
    print;
}' merge_4 > merge_5
```
-wa -wb ：输出overlap的区域所在-a和-b中的原内容
###### 分别获得正负链的原件出现次数
```
#正链
awk -F'\t' -vOFS='\t' '$7=="+"' h38.gtf > h38_1.gtf
bedtools intersect -a h38_1.gtf -b locate_11.bed -c > merge_21
##筛选出现次数大于0的（经过试验最多3次）
awk -F'\t' -vOFS='\t' '$10 > 0  {print}' merge_21 > merge_211
#负链
awk -F '\t' -vOFS='\t' '$7=="-"' h38.gtf > h38_2.gtf
bedtools intersect -a h38_2.gtf -b locate_22.bed -c > merge_22
##筛选出现次数大于0的（经过试验最多3次）
awk -F'\t' -vOFS='\t' '$10 > 0  {print}' merge_22 > merge_222
```
-c：包含着染色体位置的两个文件，分别记为A文件和B文件。对于A文件中染色体位置，输出在A文件中染色体位置中有多少B文件染色体位置与之有overlap。
###### 删除多余列和值
```
##正负链操作一致
awk -F'\t' -vOFS='\t' '{$2=$6=$8=$14="";print}' merge_211 > merge_c
awk -F'\t' -vOFS='\t' '{
    # 初始化一个新的第九列字符串
    new_col9 = "";
    # 分割第九列的键值对
    n = split($9, attributes, ";");
    for (i = 1; i <= n; i++) {
        # 去除前后空格
        gsub(/^[ \t]+|[ \t]+$/, "", attributes[i]);
        # 检查键值对是否以特定字符串开头
        if (attributes[i] ~ /^gene_id/ || attributes[i] ~ /^transcript_id/ || attributes[i] ~ /^gene_name/ || attributes[i] ~ /^transcript_name/) {
            # 如果是，则将其添加到新的第九列字符串中
            new_col9 = new_col9 (new_col9 ? "; " : "") attributes[i];
        }
    }
    # 替换原来的第九列
    $9 = new_col9;
    print;
}' merge_c > merge_cc
```
##### 转化文件格式
```
安装csvkit和xlsxwriter
pip3 install csvkit
pip3 install xlsxwriter
csv2xlsx merge_5 .xlsx
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbMjczODQyNDAxLC02NTUzMDU3NDQsLTI3OT
gwMTQsLTg5NjM2NTYwOSw5MTI4OTE3NjQsLTE2OTk5ODI3ODcs
MTI2MTkzMjQyOSw1NjAzMzIwNjksLTExMjUyMjM1NzUsMjA4Nz
EyNjQ0NSwxNDc0NzU4OTU3LC0xNzc3MjA1NzU2LDIwMDczNTMx
NiwtMTYzMDg2OTIyOCw3OTY3OTMxODIsLTM4NjQ5ODQ5MCwtMj
E4NTM0NTA0LC0xODEzOTg5OTkyLC0xNTI5MDYzMjkwLC0xMjg0
OTM5Mzg0XX0=
-->