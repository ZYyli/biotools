### 选择基因组和对应gtf文件
[RNAseq分析如何选择 参考基因组 和 gtf - 简书](https://www.jianshu.com/p/b6dd73e4264c)
选择primary_assemble
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

###  筛选编码基因transcript的注释信息
```
awk -F'\t' '$3 == "transcript" {print}' Homo_sapiens.GRCh38.113.gtf | grep 'transcript_biotype "protein_coding"' > protein_coding_transcript.gtf
grep 'tag "MANE_Select"' protein_coding_transcript.gtf > protein_coding_mane_transcript.gtf
```
###  筛选3‘UTR的基因注释信息
```
awk -F'\t' '$3 == "three_prime_utr" {print}' Homo_sapiens.GRCh38.113.gtf > h38_3utr.gtf
```
###  筛选5‘UTR的基因注释信息
```
awk -F'\t' '$3 == "five_prime_utr" {print}' Homo_sapiens.GRCh38.113.gtf > h38_5utr.gtf
```
###  筛选CDS的基因注释信息
```
awk -F'\t' '$3 == "CDS" {print}' Homo_sapiens.GRCh38.113.gtf > h38_cds.gtf
```
### 根据某段序列，从基因组fa文件筛选对应的位置
[seqkit的使用说明 - 简书](https://www.jianshu.com/p/f28bdc9a3b54)
```
seqkit locate --bed -p ATTATGCTT Homo_sapiens.GRCh38.dna.primary_assembly.fa > locate_1.bed
```
 -d, --degenerate 包含简并碱基模式和motif
  --gtf 输出为GTF格式
  -i, --ignore-case 忽视字母大小写
  -m, --max-mismatch int 通过序列匹配时允许的最大错配
  -G, --non-greedy 非贪婪模式，更快但是可能错过与其他重叠的motif
  -P, --only-positive-strand 只搜索正链
  -f, --pattern-file 模式或motif文件（fasta格式）
  -p, --pattern strings 搜索pattern或motif
### 筛选ATTTTGCTT +链 ，-链
```
grep '+' locate_1.bed > locate_c.bed
grep '-' locate_1.bed > locate_n.bed
```
### 确定序列元件在编码基因主要transcript出现次数
```
bedtools intersect -a protein_coding_transcript.gtf -b locate_1.bed -wa -wb > trans_count_1
##正负链统一 
awk -F'\t' -vOFS='\t' '$7==$15 {print}' trans_count_1 > trans_count_2

bedtools intersect -a protein_coding_mane_transcript.gtf -b locate_1.bed -wa -wb > mane_trans_count_1
##正负链统一 
awk -F'\t' -vOFS='\t' '$7==$15 {print}' mane_trans_count_1 > mane_trans_count_2
#
```
### 确定元件在内含子中出现的次数
```
##获取intrond位置信息
awk '$3 == "exon"' Homo_sapiens.GRCh38.113.gtf | grep 'transcript_biotype "protein_coding"' | grep 'tag "MANE_Select"' > mane_exons.gtf
gtftools introns mane_exons.gtf > mane_introns.bed
##元件在intron中出现次数
bedtools intersect -a mane_introns.bed -b locate_1.bed -wa -wb > mane_intron_count_1
##正负链统一 
awk -F'\t' -vOFS='\t' '$7==$15 {print}' mane_intron_count_1 > mane_intron_count_2
```
### 交集——序列在3’UTR中的区域位置及数量
[【bioinfo】bedtools之intersect命令参数_bedtools intersect-CSDN博客](https://blog.csdn.net/sinat_32872729/article/details/126541494)
###### 可以对两个基因组特征进行overlap，找到两者重合的区域。比如求两个peaks的交集，或者看很多位点信息在没在peaks或其他区域中
```
(python3.8环境)
#序列在3'UTR的位置
bedtools intersect -a h38.gtf -b locate_1.bed -wa -wb   > merge_1
##正负链统一
awk -F'\t' -vOFS='\t' '$7==$15 {print}' merge_1 > merge_2
##删除多余列和值
awk -F'\t' -vOFS='\t' '{$2=$6=$8=$14="";print}' merge_2 > merge_3
awk -F'\t' -vOFS='\t' '{
    # 初始化一个新的第九列字符串
    new_col9 = "";
    # 分割第九列的键值对
    n = split($9, attributes, ";");
    for (i = 1; i <= n; i++) {
        # 去除前后空格
        gsub(/^[ \t]+|[ \t]+$/, "", attributes[i]);
        # 检查键值对是否以特定字符串开头
        if (attributes[i] !~ /^gene_version/ && attributes[i] !~ /^transcript_version/ && attributes[i] !~ /^gene_source/ && attributes[i] !~ /^gene_biotype/ && attributes[i] !~ /^transcript_source/ && attributes[i] !~ /^transcript_biotype/ && attributes[i] !~ /^transcript_support_level/) {
            # 如果是，则将其添加到新的第九列字符串中
            new_col9 = new_col9 (new_col9 ? "; " : "") attributes[i];
        }
    }
    # 替换原来的第九列
    $9 = new_col9;
    print;
}' merge_3 > merge_4
## 筛选出tag "MANE_Select"的行
grep 'tag "MANE_Select"' merge_4 > merge_man_select
```
-wa -wb ：输出overlap的区域所在-a和-b中的原内容
###### 分别获得正负链的原件出现次数(制表)
```
#正链
awk -F'\t' -vOFS='\t' '$7=="+"' h38.gtf > h38_c.gtf
bedtools intersect -a h38_c.gtf -b locate_c.bed -c > count_c
##筛选出现次数大于0的（经过试验最多3次）
awk -F'\t' -vOFS='\t' '$10 > 0  {print}' count_c > count_c1
#负链
awk -F '\t' -vOFS='\t' '$7=="-"' h38.gtf > h38_n.gtf
bedtools intersect -a h38_n.gtf -b locate_n.bed -c > count_n
##筛选出现次数大于0的（经过试验最多3次）
awk -F'\t' -vOFS='\t' '$10 > 0  {print}' count_n > count_n1
```
-c：包含着染色体位置的两个文件，分别记为A文件和B文件。对于A文件中染色体位置，输出在A文件中染色体位置中有多少B文件染色体位置与之有overlap。
###### 删除多余列和值
```
##正负链操作一致
awk -F'\t' -vOFS='\t' '{$2=$6=$8="";print}' count_c1 > count_c2
awk -F'\t' -vOFS='\t' '{
    # 初始化一个新的第九列字符串
    new_col9 = "";
    # 分割第九列的键值对
    n = split($9, attributes, ";");
    for (i = 1; i <= n; i++) {
        # 去除前后空格
        gsub(/^[ \t]+|[ \t]+$/, "", attributes[i]);
        # 检查键值对是否以特定字符串开头
        if (attributes[i] !~ /^gene_version/ && attributes[i] !~ /^transcript_version/ && attributes[i] !~ /^gene_source/ && attributes[i] !~ /^gene_biotype/ && attributes[i] !~ /^transcript_source/ && attributes[i] !~ /^transcript_biotype/ && attributes[i] !~ /^transcript_support_level/) {
            # 如果是，则将其添加到新的第九列字符串中
            new_col9 = new_col9 (new_col9 ? "; " : "") attributes[i];
        }
    }
    # 替换原来的第九列
    $9 = new_col9;
    print;
}' count_c2 > count_c3
## 筛选出tag "MANE_Select"的行
grep 'tag "MANE_Select"' count_c3 > count_cman_select
```
##### 转化文件格式（Rstudio)
```
install.packages("openxlsx")
library("openxlsx")
merge_ms <- read.table("D:/新桌面/merge_man_select", header = FALSE, sep = "\t")
merge_ms <- merge_ms[-c(2,6,8,14)]
write.xlsx(merge_ms, "D:/新桌面/merge.xlsx")
merge <- read.table("D:/新桌面/merge_4", header = FALSE, sep = "\t")
merge <- merge[-c(2,6,8,14)]
### 读取已有的xlsx文件
merge_file <- "D:/新桌面/merge.xlsx"
### 假设你有一个名为my_table的data.frame，需要插入到Sheet2
wb <- loadWorkbook(merge_file)
# 将my_table写入到Sheet2，如果Sheet2已存在内容，这将覆盖原有内容
writeData(wb, sheet = "Sheet2", x = merge, rowNames = FALSE)
# 保存工作簿
saveWorkbook(wb, file = merge_file, overwrite = TRUE)
```
## seqkit序列处理
[seqkit序列处理神器的常用命令 - 组学大讲堂问答社区](https://www.omicsclass.com/article/1903)
[seqkit：序列梳理神器-统计、格式转换、长度筛选、质量值转换、翻译、反向互补、抽样、去重、滑窗、拆分等30项全能...-CSDN博客](https://blog.csdn.net/woodcorpse/article/details/114827537)
[使用awk随机截取细菌DNA基因组指定长度片段_微生物单菌基因组contig上截取特定基因片段如何操作-CSDN博客](https://blog.csdn.net/weixin_44022515/article/details/102889358)
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE2NjM3MTQwLDExOTI4MDg3MjMsMjA0Nj
QzMTEwNCw5NTk5Mzc3NDUsLTQ2MTE4MjMwMywtMTIxNDQxMzMw
MSwyMDYxOTY2NDE2LC0yNTQ3NDQyMzYsMjg2Nzk5NTE4LC0xOT
M0MzA2ODgyLC02ODMxMjkwNDgsMTIzOTc4MDE0NywxNDA2OTM5
MTcwLC0xMzE2Mzg3OTQ1LC0yNjQxMDA2NzAsODE0Mjk2ODE0LD
IxMjg5NDYxMDAsLTE5NDk2NzA3ODMsMTEzNjI5NTEwMywtMTEy
MDUyNDA2Nl19
-->