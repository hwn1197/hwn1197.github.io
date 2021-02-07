---
layout: page
title: 스타벅스 입지 분석
description: > 
 데이터 분석 실무 파이썬 - 6장 왜 우리 동네에는 스타벅스가 없을까?
hide_description: true
sitemap: false
---

## 왜 우리 동네에는 스타벅스가 없을까?

스타벅스가 어떤 전략으로 매장 입지를 선택하는지 살펴보고자 한다. 두가지 가설
1. 거주 인구가 많은 지역에 스타벅스 매장이 많이 입지해 있을 것이다.
2. 직장인이 많은 지역에 스타벅스 매장이 많이 입지해 있을 것이다.

실제 입지 분석은 매우 어렵고 복잡한 데이터 분석 기법들이 필요하지만, 여기서는 쉽게 수집할 수 있는 데이터를 이용해 시각화 하는 것만으로도 주요한 인사이트를 얻을 수 있다는 것을 보여주고자 한다.

#### 1. 데이터 수집
스타벅스 홈페이지에서 스타벅스 매장들의 정보 수집\
서울시 열린 데이터 광장 OPEN API를 이용해 인구통계 데이터(거주인구 수, 직장인구 수)를 수집


```python
#starbucks 매장 목록 데이터 수집
from selenium import webdriver
from bs4 import BeautifulSoup
import pandas as pd

#webdriver 실행 후 스타벅스의 지역별 매장 검색 화면에 접속
driver = webdriver.Chrome(r'c:\Users\hwn11\Data\chromedriver.exe')
url = 'https://www.istarbucks.co.kr/store/store_map.do?disp=locale'
driver.get(url)
```


```python
#'서울' 클릭
#'크롬 브라우저에서 해당 버튼 부분 찾은 후 우클릭 Copy에서 Coup selector 클릭'
seoul_btn = '#container > div > form > fieldset > div > section > article.find_store_cont > article > article:nth-child(4) > div.loca_step1 > div.loca_step1_cont > ul > li:nth-child(1) > a'
driver.find_element_by_css_selector(seoul_btn).click()
```


```python
#'전체' 클릭
all_btn = '#mCSB_2_container > ul > li:nth-child(1) > a'
driver.find_element_by_css_selector(all_btn).click()
```


```python
#BeautifulSoup로 HTML 파서 만들기

#driver.page_source를 통해 크롬 브루아주의 현재 화면에 나타난 웹 페이지의 HTML을 가져올 수 있다.
html = driver.page_source
#html.parser는 HTML 문법을 이해하고 웹 페이지의 정보를 분류하는 역할을 한다.
soup = BeautifulSoup(html, 'html.parser')
```


```python
#select()를 이용해 원하는 HTML 태그를 모두 찾아오기
starbucks_soup_list = soup.select('li.quickResultLstCon')
print(len(starbucks_soup_list))
```

    536
    


```python
#태그 문자열 살펴보기
starbucks_soup_list[0]
```




    <li class="quickResultLstCon" data-code="3762" data-hlytag="null" data-index="0" data-lat="37.501087" data-long="127.043069" data-name="역삼아레나빌딩" data-storecd="1509" style="background:#fff"> <strong>역삼아레나빌딩  <img alt="" class="setStoreFavBtn" data-my_siren_order_store_yn="N" data-name="역삼아레나빌딩" data-store="1509" data-yn="N" src="//image.istarbucks.co.kr/common/img/store/icon_fav_off.png"/></strong> <p class="result_details">서울특별시 강남구 언주로 425 (역삼동)<br/>1522-3232</p> <i class="pin_general">리저브 매장 2번</i></li>




```python
#스타벅스 매장 정보 샘플 확인
starbucks_store = starbucks_soup_list[0]
name = starbucks_store.select('strong')[0].text.strip()
lat = starbucks_store['data-lat'].strip()
lng = starbucks_store['data-long'].strip()
store_type = starbucks_store.select('i')[0]['class'][0][4:]
address = str(starbucks_store.select('p.result_details')[0]).split('<br/>')[0].split('>')[1]
tel = str(starbucks_store.select('p.result_details')[0]).split('<br/>')[1].split('<')[0]

print(name)
print(lat)
print(lng)
print(store_type)
print(address)
print(tel)
```

    역삼아레나빌딩
    37.501087
    127.043069
    general
    서울특별시 강남구 언주로 425 (역삼동)
    1522-3232
    


