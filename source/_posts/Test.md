---
title: Test
date: 2023-02-02 09:30:57
tags:
typora-root-url: Test
---

Output Spline
<!--more-->

```python
import unreal as ue
import os
import csv
from math import ceil

def output_csv(divs, csvFile):

    filePath = os.path.dirname(csvFile)
    if not filePath:
        os.makedirs(filePath)
    cf = open(csvFile, "w", newline='')
    csv_writer = csv.writer(cf)

    slActors = ue.EditorLevelLibrary.get_selected_level_actors()
    for slActor in slActors:
        spline = slActor.get_component_by_class(ue.SplineComponent)
        length = spline.get_spline_length()
        numberOfPoints = ceil(length / divs)
        for i in range(numberOfPoints+1):
            distance = i * divs
            if i == numberOfPoints:
                distance = length        
            posVector = spline.get_world_location_at_distance_along_spline(distance)
            pos = []
            pos.append(posVector.x/100)
            pos.append(posVector.z/100)
            pos.append(posVector.y/100)
            csv_writer.writerow(pos)

        csv_writer.writerow("|")
    cf.close()

if __name__ == '__main__':    
    divs = 100.0
    csvFile = "D:/spline1.csv"
    print(csvFile)
    output_csv(divs, csvFile)

```

<center>
    <img src="Pig.png" width="45%" border ="1100"> 
    <div style = "font-size: 80%"> UE5布2料1制1作流程 </div>
    <br>
</center>

阿斯蒂芬 三大法师法撒旦伤大防守打法啥地方伤大防守打法sad啥地方士大夫士大夫手动发送大根深蒂固为蒂芬 三大法师法撒旦伤蒂芬 三大法师法撒旦伤蒂芬 三大法师法撒旦伤蒂芬 三大法师法撒旦伤蒂芬 三大法师法撒旦伤蒂芬 三大法师法撒旦伤蒂芬 三大法师法撒旦伤蒂芬 三大法师法撒旦伤蒂芬 三大法师法撒旦伤

### 墙购物

#### 各位亲岗位我去过未去过为问过围墙购物券歌曲
