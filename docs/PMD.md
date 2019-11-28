> Title: methy_Haplo
> Date: 2019-03-01
> Category: methyl
> Tags: methyl, PMD
> Author: shaohuaihan
> Summary:



### 程序功能及原理说明
基于bisulfite测序比对数据（已去除duplicate和unmap数据），借助HMM模型在全基因组范围内寻找 partially methylated domains (PMD）。

为了确定是否存在PMDs，首先计算一个所选择的染色体的α-value分布，α是滑动窗口（包含100个连续的CpGs）的甲基化水平的分布。α-value小于1则反映出一个散射的分布，偏向低和高甲基化水平分布；大于等于1时分布均匀，倾向于中间甲基化水平分布状态，即PMDs。

PMDs的寻找可以使用R包MethylSeekR实现。



### 程序使用说明

```makefile
$python  generate_PMD.py -h
usage: generate_PMD.py [-h] -d DIR -m METHY -s SNV -c CHR -g GENOME -de DEPTH

generate PMD R script

optional arguments:
  -h, --help            show this help message and exit
  -d DIR, --dir DIR     work dir	工作路径
  -m METHY, --methy METHY			DNA甲基化CX_report文件
                        methyl file
  -s SNV, --snv SNV     snv file	SNP文件，用于过滤SNP-cytosine冲突结果
  -c CHR, --chr CHR     chr name	所分析的染色体名称
  -g GENOME, --genome GENOME		参考基因组版本
                        genome version
  -de DEPTH, --depth DEPTH			cytosine的深度
                        depth

其中：
-g 参数为内置的基因组版本信息（下述文件最后一列），对应关系如下： 
#ID	latin	genome_version
BSgenome.Drerio.UCSC.danRer10	Drerio	danRer10
BSgenome.Hsapiens.1000genomes.hs37d5	Hsapiens	hs37d5
BSgenome.Hsapiens.NCBI.GRCh38	Hsapiens	GRCh38
BSgenome.Hsapiens.UCSC.hg19	Hsapiens	hg19
BSgenome.Hsapiens.UCSC.hg38	Hsapiens	hg38
BSgenome.Mmusculus.UCSC.mm10	Mmusculus	mm10
```



#### 参数说明

```shell
  -d	工作路径
  
  -m	DNA甲基化CX_report文件，gz压缩格式
  
  -s	SNP文件，用于过滤SNP-cytosine冲突结果
  
  -c	所分析的染色体名称
  
  -g	参考基因组版本，见genome.ini
  
  -de	cytosine的筛选深度，默认5x
```



### 使用示例
```bash
export LD_LIBRARY_PATH=/opt/glibc-2.14/lib:$LD_LIBRARY_PATH

python generate_PMD.py -d $PWD -m test.CX_report.txt.gz -s snv.txt -c chr10 -g hg19 -de 5

```



## Resource

```shell
http://www.bioconductor.org/packages/release/bioc/vignettes/MethylSeekR/inst/doc/MethylSeekR.pdf
```