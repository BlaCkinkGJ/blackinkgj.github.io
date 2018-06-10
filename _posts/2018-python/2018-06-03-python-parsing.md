---
layout: post
title: "[파이썬] 위키피디아 간단한 파싱 방법"
date: 2018-6-3
excerpt: "위키 피디아의 내용을 파싱해보도록 한다."
tag:
- python
comments: true
---
# BeautifulSoup와 PANDAS를 활용한 파싱


```python
# 파싱할 웹 페이지 값을 가져오기 위해 import 해줍니다.
import urllib2

# 파싱할 위치를 입력합니다.
base = "https://en.wikipedia.org/wiki/
search = "Swimming_at_the_2016_Summer_Olympics#Men's_events"
wiki = base + search

# HTML 데이터를 가져오도록 합니다.
page = urllib2.urlopen(wiki)

# 파싱을 위해 import합니다.
from bs4 import BeautifulSoup

# 페이지로 가져온 데이터를 BeautifulSoup를 통해 soup 변수로 데이터를 이동합니다.
soup = BeautifulSoup(page)

# 테이블의 위치와 그것의 class를 가져오도록 합니다.
first_table = soup.find("table", class_ = "wikitable plainrowheaders")

# 리스트에서 데이터를 가져오도록 합니다.
list = [[] for i in xrange(7)] # 7개 열이므로
for row in first_table.findAll("tr"):
    cells = row.findAll(['td','th'])
    if len(cells) == 7:
        for i in xrange(len(cells)):
        sample = cells[i].find(text=True)
        if sample != None:
          list[i].append(sample.encode('UTF8'))

# 데이터 분석을 위한 판다스를 가져옵니다.
import pandas as pd
df=pd.DataFrame()
df['Event'] = list[0]
df['Gold Name'] = list[1]
df['Gold Time'] = list[2]
df['Silver Name'] = list[3]
df['Silver Time'] = list[4]
df['Bronze Name'] = list[5]
df['Bronze Time'] = list[6]

print df
```