```python
#서울시 스타벅스 매장 목록 데이터 만들기
starbucks_list = []
for item in starbucks_soup_list:
    name = item.select('strong')[0].text.strip()
    lat = item['data-lat'].strip()
    lng = item['data-long'].strip()
    store_type = item.select('i')[0]['class'][0][4:]
    address = str(item.select('p.result_details')[0]).split('<br/>')[0].split('>')[1]
    tel = str(item.select('p.result_details')[0]).split('<br/>')[1].split('<')[0]
    
    starbucks_list.append([name, lat, lng, store_type, address, tel])
```


```python
starbucks_list[0:2]
```




    [['역삼아레나빌딩',
      '37.501087',
      '127.043069',
      'general',
      '서울특별시 강남구 언주로 425 (역삼동)',
      '1522-3232'],
     ['논현역사거리',
      '37.510178',
      '127.022223',
      'general',
      '서울특별시 강남구 강남대로 538 (논현동)',
      '1522-3232']]




```python
columns = ['매장명', '위도', '경도', '매장타입', '주소', '전화번호']
seoul_starbucks_df = pd.DataFrame(starbucks_list, columns = columns)
seoul_starbucks_df.head()
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
      <th>매장명</th>
      <th>위도</th>
      <th>경도</th>
      <th>매장타입</th>
      <th>주소</th>
      <th>전화번호</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>역삼아레나빌딩</td>
      <td>37.501087</td>
      <td>127.043069</td>
      <td>general</td>
      <td>서울특별시 강남구 언주로 425 (역삼동)</td>
      <td>1522-3232</td>
    </tr>
    <tr>
      <th>1</th>
      <td>논현역사거리</td>
      <td>37.510178</td>
      <td>127.022223</td>
      <td>general</td>
      <td>서울특별시 강남구 강남대로 538 (논현동)</td>
      <td>1522-3232</td>
    </tr>
    <tr>
      <th>2</th>
      <td>신사역성일빌딩</td>
      <td>37.514132</td>
      <td>127.020563</td>
      <td>general</td>
      <td>서울특별시 강남구 강남대로 584 (논현동)</td>
      <td>1522-3232</td>
    </tr>
    <tr>
      <th>3</th>
      <td>국기원사거리</td>
      <td>37.499517</td>
      <td>127.031495</td>
      <td>general</td>
      <td>서울특별시 강남구 테헤란로 125 (역삼동)</td>
      <td>1522-3232</td>
    </tr>
    <tr>
      <th>4</th>
      <td>스탈릿대치R</td>
      <td>37.494668</td>
      <td>127.062583</td>
      <td>reserve</td>
      <td>서울특별시 강남구 남부순환로 2947 (대치동)</td>
      <td>1522-3232</td>
    </tr>
  </tbody>
</table>
</div>




```python
seoul_starbucks_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 536 entries, 0 to 535
    Data columns (total 6 columns):
     #   Column  Non-Null Count  Dtype 
    ---  ------  --------------  ----- 
     0   매장명     536 non-null    object
     1   위도      536 non-null    object
     2   경도      536 non-null    object
     3   매장타입    536 non-null    object
     4   주소      536 non-null    object
     5   전화번호    536 non-null    object
    dtypes: object(6)
    memory usage: 25.2+ KB
    


```python
seoul_starbucks_df.to_excel('1_seoul_starbucks_list.xlsx', index = False)
```

#### 2. 서울 열린 데이터 광장의 OPEN API를 활용한 공공데이터 수집


```python
import requests
import pandas as pd

#서울 열린 데이터 광장 OPEN API KEY
API_key = '6c6161704b68776e3530496c77774c'

#시군구 목록을 가져오는 API 호출 URL
url = f'http://openapi.seoul.go.kr:8088/{API_key}/json/SdeTlSccoSigW/1/25/'
print(url)
```

    http://openapi.seoul.go.kr:8088/6c6161704b68776e3530496c77774c/json/SdeTlSccoSigW/1/25/
    


