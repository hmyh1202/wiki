## **clusterProfiler**初测

------



**clusterProfiler**是一个基于基因、基因集的集统计分析和可视化的R包，支持多种over-representation检验和基因本体富集分析，可进行分析如：Gene Ontology、EGG Pathway、MeSH 、Reactome Pathway、DAVID 、Disease Ontology等。



R包说明文档参考： 

【功能概要】 https://guangchuangyu.github.io/software/clusterProfiler/

【使用说明】 https://guangchuangyu.github.io/2016/01/go-analysis-using-clusterprofiler/

【使用手册】https://yulab-smu.github.io/clusterProfiler-book/chapter1.html,，很详细

【参考文献】 Guangchuang Yu, Li-Gen Wang, Yanyan Han, Qing-Yu He. clusterProfiler: an R package for comparing biological themes among gene clusters. OMICS: A Journal of Integrative Biology. 2012, 16(5):284-287.



### Gene Ontology富集分析：

clusterProfiler提供使用物种的***OrgDb*** 对象来进行富集分析，Bioconductor官网已经提供了多达20个物种的OrgDb对象文件；当然用户也可以使用`AnnotationHub`（该基于R 3.6构建）包来自定义构建OrgDb对象。

Bioconductor OrgDb：[OrgDb](http://bioconductor.org/packages/release/BiocViews.html#___OrgDb)



### 1. 如何使用

#### 1.1 基于已有的`OrgDb`对象进行GO富集分析

使用clusterProfiler需要提前做的是安装相应物种的OrgDb包，这一点比较麻烦，可使用 BiocManager::install("包") 进行安装。 clusterProfiler所使用的OrgDb支持多种基因ID类型，不推荐使用SYMBOL类型。



需要安装的包有：

**clusterProfiler**、**AnnotationHub**（自建库）、**topGO**（DAG分析）

clusterProfiler版本：[3.12.0](http://www.bioconductor.org/packages/release/bioc/html/clusterProfiler.html)



富集分析由**enrichGO**模块功能来实现：

```R
enrichGO     GO Enrichment Analysis of a gene set. Given a vector of genes, this
             function will return the enrichment GO categories after FDR control.
Usage:
  enrichGO(gene, OrgDb, keyType = "ENTREZID", ont = "MF", pvalueCutoff = 0.05, 
           pAdjustMethod = "BH", universe, qvalueCutoff = 0.2, minGSSize = 10, 
           maxGSSize = 500, readable = FALSE, pool = FALSE)
Arguments:
  gene                 a vector of entrez gene id.
  OrgDb                OrgDb
  keyType              keytype of input gene
  ont                  One of "MF", "BP", and "CC" subontologies or 'ALL'.
  pvalueCutoff         Cutoff value of pvalue.
  pAdjustMethod        one of "holm", "hochberg", "hommel", "bonferroni", "BH", "BY", "fdr", "none"
  universe             background genes
  qvalueCutoff         qvalue cutoff
  minGSSize            minimal size of genes annotated by Ontology term for testing.
  maxGSSize            maximal size of genes annotated for testing
  readable             whether mapping gene ID to gene Name
  pool                 If ont=’ALL’, whether pool 3 GO sub-ontologies
```

**操作实例**：

```R
#载入Hs包：Genome wide annotation for Human
> library(org.Hs.eg.db)
# 看看Db库中包含哪些内容
> keytypes(org.Hs.eg.db)
 [1] "ACCNUM"       "ALIAS"        "ENSEMBL"      "ENSEMBLPROT"  "ENSEMBLTRANS"
 [6] "ENTREZID"     "ENZYME"       "EVIDENCE"     "EVIDENCEALL"  "GENENAME"
[11] "GO"           "GOALL"        "IPI"          "MAP"          "OMIM"
[16] "ONTOLOGY"     "ONTOLOGYALL"  "PATH"         "PFAM"         "PMID"
[21] "PROSITE"      "REFSEQ"       "SYMBOL"       "UCSCKG"       "UNIGENE"
[26] "UNIPROT"

# 造一个测试数据列表，内容为ENSEMBL基因ID
> DATA <- read.delim("geneList.txt", sep="\n", header=F)
> head(DATA)
               V1
1 ENSG00000196611
2 ENSG00000093009
3 ENSG00000109255
4 ENSG00000134690
5 ENSG00000065328
6 ENSG00000117399
> gene1 <- c(t(DATA))
> gene1
[1] "ENSG00000196611" "ENSG00000093009" "ENSG00000109255" "ENSG00000134690"
[5] "ENSG00000065328" "ENSG00000117399"

#通过ensembl的基因ID号可以找到Db文件中其他信息的对应关系，例如：
> cols <- c("SYMBOL", "GENENAME")
# 使用select函数读取gene1在Db文件中的对应信息
> select(org.Hs.eg.db, keys=gene1, columns=cols, keytype="ENSEMBL")

'select()' returned 1:1 mapping between keys and columns
          ENSEMBL SYMBOL
1 ENSG00000196611   MMP1
2 ENSG00000093009  CDC45
3 ENSG00000109255    NMU
4 ENSG00000134690  CDCA8
5 ENSG00000065328  MCM10
6 ENSG00000117399  CDC20
                                                     GENENAME
1                                   matrix metallopeptidase 1
2                                      cell division cycle 45
3                                                neuromedin U
4                            cell division cycle associated 8
5 minichromosome maintenance 10 replication initiation factor
6                                      cell division cycle 20

#再例如检索SOX1基因，以SYMBOL基因为属性
> select(org.Hs.eg.db, keys="SOX1", columns=c("ENSEMBL","ENTREZID","GO","GENENAME"), keytype="SYMBOL")
'select()' returned 1:many mapping between keys and columns
   SYMBOL         ENSEMBL ENTREZID         GO EVIDENCE ONTOLOGY  GENENAME
1    SOX1 ENSG00000182968     6656 GO:0000978      IDA       MF SRY-box 1
2    SOX1 ENSG00000182968     6656 GO:0000981      ISA       MF SRY-box 1
3    SOX1 ENSG00000182968     6656 GO:0000981      ISM       MF SRY-box 1
4    SOX1 ENSG00000182968     6656 GO:0000981      NAS       MF SRY-box 1
5    SOX1 ENSG00000182968     6656 GO:0001228      IEA       MF SRY-box 1
 ... ...

# 此外，提供bitr方法进行数据ID的转换
# 例如，从ENSEMBL编号转换为ENTREZID编号，但都是基于数据库记录的信息
bitr(gene1, fromType="ENSEMBL", toType=c("ENTREZID","SYMBOL"), OrgDb="org.Hs.eg.db")

1 ENSG00000120784     22835           ZFP30
2 ENSG00000120798      7181           NR2C1
... ...
Warning message:
In bitr(gene1, fromType = "ENSEMBL", toType = c("ENTREZID", "SYMBOL"),  :
  0.62% of input gene IDs are fail to map...

        
# 所以，clusterProfiler基于上述数据结构可以根据任意的获取基因“ID”和多种注释信息
# keytype支持ENSEMBL、SYMBOL、ENTREZID

# 基于gene1列表调取Db文件中的信息
> gene.df <- select(org.Hs.eg.db, keys=gene1, columns=c("ENTREZID", "ENSEMBL", "SYMBOL"), keytype="ENSEMBL")
> head(gene.df)
        ENSEMBL    ENTREZID SYMBOL
1 ENSG00000196611     4312   MMP1
2 ENSG00000093009     8318  CDC45
3 ENSG00000109255    10874    NMU
4 ENSG00000134690    55143  CDCA8
5 ENSG00000065328    55388  MCM10
6 ENSG00000117399      991  CDC20

# 载入包
> library(clusterProfiler)

# 对细胞组分大类CC进行富集分析
> go <- enrichGO(gene=gene.df$ENSEMBL, OrgDb=org.Hs.eg.db, keyType='ENSEMBL', ont="CC", pAdjustMethod="BH", pvalueCutoff=0.01, qvalueCutoff=0.05)

# ont也可设置'ALL'，即对BP，CC和MF进行整体分析，而不是单独进行富集分析

# 如果基因数目少，无富集会产生以下结果
> go
#
# over-representation test
#
#...@organism    Homo sapiens
#...@ontology    CC
#...@keytype     ENSEMBL
#...@gene        chr [1:6] "ENSG00000196611" "ENSG00000093009" "ENSG00000109255" ...
#...pvalues adjusted by 'BH' with cutoff <0.01
#...0 enriched terms found

# 若有富集则显示以下结果：
> go
#
# over-representation test
#
#...@organism    Homo sapiens
#...@ontology    CC
#...@keytype     ENSEMBL
#...@gene        chr [1:5000] "ENSG00000000003" "ENSG00000000005" "ENSG00000000419" ...
#...pvalues adjusted by 'BH' with cutoff <0.01
#...83 enriched terms found
'data.frame':   83 obs. of  9 variables:
 $ ID         : chr  "GO:0015629" "GO:0005819" "GO:0099568" "GO:0098793" ...
 $ Description: chr  "actin cytoskeleton" "spindle" "cytoplasmic region" "presynapse" ...
 $ GeneRatio  : chr  "164/4726" "126/4726" "146/4726" "157/4726" ...
 $ BgRatio    : chr  "464/20630" "348/20630" "423/20630" "467/20630" ...
 $ pvalue     : num  5.63e-10 1.09e-08 2.83e-08 6.22e-08 1.88e-07 ...
 $ p.adjust   : num  4.19e-07 4.05e-06 7.02e-06 1.16e-05 2.80e-05 ...
 $ qvalue     : num  2.82e-07 2.73e-06 4.72e-06 7.79e-06 1.88e-05 ...
 $ geneID     : chr  "ENSG00000000938/ENSG00000002834/ENSG00000005206/ENSG00000005471/ENSG00000006453/ENSG00000006747/ENSG00000006788"| __truncated__ "ENSG00000002822/ENSG00000004897/ENSG00000005022/ENSG00000007168/ENSG00000008323/ENSG00000010244/ENSG00000013573"| __truncated__ "ENSG00000002834/ENSG00000004939/ENSG00000006747/ENSG00000007168/ENSG00000007171/ENSG00000007174/ENSG00000007908"| __truncated__ "ENSG00000003147/ENSG00000005379/ENSG00000007516/ENSG00000008056/ENSG00000008282/ENSG00000011347/ENSG00000018236"| __truncated__ ...
 $ Count      : int  164 126 146 157 127 26 142 113 126 131 ...

# GO富集统计如下：
> summary(go)

# 数据输出
data <- as.data.frame(go)
write.table(data, file="go_enrich.xls", row.names=F, sep="\t") 

```



### 1.2 自己构建Db文件

通过使用[AnnotationHub](http://bioconductor.org/help/annotationhub/)包进行构建

```
What is AnnotationHub ? 

AnnotationHub is a new approach to providing annotation resources to the Bioconductor community. The initial plan is to make available fasta, GFF / GTF, BED, VCF, and similar files from entities such as Ensembl, UCSC, ENCODE, and 1000 Genomes projects as objects ready for work in Bioconductor, e.g., GRanges representations of BED files.
```



```R
# 依照作者测试
library(AnnotationHub)

hub <- AnnotationHub()
# 该步骤测试失败，可能是网络问题： 
#Error in .util_download(x, rid[i], proxy, config, "bfcadd()", ...) :
#  bfcadd() failed; see warnings()
#In addition: Warning messages:
#1: download failed
#  web resource path: #‘https://annotationhub.bioconductor.org/metadata/annotationhub.sqlite3’
#  local file path: ‘/root/.cache/AnnotationHub/4272463205a1_annotationhub.sqlite3’
#  reason: An unknown option was passed in to libcurl
#2: bfcadd() failed; resource removed
#  rid: BFC3
#  fpath: ‘https://annotationhub.bioconductor.org/metadata/annotationhub.sqlite3’
#  reason: download failed
》 确实有遇到过该问题： 手动下载，https://links.jianshu.com/go?to=https%3A%2F%2Fannotationhub.bioconductor.org%2Fmetadata%2Fannotationhub.sqlite3

# 选择一个物种
query(hub, "Cricetulus") 

## AnnotationHub with 4 records
## # snapshotDate(): 2015-12-29
## # $dataprovider: UCSC, Inparanoid8, ftp://ftp.ncbi.nlm.nih.gov/gene/DATA/
## # $species: Cricetulus griseus
## # $rdataclass: ChainFile, Inparanoid8Db, OrgDb, TwoBitFile
## # additional mcols(): taxonomyid, genome, description, tags,
## #   sourceurl, sourcetype
## # retrieve records with, e.g., 'object[["AH10393"]]'
##
##             title
##   AH10393 | hom.Cricetulus_griseus.inp8.sqlite
##   AH13980 | criGri1.2bit
##   AH14346 | criGri1ToHg19.over.chain.gz
##   AH48061 | org.Cricetulus_griseus.eg.sqlite

# 基于列出的文件进行选择
Cgriseus <- hub[["AH48061"]]
```

关于AnnotationHub使用可参考： https://www.cnblogs.com/yatouhetademao/p/8018252.html



### 1.3 富集结果可视化

```R
# 条形图
barplot(go,showCategory=20,drop=T)

# 散点图
dotplot(go, showCategory=20)
# 这步测试报错： wrong orderBy parameter; set to default `orderBy = "x"`

# DAG有向无环图绘制
# 只能单独对BP、CC或MF进行DAG绘制
plotGOgraph(go)

# 若go为整体富集，如仅对BP组分绘制DAG
plotGOgraph(go.BP)
```

散点图示意图：

![img](http://guangchuangyu.github.io/blog_images/Bioconductor/clusterProfiler/2016_GO_analysis_using_clusterProfiler_files/figure-markdown_strict/unnamed-chunk-7-1.png)



### 1.4 KEGG富集分析

```
KEGG Enrichment Analysis of a gene set. Given a vector of genes,
this function will return the enrichment KEGG categories with FDR control.

Usage:
enrichKEGG(gene, organism = "hsa", keyType = "kegg",
       pvalueCutoff = 0.05, pAdjustMethod = "BH", universe,
       minGSSize = 10, maxGSSize = 500, qvalueCutoff = 0.2,
       use_internal_data = FALSE)

Arguments:
  gene                     a vector of entrez gene id.
  organism                 supported organism listed in ’http://www.genome.jp/kegg/catalog/org_list.html’
  keyType                  one of "kegg", ’ncbi-geneid’, ’ncib-proteinid’ and ’uniprot’
  pvalueCutoff             Cutoff value of pvalue.
  pAdjustMethod            one of "holm", "hochberg", "hommel", "bonferroni", "BH", "BY", "fdr", "none"
  universe                 background genes
  minGSSize                minimal size of genes annotated by Ontology term for testing.
  maxGSSize                maximal size of genes annotated for testing
  qvalueCutoff             qvalue cutoff
  use_internal_data        logical, use KEGG.db or latest online KEGG data
```

keyType支持kegg、 ncbi-geneid、ncib-proteinid和uniprot

enrichKEGG只支持entrez gene ID，这一点比较麻烦

```R
# 数据ID转换为entrez 编号
gene2 <- bitr(gene1, fromType="ENSEMBL", toType="ENTREZID", OrgDb="org.Hs.eg.db")
data <- gene2$ENTREZID
# 
kegg <- enrichKEGG(data, 
                   organism = 'hsa', 
                   keyType = 'kegg', 
                   pvalueCutoff = 0.05,
                   pAdjustMethod = 'BH', 
                   minGSSize = 10,
                   maxGSSize = 500,
                   qvalueCutoff = 0.2,
                   use_internal_data = FALSE)

# 查看数据分析结果
head(kegg)
         ID                          Description              GeneRatio
hsa05165 hsa05165              Human papillomavirus infection  139/2395
hsa04010 hsa04010                      MAPK signaling pathway  123/2395
hsa04120 hsa04120              Ubiquitin mediated proteolysis   64/2395
hsa04510 hsa04510                              Focal adhesion   87/2395
hsa04141 hsa04141 Protein processing in endoplasmic reticulum   73/2395
hsa04668 hsa04668                       TNF signaling pathway   53/2395
~
BgRatio    pvalue     p.adjust       qvalue      geneID            Count
330/7878 2.726058e-06 0.0008696126 0.0006628626    572/51384/...    139
295/7878 1.829656e-05 0.0029183006 0.0022244760    020/10368/...    123
137/7878 3.777734e-05 0.0032083090 0.0024455350    90293/996/...    64
199/7878 4.022958e-05 0.0032083090 0.0024455350    572/3675/...     87
165/7878 1.038498e-04 0.0059955161 0.0045700849    7095/51360/...   73
112/7878 1.127683e-04 0.0059955161 0.0045700849    843/8837/...     53

```

kegg富集简单可视化：

```R
# 散点图
dotplot(kegg, showCategory=30)
```



### 1.5 总结



1. clusterProfile使用简单，功能丰富，GO、KEGG、DAVID、GSEA等，非常适合对基因/基因集进行功能研究；
2. 使用该包需要克服的问题是需要建库，最好将20种物种和生僻物种数据文件本地化，所以关键是自己构建OrgDb；
3. 作者一直在维护和更新，不时加入新功能和改进。



### 补充：非模式物种clusterProfiler进行富集分析 ：

```R
# 载入包
> library(clusterProfiler)

# 设置我的工作路径
> setwd("/home/shaohuaihan/bin/Iso-seq/image/R")

# 读取GO注释文件
> go <- read.table("test_go_annot.txt", sep="\t", quote="")
# 查看GO库数据格式
> head(go)
          V1                                V2                 V3        V4
1 GO:0000015 phosphopyruvate_hydratase_complex cellular_component gene1
2 GO:0000015 phosphopyruvate_hydratase_complex cellular_component gene2
3 GO:0000015 phosphopyruvate_hydratase_complex cellular_component gene2
# 各列分别是GO term，GO sub term，GO top term和基因ID

# 读取差异基因列表
> gene <- read.table("test_gene.txt")
# 查看基因数据
> head(gene)
    V1
1 gene100
2 gene200
3 gene300

# 处理差异基因列表，"data.frame" —> "character"
> gene <- as.vector(gene$V1)

# GO to gene数据框
> go_term2gene <- data.frame(go$V1, go$V4)
> head(go_term2gene)
   go.V1     go.V4
1 GO:0000015 gene10
2 GO:0000015 gene23
3 GO:0000015 gene32

# 表头命名
> names(go_term2gene) <- c("go_term", "gene")

# GO term to GO name数据框
> go_term2name <- data.frame(go$V1, go$V2)
> head(go_term2name)
       go.V1                 go.V2
1 GO:0000015 phosphopyruvate_hydratase_complex
2 GO:0000015 phosphopyruvate_hydratase_complex
3 GO:0000015 phosphopyruvate_hydratase_complex

# 表头命名
> names(go_term2name) <- c("go_term", "name")

# 至此，建立了GO Term与基因ID和功能的关系
 
# 使用enrichr函数进行GO富集分析
> GO <- enricher(gene=gene, pvalueCutoff = 0.05, qvalueCutoff = 0.2, pAdjustMethod = "BH", TERM2GENE = go_term2gene, TERM2NAME = go_term2name)

# 查 看 结 果
> GO
# over-representation test
#
#...@organism    UNKNOWN
#...@ontology    UNKNOWN
#...@gene        chr [1:774] "Lal000100" "Lal001330" "Lal001430" "Lal001690" "Lal001990" ...
#...pvalues adjusted by 'BH' with cutoff <0.05
#...6 enriched terms found
# 省略，与上文富集展示一样

# 文件输出
> write.table(as.data.frame(GO), file="GO_enrichment.txt", row.names = F, sep="\t")
# 表头如下：
# ID	Description	GeneRatio	BgRatio	pvalue	p.adjust	qvalue	geneID	Count

# 可视化
# 条形图
> barplot(GO, showCategory=10)
# 散点图
> dotplot(GO) #还是报错： wrong orderBy parameter;set to default `orderBy = "x"` ^_^!

# ontology追加细胞组分（CC)
> GO@ontology = "CC"


# 注意：由于不依赖于软甲的OrgDb数据库，CC，MF，BP基因需要单独去做富集分析

### KEGG套路一样 ###
# 读取KEGG注释文件
> kegg <- read.table("test_kegg_annot.txt", sep="\t", quote="") 

# 查看数据
> head(kegg)
         V1     V2        V3
1 gene1 K15397       KCS
2 gene2 K01051 E3.1.1.11
3 gene3 K14508      NPR1

# 同理
> kegg_term2gene <- data.frame(kegg$V2, kegg$V1)
> kegg_term2name <- data.frame(kegg$V2, kegg$V3)
> names(kegg_term2gene) <- c("ko_term", "gene")
> names(kegg_term2name) <- c("ko_term", "name")

# 查看数据
> head(kegg_term2gene)
  ko_term      gene
1  K15397 gene8
2  K01051 gen11
3  K14508 gene92
> head(kegg_term2name)
  ko_term      name
1  K15397       KCS
2  K01051 E3.1.1.11
3  K14508      NPR1

# 使用enrichr函数进行KEGG的富集分析
> KEGG <- enricher(gene=gene, pvalueCutoff = 0.05, qvalueCutoff = 0.2, AdjustMethod = "BH", TERM2GENE = kegg_term2gene, TERM2NAME = kegg_term2name)

# 输出到文件
> write.table(as.data.frame(KEGG), file="KEGG_enrichment.txt", row.names = F, sep="\t")

# 表头信息如下
# ID	Description	GeneRatio	BgRatio	pvalue	p.adjust	qvalue	geneID	Count

# 可视化
> barplot(KEGG, showCategory=10)
> dotplot(KEGG)
# dev.off()
```

**关于enricher方法**：

```R
enricher            package:clusterProfiler            R Documentation

enricher

Description:

     A universal enrichment analyzer

Usage:

     enricher(gene, pvalueCutoff = 0.05, pAdjustMethod = "BH", universe,
       minGSSize = 10, maxGSSize = 500, qvalueCutoff = 0.2, TERM2GENE,
       TERM2NAME = NA)

Arguments:

    gene: a vector of gene id

pvalueCutoff: pvalue cutoff

pAdjustMethod: one of "holm", "hochberg", "hommel", "bonferroni", "BH",
          "BY", "fdr", "none"

universe: background genes

minGSSize: minimal size of genes annotated for testing

maxGSSize: maximal size of genes annotated for testing

qvalueCutoff: qvalue cutoff

TERM2GENE: user input annotation of TERM TO GENE mapping, a data.frame
          of 2 column with term and gene

TERM2NAME: user input of TERM TO NAME mapping, a data.frame of 2 column
          with term and name

Value:

     A ‘enrichResult’ instance

Author(s):

     Guangchuang Yu
```

> 一个通用的超几何检统计检验方法

​																	2019/07/23-24