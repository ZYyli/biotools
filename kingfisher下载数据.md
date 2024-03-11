<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>kingfisher下载数据</title>
  <link rel="stylesheet" href="https://stackedit.cn/style.css" />
</head>

<body class="stackedit">
  <div class="stackedit__html"><h2 id="介绍"><span class="prefix"></span><span class="content">介绍</span><span class="suffix"></span></h2>
<p>一个高通量测序数据下载工具，用户提供Run accessions或者BioProject accessions，即可在<strong>ENA、SRA、Amazon AWS</strong>以及<strong>Google Cloud</strong>等数据库中下载目标数据。<strong>Kingfisher会尝试从一系列的数据源进行数据下载，直到某个源能够work。</strong><br>
此外，还能根据用户的需求<strong>将下载数据直接输出为SRA、Fastq、Fasta或Gzip等格式</strong>，非常方便，不需要自己再对SRA数据通过fasterq-dump进行拆分转换</p>
<h2 id="安装"><span class="prefix"></span><span class="content">安装</span><span class="suffix"></span></h2>
<p><code>conda create -c conda-forge -c bioconda -n kingfisher pigz python extern curl sra-tools pandas requests aria2</code><br>
<code>conda activate kingfisher</code><br>
<code>#使用conda activate不能成功激活环境时可以尝试使用：source activate kingfisher</code><br>
<code>pip install bird_tool_utils'&gt;='0.2.17</code><br>
#下载该工具需要使用清华镜像<br>
<a href="https://blog.csdn.net/JineD/article/details/124774570">完美解决 Could not find a version that satisfies the requirement 安装包名字 (from versions: )-CSDN博客</a><br>
<code>git clone https://github.com/wwood/kingfisher-download cd kingfisher-download/bin</code><br>
<code>export PATH=$PWD:$PATH</code><br>
<code>kingfisher -h</code><br>
<code>#弹出帮助文档即安装成功</code></p>
<h2 id="使用"><span class="prefix"></span><span class="content">使用</span><span class="suffix"></span></h2>
<h4 id="如果只想下载某个确定的sra数据，则使用-r参数，提供srr-number即可，如-srr12042866-；若是想批量下载某个bioproject中的所有数据，则可以使用-p参数，提供bioproject-number，如prjna640275或srp267791。"><span class="prefix"></span><span class="content">如果只想下载某个确定的SRA数据，则使用-r参数，提供SRR Number即可，如 SRR12042866 ；若是想<strong>批量下载某个BioProject中的所有数据</strong>，则可以使用-p参数，提供BioProject Number，如PRJNA640275或SRP267791。</span><span class="suffix"></span></h4>
<p><code>kingfisher get -r SRR1013438 -m ena-ascp ena-ftp prefetch aws-http</code></p>
<h4 id="ena-ascp调用aspera从ena中下载.fastq.gz数据"><span class="prefix"></span><span class="content">ena-ascp,调用Aspera从ENA中下载.fastq.gz数据</span><span class="suffix"></span></h4>
<h4 id="ena-ftp，调用curl从ena中下载.fastq.gz数据"><span class="prefix"></span><span class="content">ena-ftp，调用curl从ENA中下载.fastq.gz数据</span><span class="suffix"></span></h4>
<h4 id="prefetch，调用prefetch从ncbi-sra数据库中下载sra数据，然后默认使用fasterq-dump对其进行拆分转换"><span class="prefix"></span><span class="content">prefetch，调用prefetch从NCBI SRA数据库中下载SRA数据，然后默认使用fasterq-dump对其进行拆分转换</span><span class="suffix"></span></h4>
<h4 id="aws-http，调用aria2c从aws-open-data-program中下载sra数据，然后默认使用fasterq-dump对其进行拆分转换"><span class="prefix"></span><span class="content">aws-http，调用aria2c从AWS Open Data Program中下载SRA数据，然后默认使用fasterq-dump对其进行拆分转换</span><span class="suffix"></span></h4>
<p><strong>也就是说，如果是用的ENA源 直接下载的就是fastq，如果用的SRA或其他，那就是下载SRA数据 然后kingfisher再自动调用fasterq-dump转换成fastq</strong></p>
<h2 id="检查数据是否完整"><span class="prefix"></span><span class="content">检查数据是否完整</span><span class="suffix"></span></h2>
<p><code>vdb-validate xx</code></p>
</div>
</body>

</html>
