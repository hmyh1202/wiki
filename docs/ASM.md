## ASM（allele-specific methylation analysis）



> Title: ASM
> Date: 2019-02-18
> Category: methyl
> Tags: methyl, ASM
> Author: shaohuaihan
> Summary: for allele-specific methylation analysis



### 程序功能及原理说明
**ASM** (allele-specific methylation analysis) is an allele specific methylation analysis is primarily to compare the DNA methylation linked by two different alleles (heterozygous genotype). 

ASM works for SNP files in standard VCF format, however, may be ok for Bis-SNP output. If it does not work, you’d better reformat the Bis-SNP results.



examples allele-specific methylated region shown in  [Weilong Guo, et al., Scientifc Report, 2016](http://www.nature.com/articles/srep32207)



### 程序使用说明

```shell
$bash run_ASM.sh 

ASM: Allele specific methylated region/site calling
	Fisher exact test for site calling
	Students' t-test for region calling


USAGE: run_ASM.sh <ref_FA>  <align_BAM>  <snp_VCF>  <out_Folder>
need SAMtools and perl
```



#### 参数说明

```shell
ref_FA: 参考基因组FA文件，需要有fai的索引，samtools完成
align_BAM： 比对文件，需要有bai索引，samtools完成
snp_VCF： 标准的VCF文件，若是bisSNP检测，可能需要简单处理一下
其他参数内部默认

```



### 使用示例
```bash
bash bash run_ASM.sh test.fa test.bam test.vcf ./

shell中将所输入的文件如基因组fa、比对后BAM均连接到输出路径，重新进行索引和后续分析


### ASS（allele specific methylated site）输出示例：
Chr	SNP_Pos  Ref	Allele1	Allele2	C_Pos	Allele1_linked_C	Allele2_linked_C   	Allele1_linked_C_met	Allele2_linked_C_met	pvalue	fdr	ASM
Chr1	8949221 T	T	A	8949252	30,2	6,0	0.94	1.00	1.00e+00	1.00e+00	FALSE
各列信息如下：
Chr: 染色体编号
SNP_Pos: SNP的1-based比对位置
Ref: 参考基因组碱基
Allele1: 等位核苷酸1 [ATCG]
Allele2: 等位核苷酸2 [ATCG]
C_Pos: C的1-based位置
Allele1_linked_C: 与allele1相关联的胞嘧啶的reads深度，逗号分隔，即支持甲基化和非甲基化reads数目
Allele2_linked_C: 与allele2相关联的胞嘧啶的reads深度，逗号分隔，即支持甲基化和非甲基化reads数目
Allele1_linked_C_met: 与allele1关联的C的甲基化水平
Allele2_linked_C_met: 与allele2关联的C的甲基化水平
pvalue: t test得到的p-value
fdr: BH校正后的p-value
ASM: TRUE表示该C是allele特异甲基化的

### ASR（allele specific methylated region）输出示例
Chr	Pos	Ref	Allele1	Allele2	Allele1_linked_C	Allele2_linked_C	Allele1_linked_C_met	Allele2_linked_C_met	pvalue	fdr	ASM
chr1	8943402	A	A	T	1-1	0.8-1	1	0.9	5.01e-01	7.52e-01	FALSE
各列信息如下：
Chr: 染色体编号
Pos: SNP的1-based比对位置
Ref: 参考基因组碱基
Allele1: 等位核苷酸1 [ATCG]
Allele2: 等位核苷酸2 [ATCG]
Allele1_linked_C: 与allele1关联的Cs的甲基化水平，"-"分隔
Allele2_linked_C: 与allele2关联的Cs的甲基化水平，"-"分隔
Allele1_linked_C_met: 与allele1关联的Cs的平均甲基化水平
Allele2_linked_C_met: 与allele2关联的Cs的平均甲基化水平
pvalue: t test得到的p-value
fdr: BH校正后的p-value
ASM: TRUE表示该region是allele特异甲基化的
```



## Resource

```shell
Weilong Guo, et al. CGmapTools improves the precision of heterozygous SNV calls and supports allele-specific methylation detection and visualization in bisulfite-sequencing data. 2017, Bioinformatics.
```
