## 安装
[Download Cell Ranger - Official 10x Genomics Support](https://www.10xgenomics.com/support/software/cell-ranger/downloads)
![输入图片说明](https://raw.githubusercontent.com/ZYyli/bioinfosoft_pictures/master/imgs/2024-03-12/XAfV8AXbIBQzeil0.jpeg)
下载完后查看md5sum值，看下载是否完整。
`
#解压
tar -zxvf cellranger-7.2.0.tar.gz`
`#修改环境变量
echo 'export PATH=/cellranger-7.2.0/:$PATH' >> ~/.bashrc`
`#更新配置文件
source ~/.bashrc
cellranger count --help
`
## 使用
### 建立参考基因组(经过试验，后决定还是使用官方给定参考基因版本)
10x官方有人和小鼠的参考基因组，但我这里采用最新人类参考基因组
![输入图片说明](https://raw.githubusercontent.com/ZYyli/bioinfosoft_pictures/master/imgs/2024-03-12/iV1bwhDfgivcnMFL.png)
下载参考基因组(fasta)和注释基因(GTF)
`
wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/405/GCF_000001405.40_GRCh38.p14/GCF_000001405.40_GRCh38.p14_genomic.fna.gz
wget
https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/001/405/GCF_000001405.40_GRCh38.p14/GCF_000001405.40_GRCh38.p14_genomic.gtf.gz
`
#### Cellranger mkgtf 对GTF文件进行过滤
10x单细胞使用的polydT进行RNA逆转录，只能测到带有polyA尾的RNA序列，所以我们需要从GTF文件中过滤掉non-polyA的基因。Cellranger的mkgtf命令可以对GTF文件进行过滤，--attribute=gene_biotype:protein_coding
`cellranger mkgtf GCF_000001405.40_GRCh38.p14_genomic.gtf GCF_000001405.40_GRCh38.p14_genomic.filtered.gtf --attribute=gene_biotype:protein_coding
--attribute=gene_biotype:lincRNA --attribute=gene_biotype:antisense  --attribute=gene_biotype:IG_LV_gene --attribute=gene_biotype:IG_V_gene --attribute=gene_biotype:IG_V_pseudogene  --attribute=gene_biotype:IG_D_gene --attribute=gene_biotype:IG_J_gene --attribute=gene_biotype:IG_J_pseudogene --attribute=gene_biotype:IG_C_gene  --attribute=gene_biotype:IG_C_pseudogene --attribute=gene_biotype:TR_V_gene --attribute=gene_biotype:TR_V_pseudogene --attribute=gene_biotype:TR_D_gene --attribute=gene_biotype:TR_J_gene --attribute=gene_biotype:TR_J_pseudogene --attribute=gene_biotype:TR_C_gene`
` #这样得到的Homo_sapiens.GRCh38.ensembl.filtered.gtf结果中就不包含gene_biotype:pseudogene这部分`
#### cellranger mkref命令构建
--genome：生成索引的目录
--fasta：基因组序列
--genes：基因注释文件（gtf格式）
`cellranger mkref 
--genome=hg
--nthreads=12
--fasta=GCF_000001405.40_GRCh38.p14_genomic.fna
--genes=GCF_000001405.40_GRCh38.p14_genomic.gtf`
10X官方参考基因组
[Download Space Ranger - Official 10x Genomics Support](https://www.10xgenomics.com/support/software/space-ranger/downloads#reference-downloads)
## cellranger count定量
#### fastq文件改名
`cat SRR.txt | while read i ;do (mv ${i}_1*.gz ${i}_S1_L001_R1_001.fastq.gz;mv ${i}_2*.gz ${i}_S1_L001_R2_001.fastq.gz);done`
#### count函数解释
`
cellranger count 
--id=$i 
--transcriptome=refdata-cellranger-GRCh38-1.2.0 
--fastqs=/home/scRNA/runs/HAWT7ADXX/outs/fastq_path
--sample=mysample
--expect-cells=1000
--nosecondary
#id指定输出文件存放目录名
#transcriptome指定与CellRanger兼容的参考基因组
#fastqs指定mkfastq或者自定义的测序文件
#sample要和fastq文件的前缀中的sample保持一致，作为软件识别的标志
#expect-cells指定复现的细胞数量，这个要和实验设计结合起来
#nosecondary 只获得表达矩阵，不进行后续的降维、聚类和可视化分析(反正后续要走Seurat，为了节省计算资源，建议加上)
`
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTc0NDc0MTAwNiw2MzE3MzcyMzIsLTMzOT
Q5OTYwMiwtODgyMDM5NjgxLC03MDM5NzI0MDMsMTUzNjAyNDI3
MSwtMTE2NDA0NDIyMywtNTAxMzc1NTY3LC04ODQ5NzA2NTUsMT
cxNzgzMjg1MSwtMzE3NDE3MjkxLC0xNDUxMTA1NjEzLC0xNzU1
NDA3MjQwLC0xNzc5Nzk0MjMsLTE2Mzg0Mjc5NzAsLTk5NzA2ND
Q1MF19
-->