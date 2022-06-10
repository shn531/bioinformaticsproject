# Main Project에 관한 Readme : Figure5A
## Main Text
1. 기존에 제공된 예제 1-3의 데이터의 형식에 따르다.
2. Data의 출처는 github의 hyeshik/colab-biolab.git에 있는 데이터를 불러와서 사용한다.
```java
!git clone https://github.com/hyeshik/colab-biolab.git
!cd colab-biolab && bash tools/setup.sh
exec(open('colab-biolab/tools/activate_conda.py').read())
```
3. 해당 데이터는 나의 google colab에 불러온 후 사용한다.
```java
%cd /content/drive/MyDrive/binfo1-datapack1/
```
4. bioinfo-datapack1에서 해당 실험에 사용한 데이터는 다음과 같으며, pandas를 통해 dataframe을 불러온다.
```java
!featureCounts -a gencode.gtf -o read-counts.txt *.bam
```
```java
import pandas as pd
cnts = pd.read_csv('read-counts.txt', sep='\t', comment='#', index_col=0)
cnts.head()
cnts
```
5. Figure에 사용될 x/y 축인 clip enrichment 와 rden_change를 만들어준다.
```java
cnts['clip_enrichment'] = cnts['CLIP-35L33G.bam'] / cnts['RNA-control.bam']
cnts['rden_change'] = (cnts['RPF-siLin28a.bam'] / cnts['RNA-siLin28a.bam']) / (cnts['RPF-siLuc.bam'] / cnts['RNA-siLuc.bam'])
cnts.head()
```
6. numpy 를 가져온 후, 사용할 Geneid들만 목록으로 추린다. 이때 소수점 뒷부분은 lambda를 통해 제거해 준다.
```java
import numpy as np
cnts['Geneid'] = cnts.index
cnts_1 = cnts.reset_index(drop = True)
cnts_1['Geneid'] = cnts_1['Geneid'].apply(lambda x: x.split(".")[0])
Geneid = cnts_1['Geneid']
```
7. Geneid의 index를 지워준 후, txt file로 전송해 이후 Goterm에 이용한다.
```java
Geneid_noindex = Geneid.reset_index(drop = True)
Geneid.to_csv('Geneid.txt', index = False)
```
8. 이후 GO term 분석방법을 위해서 여러가지 시도를 해 보았는데, 각각의 방법들은 아래 주석에 달아놨다. 
8-1. Panther를 통해 Go term과 Geneid를 합쳐준 후 text file로 전송한 후 목록을 만드는 방법
* panther link : <http://pantherdb.org>
* Pantherorg 사용방법은 해당 readme에 설명해 놓았다. - [Panther](https://github.com/shn531/bioinformaticsproject/blob/main/Mainproject/Panther.md)
```java
panther = pd.read_csv('pantherGeneList.txt', sep='\t', names = ["1", "2", "3", "4", "5", "6"])
```
* Geneid는 여러가지 섞여있기 때문에, split을 통해 MGI 번호만 남겨놓고 지우기로 하였다.
* 이부분 부터는 수정이 더 필요함!
```java
goandgene = panther['2'].str.split('|', expand = True)
```
8-2. Biomart emsembl을 통해 Gene에 해당하는 GOTERM을 1:1로 정리한 이후 위에 데이터와 합쳐주는 방법
*biomart link : <https://www.ensembl.org/biomart/martview/3060b118360cd8fc256f91b8247b41bf>
*Biomart emsembl의 사용 방법은 해당 readme에 설명해 놓았다.  - [Biomart](https://github.com/shn531/bioinformaticsproject/blob/main/Mainproject/Biomart.md)
*Biomart를 통한 gene, go list는 첨부해 놓았으며, 이러한 list를 가지고 dataset을 만들기로함.

9. Biomart 방법을 따라 Gene stable ID(GENE ID), GO term accession과 GO term name을 불러온 후 pandas로 dataframe 형성한다.
```java
goanden = pd.read_csv('goanden.txt', sep=',')
```

10. 불러온 정보들을 위의 cnt data와 합쳐주기 위해서 column 이름을 변경하고 Geneid를 공통분모로 합쳐준다.
```java
goanden.rename(columns = {'Gene stable ID':'Geneid'},inplace=True)
merge = pd.merge(cnts_1, goanden, how='inner', on=None)
```

11. Gene을 GO 별로 묶은 후 count로 묶어준다.
```java
geneexport = pd.read_csv('geneexport.txt, sep = ',')
genesize = geneexport.groupy([GO term accession, GO stable ID].count().reset_index(name='count')
```

12. mmc6 (supplement 에 있는 mmc6 GO term 을 가져와서 이름을 위에 genesize 와 맞춰준 후 최종적으로 합쳐준다.
```java
mmc6 = pd.read_csv('mm6.txt', index_col = False, sep="\t")
mmc6.rename(columns = {'GO accession':'GO term accession'},inplace=True)
mergefinal = pd.merge(mmc6, genesize, how='inner', on=None)
mergefinal.rename(columns = {'RPF\r\nRibosome density change (log2)':'Rdenlogchange'},inplace=True)
```

13. MannWhitney U test 실행한다
```java
import scipy.stats as stats
stats.mannwhitneyu(x = mergefinal['CLIP-seq log2 enrichment'], y = mergefinal['Rdenlogchange'])
```


14. DATA 준비는 완료되었으며, 그 이후 cbook을 이용해 scatter plot을 만든다. 우선 필요한 프로그램들을 불러온다.
**그전에 FDR <0.05 값들만 excel을 통해서 추려 줬다.
```java
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.cbook as cbook
import matplotlib as mpl
from mpl_toolkits.axes_grid1 import make_axes_locatable
```

15. 최종적으로 그래프를 만들어준다.
```java
volume = mergefinaldrop['count']
color = mergefinaldrop['RPF\r\nFDR']

fig, ax = plt.subplots(figsize = (15,8))
figure = ax.scatter(mergefinaldrop['CLIP-seq log2 enrichment'], mergefinaldrop['Rdenlogchange'], c = color, cmap = "YlOrRd", s = volume*0.5, alpha = 0.9)

ax.set_xlabel('Enrichment level of LIN28A-bount CLIP tags(log2)')
ax.set_ylabel('Ribosome density change upon Lin28a knockdown(log2)')

ax.grid(True)
#colorlabel
cbar = plt.colorbar(figure, ticks = [np.arange(0, 10e-20, 10e-2)])
cbar.set_label('Term-specific enrichment confidence(false dicovery rate')
cbar.set_ticks(np.arange(10e-20, 1, 10e-2))

#annotation
ax.annotate('cytoplasm', xy = (-0.25, 0.53), xycoords='data', xytext = (-60, 50), size = 10, textcoords = 'offset points', bbox=dict(boxstyle = "round", fc = "0.8"), arrowprops = dict(arrowstyle = "->"))


plt.show()
```

## 사용된 데이터의 출처
1. LIN28a paper
2. LIN28a paper Figure 5A
3. LIN28 Table5 :
4. LIN28a Table6 :

## 사용된 CODE 출처
1. SCATTER PLOT 정보 :    
 * [SCATTER DEMO](https://matplotlib.org/3.5.0/gallery/lines_bars_and_markers/scatter_demo2.html#sphx-glr-gallery-lines-bars-and-markers-scatter-demo2-py)     
 * [SCATTER PLOT MARKER](https://www.pythonprogramming.in/adjust-marker-sizes-and-colors-in-scatter-plot.html)
2. DataFrame 수정 정보 :     
 * [Dataframe1](https://firedino.tistory.com/49)     
 * [Dataframe2](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.reset_index.html)      
 * [Dataframe3](https://mizykk.tistory.com/126)    
3. GO data 수정 정보 :     
 * [GSEApy_Biomart](https://gseapy.readthedocs.io/en/latest/gseapy_example.html)     
4. MATPLOTLIB 수정 정보 :     
 * [Subplot](https://www.delftstack.com/ko/howto/matplotlib/how-to-improve-subplot-size-or-spacing-with-many-subplots-in-matplotlib/)
