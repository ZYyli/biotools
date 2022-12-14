# Fastqc安装运行
## 1.安装java环境(需要root权限)
 
    1.sudo  mkdir /usr/java 
    #建立文件夹
    2.sudo  tar -zvxf ~/Biosofts/jdk-8u172-linux-x64.tar.gz -C /usr/java/
    #解压压缩包文件存入/usr/java/
    3.cd /usr/java
    #进入文件夹
    4.sudo  ln -s jdk1.8.0_172 latest
    5.sudo  ln -s /usr/java/latest default
    #创建软连接，可在不同目录用相同文件
    6.sudo  vi /etc/profile
    #末尾加上如下几行
    7.export JAVA_HOME=/usr/java/latest
    8.export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
    9.export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:JAVA_HOME/lib/tools.jar
    10.source /etc/profile
    #执行配置文件
    11.java -verison
    #查看版本
  ![](https://raw.githubusercontent.com/ZYyli/bioinfosoftware/master/2022/09/20/MHKfbter21PjTK3w.jpeg)
 
## 2.fastqc安装

    1.cd ~/Biosofts
    #下载压缩包 wget http://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.7.zip
    #创建文件夹 mkdir ~/Biosofts/fastqc
    2.unzip /disk1/shares/fastqc_v0.11.7.zip -d ~/Biosofts/
    #解压文件到Biosofts目录下
    3.chmod +x ~/Biosofts/FastQC/fastqc
    #给予执行权限
![](https://raw.githubusercontent.com/ZYyli/bioinfosoftware/master/2022/09/21/HCXXZFKfSJmlx1jZ.png)

  
    4.~/Biosofts/FastQC/fastqc -h
    #通过路径查看
    5.echo  'export PATH=~/Biosofts/FastQC:$PATH'>>~/.bashrc
    #加入环境变量，就不用再输入路径了
    6.source ~/.bashrc
    #重新执行刚修改完的初始化文档
    7.fastqc -h
    #查看fastqc所有命令和参数
![](https://raw.githubusercontent.com/ZYyli/bioinfosoftware/master/2022/09/21/wxuwYVVHVLunNYss.jpeg)
![](https://raw.githubusercontent.com/ZYyli/bioinfosoftware/master/2022/09/21/S21XaxmvputHsxqW.png)
## fastqc评价测试数据质量

    fastqc /disk1/shares/Seqs/Akle_TTAGGC_L004_R2_001.fastq.gz -o ~
    #生成的报告储存在当前目录
在Windows下载Winscp，将Linux文件拖拽至Windows，打开html文件浏览
## multiqc整合多个质控结果
#安装conda（前面已安装过）

    1.conda create --name python2 python=2.7 -c https://mirrors.ustc.edu.cn/anaconda/cloud/bioconda/ -y
    #安装python2环境
    2.conda activate python2
    #进入python2环境
    3.conda install multiqc -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda
    #conda里面下载multiqc
    #将fastqc评价测试数据质量得到报告存入fastQC文件夹中
    4.cd fastQC/
    5.multiqc .
    #生成multiqc报告并将其拖拽到本地查看
![](https://raw.githubusercontent.com/ZYyli/bioinfosoftware/master/2022/10/11/4yKeNgEB8mh1VAfz.jpeg)
 ![](https://raw.githubusercontent.com/ZYyli/bioinfosoftware/master/2022/10/11/swPO0YZSqgkpTduY.png)

## 对SRR6232298和SRR6208854实际操作

 1. 先用prefetch下载SRR6232298和SRR6208854文件
 2. 解压其为fastq.gz文件
 3. fastqc评价测试数据质量
 4. multiqc整合多个质控结果
![](https://raw.githubusercontent.com/ZYyli/bioinfosoftware/master/2022/10/17/KL0mx1CC4sxhlGlA.png)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyNzY3OTIwNDUsMjAzMDkwNjk3MV19
-->