```python
r = requests.get(url).json()
```


```python
result_data = r['SdeTlSccoSigW']
sample_df = pd.DataFrame(result_data['row'])
sample_df.head(5)
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
      <th>OBJECTID</th>
      <th>SIG_CD</th>
      <th>SIG_KOR_NM</th>
      <th>SIG_ENG_NM</th>
      <th>ESRI_PK</th>
      <th>LAT</th>
      <th>LNG</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.0</td>
      <td>11320</td>
      <td>도봉구</td>
      <td>Dobong-gu</td>
      <td>0.0</td>
      <td>37.6658609</td>
      <td>127.0317674</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2.0</td>
      <td>11380</td>
      <td>은평구</td>
      <td>Eunpyeong-gu</td>
      <td>1.0</td>
      <td>37.6176125</td>
      <td>126.9227004</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3.0</td>
      <td>11230</td>
      <td>동대문구</td>
      <td>Dongdaemun-gu</td>
      <td>2.0</td>
      <td>37.5838012</td>
      <td>127.0507003</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4.0</td>
      <td>11590</td>
      <td>동작구</td>
      <td>Dongjak-gu</td>
      <td>3.0</td>
      <td>37.4965037</td>
      <td>126.9443073</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5.0</td>
      <td>11545</td>
      <td>금천구</td>
      <td>Geumcheon-gu</td>
      <td>4.0</td>
      <td>37.4600969</td>
      <td>126.9001546</td>
    </tr>
  </tbody>
</table>
</div>




```python
API_key = '6c6161704b68776e3530496c77774c'
#서울열린데이터 광장 OPEN API 호출을 위한 공통 함수
def seoul_open_api_data(url, service):
    data_list = None
    try:
        result_dict = requests.get(url).json()
        result_data = result_dict[service]
        code = result_data['RESULT']['CODE']
        if code == 'INFO-000':
            data_list = result_data['row']
    except:
        pass
    return data_list
```


```python
sgg_data_list = seoul_open_api_data(url, 'SdeTlSccoSigW')
sgg_data_list[1]
```




    {'OBJECTID': 2.0,
     'SIG_CD': '11380',
     'SIG_KOR_NM': '은평구',
     'SIG_ENG_NM': 'Eunpyeong-gu',
     'ESRI_PK': 1.0,
     'LAT': '37.6176125',
     'LNG': '126.9227004'}




```python
ssg_df = pd.DataFrame(sgg_data_list).drop(['OBJECTID','SIG_ENG_NM', 'ESRI_PK'], axis = 1)
ssg_df.columns = ['시군구코드', '시군구명', '위도', '경도']
ssg_df.head(3)
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
      <th>시군구코드</th>
      <th>시군구명</th>
      <th>위도</th>
      <th>경도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11320</td>
      <td>도봉구</td>
      <td>37.6658609</td>
      <td>127.0317674</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11380</td>
      <td>은평구</td>
      <td>37.6176125</td>
      <td>126.9227004</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11230</td>
      <td>동대문구</td>
      <td>37.5838012</td>
      <td>127.0507003</td>
    </tr>
  </tbody>
</table>
</div>




```python
ssg_df.to_excel('2_seoul_sgg_list.xlsx', index = False)
```

#### 시군구별 인구 데이터 가져오기


```python
pop_url = f'http://openapi.seoul.go.kr:8088/{API_key}/json/octastatapi419/1/26/'
pop_data_list = seoul_open_api_data(pop_url, 'octastatapi419')
ssg_pop_df = pd.DataFrame(data = pop_data_list)
```


```python
#데이터를 들고오려 했으나 서울 열린데이터 광장 측 서버 오류로 들고와지지 않아 해결 시 추가할 예정
# sgg_pop_df_selected = ssg_pop_df[ssg_pop_df['JACHIGU'] != '합계']
# sgg_pop_df_final = sgg_pop_df_selected['JACHIGU', 'GYE_1']
# sgg_pop_df_final.columns = ['시군구명', '주민등록인구']
# sgg_pop_df_final.head()
```


