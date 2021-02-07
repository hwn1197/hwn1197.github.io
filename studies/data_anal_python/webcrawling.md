---
layout: page
title: 웹 크롤링 기초
description: > 
 데이터 분석 실무 파이썬 - 2장 웹 크롤링 기초 
hide_description: true
sitemap: false
---
## 웹 크롤링 기초

selenium의 webdriver는 크롬이나 인터넷 익스플로러 등에서 사이트 접속, 버튼 클릭, 클자 입력과 같이 웹 브라우저에서 사람이 할 수 있는 일들을 코드를 통해 제어할 수 있는 라이브러리.\
webdriver를 활용하기 위해서는 사용 중인 웹 브라우저의 종류에 따라 제어하는 드라이버가 필요하다.\

#### 1) 크롬 드라이버 활용


```python
from selenium import webdriver
# google chrome에서 제공하는 webdriver 설치 후 크롬드라이버 실행
driver = webdriver.Chrome(r'C:\Users\hwn11\Data\chromedriver.exe')
```

#### 2) 웹 페이지 접속


```python
url = 'https://www.naver.com'
driver.get(url)
```

#### 3) 웹 페이지(HTML) 다운로드
웹 페이지에서 필요한 정보를 가져온다는 것은 웹 페이지의 HTML을 다운로드한 뒤, HTML에서 정보를 찾는다는 의미. 현재 크롬 브라우저가 접속한 웹 페이지의 HTML을 다운로드


```python
#웹 페이지의 HTML 다운로드
html = driver.page_source
```

#### 4) HTML 구조 살펴보기
html에서 필요한 정보가 어디에 있는지를 찾아야 한다. 이때 HTML에서 특정 정보가 존재하는 위치를 지정할 수 있어야하며, 이를 위해서 HTML을 이해하는 것이 좋다.\
HTML의 규칙
1. 시작과 끝이 있다
<태그>로 시작하고 </태그>로 끝난다. 태그 부분에 들어가는 태그명은 div, p, span, a, tr, td 등 다양하다.
2. 태그는 다른 태그에 속할 수 있다.
태그 시작 부분과 끝 부분 사이에 다른 태그가 들어 있는지로 확인 가능하다. 
3. 태그의 시작과 끝 사이에 화면에 표시되는 정보가 들어간다.
<태그> PlaywithData </태그>의 경우 화면에 PlaywithData 부분이 표시된다.
4. 태그 기호 내에 속성을 가질 수 있다.
<태그 속성 1 = 값 1 속성2 = 값2>안녕하세요</태그>에서 속성1, 속성2와 같이 여러 개의 속성 정보를 가질 수 있다.

EX) HTML 구조
<html>
    <head>
    </head>
    <body>
        <h1>우리동네시장</h1>
            <div class = 'sale'>
                <p 1d = 'fruit1' class = 'fruits'>
                    <span class = 'name'> 바나나 </span>
                    <span class = 'price'> 3000원 </span>
                    <span class = 'inventory'> 500개 </span>
                    <span class = 'store'> 가나다상회 </span>
                    <a href = 'http://bit.ly/forPlaywithdata'> 홈페이지 </a>
                </p>
            </div>
            <div class = 'prepare>
                <p id = 'fruit2' class = 'fruits'>
                    <span class = 'name'> 파인애플 </span>
                </p>                                            
           </div>                            
    </body>
</html>

#### 5) 크롬 브라우저에서 웹 페이지의 HTML 살펴보기

개발자 도구를 이용해 HTML 구조를 살펴볼 수 있음.
방법 : [도구 더보기] -> [개발자 도구] | F12 | 웹 페이지 마우스 우클릭 [검사]

개발자 도구 창이 열린 뒤 [Elements] 탭을 클릭하면 HTML 내용 확인 가능. 원하는 정보의 태그를 확인하기 위해서는 해당 영역에서 마우스 오른쪽 버튼을 클릭한 후 [검사]를 선택하면 됨.


