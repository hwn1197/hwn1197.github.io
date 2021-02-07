---
layout: post
title: 리그오브레전드 정글 미드 듀오 티어
image: 
  path: /assets/img/lol.jpg
description: > 
 리그오브레전드 정글 미드 듀오 승률계산
hide_description: true
sitemap: false
---

#### 배경

많은 사람들이 롤을 즐기면서 듀오로 랭겜을 돌린다. 봇 듀오, 미드 탑, 탑 정글 등 많은 조합으로 듀오를 가지만 op.gg나 fow 같은 대표적인 전략 분석 사이트에서는 듀오에 대한 티어 정보를 제공하지 않는다. 그래서 어떤 조합의 챔피언들이 가장 티어가 높을까?라는 호기심에서 이 분석을 진행하기로 했다.\
나는 주로 미드를 가고 정글 가는 친구와 듀오를 자주 하기 때문에 최적의 "미드 정글 듀오 챔피언 조합"을 찾아 티어별로 정리해보는 것을 목표로 한다.\
조금 더 구체적으로 현재 내 티어가 플래티넘4 이기 때문에 플래티넘4 구간의 데이터만 불러와서 살펴 봤다.


처음으로 진행하는 분석이기도 하며 데이터 공부를 목적으로 한 분석이기에 중간 중간에 개념에 대한 정리가 포함 되어 있다.

### 1. 데이터 불러오기


```python
import numpy as np
import pandas as pd
import json
import requests
import time
import warnings
warnings.filterwarnings(action='ignore')
```


```python
# 볼 수 있는 행,열 개수 설정
pd.set_option('display.max_rows', 100)
pd.set_option('display.max_columns', 100)
```

 라이엇에서는 게임과 관련된 모든 데이터를 "developer.riotgames.com"에서 제공하고 있다.\
데이터를 불러오기 위해서는 api_key를 넣어줘야 하는데 키는 매 24시간 마다 만료되므로 매일 갱신시켜줘야한다.\
그렇지 않고 실수로 만료된 key로 데이터를 불러올 경우 블랙리스트에 올라 하루동안 아이디가 일시 정지 될 수도 있기에 주의해야 한다.

#### API란?

API는 프로그램들이 서로 상호작용하는 것을 도와주는 매개체
![image.png](attachment:image.png)
#### API의 역할은? 
 

1. API는 서버와 데이터베이스에 대한 출입구 역할을 한다.
: 데이터베이스에는 소중한 정보들이 저장된다. 모든 사람들이 이 데이터베이스에 접근할 수 있으면 안 된다. API는 이를 방지하기 위해 여러분이 가진 서버와 데이터베이스에 대한 출입구 역할을 하며, 허용된 사람들에게만 접근성을 부여해준다.

2. API는 애플리케이션과 기기가 원활하게 통신할 수 있도록 한다.
: 여기서 애플리케이션이란 우리가 흔히 알고 있는 스마트폰 어플이나 프로그램을 말합다. API는 애플리케이션과 기기가 데이터를 원활히 주고받을 수 있도록 돕는 역할을 합다.

3. API는 모든 접속을 표준화한다.
API는 모든 접속을 표준화하기 때문에 기계/ 운영체제 등과 상관없이 누구나 동일한 액세스를 얻을 수 있다. 쉽게 말해, API는 범용 플러그처럼 작동한다고 볼 수 있다.


#### 파이썬에서 request를 이용해 데이터 불러오기 
Python requests

Python에서 HTTP 요청을 보내는 모듈 = requests

0. 기본적인 사용방법
URL = 'http://whateverthewebsites.com'
response = requests.get(URL) - 이 URL에 요청을 보냈고

response.statues_code - 요청을 받아 뭔가를 처리한 후 요청자인 나에게 응답을 줬고 200, 400,429

response.text - HTML 코드 확인

1. GET 요청할때 parameter 전달법

params = {'param1': 'value1', 'params2': ' value'}\
res = requests.get(URL, params = params)

ex)\
params = {'key':'value'}\
res = requests.get('http://whateverthewebsite.com', params = parmas)\
res.url - 내가 던진 URL이 뭔지 확인하는 법

2. POST 요청할 때 data 전달법\
data = {'param1' : 'value1', 'param2' : 'value'}\
res = requests.post(URL, data = data)\
*우리가 인지하고 있는 딕셔너리 구조를 유지하면서 문자열로 바꿔서 전달할때 json 모듈\
import requests, json\
data = {'outer' : {'inner' : 'value'}}\
res = requests.post(URL, data = json.dump(data))\

3. 헤더 추가, 쿠키추가

headers = {'Content-Type': 'application/json; charset = utrf-8'}\
cookies = {'seession_id':'sorryidontcare'}\
res = requests.get(URL, headers = headers, cookies=cookies)\

4.res. 치면 기능들 다나옴
ex)\
res.requests - 내가 보낸 request 객체에 접근 가능\
res.status_code 응답 코드\
res.raise_for_status 200 OK 코드가 아닌 경우 에러 발동\
res.json() json response일 경우 딕셔너리 타입으로 바로 변환. 


#### api key 업데이트 및 game_version 확인


```python
#version 확인 및 api_key update.

api_key = 'RGAPI-bd6912a3-e2c0-48fe-846f-c8bde4f2b5c5' # Key를 갱신하여야 한다
params = {'api_key' : 'RGAPI-bd6912a3-e2c0-48fe-846f-c8bde4f2b5c5'}
r = requests.get('https://ddragon.leagueoflegends.com/api/versions.json') # version data 확인
current_version = r.json()[0] # 가장 최신 버전 확인
current_version
```




    '11.1.1'



#### 챔피언 데이터 불러오기


```python
#챔피언들에 대한 정보를 developer 사이트에서 가져옴
r = requests.get('http://ddragon.leagueoflegends.com/cdn/{}/data/ko_KR/champion.json'.format(current_version))
parsed_data = r.json() # 파싱
info_df = pd.DataFrame(parsed_data)
info_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>type</th>
      <th>format</th>
      <th>version</th>
      <th>data</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Aatrox</th>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Aatrox', 'key': '...</td>
    </tr>
    <tr>
      <th>Ahri</th>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Ahri', 'key': '10...</td>
    </tr>
    <tr>
      <th>Akali</th>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Akali', 'key': '8...</td>
    </tr>
    <tr>
      <th>Alistar</th>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Alistar', 'key': ...</td>
    </tr>
    <tr>
      <th>Amumu</th>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Amumu', 'key': '3...</td>
    </tr>
  </tbody>
</table>
</div>



DataFrame 내에 딕셔너리 형태의 column이 존재하는데 여기서에서 챔피언 'key'와 같이 필요한 column들만 빼서 기존 df에 추가하는 function 만듦


```python
# 원하는 데이터 추출해서 기존 DF에 추가하는 function. 
def extract_column(df, high_column, low_column):
    df.loc[:, low_column] = df.loc[:, high_column].map(lambda x: x[low_column])
