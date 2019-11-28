> Title: methy_Haplo
> Date: 2019-02-01
> Category: methyl
> Tags: methyl, haplotype, block 
> Author: shaohuaihan
> Summary:



### 程序功能及原理说明
基于bisulfite测序比对数据（已去除duplicate和unmap数据）提取methylation haplotype并鉴定methylation haplotype block（MHB）。

http://www.nature.com/ng/journal/v49/n4/full/ng.3805.html



### 程序使用说明

```makefile
$make -f methy_Haplo
Description:
	 Methyl Haplo Module for WGBS data
USAGE:

	 make -f makefile workDir= FA= genomePre
	 make -f makefile workDir= thread=6 samName= bamFile= hapInfo
	 make -f makefile workDir= samName= region= plotLD
cal haplo: genomePre hapInfo and region plot using plotLD
	 make -f makefile workDir= mindepth=5 samName= BAM= mappBin
	 make -f makefile workDir= samName= mindepth=5 mergeHaplo
	 make -f makefile workDir= samName= minR2=0.3 snpBED=you need to prepare calMHB
cal MHB: genomePre hapInfo mappBin mergeHaplo calMHB
```



#### 参数说明

```shell
# 准备CpG信息
genomePre：
	workDir=工作路径
	FA=参考基因组文件

# 从比对文件中提取methylation haplotype信息
hapInfo：
	samName=样本名称
	bamFile=比对后的BAM文件（不要求排序）
	thread=线程

# 绘制LD相关性图形
plotLD：
	workDir=工作路径
    samName=样本名称
    region=所需绘制的区域，eg chr1:1090:1983	

# 提取high mappability区域
mappBin:
	workDir=工作路径
    mindepth=深度阈值，默认5x
    samName=样本名称
    BAM=BAM比对文件（无需提前sort）

# 合并hapinfo文件至上述连续区域
mergeHaplo
	workDir=工作路径
    samName=样本名称
    mindepth=5

# 鉴定MHB，首先methylation haplotype被分割到连续的mappable区间中，贪婪算法选取最大可能的连续区域，同时保证有最小的连锁不平衡值，此外block中至少含有3个CpG位点。
# 参考：Shoemaker et al. 2010 Genome Research.
calMHB：
	workDir=工作路径
    samName=样本名称
    minR2=0.3 ld数值
    snpBED=snp bed文件，需要单独准备（可基于WGS或BS-seq）

```



### 使用示例
```bash
FASTA=genome.fa
BAM=test.bam
# step1
make -f methy_Haplo workDir=$PWD samName=test FA=${FASTA} genomePre
# example for cpositions file from step1
chr10:W	24	CG
chr10:W	31	CG
chr10:W	49	CG

# step2
make -f methy_Haplo workDir=$PWD thread=6 samName=test bamFile=${BAM} hapInfo
#example for haplo info from step2
chr2:127688027-127688030        CC      2       127688027:127688030
chr15:58346987-58346987 T       1       58346987
chr15:58346987-58346987 C       3       58346987

# plot for a example haplo region
chr1    68      3358    3291
chr1    3805    10700   6896
chr1    10768   11164   397
make -f methy_Haplo workDir=$PWD samName=test region=chr1:10000-20000 plotLD
# step3
make -f methy_Haplo workDir=$PWD mindepth=5 samName=test BAM=${BAM}
# mappBin
# example mappable bin from step3
chr1    68      3358    3291
chr1    3805    10700   6896
chr1    10768   11164   397

# step4
make -f methy_Haplo workDir=$PWD samName=test mindepth=5 mergeHaplo
# result as step2

# step5
make -f methy_Haplo workDir=$PWD samName=test minR2=0.3 snpBED=you need to prepare calMHB
# BED file of chr、start and end.


```

## Resource

```shell
https://github.com/dinhdiep/MONOD2
```