```python
sgg_pop_df_final.to_excel('2_seoul_sgg_pop.xlsx', index = False)
```

#### 서울시 시군구별 사업체 데이터


```python

biz_url = f'http://openapi.seoul.go.kr:8088/{API_key}/json/octastatapi104/1/450'
biz_data_list = seoul_open_api_data(biz_url, 'ocatastatapi104')

sgg_biz_df = pd.DataFrame(biz_data_list)
sgg_biz_df.head()
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
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
#데이터를 들고오려 했으나 서울 열린데이터 광장 측 서버 오류로 들고와지지 않아 해결 시 추가할 예정
condition = sgg_biz_df['DONG'] == '소계'
sgg_biz_df_selected  = sgg_biz_df[condition]

columns = ['JACHIGU', 'GYE', 'SAEOPCHESU_1']
sgg_biz_df_final = sgg_biz_df_selected[columns]
sgg_biz_df_final.columns = ['시군구명', '종사자수', '사업체수']
sgg_biz_df_final.reset_index(drop = True, inplace = True)
sgg_biz_df_final
```


```python
sgg_biz_df_final.to_excel('3_sgg_biz.xlsx', index = False)
```

### 데이터 통합 및 전처리


```python
#스타벅스 매장 목록이 담긴 엑셀 파일 불러오기
seoul_starbucks = pd.read_excel('1_seoul_starbucks_list.xlsx', header = 0)
seoul_starbucks. 
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
      <th>매장명</th>
      <th>위도</th>
      <th>경도</th>
      <th>매장타입</th>
      <th>주소</th>
      <th>전화번호</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>역삼아레나빌딩</td>
      <td>37.501087</td>
      <td>127.043069</td>
      <td>general</td>
      <td>서울특별시 강남구 언주로 425 (역삼동)</td>
      <td>1522-3232</td>
    </tr>
    <tr>
      <th>1</th>
      <td>논현역사거리</td>
      <td>37.510178</td>
      <td>127.022223</td>
      <td>general</td>
      <td>서울특별시 강남구 강남대로 538 (논현동)</td>
      <td>1522-3232</td>
    </tr>
    <tr>
      <th>2</th>
      <td>신사역성일빌딩</td>
      <td>37.514132</td>
      <td>127.020563</td>
      <td>general</td>
      <td>서울특별시 강남구 강남대로 584 (논현동)</td>
      <td>1522-3232</td>
    </tr>
    <tr>
      <th>3</th>
      <td>국기원사거리</td>
      <td>37.499517</td>
      <td>127.031495</td>
      <td>general</td>
      <td>서울특별시 강남구 테헤란로 125 (역삼동)</td>
      <td>1522-3232</td>
    </tr>
    <tr>
      <th>4</th>
      <td>스탈릿대치R</td>
      <td>37.494668</td>
      <td>127.062583</td>
      <td>reserve</td>
      <td>서울특별시 강남구 남부순환로 2947 (대치동)</td>
      <td>1522-3232</td>
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
      <th>531</th>
      <td>사가정역</td>
      <td>37.579594</td>
      <td>127.087966</td>
      <td>general</td>
      <td>서울특별시 중랑구 면목로 310</td>
      <td>1522-3232</td>
    </tr>
    <tr>
      <th>532</th>
      <td>상봉역</td>
      <td>37.596890</td>
      <td>127.086470</td>
      <td>general</td>
      <td>서울특별시 중랑구 망우로 307, ,3,4번지 (상봉동)</td>
      <td>1522-3232</td>
    </tr>
    <tr>
      <th>533</th>
      <td>묵동이마트</td>
      <td>37.613433</td>
      <td>127.077484</td>
      <td>general</td>
      <td>서울특별시 중랑구 동일로 932, 묵동이마트 B1층 (묵동)</td>
      <td>1522-3232</td>
    </tr>
    <tr>
      <th>534</th>
      <td>묵동</td>
      <td>37.615368</td>
      <td>127.076633</td>
      <td>general</td>
      <td>서울특별시 중랑구 동일로 952</td>
      <td>1522-3232</td>
    </tr>
    <tr>
      <th>535</th>
      <td>중화역</td>
      <td>37.601709</td>
      <td>127.078411</td>
      <td>general</td>
      <td>서울특별시 중랑구 봉화산로 35 1,2층</td>
      <td>1522-3232</td>
    </tr>
  </tbody>
</table>
<p>536 rows × 6 columns</p>
</div>




```python
#스타벅스 주소 정보에서 시군구명 추출
sgg_names = []
for address in seoul_starbucks['주소']:
    sgg = address.split()[1] #전체 내용중 2번째 단어 추출
    sgg_names.append(sgg)