#### 6) BeautifulSoup을 이용한 정보 찾기


```python
html = '''
<html>
    <head>
    </head>
    <body>
        <h1>우리동네시장</h1>
            <div class = 'sale'>
                <p id = 'fruit1' class = 'fruits'>
                    <span class = 'name'> 바나나 </span>
                    <span class = 'price'> 3000원 </span>
                    <span class = 'inventory'> 500개 </span>
                    <span class = 'store'> 가나다상회 </span>
                    <a href = 'http://bit.ly/forPlaywithdata'> 홈페이지 </a>
                </p>
            </div>
            <div class = 'prepare>
                <p id = 'fruit2' class = 'fruits'>
                    <span class = 'name'> 파인애플 </span>
                </p>                                            
           </div>                            
    </body>
</html>
'''
```

HTMl에서 '''은 파이썬에서 여러 줄에 걸친 문자열을 입력할 때 사용하는 기호.

웹 브라우저에서 특정 사이트에 접속한 뒤 driver.page_source를 이용해 HTML를 다운로드 했을 때 위와 같은 정보가 들어 있다고 생각하고 원하는 정보를 추출하는 작업 해보기. 내려받은 HTML은 여러 문자들이 연결된 문자열 데이터. 문자열에서 여러 개의 데이터를 찾는 것은 쉽지 않은 일이다. 여기서 이 문자열 데이터를 HTML 형식으로 읽고, 정보를 쉽게 찾을 수 있게 BeautifulSoup 라이브러리를 활용


```python
from bs4 import BeautifulSoup

#html 변수에 들어 있는 문자열 정보를 HTML 형식에 맞게 해석해서 원하는 정보를 찾을 수 있도록 준비
soup = BeautifulSoup(html, 'html.parser')
```

#### 7) HTML 정보 찾기 1 - 태그 속성 활용

BeautifulSoup 명령어인 select를 이용하면 HTML 내에서 입력한 조건을 만족하는 태그를 모두 선택할 수 있따. 조건 부분에는 해당 태그의 태그명이나 속성값을 지정하거나 태그 간의 구조를 지정할 수도 있고, 두 방법을 모두 활용할 수도 있다.


```python
#태그 명이 span인거
tags_span = soup.select('span')
#태그 명이 p인거
tags_p = soup.select('p')
```


```python
#태그명 외에 태그의 속성 중 id 값과 class명을 이용해 태그를 찾을 수도 있음. select 부분에 #id , .class
ids_fruits1 = soup.select('#fruits1')
class_price = soup.select('.price')
#태그명이 span이면서 class명이 price
tags_span_class_price = soup.select('span.price')
```

#### 8)HTML 정보 찾기 2 - 상위 구조 활용
태그의 속성만으로 찾기가 어려울 경우 어떠한 부모 태그 아래에 있는지 등의 정보를 추가해서 찾을 수 있다. 한 단계 아래를 의미할 때는 '>' 기호를 사용하며, 상위 태그는 부모 태그, 하위 태그는 자식 태그라 부름. 여러 단계 아래를 의미할 때는 띄어쓰기('')를 사용하며, 상위 태그는 부모 ㅐ그, 하위 태그는 자손 태그라고 부름.


```python
#태그 구조로 위치 찾기
tags_name = soup.select('span.name')
print(tags_name)
```

    [<span class="name"> 바나나 </span>, <span class="name"> 파인애플 </span>]
    


```python
#태그 구조로 위치 찾기 2
tags_banana1 = soup.select('#fruit1 > span.name')
print(tags_banana1)
```

    [<span class="name"> 바나나 </span>]
    


```python
#태그 구조로 위치 찾기 3
tags_banana2 = soup.select('div.sale > #fruit1 > span.name')
tags_banana3 = soup.select('div.sale span.name')
print(tags_banana2)
print(tags_banana3)
```

    [<span class="name"> 바나나 </span>]
    [<span class="name"> 바나나 </span>]
    

첫번째 줄 = 상위태그1(div.sale) 바로 아래에 있는 상위 태그 2(#fruit1)를 찾고, 상위태그2(#fruits) 바로 아래에 있는 태그 (span.name)를 찾음\
두 번째 줄 = 상위태그(div.sale)바로 아래에 있는 태그뿐 아니라 몇 단계 아래의 태그 중에서 태그 정보(span.name)를 모두 찾음.

#### 9) 정보 가져오기 1 - 태그 그룹에서 하나의 태그 선택하기

soup.select('조건') 명령을 통해 조건에 해당하는 모든 태그를 찾을 수 있으며, 그룹 형태로 결과를 확인할 수 있다.
태그 그룹에서 개별 태그에 접근하기 위해서 '인덱스 번호'를 활용하거나 '반복문'을 사용할 필요가 있다.


```python
#태그 그룹에서 하나의 태그만 선택
tags = soup.select('span.name')
tag_1 = tags[0]  #인덱스 번호로 하나의 태그 지정
print(tag_1)
```

    <span class="name"> 바나나 </span>
    


```python
#태그 그룹에서 반복문으로 태그 하나씩 선택하기
tags = soup.select('span.name')
for tag in tags: #반복문으로 태그 그룹에서 각 태그를 선택해 활용
    print(tag)
```

    <span class="name"> 바나나 </span>
    <span class="name"> 파인애플 </span>
    

#### 10) 정보 가져오기 2 - 선택한 태그에서 정보 가져오기

태그를 선택한 다음 화면에 보이는 글 부분을 가져오거나(.text) 태그 내 속성 값을 가져올 수 있다(['속성명']). 일반적으로 브라우저에 표시되는 정보를 수집하는 일이 많기에 .text 명령을 자주 활용하지만, 화면에 보이지 않는 URL 주소를 수집하기 위해서는 ['href']명령도 필요하다. (하이퍼링크는 모두 <href = 'URL주소'>형식으로 작성돼 있다.


```python
#태그에서 정보 가져오기
content = TAG.text #태그에서 화면에 보이는 텍스트 부분만 가져오기
attribute = TAG['속성명'] #태그 내 속성값 가져오기
```


```python
#선택한 태그에서 텍스트, 속성 값 가져오기
tags = soup.select('a')
tag = tags[0]
content = tag.text
print(content)
link = tag['href']
print(link)
```

     홈페이지 
    http://bit.ly/forPlaywithdata
    

#### 11) 멜론 노래 순위 정보 크롤링


```python
#크롬 드라이버 실행
from selenium import webdriver
driver = webdriver.Chrome(r'C:\Users\hwn11\Data\chromedriver.exe')
```


```python
#멜론 인기차트 웹페이지 접속하기
url = 'https://www.melon.com/chart/index.htm'
driver.get(url)
```


```python
#HTML 다운로드 및 BeautifulSoup으로 읽기
from bs4 import BeautifulSoup
html = driver.page_source
soup = BeautifulSoup(html, 'html.parser')
```


```python
#100개의 노래 태그 찾기
songs = soup.select('table > tbody > tr') #곡 정보가 포함된 tr태그를 모두 찾음
print(len(songs)) # 원소가 몇개인지 확인. 태그의 수
print(songs[0]) #태그 중 첫 번째 태그를 출력. 
```

    100
    <tr data-song-no="33077590">
    <td><div class="wrap t_right"><input class="input_check " name="input_check" title="VVS (Feat. JUSTHIS) (Prod. GroovyRoom) 곡 선택" type="checkbox" value="33077590"/></div></td>
    <td><div class="wrap">
    <a class="image_typeAll" href="javascript:melon.link.goAlbumDetail('10521601');" title="쇼미더머니 9 Episode 1">
    <img alt="쇼미더머니 9 Episode 1 - 페이지 이동" height="60" onerror="WEBPOCIMG.defaultAlbumImg(this);" src="https://cdnimg.melon.co.kr/cm2/album/images/105/21/601/10521601_20201120125511_500.jpg/melon/resize/120/quality/80/optimize" width="60"/>
    <span class="bg_album_frame"></span>
    </a>
    </div></td>
    <td><div class="wrap">
    <a class="btn button_icons type03 song_info" href="javascript:melon.link.goSongDetail('33077590');" title="VVS (Feat. JUSTHIS) (Prod. GroovyRoom) 곡정보"><span class="none">곡정보</span></a>
    </div></td>
    <td><div class="wrap">
    <div class="wrap_song_info">
    <div class="ellipsis rank01"><span>
    <a href="javascript:melon.play.playSong('19030101',33077590);" title="VVS (Feat. JUSTHIS) (Prod. GroovyRoom) 재생">VVS (Feat. JUSTHIS) (Prod. GroovyRoom)</a>
    </span></div>
    <br/>
    <div class="ellipsis rank02">
    <a href="javascript:melon.link.goArtistDetail('2866523');" title="미란이 (Mirani) - 페이지 이동">미란이 (Mirani)</a>, <a href="javascript:melon.link.goArtistDetail('2747971');" title="먼치맨(MUNCHMAN) - 페이지 이동">먼치맨(MUNCHMAN)</a>, <a href="javascript:melon.link.goArtistDetail('1703507');" title="Khundi Panda - 페이지 이동">Khundi Panda</a>, <a href="javascript:melon.link.goArtistDetail('2745413');" title="머쉬베놈 (MUSHVENOM) - 페이지 이동">머쉬베놈 (MUSHVENOM)</a><span class="checkEllipsis" style="display: none;"><a href="javascript:melon.link.goArtistDetail('2866523');" title="미란이 (Mirani) - 페이지 이동">미란이 (Mirani)</a>, <a href="javascript:melon.link.goArtistDetail('2747971');" title="먼치맨(MUNCHMAN) - 페이지 이동">먼치맨(MUNCHMAN)</a>, <a href="javascript:melon.link.goArtistDetail('1703507');" title="Khundi Panda - 페이지 이동">Khundi Panda</a>, <a href="javascript:melon.link.goArtistDetail('2745413');" title="머쉬베놈 (MUSHVENOM) - 페이지 이동">머쉬베놈 (MUSHVENOM)</a></span>
    </div>
    <div class="wrap_atist" style="">
    <button class="button_icons etc more_down" data-control="dropdown" title="아티스트 더보기" type="button"><span class="none">아티스트명 더보기</span></button>
    <div class="atist_view" style="display:none;">
    <ul>
    <li><a class="ellipsis" href="javascript:melon.link.goArtistDetail('2866523');" title="미란이 (Mirani) 페이지 이동">미란이 (Mirani)</a></li>
    <li><a class="ellipsis" href="javascript:melon.link.goArtistDetail('2747971');" title="먼치맨(MUNCHMAN) 페이지 이동">먼치맨(MUNCHMAN)</a></li>
    <li><a class="ellipsis" href="javascript:melon.link.goArtistDetail('1703507');" title="Khundi Panda 페이지 이동">Khundi Panda</a></li>
    <li><a class="ellipsis" href="javascript:melon.link.goArtistDetail('2745413');" title="머쉬베놈 (MUSHVENOM) 페이지 이동">머쉬베놈 (MUSHVENOM)</a></li>
    </ul>
    </div>
    </div>
    </div>
    </div></td>
    <td><div class="wrap">
    <div class="wrap_song_info">
    <div class="ellipsis rank03">
    <a href="javascript:melon.link.goAlbumDetail('10521601');" title="쇼미더머니 9 Episode 1 - 페이지 이동">쇼미더머니 9 Episode 1</a>
    </div>
    </div>
    </div></td>
    <td><div class="wrap">
    <button class="button_etc like" data-song-menuid="19030101" data-song-no="33077590" title="VVS (Feat. JUSTHIS) (Prod. GroovyRoom) 좋아요" type="button"><span class="odd_span">좋아요</span>
    <span class="cnt">
    <span class="none">총건수</span>
    158,398</span></button>
    </div></td>
    <td><div class="wrap t_center">
    <button class="button_icons play " onclick="melon.play.playSong('19030101',33077590);" title="듣기" type="button"><span class="none">듣기</span></button>
    </div></td>
    <td><div class="wrap t_center">
    <button class="button_icons scrap " onclick="melon.play.addPlayList('33077590');" title="담기" type="button"><span class="none">담기</span></button>
    </div></td>
    <td><div class="wrap t_center">
    <button class="button_icons download " onclick="melon.buy.goBuyProduct('frm', '33077590', '3C0001', '','0', '19030101');" title="다운로드" type="button"><span class="none">다운로드</span></button>
    </div></td>
    <td><div class="wrap t_center">
    <button class="button_icons video disabled" disabled="disabled" onclick="melon.link.goMvDetail('19030101', '33077590','song');" title="뮤직비디오" type="button"><span class="none">뮤직비디오</span></button>
    </div></td>
    </tr>
    

1위 곡의 정보를 먼저 찾은 후 반복문을 통해 100곡 전체의 제목과 가수 찾기


```python
#한 개의 곡 정보 지정하기
song = songs[0]
```


```python
#곡 제목 찾기 
title = song.select('a')
len(title)
```




    16



첫번째 코드에서 태그명이 a 인 태그 = 16개


```python
#곡 제목 찾기 2
title = song.select('span > a')
len(title)
```




    5



span태그 바로 아래에 존재하는 a 태그 = 5개


```python
#곡 제목 찾기 3 
title = song.select('div.ellipsis.rank01 > span > a')
len(title)
```




    1




```python
#곡 제목 가져오기 
title = song.select('div.ellipsis.rank01 > span > a')[0].text
title
```




    'VVS (Feat. JUSTHIS) (Prod. GroovyRoom)'




```python
#가수 찾기
singer = song.select('div.ellipsis.rank02 > a')[1].text
singer
```




    '먼치맨(MUNCHMAN)'



#### 11) 멜론 노래 순위 정보 크롤링


```python
#멜론 100위 노래 순위 정보 가져오기
for song in songs:
    title = song.select('div.ellipsis.rank01 > span > a')[0].text
    singer = song.select('div.ellipsis.rank02 > a')[0].text
    print(title, singer, sep = ' | ')
```

    VVS (Feat. JUSTHIS) (Prod. GroovyRoom) | 미란이 (Mirani)
    밤하늘의 별을(2020) | 경서
    Dynamite | 방탄소년단
    잠이 오질 않네요 | 장범준
    Lovesick Girls | BLACKPINK
    취기를 빌려 (취향저격 그녀 X 산들) | 산들
    힘든 건 사랑이 아니다 | 임창정
    내일이 오면 (Feat. 기리보이, BIG Naughty (서동현)) | 릴보이 (lIlBOI)
    Life Goes On | 방탄소년단
    오래된 노래 | 스탠딩 에그
    Savage Love (Laxed - Siren Beat) (BTS Remix) | Jawsh 685
    내 마음이 움찔했던 순간 (취향저격 그녀 X 규현) | 규현 (KYUHYUN)
    나랑 같이 걸을래 (바른연애 길잡이 X 적재) | 적재
    에잇(Prod.&Feat. SUGA of BTS) | 아이유
    화(火花) | (여자)아이들
    어떻게 이별까지 사랑하겠어, 널 사랑하는 거지 | AKMU (악동뮤지션)
    혼술하고 싶은 밤 | 벤
    CREDIT (Feat. 염따, 기리보이, Zion.T) | 릴보이 (lIlBOI)
    Rosario (Feat. CL, ZICO) | 에픽하이 (EPIK HIGH)
    내 얘기 같아 (Feat. 헤이즈) | 에픽하이 (EPIK HIGH)
    흔들리는 꽃들 속에서 네 샴푸향이 느껴진거야 | 장범준
    Blueming | 아이유
    When We Disco (Duet with 선미) | 박진영
    늦은 밤 너의 집 앞 골목길에서 | 노을
    모든 날, 모든 순간 (Every day, Every Moment) | 폴킴
    어떻게 지내 (Prod. By VAN.C) | 오반 (OVAN)
    METEOR | 창모 (CHANGMO)
    How You Like That | BLACKPINK
    Achoo (Feat. pH-1, HAON) (Prod. GroovyRoom) | 미란이 (Mirani)
    아로하 | 조정석
    사실 나는 (Feat.전건호) | 경서예지
    Dolphin | 오마이걸 (OH MY GIRL)
    서면역에서 | 순순희
    그날에 나는 맘이 편했을까 | 이예준
    마음을 드려요 | 아이유
    Freak (Prod. Slom) | 릴보이 (lIlBOI)
    딩가딩가 (Dingga) | 마마무 (Mamamoo)
    오늘도 빛나는 너에게 (To You My Light) (Feat.이라온) | 마크툽 (MAKTUB)
    ON AIR (Feat. 로꼬, 박재범 & GRAY) | 릴보이 (lIlBOI)
    뿌리 (Feat. JUSTHIS) (Prod. GroovyRoom) | Khundi Panda
    2002 | Anne-Marie
    작은 것들을 위한 시 (Boy With Luv) (Feat. Halsey) | 방탄소년단
    Memories | Maroon 5
    DON'T TOUCH ME | 환불원정대
    거짓말이라도 해서 널 보고싶어 | 백지영
    우리 왜 헤어져야 해 | 신예영
    I CAN’T STOP ME | TWICE (트와이스)
    마리아 (Maria) | 화사 (Hwa Sa)
    시작 | 가호 (Gaho)
    사랑은 지날수록 더욱 선명하게 남아 | 전상근
    봄날 | 방탄소년단
    살짝 설렜어 (Nonstop) | 오마이걸 (OH MY GIRL)
    한잔이면 지워질까 | 황인욱
    Dance Monkey | Tones And I
    홀로 | 이하이
    뻔한남자 | 이승기
    악역 (Feat. 이하이 & 사이먼 도미닉) (Prod. 코드 쿤스트) | 스윙스
    눈누난나 (NUNU NANA) | 제시 (Jessi)
    안녕 | 폴킴
    Downtown Baby | 블루 (BLOO)
    Panorama | IZ*ONE (아이즈원)
    사랑 못해, 남들 쉽게 다 하는 거 | 먼데이 키즈 (Monday Kiz)
    너의 번호를 누르고 (Prod. 영화처럼) | #안녕
    너를 만나 | 폴킴
    가을밤에 든 생각 | 잔나비
    Love poem | 아이유
    소확행 | 임창정
    처음처럼 | 엠씨더맥스 (M.C the MAX)
    고독하구만 (Feat. 수퍼비) (Prod. GroovyRoom) | 머쉬베놈 (MUSHVENOM)
    우린 어쩌다 헤어진 걸까 | 허각
    이제 나만 믿어요 | 임영웅
    Snowman | Sia
    잘할게 | 이승기
    사랑하게 될 줄 알았어 | 전미도
    행복해 | 송하예
    What Do I Call You | 태연 (TAEYEON)
    Don't Start Now | Dua Lipa
    12:45 (Stripped) | Etham
    Black Mamba | aespa
    아무노래 | 지코 (ZICO)
    Bad Boy | 청하
    나로 바꾸자 (duet with JYP) | 비
    별을 담은 시 (Ode To The Stars) | 마크툽 (MAKTUB)
    Paris In The Rain | Lauv
    Maniac | Conan Gray
    너도 아는 | 폴킴
    여백의 미 (Feat. Jessi, JUSTHIS) (Prod. GroovyRoom) | 머쉬베놈 (MUSHVENOM)
    다시 여기 바닷가 | 싹쓰리 (유두래곤, 린다G, 비룡)
    12월의 어느 겨울… | 윤도 (YoonDo)
    ON | 방탄소년단
    밝게 빛나는 별이 되어 비춰줄게 | 송이한
    정당방위 (Feat. 우원재, 넉살, 창모) | 에픽하이 (EPIK HIGH)
    놓아줘 (with 태연) | Crush
    X (걸어온 길에 꽃밭 따윈 없었죠) | 청하
    Santa Tell Me | Ariana Grande
    원해 (Feat. 팔로알토) (Prod. 코드 쿤스트) | 스윙스
    적외선 카메라 | 원슈타인
    For You (Feat. Crush) | 이하이
    Blue & Grey | 방탄소년단
    All I Want For Christmas Is You | Mariah Carey
    


```python
#멜론 인기차트 중 상위 100곡 크롤링 (정리)
#크롬 드라이버 실행
from selenium import webdriver
driver = webdriver.Chrome(r'C:\Users\hwn11\Data\chromedriver.exe')

#멜론 인기차트 웹페이지 접속하기
url = 'https://www.melon.com/chart/index.htm'
driver.get(url)

#HTML 다운로드 및 BeautifulSoup으로 읽기
from bs4 import BeautifulSoup
html = driver.page_source
soup = BeautifulSoup(html, 'html.parser')

songs = soup.select('tbody > tr') #곡 정보가 포함된 tr태그를 모두 찾음
#멜론 100위 노래 순위 정보 가져오기
for song in songs:
    title = song.select('div.ellipsis.rank01 > span > a')[0].text
    singer = song.select('div.ellipsis.rank02 > a')[0].text
    print(title, singer, sep = ' | ')
```

    VVS (Feat. JUSTHIS) (Prod. GroovyRoom) | 미란이 (Mirani)
    밤하늘의 별을(2020) | 경서
    Dynamite | 방탄소년단
    잠이 오질 않네요 | 장범준
    Lovesick Girls | BLACKPINK
    취기를 빌려 (취향저격 그녀 X 산들) | 산들
    힘든 건 사랑이 아니다 | 임창정
    내일이 오면 (Feat. 기리보이, BIG Naughty (서동현)) | 릴보이 (lIlBOI)
    Life Goes On | 방탄소년단
    오래된 노래 | 스탠딩 에그
    Savage Love (Laxed - Siren Beat) (BTS Remix) | Jawsh 685
    내 마음이 움찔했던 순간 (취향저격 그녀 X 규현) | 규현 (KYUHYUN)
    나랑 같이 걸을래 (바른연애 길잡이 X 적재) | 적재
    에잇(Prod.&Feat. SUGA of BTS) | 아이유
    화(火花) | (여자)아이들
    어떻게 이별까지 사랑하겠어, 널 사랑하는 거지 | AKMU (악동뮤지션)
    혼술하고 싶은 밤 | 벤
    CREDIT (Feat. 염따, 기리보이, Zion.T) | 릴보이 (lIlBOI)
    Rosario (Feat. CL, ZICO) | 에픽하이 (EPIK HIGH)
    흔들리는 꽃들 속에서 네 샴푸향이 느껴진거야 | 장범준
    내 얘기 같아 (Feat. 헤이즈) | 에픽하이 (EPIK HIGH)
    Blueming | 아이유
    When We Disco (Duet with 선미) | 박진영
    늦은 밤 너의 집 앞 골목길에서 | 노을
    모든 날, 모든 순간 (Every day, Every Moment) | 폴킴
    어떻게 지내 (Prod. By VAN.C) | 오반 (OVAN)
    METEOR | 창모 (CHANGMO)
    How You Like That | BLACKPINK
    Achoo (Feat. pH-1, HAON) (Prod. GroovyRoom) | 미란이 (Mirani)
    아로하 | 조정석
    사실 나는 (Feat.전건호) | 경서예지
    Dolphin | 오마이걸 (OH MY GIRL)
    서면역에서 | 순순희
    마음을 드려요 | 아이유
    그날에 나는 맘이 편했을까 | 이예준
    Freak (Prod. Slom) | 릴보이 (lIlBOI)
    딩가딩가 (Dingga) | 마마무 (Mamamoo)
    오늘도 빛나는 너에게 (To You My Light) (Feat.이라온) | 마크툽 (MAKTUB)
    ON AIR (Feat. 로꼬, 박재범 & GRAY) | 릴보이 (lIlBOI)
    뿌리 (Feat. JUSTHIS) (Prod. GroovyRoom) | Khundi Panda
    2002 | Anne-Marie
    작은 것들을 위한 시 (Boy With Luv) (Feat. Halsey) | 방탄소년단
    Memories | Maroon 5
    DON'T TOUCH ME | 환불원정대
    거짓말이라도 해서 널 보고싶어 | 백지영
    우리 왜 헤어져야 해 | 신예영
    I CAN’T STOP ME | TWICE (트와이스)
    마리아 (Maria) | 화사 (Hwa Sa)
    시작 | 가호 (Gaho)
    사랑은 지날수록 더욱 선명하게 남아 | 전상근
    봄날 | 방탄소년단
    살짝 설렜어 (Nonstop) | 오마이걸 (OH MY GIRL)
    한잔이면 지워질까 | 황인욱
    홀로 | 이하이
    Dance Monkey | Tones And I
    뻔한남자 | 이승기
    악역 (Feat. 이하이 & 사이먼 도미닉) (Prod. 코드 쿤스트) | 스윙스
    눈누난나 (NUNU NANA) | 제시 (Jessi)
    안녕 | 폴킴
    Downtown Baby | 블루 (BLOO)
    Panorama | IZ*ONE (아이즈원)
    사랑 못해, 남들 쉽게 다 하는 거 | 먼데이 키즈 (Monday Kiz)
    너의 번호를 누르고 (Prod. 영화처럼) | #안녕
    너를 만나 | 폴킴
    가을밤에 든 생각 | 잔나비
    Love poem | 아이유
    소확행 | 임창정
    처음처럼 | 엠씨더맥스 (M.C the MAX)
    고독하구만 (Feat. 수퍼비) (Prod. GroovyRoom) | 머쉬베놈 (MUSHVENOM)
    우린 어쩌다 헤어진 걸까 | 허각
    이제 나만 믿어요 | 임영웅
    Snowman | Sia
    사랑하게 될 줄 알았어 | 전미도
    잘할게 | 이승기
    행복해 | 송하예
    What Do I Call You | 태연 (TAEYEON)
    Don't Start Now | Dua Lipa
    12:45 (Stripped) | Etham
    아무노래 | 지코 (ZICO)
    Black Mamba | aespa
    Bad Boy | 청하
    나로 바꾸자 (duet with JYP) | 비
    별을 담은 시 (Ode To The Stars) | 마크툽 (MAKTUB)
    Paris In The Rain | Lauv
    Maniac | Conan Gray
    너도 아는 | 폴킴
    여백의 미 (Feat. Jessi, JUSTHIS) (Prod. GroovyRoom) | 머쉬베놈 (MUSHVENOM)
    다시 여기 바닷가 | 싹쓰리 (유두래곤, 린다G, 비룡)
    12월의 어느 겨울… | 윤도 (YoonDo)
    ON | 방탄소년단
    밝게 빛나는 별이 되어 비춰줄게 | 송이한
    X (걸어온 길에 꽃밭 따윈 없었죠) | 청하
    정당방위 (Feat. 우원재, 넉살, 창모) | 에픽하이 (EPIK HIGH)
    놓아줘 (with 태연) | Crush
    Santa Tell Me | Ariana Grande
    원해 (Feat. 팔로알토) (Prod. 코드 쿤스트) | 스윙스
    Blue & Grey | 방탄소년단
    적외선 카메라 | 원슈타인
    For You (Feat. Crush) | 이하이
    All I Want For Christmas Is You | Mariah Carey
    