```


```python
#data columns에서 필요한 'key' 값을 빼온 뒤 확인.
extract_column(info_df, 'data', 'key')
info_df = info_df.reset_index()
info_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>type</th>
      <th>format</th>
      <th>version</th>
      <th>data</th>
      <th>key</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Aatrox</td>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Aatrox', 'key': '...</td>
      <td>266</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Ahri</td>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Ahri', 'key': '10...</td>
      <td>103</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Akali</td>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Akali', 'key': '8...</td>
      <td>84</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Alistar</td>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Alistar', 'key': ...</td>
      <td>12</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Amumu</td>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Amumu', 'key': '3...</td>
      <td>32</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>148</th>
      <td>Zed</td>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Zed', 'key': '238...</td>
      <td>238</td>
    </tr>
    <tr>
      <th>149</th>
      <td>Ziggs</td>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Ziggs', 'key': '1...</td>
      <td>115</td>
    </tr>
    <tr>
      <th>150</th>
      <td>Zilean</td>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Zilean', 'key': '...</td>
      <td>26</td>
    </tr>
    <tr>
      <th>151</th>
      <td>Zoe</td>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Zoe', 'key': '142...</td>
      <td>142</td>
    </tr>
    <tr>
      <th>152</th>
      <td>Zyra</td>
      <td>champion</td>
      <td>standAloneComplex</td>
      <td>11.1.1</td>
      <td>{'version': '11.1.1', 'id': 'Zyra', 'key': '14...</td>
      <td>143</td>
    </tr>
  </tbody>
</table>
<p>153 rows × 6 columns</p>
</div>



#### 스펠 데이터 불러오기


```python
#마찬가지로 스펠 데이터 불러옴
r = requests.get('http://ddragon.leagueoflegends.com/cdn/10.25.1/data/en_US/summoner.json')
parsed_data = r.json()
spell_df = pd.DataFrame(parsed_data)
extract_column(spell_df, 'data', 'key')
extract_column(spell_df, 'data', 'name')
spell_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>type</th>
      <th>version</th>
      <th>data</th>
      <th>key</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>SummonerBarrier</th>
      <td>summoner</td>
      <td>10.25.1</td>
      <td>{'id': 'SummonerBarrier', 'name': 'Barrier', '...</td>
      <td>21</td>
      <td>Barrier</td>
    </tr>
    <tr>
      <th>SummonerBoost</th>
      <td>summoner</td>
      <td>10.25.1</td>
      <td>{'id': 'SummonerBoost', 'name': 'Cleanse', 'de...</td>
      <td>1</td>
      <td>Cleanse</td>
    </tr>
    <tr>
      <th>SummonerDot</th>
      <td>summoner</td>
      <td>10.25.1</td>
      <td>{'id': 'SummonerDot', 'name': 'Ignite', 'descr...</td>
      <td>14</td>
      <td>Ignite</td>
    </tr>
    <tr>
      <th>SummonerExhaust</th>
      <td>summoner</td>
      <td>10.25.1</td>
      <td>{'id': 'SummonerExhaust', 'name': 'Exhaust', '...</td>
      <td>3</td>
      <td>Exhaust</td>
    </tr>
    <tr>
      <th>SummonerFlash</th>
      <td>summoner</td>
      <td>10.25.1</td>
      <td>{'id': 'SummonerFlash', 'name': 'Flash', 'desc...</td>
      <td>4</td>
      <td>Flash</td>
    </tr>
  </tbody>
</table>
</div>



#### 게임 데이터 불러오기

라이엇에서는 데이터를 불러오는데 2분에 100개씩이라는 제한을 걸어놓았다. \
아마 사람들이 한번에 너무 많은 데이터를 불러오려고 하면 트래픽이 초과되어 서버에 과부화가 올 수 있기 때문인 것 같다.\
그래서 모든 데이터들을 불러온 뒤 csv 파일로 저장해서 시간을 줄였다.\
물론 데이터가 많으면 많을 수록 좋겠지만 이 분석의 목적은 공부이기 때문에 시간을 줄이기 위해 일단 한 20000개 데이터 정도 불러와서 보고 데이터 수가 너무 작은지 판단해 보기로 했다.

플래티넘 유저들의 데이터만 들고오지만 최종적으로 필요한 정보는 게임 내의 미드와 정글 챔피언들의 조합 및 승률, 픽률, 밴률 등이기 때문에 몇번의 과정을 통해 데이터들을 불러와야 한다.

데이터 불러오는 순서
1) league data -> encrypted summonerId\
2) encrypted summnerId -> encrypted account ID\
3) encrypted accountId -> match data의 gameId\
4) gameId -> match 세부데이터


```python
# #플레4 유저 데이터 불러오기.

# platinum_URL1 = 'https://kr.api.riotgames.com/lol/league/v4/entries/RANKED_SOLO_5x5/PLATINUM/IV?page=1'
# r=requests.get(platinum_URL1, params = params)
# user_df = pd.DataFrame(r.json())

# pagenum = range(2,50)

# for i in pagenum: 
#     platinum_URL = 'https://kr.api.riotgames.com/lol/league/v4/entries/RANKED_SOLO_5x5/PLATINUM/IV?page={}'.format(i)
#     r= requests.get(platinum_URL, params = params)
#     while r.status_code!=200: # 요청 제한 또는 오류로 인해 정상적으로 받아오지 않는 상태라면, 60초 지연
#         time.sleep(60)
#         r = requests.get(platinum_URL, params = params)
#     user_df = pd.concat([user_df, pd.DataFrame(r.json())])

# user_df.to_csv('PlatinumIVData.csv')

# 저장된 데이터 받아오기
user_df = pd.read_csv('PlatinumIVData.csv', index_col=0)
user_df.reset_index(inplace=True)
user_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>leagueId</th>
      <th>queueType</th>
      <th>tier</th>
      <th>rank</th>
      <th>summonerId</th>
      <th>summonerName</th>
      <th>leaguePoints</th>
      <th>wins</th>
      <th>losses</th>
      <th>veteran</th>
      <th>inactive</th>
      <th>freshBlood</th>
      <th>hotStreak</th>
      <th>miniSeries</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>7f6e3f73-8959-479d-93e7-55f53723504a</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>lo-VVRltoxof-68ApOYeCxkaJkzt319W6xDpaP39VtOctPVo</td>
      <td>마왕Ex 볼디고드</td>
      <td>0</td>
      <td>97</td>
      <td>81</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
      <td>True</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>cff06d45-789d-4219-a5b7-c7c9fbce8bf0</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>5FqJ3WNHs3M2WtCDe8r_PV0GJVC61LblAott9azgPghg7i5V</td>
      <td>Nxsty</td>
      <td>97</td>
      <td>53</td>
      <td>29</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>4658c99e-d299-448c-a614-17424d027b66</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>GlgC2UQt1uYDgy8N50fl3zB-xeltpPIgVEmPOsfVoy_gqqru</td>
      <td>2003 09 05</td>
      <td>7</td>
      <td>61</td>
      <td>43</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>e17bb48c-52ca-4f2f-828e-c31b87910315</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>Ho0P2jO0d5UYnMqVccejvWrQMvVuIRk3_nEST09XxWLcGAtf</td>
      <td>H2641</td>
      <td>75</td>
      <td>228</td>
      <td>214</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>b72a45a9-12a2-4602-8341-ae5e56fa8828</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>NFBwk7UVsvig6D_y8WRuRAnvo2-EEhspVNtwZA9o_2vH4SQO</td>
      <td>LOL China</td>
      <td>52</td>
      <td>102</td>
      <td>100</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>20495</th>
      <td>200</td>
      <td>4034b7e7-f025-4ce4-a6a9-5c803ae09b01</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>Vg75V5aZUkxSaARjsgNKOb-K67pddGMhVyV8W25UfLNp4Q</td>
      <td>고니우기</td>
      <td>0</td>
      <td>227</td>
      <td>226</td>
      <td>True</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20496</th>
      <td>201</td>
      <td>a27c8fb7-b742-47aa-a584-7d7dbbc401a5</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>sFA4zKDeSrP20ivkNLT7ERWVc_-BdLLcKsbYVgYly9DNnw</td>
      <td>파란나라를보았니</td>
      <td>31</td>
      <td>745</td>
      <td>732</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20497</th>
      <td>202</td>
      <td>15510391-d370-459c-9a65-1f9617bb8b1f</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>kfnFRSNyiIQJBN80WgsRUANpvJRuLilAzQHesH6MXynFaA</td>
      <td>그냥딱</td>
      <td>17</td>
      <td>345</td>
      <td>331</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20498</th>
      <td>203</td>
      <td>a73844e0-43f7-446a-b1e0-2d82981745b1</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>vlRn5oCIeNmnegHEps_1iRLdrJ1FHLG1wih_NMi77QQqlw</td>
      <td>jakson</td>
      <td>61</td>
      <td>93</td>
      <td>111</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20499</th>
      <td>204</td>
      <td>aea2b11c-4187-49a1-b56b-1e55008b091c</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>IsWpWbpVbWm5aQ7fg3Xcq86dOicGxGrwvkz4iq9SwJGp7A</td>
      <td>동녘군</td>
      <td>0</td>
      <td>31</td>
      <td>30</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>20500 rows × 15 columns</p>
</div>




```python
# #User_df의 'summonerID'로 'encrypted accountID' 불러옴.
# summonerId를 이용하여 accountId를 모두 받아와야 한다. 요청 제한으로 인해 오래 걸리므로 파일로 저장하여 관리 to_csv
# API 요청 제한 : 1초에 20, 2분에 100
#생각보다 시간이 오래 걸려서 10,000개의 유저 데이터만 가지고 진행.

# user_df['account_id'] = np.nan # account_id 초기화
# for i, summoner_id in enumerate(user_df['summonerId']):
# #각 소환사의 SummonerId와 API Key를 포함한 url을 만들고, Summoner API에서 AccountId를 가져와 채워넣는다.
#     api_url = 'https://kr.api.riotgames.com/lol/summoner/v4/summoners/' + summoner_id + '?api_key=' + api_key
#     r = requests.get(api_url)
#     while r.status_code!=200: # 요청 제한 또는 오류로 인해 정상적으로 받아오지 않는 상태라면, 60초 지연
#         time.sleep(60)
#         r = requests.get(api_url)
#     account_id = r.json()['accountId']
#     user_df.iloc[i, -1] = account_id

# user_df.to_csv('LeagueData.csv')

league_df = pd.read_csv('LeagueData.csv',index_col=0)
league_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>leagueId</th>
      <th>queueType</th>
      <th>tier</th>
      <th>rank</th>
      <th>summonerId</th>
      <th>summonerName</th>
      <th>leaguePoints</th>
      <th>wins</th>
      <th>losses</th>
      <th>veteran</th>
      <th>inactive</th>
      <th>freshBlood</th>
      <th>hotStreak</th>
      <th>miniSeries</th>
      <th>account_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>7f6e3f73-8959-479d-93e7-55f53723504a</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>rFzHZfm0zb-iqDiJnBQIGtoWd1GTsei4GrGymQoyXYu2idu1</td>
      <td>마왕Ex 볼디고드</td>
      <td>0</td>
      <td>97</td>
      <td>81</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
      <td>NaN</td>
      <td>zW2aj1YTjOapsvPS0vOvulMla0zgYdDH-Z44vSPqwEcukC...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>fbe3f3fc-4f22-44ba-976f-47b2c7e01619</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>u1C4TM6NNLPDMSl9xb2dbaeXh92wOZnntKCdb4Gq-WU_hKjF</td>
      <td>응기이이이이이이이이이이이이이잇</td>
      <td>92</td>
      <td>220</td>
      <td>202</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
      <td>kQUw1aSZ5Uv1Cq53SUdynKrm1TWWgvB4mmT7JhwO3NXrXX...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>4658c99e-d299-448c-a614-17424d027b66</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>zss8bYwSlzXB7D9N7L9N92Cs4VnexshlvUenT7QVooHH2F6k</td>
      <td>2003 09 05</td>
      <td>0</td>
      <td>61</td>
      <td>44</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
      <td>cVOOLrTTAmIzwaOs7PJpt8ZCPP22cTGt90-Xu1C_8grpy7...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>d5c66391-7ae0-489f-9bd5-b6ba07070d34</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>KMPJBTofamUmJLfWodHLvd9jVN1X7qVtSZ3Md_TO0YIWlwJ9</td>
      <td>또물보라를일으켜다다다다다다다</td>
      <td>2</td>
      <td>44</td>
      <td>33</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
      <td>FS-gRP7MdHcTYBx-OK2KFK1SgBloytDdZGcOacfJd6pnHx...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>e2ae626b-15f0-4fad-9985-6c8199cf92ac</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>T4YekmYDKTt51pyFKOtfCimahjZuU4LftAxix792cZoz4iVn</td>
      <td>연천 마풍강</td>
      <td>0</td>
      <td>205</td>
      <td>191</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
      <td>KKEZg-xbCasosy2QBhvSvQ6nXyxvoEA_yvGGNDX3FlxYeH...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>10040</th>
      <td>200</td>
      <td>c014d464-b6ee-4803-8fe4-5d393e7852bd</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>_BxjinivJmU_f7gapmw7-h_cTW4HXuO2T4V74Gf5eRa1-q0</td>
      <td>덤불해오라기</td>
      <td>12</td>
      <td>267</td>
      <td>245</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
      <td>aZ4FeD5AN0nM23x5Wi-zebcNc9R9c5TsRGjfwiMoBFDJ</td>
    </tr>
    <tr>
      <th>10041</th>
      <td>201</td>
      <td>8ed41b3b-4bbd-4fd4-aab8-25821e7dce3f</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>Pv3WTs6PB3DTagiD_CmGL4B3Nym70qpgD30V4MWWD_ZyuRc</td>
      <td>sky시나모로</td>
      <td>70</td>
      <td>36</td>
      <td>17</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
      <td>sbVl40rSBsJW2Caod4l7nZo4D0s-I5Ji1zKxoxis_NBF</td>
    </tr>
    <tr>
      <th>10042</th>
      <td>202</td>
      <td>4c35e182-519a-4956-ba8d-a07b666e5e79</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>TaNjeJBwTbrSfEohKifdfSIOe7EdjSPM0XMy-8J6UidMgRM</td>
      <td>chuparla vajina</td>
      <td>0</td>
      <td>208</td>
      <td>207</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>NaN</td>
      <td>FlcJ8RZDZj_bzpizBB-ETJ9XiQrhJZCYDmwBOYGRYm6w</td>
    </tr>
    <tr>
      <th>10043</th>
      <td>203</td>
      <td>71329f98-44f0-4fba-8975-226453d0075c</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>M1tBL_oCXZmZCsySCj5faIMb5ndD8BF3BDrw0_bH9rBkWMg</td>
      <td>산부인과 간호사</td>
      <td>0</td>
      <td>49</td>
      <td>21</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
      <td>NaN</td>
      <td>AiME2cqvE9KrFhIHTrY8kJSslIw0g1GNbfgQ3iFdnWnl</td>
    </tr>
    <tr>
      <th>10044</th>
      <td>204</td>
      <td>2349f1d0-5719-4288-a3b2-fbd87e2fb7db</td>
      <td>RANKED_SOLO_5x5</td>
      <td>PLATINUM</td>
      <td>IV</td>
      <td>UyLecaB5nP52qKzReb4JHiVulpc6FgJxgDBGR9pKgfZUMTA</td>
      <td>리얼몽몽</td>
      <td>0</td>
      <td>52</td>
      <td>29</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
      <td>NaN</td>
      <td>CRBab4vx8-1wmqkR3X-HrHPsi2y3tog5MlZ6GIRjVfXd</td>
    </tr>
  </tbody>
</table>
<p>10045 rows × 16 columns</p>
</div>




```python
# #AccountId를 이용해 Match info 데이터 받기 (gameId를 얻기 위해서)
#platinum_URL1 = 'https://kr.api.riotgames.com/lol/league/v4/entries/RANKED_SOLO_5x5/PLATINUM/IV?page=1'
#r=requests.get(platinum_URL1, params = params)
#user_df = pd.DataFrame(r.json())

# season = str(13)

# match_url = 'https://kr.api.riotgames.com/lol/match/v4/matchlists/by-account/Mj2S3rdX4NO4y-dJHDr2B3d2vYwNo0B5o7GIPptv1KLb?season=13&api_key=RGAPI-19ded58b-fb57-4de9-b0a4-d33c4b67eac9'
# r = requests.get(match_url)
# match_info_df = pd.DataFrame(r.json()['matches'])
# match_info_df

# for account_id in league_df1['account_id']:
#     api_url = f'https://kr.api.riotgames.com/lol/match/v4/matchlists/by-account/{account_id}?season={season}&api_key={api_key}'
#     r = requests.get(api_url)
#     while r.status_code!=200: # 요청 제한 또는 오류로 인해 정상적으로 받아오지 않는 상태라면, 5초 간 시간을 지연
#         time.sleep(60)
#         r = requests.get(api_url)
#     match_info_df = pd.concat([match_info_df, pd.DataFrame(r.json()['matches'])])

# match_info_df.to_csv('MatchInfoData.csv')

# 저장된 데이터 받아오기
match_info_df = pd.read_csv('MatchInfoData.csv', index_col=0)
match_info_df.reset_index(inplace=True)
match_info_df

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>platformId</th>
      <th>gameId</th>
      <th>champion</th>
      <th>queue</th>
      <th>season</th>
      <th>timestamp</th>
      <th>role</th>
      <th>lane</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>KR</td>
      <td>4775472728</td>
      <td>81</td>
      <td>420</td>
      <td>13</td>
      <td>1605055261753</td>
      <td>DUO_CARRY</td>
      <td>BOTTOM</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>KR</td>
      <td>4775291889</td>
      <td>51</td>
      <td>420</td>
      <td>13</td>
      <td>1605053885302</td>
      <td>DUO</td>
      <td>NONE</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>KR</td>
      <td>4772178774</td>
      <td>81</td>
      <td>420</td>
      <td>13</td>
      <td>1604907524330</td>
      <td>DUO_CARRY</td>
      <td>BOTTOM</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>KR</td>
      <td>4766871878</td>
      <td>67</td>
      <td>420</td>
      <td>13</td>
      <td>1604708786840</td>
      <td>DUO_CARRY</td>
      <td>BOTTOM</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>KR</td>
      <td>4766870521</td>
      <td>81</td>
      <td>420</td>
      <td>13</td>
      <td>1604707101240</td>
      <td>DUO_CARRY</td>
      <td>BOTTOM</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>30095</th>
      <td>95</td>
      <td>KR</td>
      <td>4174638898</td>
      <td>412</td>
      <td>420</td>
      <td>13</td>
      <td>1582563659775</td>
      <td>DUO_SUPPORT</td>
      <td>BOTTOM</td>
    </tr>
    <tr>
      <th>30096</th>
      <td>96</td>
      <td>KR</td>
      <td>4174634161</td>
      <td>412</td>
      <td>420</td>
      <td>13</td>
      <td>1582561707791</td>
      <td>DUO_SUPPORT</td>
      <td>BOTTOM</td>
    </tr>
    <tr>
      <th>30097</th>
      <td>97</td>
      <td>KR</td>
      <td>4171831444</td>
      <td>154</td>
      <td>430</td>
      <td>13</td>
      <td>1582458029617</td>
      <td>SOLO</td>
      <td>TOP</td>
    </tr>
    <tr>
      <th>30098</th>
      <td>98</td>
      <td>KR</td>
      <td>4171724714</td>
      <td>98</td>
      <td>430</td>
      <td>13</td>
      <td>1582456379970</td>
      <td>DUO_SUPPORT</td>
      <td>BOTTOM</td>
    </tr>
    <tr>
      <th>30099</th>
      <td>99</td>
      <td>KR</td>
      <td>4171468204</td>
      <td>412</td>
      <td>430</td>
      <td>13</td>
      <td>1582454914199</td>
      <td>DUO_SUPPORT</td>
      <td>BOTTOM</td>
    </tr>
  </tbody>
</table>
<p>30100 rows × 9 columns</p>
</div>




```python
# #gameId를 통해 Match 데이터 받기 (경기의 승패, 팀원과 같은 정보들이 담겨있다.)

# match_info_df = match_info_df.drop_duplicates('gameId')

# match_df = pd.DataFrame()
# for game_id in match_info_df['gameId']: # 이전의 매치에 대한 정보 데이터에서 게임 아이디를 가져온다
#     api_url = 'https://kr.api.riotgames.com/lol/match/v4/matches/' + str(game_id) + '?api_key=' + api_key
#     r = requests.get(api_url)
#     while r.status_code!=200: # 요청 제한 또는 오류로 인해 정상적으로 받아오지 않는 상태라면, 5초 간 시간을 지연
#         time.sleep(60)
#         r = requests.get(api_url)
#     r_json = r.json()
#     temp_df = pd.DataFrame(list(r_json.values()), index=list(r_json.keys())).T # 게임 아이디에 대한 매치 데이터를 받아서 추가
#     match_df = pd.concat([match_df, temp_df])

# match_df.to_csv('MatchData.csv') # 파일로 저장

match_df = pd.read_csv('MatchData.csv', index_col=0)

#정확한 통계를 위해 가장 최신의 버전과 클래식 게임에 대한 데이터만 가져오자
match_df = match_df.loc[match_df['gameMode']=='CLASSIC', :]

# 그 중에서도 이번 분석에서는 소환사의 협곡 솔로 랭크와 팀 랭크 게임만 사용한다.
select_indices = (match_df['queueId']==420) | (match_df['queueId']==440) 
match_df = match_df.loc[select_indices, :].reset_index(drop=True)

# DataFrame 내의 리스트들이 파일로 저장되었다가 불러지는 과정에서 문자로 인식됨
for column in ['teams', 'participants', 'participantIdentities']:
    match_df[column] = match_df[column].map(lambda v: eval(v)) # 각 값에 대해 eval 함수를 적용

match_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>gameId</th>
      <th>platformId</th>
      <th>gameCreation</th>
      <th>gameDuration</th>
      <th>queueId</th>
      <th>mapId</th>
      <th>seasonId</th>
      <th>gameVersion</th>
      <th>gameMode</th>
      <th>gameType</th>
      <th>teams</th>
      <th>participants</th>
      <th>participantIdentities</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4775472728</td>
      <td>KR</td>
      <td>1605055261753</td>
      <td>1865</td>
      <td>420</td>
      <td>11</td>
      <td>13</td>
      <td>10.23.343.2581</td>
      <td>CLASSIC</td>
      <td>MATCHED_GAME</td>
      <td>[{'teamId': 100, 'win': 'Win', 'firstBlood': F...</td>
      <td>[{'participantId': 1, 'teamId': 100, 'champion...</td>
      <td>[{'participantId': 1, 'player': {'platformId':...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4775291889</td>
      <td>KR</td>
      <td>1605053885302</td>
      <td>969</td>
      <td>420</td>
      <td>11</td>
      <td>13</td>
      <td>10.23.343.2581</td>
      <td>CLASSIC</td>
      <td>MATCHED_GAME</td>
      <td>[{'teamId': 100, 'win': 'Win', 'firstBlood': F...</td>
      <td>[{'participantId': 1, 'teamId': 100, 'champion...</td>
      <td>[{'participantId': 1, 'player': {'platformId':...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4772178774</td>
      <td>KR</td>
      <td>1604907524330</td>
      <td>1566</td>
      <td>420</td>
      <td>11</td>
      <td>13</td>
      <td>10.22.341.643</td>
      <td>CLASSIC</td>
      <td>MATCHED_GAME</td>
      <td>[{'teamId': 100, 'win': 'Win', 'firstBlood': T...</td>
      <td>[{'participantId': 1, 'teamId': 100, 'champion...</td>
      <td>[{'participantId': 1, 'player': {'platformId':...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4766871878</td>
      <td>KR</td>
      <td>1604708786840</td>
      <td>1596</td>
      <td>420</td>
      <td>11</td>
      <td>13</td>
      <td>10.22.341.643</td>
      <td>CLASSIC</td>
      <td>MATCHED_GAME</td>
      <td>[{'teamId': 100, 'win': 'Fail', 'firstBlood': ...</td>
      <td>[{'participantId': 1, 'teamId': 100, 'champion...</td>
      <td>[{'participantId': 1, 'player': {'platformId':...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4766870521</td>
      <td>KR</td>
      <td>1604707101240</td>
      <td>1287</td>
      <td>420</td>
      <td>11</td>
      <td>13</td>
      <td>10.22.341.643</td>
      <td>CLASSIC</td>
      <td>MATCHED_GAME</td>
      <td>[{'teamId': 100, 'win': 'Win', 'firstBlood': F...</td>
      <td>[{'participantId': 1, 'teamId': 100, 'champion...</td>
      <td>[{'participantId': 1, 'player': {'platformId':...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>21348</th>
      <td>4572984192</td>
      <td>KR</td>
      <td>1597249145611</td>
      <td>1648</td>
      <td>420</td>
      <td>11</td>
      <td>13</td>
      <td>10.16.330.9186</td>
      <td>CLASSIC</td>
      <td>MATCHED_GAME</td>
      <td>[{'teamId': 100, 'win': 'Win', 'firstBlood': T...</td>
      <td>[{'participantId': 1, 'teamId': 100, 'champion...</td>
      <td>[{'participantId': 1, 'player': {'platformId':...</td>
    </tr>
    <tr>
      <th>21349</th>
      <td>4572902272</td>
      <td>KR</td>
      <td>1597244549656</td>
      <td>1316</td>
      <td>420</td>
      <td>11</td>
      <td>13</td>
      <td>10.16.330.9186</td>
      <td>CLASSIC</td>
      <td>MATCHED_GAME</td>
      <td>[{'teamId': 100, 'win': 'Fail', 'firstBlood': ...</td>
      <td>[{'participantId': 1, 'teamId': 100, 'champion...</td>
      <td>[{'participantId': 1, 'player': {'platformId':...</td>
    </tr>
    <tr>
      <th>21350</th>
      <td>4572815236</td>
      <td>KR</td>
      <td>1597242450200</td>
      <td>1751</td>
      <td>420</td>
      <td>11</td>
      <td>13</td>
      <td>10.16.330.9186</td>
      <td>CLASSIC</td>
      <td>MATCHED_GAME</td>
      <td>[{'teamId': 100, 'win': 'Win', 'firstBlood': F...</td>
      <td>[{'participantId': 1, 'teamId': 100, 'champion...</td>
      <td>[{'participantId': 1, 'player': {'platformId':...</td>
    </tr>
    <tr>
      <th>21351</th>
      <td>4572697671</td>
      <td>KR</td>
      <td>1597240316310</td>
      <td>1745</td>
      <td>420</td>
      <td>11</td>
      <td>13</td>
      <td>10.16.330.9186</td>
      <td>CLASSIC</td>
      <td>MATCHED_GAME</td>
      <td>[{'teamId': 100, 'win': 'Win', 'firstBlood': F...</td>
      <td>[{'participantId': 1, 'teamId': 100, 'champion...</td>
      <td>[{'participantId': 1, 'player': {'platformId':...</td>
    </tr>
    <tr>
      <th>21352</th>
      <td>4569120544</td>
      <td>KR</td>
      <td>1597111927527</td>
      <td>1187</td>
      <td>440</td>
      <td>11</td>
      <td>13</td>
      <td>10.16.330.9186</td>
      <td>CLASSIC</td>
      <td>MATCHED_GAME</td>
      <td>[{'teamId': 100, 'win': 'Fail', 'firstBlood': ...</td>
      <td>[{'participantId': 1, 'teamId': 100, 'champion...</td>
      <td>[{'participantId': 1, 'player': {'platformId':...</td>
    </tr>
  </tbody>
</table>
<p>21353 rows × 13 columns</p>
</div>



패치 마다 승률이 바뀌는 경우들이 많이 있으나 좀 더 개괄적인 챔프들의 시너지를 보기 위해서 데이터를 많이 필요로 한다고 생각했다.\
만약 시간이 더 많았다면 훨씬 더 많은 데이터들을 riot 서버로 부터 들고와서 패치 기준으로 데이터들을 보면 더 정확했겠지만 시간 관계상 주어진 데이터내에서 조금더 많은 데이터를 확보하기 위해서 시즌을 기준점으로 잡고 분석을 진행했다.

### 2. 데이터 정제

챔피언 조합의 티어를 계산하기 위해서는 매 게임당 미드와 정글 챔피언의 승률, 밴률, 픽률을 알아내야 하기에 이에 대한 작업을 실시 하였다.

#### 2.1 밴 챔프 확인


```python
# # 매치 데이터에서 teams, participants, participantIdentities가 최종적으로 원하는 데이터이다.

# match_teams_df = pd.DataFrame()
# for i in range(len(match_df)):
#     temp_df = pd.DataFrame(match_df['teams'].iloc[i]) # teams 데이터를 2행 짜리 데이터프레임으로 변환
#     temp_df['gameId'] = match_df['gameId'].iloc[i] # teams 데이터에 각 게임의 gameId 추가 (2행 마다 같은 값)
#      # teams 데이터에 있는 bans 데이터를 5개의 변수로 추가한다
#     ban_dict = {i: pd.DataFrame(temp_df['bans'][i]).iloc[:, 0] for i in range(2)} 
#     # 각 팀의 밴픽을 저장
#     temp_ban = pd.DataFrame(ban_dict).T
#     temp_ban.columns = [f'ban{i}' for i in range(1, 6)] # 열 이름 변경
#     temp_df = pd.concat([temp_df, temp_ban], axis=1)

#     match_teams_df = pd.concat([match_teams_df, temp_df])

# match_teams_df.to_csv('MatchTeamsData.csv')

match_teams_df = pd.read_csv('MatchTeamsData.csv', index_col=0).reset_index()
match_teams_df.drop('bans', axis=1, inplace=True)
match_teams_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>teamId</th>
      <th>win</th>
      <th>firstBlood</th>
      <th>firstTower</th>
      <th>firstInhibitor</th>
      <th>firstBaron</th>
      <th>firstDragon</th>
      <th>firstRiftHerald</th>
      <th>towerKills</th>
      <th>inhibitorKills</th>
      <th>baronKills</th>
      <th>dragonKills</th>
      <th>vilemawKills</th>
      <th>riftHeraldKills</th>
      <th>dominionVictoryScore</th>
      <th>gameId</th>
      <th>ban1</th>
      <th>ban2</th>
      <th>ban3</th>
      <th>ban4</th>
      <th>ban5</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>100</td>
      <td>Win</td>
      <td>False</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>11</td>
      <td>3</td>
      <td>1</td>
      <td>5</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>4775472728</td>
      <td>117</td>
      <td>147</td>
      <td>58</td>
      <td>11</td>
      <td>157</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>200</td>
      <td>Fail</td>
      <td>True</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4775472728</td>
      <td>147</td>
      <td>876</td>
      <td>80</td>
      <td>84</td>
      <td>122</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>100</td>
      <td>Win</td>
      <td>False</td>
      <td>True</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>4775291889</td>
      <td>17</td>
      <td>8</td>
      <td>25</td>
      <td>147</td>
      <td>360</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>200</td>
      <td>Fail</td>
      <td>True</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
      <td>False</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4775291889</td>
      <td>82</td>
      <td>80</td>
      <td>157</td>
      <td>360</td>
      <td>164</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>100</td>
      <td>Win</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>False</td>
      <td>True</td>
      <td>True</td>
      <td>7</td>
      <td>1</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>4772178774</td>
      <td>360</td>
      <td>420</td>
      <td>79</td>
      <td>142</td>
      <td>80</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>42701</th>
      <td>1</td>
      <td>200</td>
      <td>Fail</td>
      <td>True</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
      <td>True</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>4572815236</td>
      <td>58</td>
      <td>17</td>
      <td>777</td>
      <td>25</td>
      <td>245</td>
    </tr>
    <tr>
      <th>42702</th>
      <td>0</td>
      <td>100</td>
      <td>Win</td>
      <td>False</td>
      <td>True</td>
      <td>True</td>
      <td>False</td>
      <td>True</td>
      <td>True</td>
      <td>11</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>4572697671</td>
      <td>157</td>
      <td>7</td>
      <td>86</td>
      <td>777</td>
      <td>51</td>
    </tr>
    <tr>
      <th>42703</th>
      <td>1</td>
      <td>200</td>
      <td>Fail</td>
      <td>True</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4572697671</td>
      <td>-1</td>
      <td>114</td>
      <td>51</td>
      <td>91</td>
      <td>53</td>
    </tr>
    <tr>
      <th>42704</th>
      <td>0</td>
      <td>100</td>
      <td>Fail</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>False</td>
      <td>True</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>4569120544</td>
      <td>51</td>
      <td>22</td>
      <td>67</td>
      <td>58</td>
      <td>2</td>
    </tr>
    <tr>
      <th>42705</th>
      <td>1</td>
      <td>200</td>
      <td>Win</td>
      <td>True</td>
      <td>True</td>
      <td>True</td>
      <td>False</td>
      <td>True</td>
      <td>False</td>
      <td>6</td>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>4569120544</td>
      <td>238</td>
      <td>164</td>
      <td>25</td>
      <td>58</td>
      <td>245</td>
    </tr>
  </tbody>
</table>
<p>42706 rows × 22 columns</p>
</div>



#### 2.2 승/패 챔프 확인 


```python
# #빈 데이터 프레임 만듦
# match_part_df = pd.DataFrame()

# #lane이랑 win 데이터 뽑아서 미드 정글만 남김.
# for i in range(len(match_df)):
#     a = pd.DataFrame(match_df['participants'].iloc[i])
#     a['gameId'] = match_df['gameId'].iloc[i]
#     extract_column(a, 'timeline', 'lane')
#     extract_column(a, 'stats', 'win')
#     extract_column(a, 'stats', 'neutralMinionsKilled')
#     a = a.loc[(a['lane'] == 'MIDDLE') | (a['lane'] == 'JUNGLE')]
#     match_part_df = pd.concat([match_part_df, a])

# match_part_df.to_csv('MatchParticipantsData.csv')

match_part_df = pd.read_csv('MatchParticipantsData.csv', index_col = 0)
match_part_df.drop(['stats', 'timeline', 'highestAchievedSeasonTier'], axis = 1, inplace = True)
match_part_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>participantId</th>
      <th>teamId</th>
      <th>championId</th>
      <th>spell1Id</th>
      <th>spell2Id</th>
      <th>gameId</th>
      <th>lane</th>
      <th>win</th>
      <th>neutralMinionsKilled</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>100</td>
      <td>104</td>
      <td>11</td>
      <td>4</td>
      <td>4775472728</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>150</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>100</td>
      <td>4</td>
      <td>14</td>
      <td>4</td>
      <td>4775472728</td>
      <td>MIDDLE</td>
      <td>True</td>
      <td>24</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>200</td>
      <td>64</td>
      <td>11</td>
      <td>4</td>
      <td>4775472728</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>116</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>200</td>
      <td>105</td>
      <td>14</td>
      <td>4</td>
      <td>4775472728</td>
      <td>MIDDLE</td>
      <td>False</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>100</td>
      <td>64</td>
      <td>11</td>
      <td>4</td>
      <td>4772178774</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>123</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>200</td>
      <td>76</td>
      <td>11</td>
      <td>4</td>
      <td>4572815236</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>151</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>100</td>
      <td>245</td>
      <td>11</td>
      <td>4</td>
      <td>4572697671</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>69</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>100</td>
      <td>142</td>
      <td>14</td>
      <td>4</td>
      <td>4572697671</td>
      <td>MIDDLE</td>
      <td>True</td>
      <td>6</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>200</td>
      <td>517</td>
      <td>1</td>
      <td>4</td>
      <td>4572697671</td>
      <td>MIDDLE</td>
      <td>False</td>
      <td>12</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>200</td>
      <td>104</td>
      <td>11</td>
      <td>4</td>
      <td>4572697671</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>168</td>
    </tr>
  </tbody>
</table>
<p>79036 rows × 9 columns</p>
</div>



게임별 챔피언들의 라인과 승, 패 등 필요한 정보들이 모두 담겨 있으나 여기에 명시된 라인들이 진짜로 맞는건지 확인하기 위해 승패의 갯수를 확인 하였다. \
리그오브레전드에서는 각 팀당 정글과 미드 라이너가 1명씩 존재해야하기 때문에 한 게임당 2명의 미드라이너와 2명의 정글러 총 4명의 정보가 존재해야 하는데 만약 이것 보다 작거나 많다면 문제가 있는 것이라 판단되기에 확인을 해 보았다.




```python
#총 17991판의 게임이 있다 
check = pd.DataFrame(match_part_df['gameId'].value_counts()).reset_index()
check.shape
```




    (17991, 2)




```python
#그중 7754개가 정글 미드 총합이 4가 아니다
uncertain_df = check.loc[check['gameId'] != 4]
uncertain_df.rename({'index': 'gameId', 'gameId':'counts'}, axis = 'columns', inplace = True)
uncertain_df.shape
```




    (7754, 2)



확인해본 결과 17991판 중 7754판은 정글과 미드의 총합이 4개가 아니였다. \
어떤 판은 1, 2개인 경우도 있고 많은 경우는 8, 9개 인 판도 있어 그냥 보고 정글과 미드를 구별하기는 어려워 보인다.\
그래서 꽤 많은 데이터 이기는 하지만 시간상의 문제도 있으며 10000개 정도의 데이터 만으로도 충분히 결과를 도출 해 낼 수 있을 것 같아 모두 지웠다. 

만약 시간이 있었다면 2가지 정도의 확실한 방법으로 미드와 정글을 구별 할 수 있을 것 같은데
1. 타임라인에 따라 10분 이내에 미드 라인 쪽에 스탬프가 많이 찍히면 미드, 정글 내에서 스탬프가 많이 찍히면 정글이라고 판단하기\
2. 미드나 정글의 제약 조건을 걸어 두 라인을 확인.ex) 정글 챔프와 미드 챔프 별로 championId를 나누고 정글은 강타를 들어야 하며 정글몹 몇마리 이상 등 등

추후에 있을 다음번 프로젝트에서는 좀 더 시간을 가지고 확실한 방법으로 라인을 구별 해 볼 예정이다.


```python
# 미드 정글 총합이 4개가 아닌 정보는 모두 제외
match_part_df = pd.merge(match_part_df, uncertain_df, how = 'left')
match_part_df = match_part_df.loc[match_part_df['counts'].isnull()]
match_part_df.reset_index(drop = True, inplace = True)
match_part_df.drop(['counts'], axis = 1, inplace = True)
match_part_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>participantId</th>
      <th>teamId</th>
      <th>championId</th>
      <th>spell1Id</th>
      <th>spell2Id</th>
      <th>gameId</th>
      <th>lane</th>
      <th>win</th>
      <th>neutralMinionsKilled</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>100</td>
      <td>104</td>
      <td>11</td>
      <td>4</td>
      <td>4775472728</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>150</td>
    </tr>
    <tr>
      <th>1</th>
      <td>5</td>
      <td>100</td>
      <td>4</td>
      <td>14</td>
      <td>4</td>
      <td>4775472728</td>
      <td>MIDDLE</td>
      <td>True</td>
      <td>24</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6</td>
      <td>200</td>
      <td>64</td>
      <td>11</td>
      <td>4</td>
      <td>4775472728</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>116</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10</td>
      <td>200</td>
      <td>105</td>
      <td>14</td>
      <td>4</td>
      <td>4775472728</td>
      <td>MIDDLE</td>
      <td>False</td>
      <td>5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3</td>
      <td>100</td>
      <td>154</td>
      <td>11</td>
      <td>4</td>
      <td>4766871878</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>95</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>40943</th>
      <td>10</td>
      <td>200</td>
      <td>76</td>
      <td>11</td>
      <td>4</td>
      <td>4572815236</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>151</td>
    </tr>
    <tr>
      <th>40944</th>
      <td>2</td>
      <td>100</td>
      <td>245</td>
      <td>11</td>
      <td>4</td>
      <td>4572697671</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>69</td>
    </tr>
    <tr>
      <th>40945</th>
      <td>5</td>
      <td>100</td>
      <td>142</td>
      <td>14</td>
      <td>4</td>
      <td>4572697671</td>
      <td>MIDDLE</td>
      <td>True</td>
      <td>6</td>
    </tr>
    <tr>
      <th>40946</th>
      <td>9</td>
      <td>200</td>
      <td>517</td>
      <td>1</td>
      <td>4</td>
      <td>4572697671</td>
      <td>MIDDLE</td>
      <td>False</td>
      <td>12</td>
    </tr>
    <tr>
      <th>40947</th>
      <td>10</td>
      <td>200</td>
      <td>104</td>
      <td>11</td>
      <td>4</td>
      <td>4572697671</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>168</td>
    </tr>
  </tbody>
</table>
<p>40948 rows × 9 columns</p>
</div>



제외 하고 난 후 또 확인해 봐야 할 것이 매 게임당 미드 2명 정글 2명이 있는지이다. 물론 한 팀에서 미드를 2명이서 가서 미드가 3명으로 집계될 수는 있으나 그런 판들은 정상적인 판이라고 보기는 어렵다. 이 분석에서는 정상적인 판에서의 미드와 정글 듀오 조합을 보는 것이기 때문에 매판마다 미드와 정글이 각 각 2명씩 있어야 한다.


```python
match_part_df['lane'].value_counts()
```




    JUNGLE    20822
    MIDDLE    20126
    Name: lane, dtype: int64



정글과 미드 숫자가 동일하지 않다.\
정상적인 게임이라면 정글러들은 강타를 들어야 한다 그래서 강타를 든 챔피언이 정글이라고 볼 수 있겠지만 잠시동안 미드에서 강타를 들고 플레이하는 메타가 있었기에 정글 몹을 잡은 갯수도 함께 고려 하기로 했다.


```python
#df_jg = 정글
df_jg = match_part_df.loc[match_part_df['lane'] == 'JUNGLE']
df_jg.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>participantId</th>
      <th>teamId</th>
      <th>championId</th>
      <th>spell1Id</th>
      <th>spell2Id</th>
      <th>gameId</th>
      <th>neutralMinionsKilled</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>20822.000000</td>
      <td>20822.000000</td>
      <td>20822.000000</td>
      <td>20822.000000</td>
      <td>20822.000000</td>
      <td>2.082200e+04</td>
      <td>20822.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>5.486168</td>
      <td>149.961579</td>
      <td>147.820382</td>
      <td>9.555470</td>
      <td>5.749448</td>
      <td>4.647829e+09</td>
      <td>123.280809</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2.884065</td>
      <td>50.001186</td>
      <td>192.322352</td>
      <td>2.900603</td>
      <td>3.080199</td>
      <td>1.474464e+08</td>
      <td>36.726744</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>100.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>3.567652e+09</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>3.000000</td>
      <td>100.000000</td>
      <td>59.000000</td>
      <td>11.000000</td>
      <td>4.000000</td>
      <td>4.567308e+09</td>
      <td>101.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>5.000000</td>
      <td>100.000000</td>
      <td>84.000000</td>
      <td>11.000000</td>
      <td>4.000000</td>
      <td>4.697078e+09</td>
      <td>122.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>8.000000</td>
      <td>200.000000</td>
      <td>141.000000</td>
      <td>11.000000</td>
      <td>6.000000</td>
      <td>4.758533e+09</td>
      <td>146.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>10.000000</td>
      <td>200.000000</td>
      <td>876.000000</td>
      <td>21.000000</td>
      <td>21.000000</td>
      <td>4.798143e+09</td>
      <td>332.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#df_mid = 미드
df_mid = match_part_df.loc[match_part_df['lane'] == 'MIDDLE']
df_mid.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>participantId</th>
      <th>teamId</th>
      <th>championId</th>
      <th>spell1Id</th>
      <th>spell2Id</th>
      <th>gameId</th>
      <th>neutralMinionsKilled</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>20126.000000</td>
      <td>20126.000000</td>
      <td>20126.000000</td>
      <td>20126.000000</td>
      <td>20126.000000</td>
      <td>2.012600e+04</td>
      <td>20126.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>5.492000</td>
      <td>149.607473</td>
      <td>168.482709</td>
      <td>10.656266</td>
      <td>6.254199</td>
      <td>4.648302e+09</td>
      <td>11.732933</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2.870542</td>
      <td>49.999701</td>
      <td>194.152013</td>
      <td>4.659561</td>
      <td>4.115014</td>
      <td>1.467006e+08</td>
      <td>12.422083</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>100.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>3.567652e+09</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>3.000000</td>
      <td>100.000000</td>
      <td>55.000000</td>
      <td>4.000000</td>
      <td>4.000000</td>
      <td>4.567889e+09</td>
      <td>4.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>5.000000</td>
      <td>100.000000</td>
      <td>103.000000</td>
      <td>12.000000</td>
      <td>4.000000</td>
      <td>4.697199e+09</td>
      <td>8.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>8.000000</td>
      <td>200.000000</td>
      <td>236.000000</td>
      <td>14.000000</td>
      <td>4.000000</td>
      <td>4.758459e+09</td>
      <td>16.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>10.000000</td>
      <td>200.000000</td>
      <td>876.000000</td>
      <td>21.000000</td>
      <td>21.000000</td>
      <td>4.798143e+09</td>
      <td>250.000000</td>
    </tr>
  </tbody>
</table>
</div>



미드와 정글로 나누어서 데이터를 살펴본 결과 정글러의 정글몹 25%가 101개고 미드의 75%가 16개 이다. 확연하게 차이가 나는 것으로 보아 대부분의 경우는 이 것으로 나뉠 수 있다고 판단했다. 그래서 101과 16의 중간인 58개를 기준으로 판별해 보았다.

더 확실한 방법으로는 정상적인 게임이라고 판단되는 기준 시간인 15분에서 20분 이 전에 끝나버린 게임들은 날려버리고\
남은 게임 데이터들 내에서 15분 내에 먹은 정글 몹의 상위 75%를 기준으로 정글과 미드를 구분하면 더 정확한 결과가 나올 것이라고 생각한다.


```python
#JUNGLE 중 미니언을 58개 이하 먹었다 + 스펠이 강타가 아니다 -> 정글 아웃
idx_nm_1 = df_jg[df_jg['neutralMinionsKilled'] <= 58].index
idx_nm_2 = df_jg.loc[(df_jg['spell1Id'] != 11) & (df_jg['spell2Id'] != 11)].index
df_jg.drop(idx_nm_1, inplace = True)
df_jg = df_jg.drop(idx_nm_2).reset_index(drop = True)
df_jg
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>participantId</th>
      <th>teamId</th>
      <th>championId</th>
      <th>spell1Id</th>
      <th>spell2Id</th>
      <th>gameId</th>
      <th>lane</th>
      <th>win</th>
      <th>neutralMinionsKilled</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>100</td>
      <td>104</td>
      <td>11</td>
      <td>4</td>
      <td>4775472728</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>150</td>
    </tr>
    <tr>
      <th>1</th>
      <td>6</td>
      <td>200</td>
      <td>64</td>
      <td>11</td>
      <td>4</td>
      <td>4775472728</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>116</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>100</td>
      <td>154</td>
      <td>11</td>
      <td>4</td>
      <td>4766871878</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>95</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7</td>
      <td>200</td>
      <td>60</td>
      <td>4</td>
      <td>11</td>
      <td>4766871878</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>147</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3</td>
      <td>100</td>
      <td>64</td>
      <td>11</td>
      <td>4</td>
      <td>4766870521</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>81</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>20172</th>
      <td>6</td>
      <td>200</td>
      <td>141</td>
      <td>11</td>
      <td>4</td>
      <td>4572902272</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>101</td>
    </tr>
    <tr>
      <th>20173</th>
      <td>2</td>
      <td>100</td>
      <td>517</td>
      <td>11</td>
      <td>4</td>
      <td>4572815236</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>84</td>
    </tr>
    <tr>
      <th>20174</th>
      <td>10</td>
      <td>200</td>
      <td>76</td>
      <td>11</td>
      <td>4</td>
      <td>4572815236</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>151</td>
    </tr>
    <tr>
      <th>20175</th>
      <td>2</td>
      <td>100</td>
      <td>245</td>
      <td>11</td>
      <td>4</td>
      <td>4572697671</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>69</td>
    </tr>
    <tr>
      <th>20176</th>
      <td>10</td>
      <td>200</td>
      <td>104</td>
      <td>11</td>
      <td>4</td>
      <td>4572697671</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>168</td>
    </tr>
  </tbody>
</table>
<p>20177 rows × 9 columns</p>
</div>




```python
a = pd.merge(df_jg, df_mid, on = ['gameId', 'win', 'teamId'])
a['win'].value_counts()
```




    True     9945
    False    9834
    Name: win, dtype: int64



그래도 짝이 안맞다. 그래서 안맞는 나머지 애들은 그냥 삭제했다.


```python
#게임당 정글 2개 데이터
df_jg_only2 = pd.DataFrame(df_jg['gameId'].value_counts())
df_jg_only2.reset_index(inplace = True)

#게임당 미드 2개 데이터
df_mid_only2 = pd.DataFrame(df_mid['gameId'].value_counts())
df_mid_only2.reset_index(inplace = True)
df_mid_only2 = df_mid_only2.loc[df_mid_only2['gameId'] == 2]

#두 DF 합쳐서 기존 데이터와 합치려고 컬럼명 변경
df_only2 = pd.merge(df_jg_only2, df_mid_only2)
df_only2 = df_only2.rename({'index' : 'gameId', 'gameId' : 'counts'}, axis = 'columns')
#gameId로 merge
df_part = pd.merge(match_part_df, df_only2, on = 'gameId')
df_part
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>participantId</th>
      <th>teamId</th>
      <th>championId</th>
      <th>spell1Id</th>
      <th>spell2Id</th>
      <th>gameId</th>
      <th>lane</th>
      <th>win</th>
      <th>neutralMinionsKilled</th>
      <th>counts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>100</td>
      <td>104</td>
      <td>11</td>
      <td>4</td>
      <td>4775472728</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>150</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>5</td>
      <td>100</td>
      <td>4</td>
      <td>14</td>
      <td>4</td>
      <td>4775472728</td>
      <td>MIDDLE</td>
      <td>True</td>
      <td>24</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6</td>
      <td>200</td>
      <td>64</td>
      <td>11</td>
      <td>4</td>
      <td>4775472728</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>116</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10</td>
      <td>200</td>
      <td>105</td>
      <td>14</td>
      <td>4</td>
      <td>4775472728</td>
      <td>MIDDLE</td>
      <td>False</td>
      <td>5</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3</td>
      <td>100</td>
      <td>154</td>
      <td>11</td>
      <td>4</td>
      <td>4766871878</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>95</td>
      <td>2</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>38147</th>
      <td>10</td>
      <td>200</td>
      <td>76</td>
      <td>11</td>
      <td>4</td>
      <td>4572815236</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>151</td>
      <td>2</td>
    </tr>
    <tr>
      <th>38148</th>
      <td>2</td>
      <td>100</td>
      <td>245</td>
      <td>11</td>
      <td>4</td>
      <td>4572697671</td>
      <td>JUNGLE</td>
      <td>True</td>
      <td>69</td>
      <td>2</td>
    </tr>
    <tr>
      <th>38149</th>
      <td>5</td>
      <td>100</td>
      <td>142</td>
      <td>14</td>
      <td>4</td>
      <td>4572697671</td>
      <td>MIDDLE</td>
      <td>True</td>
      <td>6</td>
      <td>2</td>
    </tr>
    <tr>
      <th>38150</th>
      <td>9</td>
      <td>200</td>
      <td>517</td>
      <td>1</td>
      <td>4</td>
      <td>4572697671</td>
      <td>MIDDLE</td>
      <td>False</td>
      <td>12</td>
      <td>2</td>
    </tr>
    <tr>
      <th>38151</th>
      <td>10</td>
      <td>200</td>
      <td>104</td>
      <td>11</td>
      <td>4</td>
      <td>4572697671</td>
      <td>JUNGLE</td>
      <td>False</td>
      <td>168</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
<p>38152 rows × 10 columns</p>
</div>




```python
#챔피언들을 한 행에 합치기 위해 정글 미드 두 DF로 나눠서 merge한 후 필요한 데이터만 남김.
df_jg_temp = df_part.loc[df_temp['lane'] == 'JUNGLE']
df_mid_temp = df_part.loc[df_temp['lane'] == 'MIDDLE']
df_win = pd.merge(df_jg_temp, df_mid_temp, on = ['gameId', 'win'])
df_win = df_win.loc[:, ['championId_x', 'championId_y', 'win']]
df_win
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>championId_x</th>
      <th>championId_y</th>
      <th>win</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>104</td>
      <td>4</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>64</td>
      <td>105</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2</th>
      <td>154</td>
      <td>245</td>
      <td>False</td>
    </tr>
    <tr>
      <th>3</th>
      <td>60</td>
      <td>20</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>64</td>
      <td>69</td>
      <td>True</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>19071</th>
      <td>141</td>
      <td>238</td>
      <td>True</td>
    </tr>
    <tr>
      <th>19072</th>
      <td>517</td>
      <td>104</td>
      <td>True</td>
    </tr>
    <tr>
      <th>19073</th>
      <td>76</td>
      <td>91</td>
      <td>False</td>
    </tr>
    <tr>
      <th>19074</th>
      <td>245</td>
      <td>142</td>
      <td>True</td>
    </tr>
    <tr>
      <th>19075</th>
      <td>104</td>
      <td>517</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
<p>19076 rows × 3 columns</p>
</div>



### 3. 승률, 픽률, 밴률 계산


```python
# #승률 계산
# df_win_rate = pd.DataFrame()
# df_win['win_rate'] = np.nan

# for i in range(len(df_win)):
#     e = df_win.loc[(df_win['championId_x'] == df_win['championId_x'][i]) & (df_win['championId_y'] == df_win['championId_y'][i])]
#     f = (e['win'].loc[e['win'] == True].count()/e['win'].count())*100
#     e['win_rate'] = f  
#     e.drop('win', axis = 1, inplace = True)
#     e = e.iloc[:1]
#     df_win_rate = pd.concat([df_win_rate, e])

# df_win_rate.to_csv('df_win_rate.csv')

df_win_rate = pd.read_csv('df_win_rate.csv', index_col = 0)
df_win_rate.reset_index(drop = True, inplace = True)
df_win_rate

```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>championId_x</th>
      <th>championId_y</th>
      <th>win_rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>104</td>
      <td>4</td>
      <td>48.235294</td>
    </tr>
    <tr>
      <th>1</th>
      <td>64</td>
      <td>105</td>
      <td>35.937500</td>
    </tr>
    <tr>
      <th>2</th>
      <td>154</td>
      <td>245</td>
      <td>66.666667</td>
    </tr>
    <tr>
      <th>3</th>
      <td>60</td>
      <td>20</td>
      <td>28.571429</td>
    </tr>
    <tr>
      <th>4</th>
      <td>64</td>
      <td>69</td>
      <td>44.736842</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>19071</th>
      <td>141</td>
      <td>238</td>
      <td>45.283019</td>
    </tr>
    <tr>
      <th>19072</th>
      <td>517</td>
      <td>104</td>
      <td>50.000000</td>
    </tr>
    <tr>
      <th>19073</th>
      <td>76</td>
      <td>91</td>
      <td>37.931034</td>
    </tr>
    <tr>
      <th>19074</th>
      <td>245</td>
      <td>142</td>
      <td>53.333333</td>
    </tr>
    <tr>
      <th>19075</th>
      <td>104</td>
      <td>517</td>
      <td>53.333333</td>
    </tr>
  </tbody>
</table>
<p>19076 rows × 3 columns</p>
</div>




```python
#픽률 계산
#판수 확인해서 = 픽률 확인.
df_pick_rate = pd.DataFrame(df_win_rate.value_counts())
df_pick_rate.reset_index(inplace = True)
df_pick_rate.rename({0:'pick_counts'}, axis = 'columns', inplace = True)

df_pick_rate['pick_rate'] = (df_pick_rate['pick_counts']/df_pick_rate['pick_counts'].count())*100
df_pick_rate['win_rate'] = df_pick_rate['win_rate']

#20판 이하로 플레이 된 애들은 큰 의미 없을 것 같아서 제외 -----> 이거 같은 경우에도 데이터 기반으로 결정을 내리는게 좋음.
df_pick_rate = df_pick_rate.loc[df_pick_rate['pick_counts'] >= 20]
df_pick_rate
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>championId_x</th>
      <th>championId_y</th>
      <th>win_rate</th>
      <th>pick_counts</th>
      <th>pick_rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>64</td>
      <td>517</td>
      <td>51.655629</td>
      <td>151</td>
      <td>4.775459</td>
    </tr>
    <tr>
      <th>1</th>
      <td>64</td>
      <td>84</td>
      <td>39.669421</td>
      <td>121</td>
      <td>3.826692</td>
    </tr>
    <tr>
      <th>2</th>
      <td>64</td>
      <td>7</td>
      <td>48.717949</td>
      <td>117</td>
      <td>3.700190</td>
    </tr>
    <tr>
      <th>3</th>
      <td>64</td>
      <td>238</td>
      <td>50.000000</td>
      <td>112</td>
      <td>3.542062</td>
    </tr>
    <tr>
      <th>4</th>
      <td>64</td>
      <td>4</td>
      <td>39.090909</td>
      <td>110</td>
      <td>3.478811</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>203</th>
      <td>121</td>
      <td>61</td>
      <td>65.000000</td>
      <td>20</td>
      <td>0.632511</td>
    </tr>
    <tr>
      <th>204</th>
      <td>245</td>
      <td>56</td>
      <td>50.000000</td>
      <td>20</td>
      <td>0.632511</td>
    </tr>
    <tr>
      <th>205</th>
      <td>64</td>
      <td>54</td>
      <td>50.000000</td>
      <td>20</td>
      <td>0.632511</td>
    </tr>
    <tr>
      <th>206</th>
      <td>245</td>
      <td>8</td>
      <td>45.000000</td>
      <td>20</td>
      <td>0.632511</td>
    </tr>
    <tr>
      <th>207</th>
      <td>141</td>
      <td>38</td>
      <td>80.000000</td>
      <td>20</td>
      <td>0.632511</td>
    </tr>
  </tbody>
</table>
<p>208 rows × 5 columns</p>
</div>




```python
#밴률 계산
df_ban_rate = match_teams_df.loc[:, ['ban1', 'ban2', 'ban3', 'ban4', 'ban5']]
ban = pd.DataFrame()
for i in range(1,6):
    ban1 = pd.DataFrame(df_ban_rate[f'ban{i}'].value_counts())
    ban1 = ban1.reset_index()
    ban1 = ban1.rename({'index' : 'champ_name', f'ban{i}' : 'counts'}, axis = 'columns')
    ban = pd.concat([ban1, ban])

ban = ban.groupby('champ_name')[['counts']].sum()
ban = ban.sort_values(by = 'counts', ascending = False).reset_index()
ban['ban_rate'] = (ban['counts']/ban['counts'].sum())*100
ban
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>champ_name</th>
      <th>counts</th>
      <th>ban_rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>360</td>
      <td>13467</td>
      <td>6.306842</td>
    </tr>
    <tr>
      <th>1</th>
      <td>122</td>
      <td>9825</td>
      <td>4.601227</td>
    </tr>
    <tr>
      <th>2</th>
      <td>238</td>
      <td>9779</td>
      <td>4.579684</td>
    </tr>
    <tr>
      <th>3</th>
      <td>157</td>
      <td>8037</td>
      <td>3.763874</td>
    </tr>
    <tr>
      <th>4</th>
      <td>104</td>
      <td>7868</td>
      <td>3.684728</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>148</th>
      <td>136</td>
      <td>17</td>
      <td>0.007961</td>
    </tr>
    <tr>
      <th>149</th>
      <td>42</td>
      <td>14</td>
      <td>0.006556</td>
    </tr>
    <tr>
      <th>150</th>
      <td>163</td>
      <td>12</td>
      <td>0.005620</td>
    </tr>
    <tr>
      <th>151</th>
      <td>31</td>
      <td>10</td>
      <td>0.004683</td>
    </tr>
    <tr>
      <th>152</th>
      <td>115</td>
      <td>9</td>
      <td>0.004215</td>
    </tr>
  </tbody>
</table>
<p>153 rows × 3 columns</p>
</div>




```python
#승률 픽률 밴률 합친 전체 데이터 프레임
df_all_info = pd.concat([df_pick_rate, ban], axis =1)
df_all_info['champ_name'] = df_all_info['champ_name'].fillna(0)
df_all_info['champ_name'].astype(int)
df_all_info['Ban_rate'] = np.nan
for i in range(len(df_all_info)):
    a = ban.loc[ban['champ_name'] == df_all_info['championId_x'][i]]['ban_rate'].tolist()
    b = ban.loc[ban['champ_name'] == df_all_info['championId_y'][i]]['ban_rate']
    #밴률은 2가지 챔피언이 동시에 밴되는 경우로 할 경우 너무 확률이 적을 것 같아 각 챔피언당 밴률의 반 더했음.
    df_all_info['Ban_rate'][i] = (a+b)/2
df_all_info = df_all_info[['championId_x', 'championId_y', 'win_rate', 'pick_rate', 'Ban_rate', 'pick_counts']]
df_all_info
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>championId_x</th>
      <th>championId_y</th>
      <th>win_rate</th>
      <th>pick_rate</th>
      <th>Ban_rate</th>
      <th>pick_counts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>64</td>
      <td>517</td>
      <td>51.655629</td>
      <td>4.775459</td>
      <td>2.099939</td>
      <td>151</td>
    </tr>
    <tr>
      <th>1</th>
      <td>64</td>
      <td>84</td>
      <td>39.669421</td>
      <td>3.826692</td>
      <td>2.726783</td>
      <td>121</td>
    </tr>
    <tr>
      <th>2</th>
      <td>64</td>
      <td>7</td>
      <td>48.717949</td>
      <td>3.700190</td>
      <td>1.852199</td>
      <td>117</td>
    </tr>
    <tr>
      <th>3</th>
      <td>64</td>
      <td>238</td>
      <td>50.000000</td>
      <td>3.542062</td>
      <td>3.317332</td>
      <td>112</td>
    </tr>
    <tr>
      <th>4</th>
      <td>64</td>
      <td>4</td>
      <td>39.090909</td>
      <td>3.478811</td>
      <td>1.163303</td>
      <td>110</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>203</th>
      <td>121</td>
      <td>61</td>
      <td>65.000000</td>
      <td>0.632511</td>
      <td>0.340467</td>
      <td>20</td>
    </tr>
    <tr>
      <th>204</th>
      <td>245</td>
      <td>56</td>
      <td>50.000000</td>
      <td>0.632511</td>
      <td>1.264459</td>
      <td>20</td>
    </tr>
    <tr>
      <th>205</th>
      <td>64</td>
      <td>54</td>
      <td>50.000000</td>
      <td>0.632511</td>
      <td>1.447572</td>
      <td>20</td>
    </tr>
    <tr>
      <th>206</th>
      <td>245</td>
      <td>8</td>
      <td>45.000000</td>
      <td>0.632511</td>
      <td>1.318082</td>
      <td>20</td>
    </tr>
    <tr>
      <th>207</th>
      <td>141</td>
      <td>38</td>
      <td>80.000000</td>
      <td>0.632511</td>
      <td>0.654943</td>
      <td>20</td>
    </tr>
  </tbody>
</table>
<p>208 rows × 6 columns</p>
</div>



승률과 픽률은 챔프들이 동시에 픽 되었을 때의 경우의 수로 계산을 했지만\
밴률 같은 경우에는 한경기당 10개의 챔피언들이 밴 되기 때문에 동시에 밴되는 것이 의미가 없다고 판단 했다. 그래서 밴률은 두 챔피언의 밴률을 더한 값에 2를 나눠준 값으로 설정 했다.

승률과 픽률은 두 챔피언이 동시에 픽되었을때를 기준으로 함\
승률 = 두 챔피언이 동시에 픽되었을때 승리 판수/ 두 챔피언이 픽되었을 때 전체 판수\
픽률 = 두 챔피언이 픽된 판수/전체 판수\
밴률 = (X챔프 밴율 + Y 챔프 밴율)/2


### 4. 티어 계산

Score = winRate2 + pickRate2 + banRate0.1 + kdaMean1 - 40

티어 점수를 계산하는 방법이 정해진게 없는것 같아서 한 블로거가 제시한 점수 계산 법을 참고하여 가중치들을 변경해 가면서 적당해보이는 가중치를 줬다. \
승률도 중요하긴 하지만 밴률과 픽률의 절대치가 너무 적어서 승률에 너무 많은 가중치를 주면\
픽이 적게 돼도 승률만 높은것들이 티어가 너무 높아 질 것같아서 승률과 밴률에 더 높은 가중치를 줬다.

*다음 번에 할때는 데이터에 기반한 가중치 사용하자 ex)다른 사이트에서 올라온 티어 계산 점수를 보고 회귀를 이용하여 가중치 분석 


```python
#티어 계산
df_all_info['tier_score'] = df_all_info['win_rate']*1.5 +  df_all_info['pick_rate']*8 + df_all_info['Ban_rate']*8 - 40

df_all_info['tier'] = np.nan
df_all_info.loc[df_all_info['tier_score'] < 60, 'tier'] = 5
df_all_info.loc[df_all_info['tier_score'] >= 60, 'tier'] = 4
df_all_info.loc[df_all_info['tier_score'] >= 70, 'tier'] = 3
df_all_info.loc[df_all_info['tier_score'] >= 80, 'tier'] = 2
df_all_info.loc[df_all_info['tier_score'] >= 90, 'tier'] = 1
df_all_info['tier'] = df_all_info['tier'].astype(int)
df_all_info = df_all_info.sort_values(by = 'tier_score', ascending = False).reset_index(drop = True)
df_all_info
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>championId_x</th>
      <th>championId_y</th>
      <th>win_rate</th>
      <th>pick_rate</th>
      <th>Ban_rate</th>
      <th>pick_counts</th>
      <th>tier_score</th>
      <th>tier</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>76</td>
      <td>238</td>
      <td>64.285714</td>
      <td>2.656546</td>
      <td>2.536646</td>
      <td>84</td>
      <td>97.974111</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>104</td>
      <td>238</td>
      <td>53.846154</td>
      <td>2.466793</td>
      <td>4.132206</td>
      <td>78</td>
      <td>93.561226</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>64</td>
      <td>517</td>
      <td>51.655629</td>
      <td>4.775459</td>
      <td>2.099939</td>
      <td>151</td>
      <td>92.486625</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>154</td>
      <td>157</td>
      <td>65.517241</td>
      <td>1.834282</td>
      <td>2.264553</td>
      <td>58</td>
      <td>91.066543</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>141</td>
      <td>38</td>
      <td>80.000000</td>
      <td>0.632511</td>
      <td>0.654943</td>
      <td>20</td>
      <td>90.299633</td>
      <td>1</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>203</th>
      <td>120</td>
      <td>61</td>
      <td>37.500000</td>
      <td>0.759013</td>
      <td>0.329462</td>
      <td>24</td>
      <td>24.957801</td>
      <td>5</td>
    </tr>
    <tr>
      <th>204</th>
      <td>11</td>
      <td>4</td>
      <td>38.095238</td>
      <td>0.664137</td>
      <td>0.292933</td>
      <td>21</td>
      <td>24.799415</td>
      <td>5</td>
    </tr>
    <tr>
      <th>205</th>
      <td>876</td>
      <td>39</td>
      <td>28.571429</td>
      <td>0.885515</td>
      <td>1.749637</td>
      <td>28</td>
      <td>23.938363</td>
      <td>5</td>
    </tr>
    <tr>
      <th>206</th>
      <td>141</td>
      <td>61</td>
      <td>36.000000</td>
      <td>0.790639</td>
      <td>0.379572</td>
      <td>25</td>
      <td>23.361686</td>
      <td>5</td>
    </tr>
    <tr>
      <th>207</th>
      <td>120</td>
      <td>3</td>
      <td>35.000000</td>
      <td>0.632511</td>
      <td>0.462464</td>
      <td>20</td>
      <td>21.259803</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
<p>208 rows × 8 columns</p>
</div>



#### 미드 정글 티어


```python
#info_df와 merge하고 챔피언 이름을 가져오기 위해 champ_x, champ_y로 나눔
df_champ_x = df_all_info[['championId_x']]
df_champ_x = df_champ_x.rename({'championId_x' : 'key'}, axis = 'columns')
df_champ_y = df_all_info[['championId_y']]
df_champ_y = df_champ_y.rename({'championId_y' : 'key'}, axis = 'columns')
#info_df의 key 변수들을 int로 변경 + index에서 챔프 이름 빼오기
info_df = info_df.reset_index(drop = True)
info_df['key'] = info_df['key'].astype(int)

#info_df와 champ_x, y merge
df_champ_x = pd.merge(df_champ_x, info_df, how = 'left')
df_champ_x = df_champ_x.rename({'key' : 'championId_x'}, axis = 'columns')
df_champ_y =  pd.merge(df_champ_y, info_df, how = 'left')
df_champ_y = df_champ_y.rename({'key' : 'championId_y'}, axis = 'columns')

# 원래 데이터 프레임하고 합체
df_champ_x = df_champ_x[['championId_x', 'index']]
df_tier = pd.merge(df_all_info , df_champ_x, on = 'championId_x').drop_duplicates()
df_tier = df_tier.rename({'index' : 'champ_JG'}, axis = 'columns')
df_champ_y = df_champ_y[['championId_y', 'index']]
df_tier = pd.merge(df_tier , df_champ_y, on = 'championId_y').drop_duplicates()
df_tier = df_tier.rename({'index' : 'champ_MID'}, axis = 'columns')
df_tier = df_tier.sort_values(by = 'tier_score', ascending = False)
df_tier = df_tier[['champ_MID', 'champ_JG','pick_counts', 'win_rate', 'pick_rate', 'Ban_rate', 'tier_score', 'tier']]
df_tier.reset_index(drop = True, inplace = True)
df_tier.head(50)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>champ_MID</th>
      <th>champ_JG</th>
      <th>pick_counts</th>
      <th>win_rate</th>
      <th>pick_rate</th>
      <th>Ban_rate</th>
      <th>tier_score</th>
      <th>tier</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Zed</td>
      <td>Nidalee</td>
      <td>84</td>
      <td>64.285714</td>
      <td>2.656546</td>
      <td>2.536646</td>
      <td>97.974111</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Zed</td>
      <td>Graves</td>
      <td>78</td>
      <td>53.846154</td>
      <td>2.466793</td>
      <td>4.132206</td>
      <td>93.561226</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sylas</td>
      <td>LeeSin</td>
      <td>151</td>
      <td>51.655629</td>
      <td>4.775459</td>
      <td>2.099939</td>
      <td>92.486625</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Yasuo</td>
      <td>Zac</td>
      <td>58</td>
      <td>65.517241</td>
      <td>1.834282</td>
      <td>2.264553</td>
      <td>91.066543</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Kassadin</td>
      <td>Kayn</td>
      <td>20</td>
      <td>80.000000</td>
      <td>0.632511</td>
      <td>0.654943</td>
      <td>90.299633</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Yone</td>
      <td>LeeSin</td>
      <td>68</td>
      <td>60.294118</td>
      <td>2.150538</td>
      <td>2.791177</td>
      <td>89.974893</td>
      <td>2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Sylas</td>
      <td>Graves</td>
      <td>105</td>
      <td>53.333333</td>
      <td>3.320683</td>
      <td>2.914813</td>
      <td>89.883968</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Zed</td>
      <td>LeeSin</td>
      <td>112</td>
      <td>50.000000</td>
      <td>3.542062</td>
      <td>3.317332</td>
      <td>89.875156</td>
      <td>2</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Zoe</td>
      <td>Zac</td>
      <td>24</td>
      <td>79.166667</td>
      <td>0.759013</td>
      <td>0.625439</td>
      <td>89.825619</td>
      <td>2</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Yone</td>
      <td>Graves</td>
      <td>59</td>
      <td>55.932203</td>
      <td>1.865908</td>
      <td>3.606051</td>
      <td>87.673972</td>
      <td>2</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Malzahar</td>
      <td>Graves</td>
      <td>21</td>
      <td>71.428571</td>
      <td>0.664137</td>
      <td>1.893177</td>
      <td>87.601363</td>
      <td>2</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Yasuo</td>
      <td>Elise</td>
      <td>30</td>
      <td>63.333333</td>
      <td>0.948767</td>
      <td>3.103779</td>
      <td>87.420367</td>
      <td>2</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Yasuo</td>
      <td>Shyvana</td>
      <td>36</td>
      <td>66.666667</td>
      <td>1.138520</td>
      <td>2.022667</td>
      <td>85.289492</td>
      <td>2</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Leblanc</td>
      <td>Graves</td>
      <td>72</td>
      <td>56.944444</td>
      <td>2.277040</td>
      <td>2.667073</td>
      <td>84.969566</td>
      <td>2</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Diana</td>
      <td>Graves</td>
      <td>31</td>
      <td>67.741935</td>
      <td>0.980392</td>
      <td>1.929003</td>
      <td>84.888064</td>
      <td>2</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Zed</td>
      <td>Lillia</td>
      <td>56</td>
      <td>55.357143</td>
      <td>1.771031</td>
      <td>3.331616</td>
      <td>83.856892</td>
      <td>2</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Yasuo</td>
      <td>LeeSin</td>
      <td>108</td>
      <td>48.148148</td>
      <td>3.415560</td>
      <td>2.909427</td>
      <td>82.822118</td>
      <td>2</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Yasuo</td>
      <td>Khazix</td>
      <td>21</td>
      <td>66.666667</td>
      <td>0.664137</td>
      <td>2.171358</td>
      <td>82.683954</td>
      <td>2</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Talon</td>
      <td>Elise</td>
      <td>26</td>
      <td>69.230769</td>
      <td>0.822264</td>
      <td>1.420643</td>
      <td>81.789417</td>
      <td>2</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Ekko</td>
      <td>Kayn</td>
      <td>23</td>
      <td>69.565217</td>
      <td>0.727388</td>
      <td>1.447572</td>
      <td>81.747502</td>
      <td>2</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Yone</td>
      <td>Khazix</td>
      <td>21</td>
      <td>66.666667</td>
      <td>0.664137</td>
      <td>2.053107</td>
      <td>81.737951</td>
      <td>2</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Zed</td>
      <td>Karthus</td>
      <td>48</td>
      <td>58.333333</td>
      <td>1.518027</td>
      <td>2.692362</td>
      <td>81.183106</td>
      <td>2</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Fizz</td>
      <td>Graves</td>
      <td>71</td>
      <td>54.929577</td>
      <td>2.245414</td>
      <td>2.554910</td>
      <td>80.796963</td>
      <td>2</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Akali</td>
      <td>Graves</td>
      <td>102</td>
      <td>44.117647</td>
      <td>3.225806</td>
      <td>3.541657</td>
      <td>80.316177</td>
      <td>2</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Yasuo</td>
      <td>Graves</td>
      <td>65</td>
      <td>49.230769</td>
      <td>2.055661</td>
      <td>3.724301</td>
      <td>80.085850</td>
      <td>2</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Irelia</td>
      <td>Elise</td>
      <td>40</td>
      <td>62.500000</td>
      <td>1.265022</td>
      <td>1.929705</td>
      <td>79.307821</td>
      <td>3</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Yasuo</td>
      <td>Karthus</td>
      <td>22</td>
      <td>63.636364</td>
      <td>0.695762</td>
      <td>2.284457</td>
      <td>79.296295</td>
      <td>3</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Yone</td>
      <td>Elise</td>
      <td>47</td>
      <td>55.319149</td>
      <td>1.486401</td>
      <td>2.985529</td>
      <td>78.754163</td>
      <td>3</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Akali</td>
      <td>Elise</td>
      <td>29</td>
      <td>58.620690</td>
      <td>0.917141</td>
      <td>2.921135</td>
      <td>78.637245</td>
      <td>3</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Yasuo</td>
      <td>Sylas</td>
      <td>31</td>
      <td>58.064516</td>
      <td>0.980392</td>
      <td>2.954386</td>
      <td>78.574998</td>
      <td>3</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Zoe</td>
      <td>Graves</td>
      <td>61</td>
      <td>57.377049</td>
      <td>1.929159</td>
      <td>2.085187</td>
      <td>78.180341</td>
      <td>3</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Leblanc</td>
      <td>LeeSin</td>
      <td>117</td>
      <td>48.717949</td>
      <td>3.700190</td>
      <td>1.852199</td>
      <td>77.496031</td>
      <td>3</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Kassadin</td>
      <td>Graves</td>
      <td>50</td>
      <td>58.000000</td>
      <td>1.581278</td>
      <td>2.168782</td>
      <td>77.000477</td>
      <td>3</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Zed</td>
      <td>Evelynn</td>
      <td>47</td>
      <td>55.319149</td>
      <td>1.486401</td>
      <td>2.624924</td>
      <td>75.869323</td>
      <td>3</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Zed</td>
      <td>Elise</td>
      <td>77</td>
      <td>45.454545</td>
      <td>2.435168</td>
      <td>3.511685</td>
      <td>75.756635</td>
      <td>3</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Yasuo</td>
      <td>Ekko</td>
      <td>80</td>
      <td>47.500000</td>
      <td>2.530044</td>
      <td>3.000983</td>
      <td>75.498222</td>
      <td>3</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Sylas</td>
      <td>Zac</td>
      <td>44</td>
      <td>61.363636</td>
      <td>1.391524</td>
      <td>1.455065</td>
      <td>74.818168</td>
      <td>3</td>
    </tr>
    <tr>
      <th>37</th>
      <td>TwistedFate</td>
      <td>Elise</td>
      <td>33</td>
      <td>63.636364</td>
      <td>1.043643</td>
      <td>1.357655</td>
      <td>74.664929</td>
      <td>3</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Yasuo</td>
      <td>Lillia</td>
      <td>52</td>
      <td>51.923077</td>
      <td>1.644529</td>
      <td>2.923711</td>
      <td>74.430533</td>
      <td>3</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Viktor</td>
      <td>MasterYi</td>
      <td>21</td>
      <td>71.428571</td>
      <td>0.664137</td>
      <td>0.210977</td>
      <td>74.143769</td>
      <td>3</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Katarina</td>
      <td>Graves</td>
      <td>43</td>
      <td>58.139535</td>
      <td>1.359899</td>
      <td>2.004636</td>
      <td>74.125584</td>
      <td>3</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Yone</td>
      <td>Fiddlesticks</td>
      <td>26</td>
      <td>61.538462</td>
      <td>0.822264</td>
      <td>1.867185</td>
      <td>73.823287</td>
      <td>3</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Lucian</td>
      <td>Lillia</td>
      <td>41</td>
      <td>58.536585</td>
      <td>1.296648</td>
      <td>1.883108</td>
      <td>73.242922</td>
      <td>3</td>
    </tr>
    <tr>
      <th>43</th>
      <td>Yone</td>
      <td>Ekko</td>
      <td>46</td>
      <td>52.173913</td>
      <td>1.454775</td>
      <td>2.882733</td>
      <td>72.960938</td>
      <td>3</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Xerath</td>
      <td>LeeSin</td>
      <td>33</td>
      <td>63.636364</td>
      <td>1.043643</td>
      <td>1.119281</td>
      <td>72.757937</td>
      <td>3</td>
    </tr>
    <tr>
      <th>45</th>
      <td>Akali</td>
      <td>LeeSin</td>
      <td>121</td>
      <td>39.669421</td>
      <td>3.826692</td>
      <td>2.726783</td>
      <td>71.931933</td>
      <td>3</td>
    </tr>
    <tr>
      <th>46</th>
      <td>Sylas</td>
      <td>Nunu</td>
      <td>22</td>
      <td>63.636364</td>
      <td>0.695762</td>
      <td>1.349693</td>
      <td>71.818189</td>
      <td>3</td>
    </tr>
    <tr>
      <th>47</th>
      <td>Zoe</td>
      <td>LeeSin</td>
      <td>73</td>
      <td>54.794521</td>
      <td>2.308665</td>
      <td>1.270313</td>
      <td>70.823610</td>
      <td>3</td>
    </tr>
    <tr>
      <th>48</th>
      <td>Zed</td>
      <td>Fiddlesticks</td>
      <td>43</td>
      <td>53.488372</td>
      <td>1.359899</td>
      <td>2.393341</td>
      <td>70.258473</td>
      <td>3</td>
    </tr>
    <tr>
      <th>49</th>
      <td>Pantheon</td>
      <td>LeeSin</td>
      <td>28</td>
      <td>53.571429</td>
      <td>0.885515</td>
      <td>2.785089</td>
      <td>69.721977</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>



#### 승률


```python
df_winrate = df_tier.sort_values(by = 'win_rate', ascending = False)
df_winrate = df_winrate.reset_index(drop = True)
df_winrate.head(30)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>champ_MID</th>
      <th>champ_JG</th>
      <th>pick_counts</th>
      <th>win_rate</th>
      <th>pick_rate</th>
      <th>Ban_rate</th>
      <th>tier_score</th>
      <th>tier</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Kassadin</td>
      <td>Kayn</td>
      <td>20</td>
      <td>80.000000</td>
      <td>0.632511</td>
      <td>0.654943</td>
      <td>90.299633</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Zoe</td>
      <td>Zac</td>
      <td>24</td>
      <td>79.166667</td>
      <td>0.759013</td>
      <td>0.625439</td>
      <td>89.825619</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Viktor</td>
      <td>MasterYi</td>
      <td>21</td>
      <td>71.428571</td>
      <td>0.664137</td>
      <td>0.210977</td>
      <td>74.143769</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Malzahar</td>
      <td>Graves</td>
      <td>21</td>
      <td>71.428571</td>
      <td>0.664137</td>
      <td>1.893177</td>
      <td>87.601363</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Ekko</td>
      <td>Kayn</td>
      <td>23</td>
      <td>69.565217</td>
      <td>0.727388</td>
      <td>1.447572</td>
      <td>81.747502</td>
      <td>2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Talon</td>
      <td>Elise</td>
      <td>26</td>
      <td>69.230769</td>
      <td>0.822264</td>
      <td>1.420643</td>
      <td>81.789417</td>
      <td>2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Diana</td>
      <td>Graves</td>
      <td>31</td>
      <td>67.741935</td>
      <td>0.980392</td>
      <td>1.929003</td>
      <td>84.888064</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Yasuo</td>
      <td>Khazix</td>
      <td>21</td>
      <td>66.666667</td>
      <td>0.664137</td>
      <td>2.171358</td>
      <td>82.683954</td>
      <td>2</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Yasuo</td>
      <td>Shyvana</td>
      <td>36</td>
      <td>66.666667</td>
      <td>1.138520</td>
      <td>2.022667</td>
      <td>85.289492</td>
      <td>2</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Yone</td>
      <td>Khazix</td>
      <td>21</td>
      <td>66.666667</td>
      <td>0.664137</td>
      <td>2.053107</td>
      <td>81.737951</td>
      <td>2</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Orianna</td>
      <td>Nidalee</td>
      <td>21</td>
      <td>66.666667</td>
      <td>0.664137</td>
      <td>0.297850</td>
      <td>67.695896</td>
      <td>4</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Yasuo</td>
      <td>Zac</td>
      <td>58</td>
      <td>65.517241</td>
      <td>1.834282</td>
      <td>2.264553</td>
      <td>91.066543</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Orianna</td>
      <td>Khazix</td>
      <td>20</td>
      <td>65.000000</td>
      <td>0.632511</td>
      <td>0.340467</td>
      <td>65.283828</td>
      <td>4</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Galio</td>
      <td>JarvanIV</td>
      <td>20</td>
      <td>65.000000</td>
      <td>0.632511</td>
      <td>0.197396</td>
      <td>64.139258</td>
      <td>4</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Zed</td>
      <td>Nidalee</td>
      <td>84</td>
      <td>64.285714</td>
      <td>2.656546</td>
      <td>2.536646</td>
      <td>97.974111</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Xerath</td>
      <td>LeeSin</td>
      <td>33</td>
      <td>63.636364</td>
      <td>1.043643</td>
      <td>1.119281</td>
      <td>72.757937</td>
      <td>3</td>
    </tr>
    <tr>
      <th>16</th>
      <td>TwistedFate</td>
      <td>Elise</td>
      <td>33</td>
      <td>63.636364</td>
      <td>1.043643</td>
      <td>1.357655</td>
      <td>74.664929</td>
      <td>3</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Yasuo</td>
      <td>Karthus</td>
      <td>22</td>
      <td>63.636364</td>
      <td>0.695762</td>
      <td>2.284457</td>
      <td>79.296295</td>
      <td>3</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Sylas</td>
      <td>Nunu</td>
      <td>22</td>
      <td>63.636364</td>
      <td>0.695762</td>
      <td>1.349693</td>
      <td>71.818189</td>
      <td>3</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Yasuo</td>
      <td>Elise</td>
      <td>30</td>
      <td>63.333333</td>
      <td>0.948767</td>
      <td>3.103779</td>
      <td>87.420367</td>
      <td>2</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Irelia</td>
      <td>Elise</td>
      <td>40</td>
      <td>62.500000</td>
      <td>1.265022</td>
      <td>1.929705</td>
      <td>79.307821</td>
      <td>3</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Katarina</td>
      <td>Kayn</td>
      <td>29</td>
      <td>62.068966</td>
      <td>0.917141</td>
      <td>0.490798</td>
      <td>64.366957</td>
      <td>4</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Yone</td>
      <td>Fiddlesticks</td>
      <td>26</td>
      <td>61.538462</td>
      <td>0.822264</td>
      <td>1.867185</td>
      <td>73.823287</td>
      <td>3</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Sylas</td>
      <td>Zac</td>
      <td>44</td>
      <td>61.363636</td>
      <td>1.391524</td>
      <td>1.455065</td>
      <td>74.818168</td>
      <td>3</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Ahri</td>
      <td>Hecarim</td>
      <td>31</td>
      <td>61.290323</td>
      <td>0.980392</td>
      <td>0.323842</td>
      <td>62.369358</td>
      <td>4</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Taric</td>
      <td>MasterYi</td>
      <td>23</td>
      <td>60.869565</td>
      <td>0.727388</td>
      <td>0.180537</td>
      <td>58.567743</td>
      <td>5</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Yone</td>
      <td>LeeSin</td>
      <td>68</td>
      <td>60.294118</td>
      <td>2.150538</td>
      <td>2.791177</td>
      <td>89.974893</td>
      <td>2</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Katarina</td>
      <td>Hecarim</td>
      <td>25</td>
      <td>60.000000</td>
      <td>0.790639</td>
      <td>0.440687</td>
      <td>59.850611</td>
      <td>5</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Nocturne</td>
      <td>LeeSin</td>
      <td>20</td>
      <td>60.000000</td>
      <td>0.632511</td>
      <td>1.172903</td>
      <td>64.443313</td>
      <td>4</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Zoe</td>
      <td>RekSai</td>
      <td>20</td>
      <td>60.000000</td>
      <td>0.632511</td>
      <td>0.452161</td>
      <td>58.677379</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>



#### 픽률


```python
df_pickrate = df_tier.sort_values(by = 'pick_rate', ascending = False)
df_pickrate = df_pickrate.reset_index(drop = True)
df_pickrate.head(30)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>champ_MID</th>
      <th>champ_JG</th>
      <th>pick_counts</th>
      <th>win_rate</th>
      <th>pick_rate</th>
      <th>Ban_rate</th>
      <th>tier_score</th>
      <th>tier</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sylas</td>
      <td>LeeSin</td>
      <td>151</td>
      <td>51.655629</td>
      <td>4.775459</td>
      <td>2.099939</td>
      <td>92.486625</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Akali</td>
      <td>LeeSin</td>
      <td>121</td>
      <td>39.669421</td>
      <td>3.826692</td>
      <td>2.726783</td>
      <td>71.931933</td>
      <td>3</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Leblanc</td>
      <td>LeeSin</td>
      <td>117</td>
      <td>48.717949</td>
      <td>3.700190</td>
      <td>1.852199</td>
      <td>77.496031</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Zed</td>
      <td>LeeSin</td>
      <td>112</td>
      <td>50.000000</td>
      <td>3.542062</td>
      <td>3.317332</td>
      <td>89.875156</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>TwistedFate</td>
      <td>LeeSin</td>
      <td>110</td>
      <td>39.090909</td>
      <td>3.478811</td>
      <td>1.163303</td>
      <td>55.773271</td>
      <td>5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Yasuo</td>
      <td>LeeSin</td>
      <td>108</td>
      <td>48.148148</td>
      <td>3.415560</td>
      <td>2.909427</td>
      <td>82.822118</td>
      <td>2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Sylas</td>
      <td>Graves</td>
      <td>105</td>
      <td>53.333333</td>
      <td>3.320683</td>
      <td>2.914813</td>
      <td>89.883968</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Akali</td>
      <td>Graves</td>
      <td>102</td>
      <td>44.117647</td>
      <td>3.225806</td>
      <td>3.541657</td>
      <td>80.316177</td>
      <td>2</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Orianna</td>
      <td>LeeSin</td>
      <td>92</td>
      <td>39.130435</td>
      <td>2.909551</td>
      <td>1.078537</td>
      <td>50.600355</td>
      <td>5</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Zed</td>
      <td>Ekko</td>
      <td>87</td>
      <td>39.080460</td>
      <td>2.751423</td>
      <td>3.408889</td>
      <td>67.903184</td>
      <td>4</td>
    </tr>
    <tr>
      <th>10</th>
      <td>TwistedFate</td>
      <td>Graves</td>
      <td>85</td>
      <td>48.235294</td>
      <td>2.688172</td>
      <td>1.978176</td>
      <td>69.683728</td>
      <td>4</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Zed</td>
      <td>Nidalee</td>
      <td>84</td>
      <td>64.285714</td>
      <td>2.656546</td>
      <td>2.536646</td>
      <td>97.974111</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Ahri</td>
      <td>LeeSin</td>
      <td>82</td>
      <td>47.560976</td>
      <td>2.593295</td>
      <td>1.072917</td>
      <td>60.671164</td>
      <td>4</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Yasuo</td>
      <td>Ekko</td>
      <td>80</td>
      <td>47.500000</td>
      <td>2.530044</td>
      <td>3.000983</td>
      <td>75.498222</td>
      <td>3</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Zed</td>
      <td>Graves</td>
      <td>78</td>
      <td>53.846154</td>
      <td>2.466793</td>
      <td>4.132206</td>
      <td>93.561226</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Katarina</td>
      <td>LeeSin</td>
      <td>78</td>
      <td>48.717949</td>
      <td>2.466793</td>
      <td>1.189763</td>
      <td>62.329369</td>
      <td>4</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Zed</td>
      <td>Elise</td>
      <td>77</td>
      <td>45.454545</td>
      <td>2.435168</td>
      <td>3.511685</td>
      <td>75.756635</td>
      <td>3</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Orianna</td>
      <td>Graves</td>
      <td>73</td>
      <td>43.835616</td>
      <td>2.308665</td>
      <td>1.893411</td>
      <td>59.370034</td>
      <td>5</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Zoe</td>
      <td>LeeSin</td>
      <td>73</td>
      <td>54.794521</td>
      <td>2.308665</td>
      <td>1.270313</td>
      <td>70.823610</td>
      <td>3</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Leblanc</td>
      <td>Graves</td>
      <td>72</td>
      <td>56.944444</td>
      <td>2.277040</td>
      <td>2.667073</td>
      <td>84.969566</td>
      <td>2</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Ahri</td>
      <td>Graves</td>
      <td>71</td>
      <td>47.887324</td>
      <td>2.245414</td>
      <td>1.887791</td>
      <td>64.896628</td>
      <td>4</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Fizz</td>
      <td>Graves</td>
      <td>71</td>
      <td>54.929577</td>
      <td>2.245414</td>
      <td>2.554910</td>
      <td>80.796963</td>
      <td>2</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Zed</td>
      <td>Sylas</td>
      <td>71</td>
      <td>39.436620</td>
      <td>2.245414</td>
      <td>3.362291</td>
      <td>64.016572</td>
      <td>4</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Yone</td>
      <td>LeeSin</td>
      <td>68</td>
      <td>60.294118</td>
      <td>2.150538</td>
      <td>2.791177</td>
      <td>89.974893</td>
      <td>2</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Lucian</td>
      <td>LeeSin</td>
      <td>66</td>
      <td>42.424242</td>
      <td>2.087287</td>
      <td>1.868824</td>
      <td>55.285248</td>
      <td>5</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Yasuo</td>
      <td>Graves</td>
      <td>65</td>
      <td>49.230769</td>
      <td>2.055661</td>
      <td>3.724301</td>
      <td>80.085850</td>
      <td>2</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Fizz</td>
      <td>LeeSin</td>
      <td>64</td>
      <td>35.937500</td>
      <td>2.024035</td>
      <td>1.740037</td>
      <td>44.018826</td>
      <td>5</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Yone</td>
      <td>Lillia</td>
      <td>63</td>
      <td>42.857143</td>
      <td>1.992410</td>
      <td>2.805461</td>
      <td>62.668678</td>
      <td>4</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Galio</td>
      <td>LeeSin</td>
      <td>62</td>
      <td>53.225806</td>
      <td>1.960784</td>
      <td>1.211539</td>
      <td>65.217299</td>
      <td>4</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Zoe</td>
      <td>Graves</td>
      <td>61</td>
      <td>57.377049</td>
      <td>1.929159</td>
      <td>2.085187</td>
      <td>78.180341</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>



#### 밴률


```python
df_banrate = df_tier.sort_values(by = 'Ban_rate', ascending = False)
df_banrate = df_banrate.reset_index(drop = True)
df_banrate.head(30)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>champ_MID</th>
      <th>champ_JG</th>
      <th>pick_counts</th>
      <th>win_rate</th>
      <th>pick_rate</th>
      <th>Ban_rate</th>
      <th>tier_score</th>
      <th>tier</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Zed</td>
      <td>Graves</td>
      <td>78</td>
      <td>53.846154</td>
      <td>2.466793</td>
      <td>4.132206</td>
      <td>93.561226</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Yasuo</td>
      <td>Graves</td>
      <td>65</td>
      <td>49.230769</td>
      <td>2.055661</td>
      <td>3.724301</td>
      <td>80.085850</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Yone</td>
      <td>Graves</td>
      <td>59</td>
      <td>55.932203</td>
      <td>1.865908</td>
      <td>3.606051</td>
      <td>87.673972</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Pantheon</td>
      <td>Graves</td>
      <td>22</td>
      <td>45.454545</td>
      <td>0.695762</td>
      <td>3.599963</td>
      <td>62.547616</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Akali</td>
      <td>Graves</td>
      <td>102</td>
      <td>44.117647</td>
      <td>3.225806</td>
      <td>3.541657</td>
      <td>80.316177</td>
      <td>2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Zed</td>
      <td>Elise</td>
      <td>77</td>
      <td>45.454545</td>
      <td>2.435168</td>
      <td>3.511685</td>
      <td>75.756635</td>
      <td>3</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Zed</td>
      <td>Ekko</td>
      <td>87</td>
      <td>39.080460</td>
      <td>2.751423</td>
      <td>3.408889</td>
      <td>67.903184</td>
      <td>4</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Zed</td>
      <td>Sylas</td>
      <td>71</td>
      <td>39.436620</td>
      <td>2.245414</td>
      <td>3.362291</td>
      <td>64.016572</td>
      <td>4</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Zed</td>
      <td>Lillia</td>
      <td>56</td>
      <td>55.357143</td>
      <td>1.771031</td>
      <td>3.331616</td>
      <td>83.856892</td>
      <td>2</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Zed</td>
      <td>LeeSin</td>
      <td>112</td>
      <td>50.000000</td>
      <td>3.542062</td>
      <td>3.317332</td>
      <td>89.875156</td>
      <td>2</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Yasuo</td>
      <td>Elise</td>
      <td>30</td>
      <td>63.333333</td>
      <td>0.948767</td>
      <td>3.103779</td>
      <td>87.420367</td>
      <td>2</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Yasuo</td>
      <td>Ekko</td>
      <td>80</td>
      <td>47.500000</td>
      <td>2.530044</td>
      <td>3.000983</td>
      <td>75.498222</td>
      <td>3</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Yone</td>
      <td>Elise</td>
      <td>47</td>
      <td>55.319149</td>
      <td>1.486401</td>
      <td>2.985529</td>
      <td>78.754163</td>
      <td>3</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Ekko</td>
      <td>Graves</td>
      <td>52</td>
      <td>42.307692</td>
      <td>1.644529</td>
      <td>2.961411</td>
      <td>60.309053</td>
      <td>4</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Yasuo</td>
      <td>Sylas</td>
      <td>31</td>
      <td>58.064516</td>
      <td>0.980392</td>
      <td>2.954386</td>
      <td>78.574998</td>
      <td>3</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Yasuo</td>
      <td>Lillia</td>
      <td>52</td>
      <td>51.923077</td>
      <td>1.644529</td>
      <td>2.923711</td>
      <td>74.430533</td>
      <td>3</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Akali</td>
      <td>Elise</td>
      <td>29</td>
      <td>58.620690</td>
      <td>0.917141</td>
      <td>2.921135</td>
      <td>78.637245</td>
      <td>3</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Sylas</td>
      <td>Graves</td>
      <td>105</td>
      <td>53.333333</td>
      <td>3.320683</td>
      <td>2.914813</td>
      <td>89.883968</td>
      <td>2</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Yasuo</td>
      <td>LeeSin</td>
      <td>108</td>
      <td>48.148148</td>
      <td>3.415560</td>
      <td>2.909427</td>
      <td>82.822118</td>
      <td>2</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Yone</td>
      <td>Ekko</td>
      <td>46</td>
      <td>52.173913</td>
      <td>1.454775</td>
      <td>2.882733</td>
      <td>72.960938</td>
      <td>3</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Yone</td>
      <td>Sylas</td>
      <td>33</td>
      <td>36.363636</td>
      <td>1.043643</td>
      <td>2.836135</td>
      <td>45.583684</td>
      <td>5</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Akali</td>
      <td>Ekko</td>
      <td>45</td>
      <td>46.666667</td>
      <td>1.423150</td>
      <td>2.818339</td>
      <td>63.931914</td>
      <td>4</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Yone</td>
      <td>Lillia</td>
      <td>63</td>
      <td>42.857143</td>
      <td>1.992410</td>
      <td>2.805461</td>
      <td>62.668678</td>
      <td>4</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Yone</td>
      <td>LeeSin</td>
      <td>68</td>
      <td>60.294118</td>
      <td>2.150538</td>
      <td>2.791177</td>
      <td>89.974893</td>
      <td>2</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Pantheon</td>
      <td>LeeSin</td>
      <td>28</td>
      <td>53.571429</td>
      <td>0.885515</td>
      <td>2.785089</td>
      <td>69.721977</td>
      <td>4</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Akali</td>
      <td>Sylas</td>
      <td>25</td>
      <td>36.000000</td>
      <td>0.790639</td>
      <td>2.771742</td>
      <td>42.499044</td>
      <td>5</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Akali</td>
      <td>Lillia</td>
      <td>27</td>
      <td>44.444444</td>
      <td>0.853890</td>
      <td>2.741067</td>
      <td>55.426321</td>
      <td>5</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Akali</td>
      <td>LeeSin</td>
      <td>121</td>
      <td>39.669421</td>
      <td>3.826692</td>
      <td>2.726783</td>
      <td>71.931933</td>
      <td>3</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Zed</td>
      <td>Karthus</td>
      <td>48</td>
      <td>58.333333</td>
      <td>1.518027</td>
      <td>2.692362</td>
      <td>81.183106</td>
      <td>2</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Lucian</td>
      <td>Graves</td>
      <td>51</td>
      <td>41.176471</td>
      <td>1.612903</td>
      <td>2.683698</td>
      <td>56.137514</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>