seoul_starbucks['시군구명'] = sgg_names
seoul_starbucks.head()
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
      <th>매장명</th>
      <th>위도</th>
      <th>경도</th>
      <th>매장타입</th>
      <th>주소</th>
      <th>전화번호</th>
      <th>시군구명</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>역삼아레나빌딩</td>
      <td>37.501087</td>
      <td>127.043069</td>
      <td>general</td>
      <td>서울특별시 강남구 언주로 425 (역삼동)</td>
      <td>1522-3232</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>1</th>
      <td>논현역사거리</td>
      <td>37.510178</td>
      <td>127.022223</td>
      <td>general</td>
      <td>서울특별시 강남구 강남대로 538 (논현동)</td>
      <td>1522-3232</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>2</th>
      <td>신사역성일빌딩</td>
      <td>37.514132</td>
      <td>127.020563</td>
      <td>general</td>
      <td>서울특별시 강남구 강남대로 584 (논현동)</td>
      <td>1522-3232</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>3</th>
      <td>국기원사거리</td>
      <td>37.499517</td>
      <td>127.031495</td>
      <td>general</td>
      <td>서울특별시 강남구 테헤란로 125 (역삼동)</td>
      <td>1522-3232</td>
      <td>강남구</td>
    </tr>
    <tr>
      <th>4</th>
      <td>스탈릿대치R</td>
      <td>37.494668</td>
      <td>127.062583</td>
      <td>reserve</td>
      <td>서울특별시 강남구 남부순환로 2947 (대치동)</td>
      <td>1522-3232</td>
      <td>강남구</td>
    </tr>
  </tbody>
</table>
</div>




```python
seoul_sgg = pd.read_excel('2_seoul_sgg_list.xlsx')
seoul_sgg.head()
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
      <th>시군구코드</th>
      <th>시군구명</th>
      <th>위도</th>
      <th>경도</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11320</td>
      <td>도봉구</td>
      <td>37.665861</td>
      <td>127.031767</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11380</td>
      <td>은평구</td>
      <td>37.617612</td>
      <td>126.922700</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11230</td>
      <td>동대문구</td>
      <td>37.583801</td>
      <td>127.050700</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11590</td>
      <td>동작구</td>
      <td>37.496504</td>
      <td>126.944307</td>
    </tr>
    <tr>
      <th>4</th>
      <td>11545</td>
      <td>금천구</td>
      <td>37.460097</td>
      <td>126.900155</td>
    </tr>
  </tbody>
</table>
</div>




```python
#시군구별 스타벅스 매장 수 세기
starbucks_sgg_count = seoul_starbucks.pivot_table(index = '시군구명', values = '매장명',
                                                 aggfunc = 'count')
starbucks_sgg_count.rename(columns = {'매장명':'스타벅스_매장수'}).head()
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
      <th>스타벅스_매장수</th>
    </tr>
    <tr>
      <th>시군구명</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>강남구</th>
      <td>84</td>
    </tr>
    <tr>
      <th>강동구</th>
      <td>16</td>
    </tr>
    <tr>
      <th>강북구</th>
      <td>5</td>
    </tr>
    <tr>
      <th>강서구</th>
      <td>16</td>
    </tr>
    <tr>
      <th>관악구</th>
      <td>11</td>
    </tr>
  </tbody>
</table>
</div>




