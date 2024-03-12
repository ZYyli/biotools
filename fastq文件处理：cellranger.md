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
### 建立参考基因组
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
`cellranger mkgtf GCF_000001405.40_GRCh38.p14_genomic.gtf GCF_000001405.40_GRCh38.p14_genomic.filtered.gtf
              --attribute=gene_biotype:protein_coding
              --attribute=gene_biotype:lincRNA
              --attribute=gene_biotype:antisense
              --attribute=gene_biotype:IG_LV_gene
              --attribute=gene_biotype:IG_V_gene
              --attribute=gene_biotype:IG_V_pseudogene
              --attribute=gene_biotype:IG_D_gene
              --attribute=gene_biotype:IG_J_gene
             --attribute=gene_biotype:IG_J_pseudogene
            --attribute=gene_biotype:IG_C_gene
             --attribute=gene_biotype:IG_C_pseudogene
              --attribute=gene_biotype:TR_V_gene
               --attribute=gene_biotype:TR_V_pseudogene
                --attribute=gene_biotype:TR_D_gene
                 --attribute=gene_biotype:TR_J_gene
              --attribute=gene_biotype:TR_J_pseudogene
              --attribute=gene_biotype:TR_C_gene`
` #这样得到的Homo_sapiens.GRCh38.ensembl.filtered.gtf结果中就不包含gene_biotype:pseudogene这部分`
#### cellranger mkref命令构建
--genome：生成索引的目录
--fasta：基因组序列
--genes：基因注释文件（gtf格式）
`cellranger mkref 
--genome=./ref/
--nthreads=12
--fasta=GCF_000001405.40_GRCh38.p14_genomic.fna
--genes=GCF_000001405.40_GRCh38.p14_genomic.gtf`
### cellranger count定量
#### fastq文件改名
`1.  cat SRR.txt | while read i ;do (mv ${i}_1*.gz ${i}_S1_L001_I1_001.fastq.gz;mv ${i}_2*.gz ${i}_S1_L001_R1_001.fastq.gz;mv ${i}_3*.gz ${i}_S1_L001_R2_001.fastq.gz);done`
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0ODgwMTU4MTcsLTUwMTM3NTU2NywtOD
g0OTcwNjU1LDE3MTc4MzI4NTEsLTMxNzQxNzI5MSwtMTQ1MTEw
NTYxMywtMTc1NTQwNzI0MCwtMTc3OTc5NDIzLC0xNjM4NDI3OT
cwLC05OTcwNjQ0NTBdfQ==
-->