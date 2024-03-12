## 安装
[Download Cell Ranger - Official 10x Genomics Support](https://www.10xgenomics.com/support/software/cell-ranger/downloads)
![输入图片说明](https://raw.githubusercontent.com/ZYyli/bioinfosoft_pictures/master/imgs/2024-03-12/XAfV8AXbIBQzeil0.jpeg)
下载完后查看md5sum值，看下载是否完整。
```
#解压
tar -zxvf cellranger-7.2.0.tar.gz
#修改环境变量
echo 'export PATH=/cellranger-7.2.0/:$PATH' >> ~/.bashrc
#更新配置文件
source ~/.bashrc
cellranger count --help
`

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk5NzA2NDQ1MF19
-->