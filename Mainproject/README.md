# Main Project에 관한 Readme : Figure5A
## Main Source
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
8. panther를 통해 GO term과 Geneid를 합쳐준 후 txt file로 전송한 후 목록을 만들었다.   
* panther link : <http://pantherdb.org>
```java
panther = pd.read_csv('pantherGeneList.txt', sep='\t', names = ["1", "2", "3", "4", "5", "6"])
```
## 사용된
