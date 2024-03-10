<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>prefetch 批量下载</title>
  <link rel="stylesheet" href="https://stackedit.cn/style.css" />
</head>

<body class="stackedit">
  <div class="stackedit__html"><h1 id="prefetch批量下载sra数据并转化为fastq.gz的脚本"><span class="prefix"></span><span class="content">prefetch批量下载sra数据并转化为fastq.gz的脚本</span><span class="suffix"></span></h1>
<pre><code>#需要先安装esearch，efetch，可以用conda安装
conda install bioconda::entrez-direct
#下载runinfo 
esearch -db sra -query PRJNA562645 | efetch -format runinfo &gt; runinfo.csv 
#提取其中的SRR 
cat runinfo.csv | cut -d, -f1 | tail -n 8 &gt; runids.txt
#查看获得的SRR号
less runids.txt
  SRR10134390
  SRR10134389
  SRR25112328
  SRR25112330
  SRR25112332
  SRR25112331
  SRR25112329
  SRR25112327
  
#可以查看更多信息，比如数据大小，类型，单端还是双端测序等
cat runinfo.csv | cut -d, -f1,8,13,16
 Run,size_MB,LibraryStrategy,LibraryLayout
 SRR10134388,55895,RNA-Seq,PAIRED
 SRR10134387,56516,RNA-Seq,PAIRED
 SRR10134386,61231,RNA-Seq,PAIRED
 SRR10134385,55880,RNA-Seq,PAIRED
 SRR10134390,56230,RNA-Seq,PAIRED
 SRR10134389,60466,RNA-Seq,PAIRED
 SRR25112328,63671,RNA-Seq,PAIRED
 SRR25112330,71834,RNA-Seq,PAIRED
 SRR25112332,71837,RNA-Seq,PAIRED
 SRR25112331,69031,RNA-Seq,PAIRED
 SRR25112329,69215,RNA-Seq,PAIRED
 SRR25112327,69926,RNA-Seq,PAIRED

</code></pre>
<h3 id="获取样本名称和srr的对应关系"><span class="prefix"></span><span class="content">获取样本名称和SRR的对应关系</span><span class="suffix"></span></h3>
<pre><code>#需要先安装xtract
#先下载信息
esearch -db sra -query PRJNA290779 | efetch -format docsum &gt; docsum.txt
#提取有用信息
cat docsum.txt | xtract -pattern DocumentSummary -element Title,Bioproject,Biosample,Run@acc
</code></pre>
<pre><code>##nano prefetch.sh 
#!/bin/bash 

#SBATCH --job-name=zyjob
#SBATCH -n 4 # 指定核心数量
#SBATCH -N 1 # 指定node的数量
#SBATCH --time=04:00:00 # 运行总时间，天数-小时数-分钟， D-HH:MM
#SBATCH -p normal* # 提交到哪一个分区
#SBATCH --mem=4000 # 所有核心可以使用的内存池大小，MB为单位
#SBATCH -o SRA # 把输出结果STDOUT保存在哪一个文件
#SBATCH -e SRAerror # 把报错结果STDERR保存在哪一个文件
#SBATCH --mail-type=ALL # 发送哪一种email通知：BEGIN,END,FAIL,ALL
#SBATCH --mail-user=zuoyiyi1010@163.com # 把通知发送到哪一个邮箱
#SBATCH --constraint=2630v3  # the Features of the nodes, using command " showcluster " could find them.
#SBATCH --gres=gpu:n # 需要使用多少GPU，n是需要的数量

for i in `cat runids.txt` 
do prefetch $i
done 
for j in SRR*/*sra
do
fasterq-dump --split-files $j
done
##ctrl+x 保存文件并退出 
##chmod +x prefetch.sh 
##sbatch prefetch.sh #提交作业
##sqcct -j jobid
##squeue -j jobid
</code></pre>
<p>sinfo 查看显示集群的所有分区节点的空闲状态，<code>idel</code>为空闲，<code>mix</code>为节点部分核心可以使用，<code>alloc</code>为已被占用<br>
squeue -u <code>whoami</code>#在队列中显示自己的作业<br>
scontrol show job xx 只能显示正在运行或者刚结束没多久的作业信息<br>
sqcct -j xx 查询已经结束作业的相关信息<br>
scancel idxx  取消作业<br>
scancel -u <code>whoami</code><br>
scancel -t PENDING -u <code>whoami</code>  取消自己上机账号上所有状态为PENDING的作业<br>
如果有10台计算服务器每个服务器有两个八核的cpu；那么节点数目总的就是10，调用两个节点计算，cpu就是4，核数就是32核</p>
<p>#!/bin/bash</p>
<p>#SBATCH --job-name=zyjob<br>
#SBATCH -n 4<br>
#SBATCH -N 1<br>
#SBATCH --time=04:00:00<br>
#SBATCH -p normal<br>
#SBATCH --mem=4000<br>
#SBATCH -o SRA<br>
#SBATCH -e SRAerror<br>
#SBATCH --mail-type=ALL<br>
#SBATCH <a href="mailto:--mail-user=zuoyiyi1010@163.com">--mail-user=zuoyiyi1010@163.com</a></p>
<p>for i in <code>cat runids.txt</code><br>
do prefetch $i --max-size 80GB<br>
done</p>
<p>for j in SRR*/*sra<br>
do fasterq-dump --split-files $j<br>
done</p>
<h3 id="aspera高速下载"><span class="prefix"></span><span class="content">aspera高速下载</span><span class="suffix"></span></h3>
<p><a href="https://www.jianshu.com/p/fdbd4c13fa29">Aspera数据下载快速应用 - 简书 (jianshu.com)</a><br>
NCBI的ftp目前无法下载，可通过<strong>ENA 下载数据</strong>，上文中有介绍<br>
<code>ascp -v -k 1 -T -l 200m -P 33001 -i /root/miniconda3/envs/bioinfo/etc/asperaweb_id_dsa.openssh era-fasp@fasp.sra.ebi.ac.uk:vol1/fastq/SRR212/006/SRR2125156/SRR2125156.fastq.gz ./</code></p>
<h5 id="多线程同时下载命令"><span class="prefix"></span><span class="content">多线程同时下载命令</span><span class="suffix"></span></h5>
<p><code>seq 6 9 | parallel --verbose "ascp -v -k 1 -T -l 200m -P 33001 -i /root/miniconda3/envs/bioinfo/etc/asperaweb_id_dsa.openssh era-fasp@fasp.sra.ebi.ac.uk:vol1/fastq/SRR212/00{}/SRR212515{}/SRR212515{}.fastq.gz ./ "</code></p>
</div>
</body>

</html>
