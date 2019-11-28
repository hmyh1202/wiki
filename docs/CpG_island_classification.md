> Title: CpG_island_classification
> Date: 2018-12-12
> Category: methyl
> Tags: CpG island, shore, shelf, BED
> Author: shaohuaihan
> Summary:



### 程序功能及原理说明
基于给定的CpG Island文件，提取CpG island侧翼的区域，诸如CpG Shore、CpG Shelf区域的位置信息。

CpG Island文件可以从EPI基因组库直接获取，也可直接下载。

CpG Island侧翼的示意图如下：

|——N Shelf(2kb)——|—N Shore(2kb)—|——**CpG Island**——|—S Shore(2kb)—|——S Shelf(2kb)——|



### 程序使用说明**

```python
python3 CpG_island_classification.py  -h
usage: CpG_island_classification.py [-h] 
    --input_BED INPUT_BED --outdir OUTDIR
    [--CpG_shore {yes,no}]
    [--CpG_shelf {yes,no}] --chrlist CHRLIST

optional arguments:
  -h, --help            show this help message and exit
  --input_BED INPUT_BED
                        input BED file of ori CGI bed
  --outdir OUTDIR       out dir
  --CpG_shore {yes,no}  whether to select CpG Shore or not
  --CpG_shelf {yes,no}  whether to select CpG Shelf or not
  --chrlist CHRLIST     input chrlist list, including chr ID and chr length
```

#### 参数说明

```bash
--input_BED	输入的CpG island文件，3列BED格式
--outdir	文件输出路径
--CpG_shore	是否输出CpG shore信息
--CpG_shelf	是否输出CpG shelf信息
--chrlist	包含染色体名称和长度信息的2列文件
```



### 结果示例与解读
```
输出文件包含：
CpG_flanking.bed	整合CpG Island、Shore、Shelf信息的文件
CpG_Island.bed	CpG Island文件
CpG_Shelf.bed	CpG Shore文件
CpG_Shore.bed	CpG Shelf文件

以CpG_flanking为例子，各列信息如下：
NC_000067.6     3527624 3529624 NC_000067.6:3527624:3529624|Shelf       +
NC_000067.6     3529624 3531624 NC_000067.6:3529624:3531624|Shore       +
NC_000067.6     3531624 3531843 NC_000067.6:3531624:3531843|CGI +
NC_000067.6     3531843 3533843 NC_000067.6:3531843:3533843|Shore       +
NC_000067.6     3533843 3535843 NC_000067.6:3533843:3535843|Shelf       +
```

- 第1列：染色体/Scaffold名称
- 第2列：区域的起始位置
- 第3列：区域的终止位置
- 第4列：区域的基本信息，包含坐标和区域类型
- 第5列：CpG Island本身不区分正负链，统一使用正链