```python
#서울시 시군구 목록 데이터에 스타벅스 매장 수 데이터를 병합
seoul_sgg = pd.merge(seoul_sgg, starbucks_sgg_count, how = 'left', on = '시군구명')
seoul_sgg.head()
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
      <th>시군구코드</th>
      <th>시군구명</th>
      <th>위도</th>
      <th>경도</th>
      <th>매장명</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11320</td>
      <td>도봉구</td>
      <td>37.665861</td>
      <td>127.031767</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11380</td>
      <td>은평구</td>
      <td>37.617612</td>
      <td>126.922700</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11230</td>
      <td>동대문구</td>
      <td>37.583801</td>
      <td>127.050700</td>
      <td>8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11590</td>
      <td>동작구</td>
      <td>37.496504</td>
      <td>126.944307</td>
      <td>11</td>
    </tr>
    <tr>
      <th>4</th>
      <td>11545</td>
      <td>금천구</td>
      <td>37.460097</td>
      <td>126.900155</td>
      <td>11</td>
    </tr>
  </tbody>
</table>
</div>




```python
#서울시 시군구별 인구통계 & 사업체 수 통계 데이터 병합
seoul_sgg_pop = pd.read_excel('3_sgg_pop.xlsx')
seoul_sgg = pd.merge(seoul_sgg, seoul_sgg_pop, how = 'left', on = '시군구명')
seoul_sgg_biz = pd.read_excel('3_sgg_biz.xlsx')
seoul_sgg = pd.merge(seoul_sgg, seoul_sgg_biz, how = 'left', on = '시군구명')
seoul_sgg.head()
```


```python
#병합 결과를 엑셀 파일로 저장
seoul_sgg.to_excel('5_seoul_sgg_stat.xlsx', index = False)
```

## 데이터 시각화

folium을 이용한 지도가 github page에서 지원이 되지 않는다. 원한다면 코드를 그대로 따라해보면 바로 볼 수 있을 것이다.

#### 스타벅스 매장분포 시각화


```python
import pandas as pd
import folium
import json

#folium을 이용한 지도생성
starbucks_map = folium.Map(
    location = [37.553050, 127.009189], #서울시 한눈에 보이게 좌표 찍기
    tiles = 'Stamen Terrain',
    zoom_start = 11)
#starbucks_map
```


```python
seoul_starbucks.loc[seoul_starbucks.index, '경도']
```




    0      127.043069
    1      127.022223
    2      127.020563
    3      127.031495
    4      127.062583
              ...    
    531    127.087966
    532    127.086470
    533    127.077484
    534    127.076633
    535    127.078411
    Name: 경도, Length: 536, dtype: float64




```python
seoul_starbucks['경도']
```




    0      127.043069
    1      127.022223
    2      127.020563
    3      127.031495
    4      127.062583
              ...    
    531    127.087966
    532    127.086470
    533    127.077484
    534    127.076633
    535    127.078411
    Name: 경도, Length: 536, dtype: float64




```python
#지도에 스타벅스 매장 위치를 나타내는 서클 마커 그리기
#내방법
for i in range(len(seoul_starbucks)):
    lat = seoul_starbucks['위도'][i]
    lng = seoul_starbucks['경도'][i]
    
    folium.CircleMarker(
        location = [lat, lng],
        fill = True,
        fill_color = 'green',
        fill_opacity = 1,
        color = 'yellow',
        weight = 1,
        radius = 3).add_to(starbucks_map)

# starbucks_map
```


```python
#스타벅스 매장 타입별 위치 서클 마커 그리기 
starbucks_map2 =folium.Map(
    location = [37.553050, 127.009189], #서울시 한눈에 보이게 좌표 찍기
    tiles = 'Stamen Terrain',
    zoom_start = 11)
#책방법
for idx in seoul_starbucks.index:
    lat = seoul_starbucks.loc[idx, '위도']
    lng = seoul_starbucks.loc[idx, '경도']
    store_type = seoul_starbucks.loc[idx, '매장타입']
    
    #매장 타입별 색상 선택을 위한 조ㅓㄴ문
    fillColor = ''
    if store_type == 'general':
        fillColor = 'black'
        size = 1
    elif store_type == 'reserve':
        fillColor = 'blue'
        size = 4
    else:
        fillColor = 'red'
        size = 4
        
    folium.CircleMarker(
        location = [lat, lng],
        color = fillColor,
        fill = True,
        fill_color = fillColor,
        fill_opacity = 1,
        weight = 1,
        radius = size).add_to(starbucks_map2)

# starbucks_map2
```

#### 시군구별 스타벅스 매장수 시각화


```python
seoul_sgg.head()
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
      <th>시군구코드</th>
      <th>시군구명</th>
      <th>위도</th>
      <th>경도</th>
      <th>매장명</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11320</td>
      <td>도봉구</td>
      <td>37.665861</td>
      <td>127.031767</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>11380</td>
      <td>은평구</td>
      <td>37.617612</td>
      <td>126.922700</td>
      <td>8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>11230</td>
      <td>동대문구</td>
      <td>37.583801</td>
      <td>127.050700</td>
      <td>8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11590</td>
      <td>동작구</td>
      <td>37.496504</td>
      <td>126.944307</td>
      <td>11</td>
    </tr>
    <tr>
      <th>4</th>
      <td>11545</td>
      <td>금천구</td>
      <td>37.460097</td>
      <td>126.900155</td>
      <td>11</td>
    </tr>
  </tbody>
</table>
</div>




```python
#서울시 시군구 행정 경계 지도 파일 불러오기
sgg_geojson_file_path = 'seoul_sgg.geojson'
seoul_sgg_geo = json.load(open(sgg_geojson_file_path, encoding = 'utf-8'))
seoul_sgg_geo['features'][0]['properties']
```




    {'SIG_CD': '11320',
     'SIG_KOR_NM': '도봉구',
     'SIG_ENG_NM': 'Dobong-gu',
     'ESRI_PK': 0,
     'SHAPE_AREA': 0.00210990544544,
     'SHAPE_LEN': 0.239901251347}




```python
#folium 지도 생성
starbucks_bubble = folium.Map(
    location = [37.553050, 127.009189], #서울시 한눈에 보이게 좌표 찍기
    tiles = 'CartoDB dark_matter',
    zoom_start = 11)
```


```python
#서울시 시군구 경계 지도 그리기
def style_function(feature):
    return {
        'opacity' : 0.7,
        'weight' : 1,
        'color' : 'white',
        'fillOpacity' : 0,
        'dashArray' : '5, 5',
    }
folium.GeoJson(
seoul_sgg_geo,
style_function = style_function).add_to(starbucks_bubble)

# starbucks_bubble
```




    <folium.features.GeoJson at 0x25c6d48c688>




```python
#서울시 시군구별 스타벅스 평균 매장 수 계산
starbucks_mean = seoul_sgg['매장명'].mean()
starbucks_mean
```




    21.44




```python
for idx in seoul_sgg.index:
    lat = seoul_sgg.loc[idx, '위도']
    lng = seoul_sgg.loc[idx, '경도']
    count = seoul_sgg.loc[idx, '매장명']
    
    if count > starbucks_mean:
        fillColor = '#FF0000'
    else:
        fillColor = 'blue'
        
    folium.CircleMarker(
        location = [lat, lng],
        color = '#FFFF00',
        fill_color = fillColor,
        fill_opacity = 0.5,
        weight = 1.5,
        radius = count/2).add_to(starbucks_bubble)

# starbucks_bubble
```


```python
#서울시 시군구별 스타벅스 매장 수를 단계구분도로 시각화
starbucks_choropleth = folium.Map(
    location = [37.553050, 127.009189], #서울시 한눈에 보이게 좌표 찍기
    tiles = 'CartoDB dark_matter',
    zoom_start = 11)

folium.Choropleth(
    geo_data = seoul_sgg_geo,
    data = seoul_sgg,
    columns = ['시군구명', '매장명'],
    fill_color = 'YlGn',
    fill_opacity = 1,
    line_opacity = 0.7,
    key_on = 'properties.SIG_KOR_NM').add_to(starbucks_choropleth)
# starbucks_choropleth
```




    <folium.features.Choropleth at 0x25c6d7ddac8>


