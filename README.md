# 현재의 음악 장르 분류기를 개선하는 모델

류다윤, 경영학부, <ryudy4977@naver.com> - 음성 데이터 확보 및 피처 추출, 모델 구축 및 적용

최재훈, 경영학부, <bang8649@naver.com> - 주제 제안/기획, 멜론 차트 데이터 셋 구축, 블로그 작성 및 발표

---

## I. Proposal
### A.	Motivation: Why are you doing this?
#### 오디오 데이터에 대한 연구의 부족
컴퓨팅 성능이 향상되고, 인공지능(AI), 머신러닝(ML), 딥러닝(DL)과 같은 기술이 발전하면서 최근 NLP(Natural Language Processing, 자연어 처리), CV(Computer Vision, 컴퓨터 비전)와 같이 텍스트와 이미지/영상 데이터를 분석, 처리하는 연구가 쏟아지고 있다. 하지만, 오디오/음성 데이터와 관련된 연구는 상대적으로 부족한 실정이다. 왜 그럴까?

#### 전처리가 까다로운 오디오 데이터의 특징
오디오 데이터는 매질(물체)의 진동을 통해 전달되는 일종의 아날로그 신호이다. 컴퓨터에게 이러한 아날로그 신호를 입력시키려면, 우리는 오디오 데이터를 디지털 신호로 변환해주어야 한다.
즉, [그림 1]과 같은 과정을 통해 아날로그 신호를 잘게 쪼개고, 각각의 값을 디지털화(숫자화) 해야 한다. 이러한 과정을 ‘샘플링(Sampling)'이라고 부른다. [그림 2]

![다운로드1](https://user-images.githubusercontent.com/97533303/171172021-a77e0ff2-c331-4f9e-89e2-65d996f1afda.png)
![다운로드2](https://user-images.githubusercontent.com/97533303/171172151-a7ca132e-0c0b-4baf-b0c2-2ddbaaa08f3d.jpeg)

하지만, 오디오 데이터가 지니는 2가지 근본적인 문제가 있다.
1. 첫 번째 문제 - 데이터의 크기가 매우 커진다.

샘플링이란 1초 분량의 오디오 데이터에서 n개의 샘플을 추출한다는 것을 의미한다. 일반적으로 많은 사람들은 1초 분량의 오디오 데이터에서 44100개의 샘플을 추출하며, 이는 44100Hz, 44.1kHz의 샘플링 레이트(Sampling Rate, SR)로 샘플링을 진행했다고 말할 수 있다.

30초 분량의 오디오 데이터를 분석한다고 예를 들면, 우리는 44100 * 30 = 1323000개의 샘플이 필요하다. 이처럼 디지털화된 오디오 데이터는 매우 큰 크기를 지니는데, 이는 컴퓨터가 데이터를 처리하는 것을 어렵게 만드는 요인이 된다. 우리는 ‘나이퀴스트 정리'를 통해 적절한 SR를 도출할 수 있다.

2. 두 번째 문제 - 오디오 데이터는 서로 다른 주파수가 결합되어 있는 형태로 나타난다.

오디오 데이터는 ‘시간 영역(x축)’과 ‘주파수 영역(y축)’으로 구성되며, [그림 3]에서 볼 수 있듯 오디오 데이터는 다양한 주파수로 구성된다. 이러한 오디오 데이터의 형태적 특징은 ‘Feature Engineering(특징 추출)’을 어렵게 만든다.

<img width="700" alt="스크린샷 2022-05-31 21 23 29" src="https://user-images.githubusercontent.com/97533303/171172368-f7c25e90-7445-43bd-89b4-2ea24b5c829a.png">

우리는 ‘고속 푸리에 변환'을 통해 오디오 데이터의 ‘시간 영역'을 ‘주파수 영역'으로 변환하고, 추가로 사람이 인식할 수 있도록 주파수를 mel-scale로 변환할 수 있다.

이러한 과정을 거치면, 우리는 Waveform 형태를 갖는 신호를 Mel-Spectrogram(멜-스펙트로그램) 이라는 형태로 변환할 수 있고, 음향 신호를 시간(x축), 주파수(y축), 진폭(z축)으로 살펴볼 수 있다.

#### 정리
이처럼, 오디오 데이터는 활용하기에 매우 까다롭다. 하지만 연산 속도가 점점 빨라지고 있으며 다양한 오디오 데이터를 위한 다양한 라이브러리가 존재하기 때문에, 오디오 데이터와 관련된 연구는 잠재적인 가능성이 많다.

오디오 데이터와 관련된 대표적인 분야로는 ‘음성 인식(Speech Recognition)', ‘오디오 분류(Audio Claasification)’, ‘오디오 캡셔닝(Audio Captioning)' 등이 있다.

우리는 딥러닝을 통해 ‘오디오 분류' 문제를 해결하고자 하며, 주변에서 흔히 접할 수 있는 대표적인 오디오 데이터, 음악을 주제로 연구를 진행하고자 한다. 

#### 국내외 디지털 음원시장 현황 
ICT 기술의 발전으로 음악 시장이 다시 부활하고 있다. 디지털 음원 시장의 경우, 스트리밍 서비스가 등장하면서 꾸준한 상승세를 보이고 있고 앨범, 레코드와 같은 실물 음반 시장도 꾸준히 증가하고 있다.

특히 한국의 경우, K-Pop이 글로벌하게 확산되면서 음악 시장이 폭발적으로 성장하고 있다. 디지털 음원 시장의 경우, ‘멜론’, ‘지니’, ‘벅스’와 같은 국내 음원 스트리밍 서비스에서 많이 재생되던 한국 음원들이 글로벌 음원 스트리밍 서비스 업체인 ‘스포티파이', ‘유튜브 뮤직'에서도 높은 스트리밍 수를 기록하고 있으며, 실물 음반 시장의 경우, K-Pop 아티스트의 글로벌 팬덤들이 음반을 다량 소비하고 있다.

이는 한국의 디지털 콘텐츠 시장이 세계적으로 주목 받고 있는 시점에서, 한국의 음악 시장도 크게 성장하고 있다고 살펴볼 수 있다. 따라서, 우리는 이렇게 성장하고 있는 한국 음악 시장에서 주제를 선정하여 프로젝트를 진행하고자 한다.

#### K-POP의 장르적 다양성
K-POP은 흔히 세대적인 관점에서 분석되곤 한다. H.O.T/젝스키스/S.E.S/핑클 등의 그룹이 활동하던 시절을 1세대, 동방신기/슈퍼주니어/빅뱅/원더걸스/소녀시대/카라 등의 그룹이 활동하던 시절을 2세대라고 하는 것처럼 말이다. 그렇게 2.5세대, 3세대, 3.5세대를 거쳐 2022년 현재에는 4세대 아이돌이라고 불리어지는 아이브/에스파/스테이씨 등이 활동하고 있다.

우선, 근본적으로 K-POP은 해외에서 유입된 다양한 장르들을 수용하면서 발전했다. 국내에서는 흔하지 않았던 장르를 1세대 케이팝 아티스트들이 가져왔고 이러한 경향은 점점 짙어져갔다. 그러던 중, 후크송을 내세운 2세대부터 케이팝이 특정 팬층이 향유하는 장르를 넘어 전 국민이 즐기는 장르로 도약했다. 이러한 영향으로 인해, 2.5세대에 들어서 케이팝은 많은 변화를 하기 시작했다.

흔히 2.5 세대 케이팝으로 분류되는, 2000년대 후반 데뷔 그룹부터 시작되는 세대의 케이팝은 사후적인 시선에서 보았을 때 현재의 케이팝의 근원이 되는 가장 가까운 기반으로서 자리하고 있다. 팬덤의 발전과 케이팝 시장 전반의 양적 성장 현장들이 2.5세대에 들어 더욱 확실하게 정착하기 시작했으며, 케이팝은 이전까지의 주 무대였던 국내와 동아시아 시장을 넘어 전 세계의 음악 시장으로 손을 뻗치기 시작했다.

이와 같은 성장세의 가속화에 따라 동시에 케이팝이라는 장르가 가지는 음악적 퀄리티와 그 특성은 발전을 거듭하게 되었고, 다양한 음악, 기획적 시도가 이루어졌다. 가령, 현재의 케이팝에서 활용되는 일렉트로닉 장르의 도입의 기저를 제시한 샤이니(SHINee), f(x)등의 그룹 역시 2.5 세대로 분류하고 있다.

이러한 케이팝의 장르적인 특징은 3세대를 시작으로 심화되었으며, 이는 케이팝만의 구별되는 특징이 되었다. 최근에 발매되는 케이팝에는 하나의 곡 내에서 다양한 장르가 맞물려 등장하는 경우도 자주 활용되곤 한다. 후렴 이후 갑작스러운 트랩 비트의 등장을 통해 랩 파트를 삽입하는 방식은 이제는 셀 수 없이 많은 사례가 존재하며, 강렬한 힙합, 혹은 일렉트로닉 비트와 부드러운 알앤비 풍의 멜로디가 겹쳐지며 그러한 낙차를 통해 역동성을 만드는 경우 역시 수많은 곡에서 재현되고 있다. 그렇게 3세대 이후, 특히 2010년대의 후반에 들어 더욱 거세지고 있는 케이팝의 역동성은 후크 송의 단순함을 탈피한다는 연유를 시작으로 그것만이 선사할 수 있는 재미를 함축하고 있다.

#### 이러한 다양성을 반영하지 못하는 현재의 음악 장르 분류기
하지만 이러한 K-Pop의 장르적 다양성에도 불구하고, 현재의 음악 장르 분류기는 이를 반영하지 못한다.

국내 최대 음원 스트리밍 사이트 ‘멜론’은 곡의 장르를 발라드, 댄스, 랩/힙합, R&B/소울, 인디음악, 록/메탈, 트로트, 포크/블루스 총 8가지로 분류하고 있다. 하지만 멜론은 대다수의 K-POP 곡들을 댄스, 랩/힙합이라는 단일한 카테고리로 음악을 분류하고 있다.

ex) 방탄소년단의 대표곡 - ‘DNA’, ‘FAKE LOVE’ ,’IDOL’ → '랩/힙합'으로 분류됨

ex) 아이돌 앨범에 수록된 발라드, R&B 풍의 노래들 - '댄스'로 분류됨

앞서 언급했듯이 같은 댄스 장르라고 하더라도, K-POP은 하나의 곡 내에서도 매우 다양한 요소들이 결합되어 있다. 이러한 장르적인 정보는 음원 사이트에서 제공하는 '앨범 소개'에서 찾아볼 수 있다. 모든 앨범 소개가 그런 것은 아니지만, 일반적으로 아티스트 및 창작자는 '앨범 소개'를 통해 음악에 대해서 상세하고 있다.

ex) 방탄소년단 ‘DNA’ - 일렉트로 팝을 기반으로 중독적인 휘파람 소리와 어쿠스틱한 기타 사운드가 어우러지면서 분명 방탄스럽지만 기존의 팀 컬러와 차별화되기도 하는 업그레이드된 사운드를 선사한다. 특히 Break Down 이후 Drop 파트에서 Future Bass 기반의 EDM 사운드로 넘어가는 곡의 구성은 '방탄소년단'만의 파워풀한 퍼포먼스를 극대화시키는 역할을 한다.

### B. What do you want to see at the end?
이렇듯 우리는 장르적인 다양성이 K-POP을 관통하는 특징임에도 불구하고, 기존의 분류 체제를 따르는 것이 현 시대를 잘 반영한다고 볼 수 있을지 의문이 들었다.

따라서 본 프로젝트는 기존의 광범위한 음악 장르 분류기를 현 시대에 맞춘 세분화된 분류기로 개선시키는 것을 목적으로 한다. 궁극적으로, 이를 통해 사용자가 자신이 원하는 음악을 더욱 디테일하게 찾을 수 있게 함을 목표로 한다.

#### 연구 문제 1
- 멜론 등에서 사용하는 것처럼, 기존의 음원 분류기를 재현한다.
- 8개의 기존 분류 중 발라드, 댄스, 랩/힙합, R&B/소울 라는 4가지 장르가 유의미하다고 판단하였다.

#### 연구 문제 2
- 댄스 곡에 포함되어 있는 음악적인 특징을 감지하는 새로운 모델을 재현한다.
- 텍스트 마이닝을 통해 앨범 소개에 등장하는 음악적 특징을 도출해내고, 이 중 빈도가 높은 것을 종속 변수로 선정할 예정이다.

## II. Datasets 
[연구 문제 1]
1. 2020년 1월부터 2022년 5월까지의 멜론 월간 차트 장르별 TOP100 음악 데이터를 수집한다.

==> 엑셀 파일 구축 완료 (y20d/b/h/r.xlsx, y21d/b/h/r.xlsx, y22d/b/h/r.xlsx)

==> Columns: 'Year', 'Month', 'Genre', 'Rank', 'Title', 'Artist', 'Album'

2. 엑셀 데이터 내의 '아티스트 명(Artist)'와 '노래 제목(Title)' 열을 불러와, Youtube에서 음원을 검색, 추출한다.

==> 음원 데이터(wav) 추출 완료 (Google Drive)

[연구 문제 2]
1. 2017년 1월부터 2022년 5월까지의 멜론 주간 댄스 차트 TOP100 음악 데이터를 수집한다

==> 엑셀 파일 구축 완료 (댄스 주간 차트 신규.xlsx)

==> Columns: 'Year', 'Month', 'Genre', 'Rank', 'Title', 'Artist', 'Album', 'Album ID', 'Track Num', 'Description'

2. 엑셀 데이터 내의 '아티스트 명(Artist)'와 '노래 제목(Title)' 열을 불러와, Youtube에서 음원을 검색, 추출한다.

==> 음원 데이터(wav) 추출 완료 (Google Drive)

## III. Methodology 

### A. Explaining your choice of features and algorithms

### i. Datasets 
#### 연구 문제 1 - 멜론 월간 차트 크롤링
멜론이 제공하는 차트 파인더 페이지에서 2020, 2021, 2022년의 장르별 월간 차트 TOP 100을 찾을 수 있었다.
셀레니움을 활용해서 멜론 차트 파인더 페이지에 접근했으며, 관련된 음악 데이터(연도, 월, 장르, 순위, 노래 제목, 아티스트 명, 앨범 명)를 크롤링해 엑셀 파일로 저장했다.

코드는 레포지토리에 첨부해두었다.

#### 연구 문제 2 - 멜론 주간 차트 크롤링
멜론이 제공하는 차트 파인더 페이지에서 2017, 2018, 2019, 2020, 2021, 2022년의 장르별 주간 차트 TOP 100을 찾을 수 있었다.
셀레니움을 활용해서 멜론 차트 파인더 페이지에 접근했으며, 관련된 음악 데이터(연도, 월, 장르, 순위, 노래 제목, 아티스트 명, 앨범 명, 앨범 ID)를 크롤링해 엑셀 파일로 저장했다.

연구 문제 1과 달리, 앨범 소개를 크롤링해야 하기 때문에 '앨범 ID'를 추가로 크롤링했다. '앨범 ID'를 통해 앨범 페이지에 각각 들어가서 '트랙 번호', '앨범 소개'를 추가로 가져왔다.
'트랙 번호'로 앨범 소개에서 필요한 부분을 자동으로 가져오려고 노력했으며, 누락된 부분은 직접 추가하는 과정을 거쳤다. 이러한 과정을 통해 데이터 정확도를 높였다.

코드는 레포지토리에 첨부해두었다.

#### 연구 문제 1,2 - 유튜브 음원 추출
'아티스트 명' + '노래 제목' + 'lyrics' 을 키워드로 검색하여 해당 음악의 음원 데이터를 찾을 수 있었다.
뮤직비디오나 무대 영상의 경우 다양한 잡음이 존재하기 때문에, 음원만 흘러 나오는 데이터를 추출하기 위해 키워드에 'lyrics'를 추가했다.

셀레니움을 활용해서 유튜브 페이지에 접근했으며, 키워드로 검색된 결과 중 최상단에 위치한 콘텐츠의 제목과 url을 추출하여 csv 파일로 저장했다.
그리고 유튜브 영상을 다운로드 할 수 있게 해주는 파이썬 모듈 'YT-DLP'를 통해 유튜브 영상을 음원 형태(wav 파일)로 다운로드 했다.

코드는 레포지토리에 첨부해두었다.

### ii. Features

#### 연구 문제 1,2 - Librosa를 활용한 Feature 도출 (총 19개)
Librosa는 음원 데이터를 분석할 수 있도록 도와주는 파이썬의 라이브러리다.
Librosa를 활용하면 음원 데이터의 다양한 특징을 발견할 수 있는데, 우리는 총 19개의 Feature를 발굴했다.

1. mfcc_mean
- MFCC(Mel Frequency Cepstral Coefficient) Feature는 음원 데이터의 특징 벡터를 나타내며, 기존 주파수를 인간이 들을 수 있는 주파수(Mel-Scale)로 변환하는 알고리즘의 결과 값이다. 이 결과 값들의 평균을 구해 Feature로 추가해주었다.

2. mfcc_var
- MFCC(Mel Frequency Cepstral Coefficient) Feature는 음원 데이터의 특징 벡터를 나타내며, 기존 주파수를 인간이 들을 수 있는 주파수(Mel-Scale)로 변환하는 알고리즘의 결과 값이다. 이 결과 값들의 분산을 구해 Feature로 추가해주었다.

3. zero_crossings_mean
- Zero Crossing Rate는 음파가 양에서 음으로, 또는 음에서 양으로 바뀌는 비율을 나타낸다. 이 값들의 평균을 구해 Feature로 추가해주었다.

4. zero_crossings_var
- Zero Crossing Rate는 음파가 양에서 음으로, 또는 음에서 양으로 바뀌는 비율을 나타낸다. 이 값들의 분산을 구해 Feature로 추가해주었다.

5. chroma_mean
- Chroma는 12 음계를 나타낸다. 이 값의 평균을 구해 Feature로 추가해주었다.

6. chroma_var
- Chroma는 12 음계를 나타낸다. 이 값의 분산을 구해 Feature로 추가해주었다.

7. tempo
- Tempo는 속도(BPM)를 나타낸다.

8. rolloff_mean
- Spectral Rolloff은 총 스펙트럴 에너지 중 낮은 주파수(85% 이하)에 얼마나 많이 집중되어 있는가를 알려준다. 이 값의 평균을 구해 Feature로 추가해주었다.

9. rollof_var
- Spectral Rolloff은 총 스펙트럴 에너지 중 낮은 주파수(85% 이하)에 얼마나 많이 집중되어 있는가를 알려준다. 이 값의 분산을 구해 Feature로 추가해주었다.

10. centrodis_mean
- Spectral Centrodis는 소리의 "무게 중심"이 어딘지를 알려주는 지표다. 이 값의 평균을 구해 Feature로 추가해주었다.

11. centrodis_var
- Spectral Centrodis는 소리의 "무게 중심"이 어딘지를 알려주는 지표다. 이 값의 분산을 구해 Feature로 추가해주었다.

12. bandwidth_mean
- Bandwidth는 소리의 대역폭을 알려주는 지표다. 이 값의 평균을 구해 Feature로 추가해주었다.

13. bandwidth_var
- Bandwidth는 소리의 대역폭을 알려주는 지표다. 이 값의 분산을 구해 Feature로 추가해주었다.

14. contrast_mean
- Spectral Contrast는 총 스펙트럴의 대비를 알려주는 지표다. 이 값의 평균을 구해 Feature로 추가해주었다.

15. contrast_var
- Spectral Contrast는 총 스펙트럴의 대비를 알려주는 지표다. 이 값의 분산을 구해 Feature로 추가해주었다.

16. harm_mean
- librosa.effects.hpss는 오디오 시계열을 고조파(harmonic) 구성요소로 분해한다. 분해된 harmonic 값의 평균을 구해 Feature로 추가해주었다.

17. harm_var
- librosa.effects.hpss는 오디오 시계열을 고조파(harmonic) 구성요소로 분해한다. 분해된 harmonic 값의 분산을 구해 Feature로 추가해주었다. 

18. perc_maen
- librosa.effects.hpss는 오디오 시계열을 타악기(percussive) 구성요소로 분해한다. 분해된 percussive 값의 평균을 구해 Feature로 추가해주었다.

19. perc_var
- librosa.effects.hpss는 오디오 시계열을 타악기(percussive) 구성요소로 분해한다. 분해된 percussive 값의 분산을 구해 Feature로 추가해주었다.

#### 연구 문제 2 - 텍스트 마이닝을 통한 세분화 Feature 도출 (총 5개)
앨범 소개에는 주로 전반적인 앨범의 컨셉, 창작자에 대한 정보, 개별 트랙들의 곡 소개가 들어 있다.
우리는, 트랙 번호로 개별 트랙들의 소개만을 잘라와서 'Description' 열에 이를 저장했다.
그리고, 파이썬의 KONLPY 라이브러리의 Okt(Open Korea Text) 모듈로 개별 트랙 소개에 나타난 명사의 빈도 분석을 실시했다.
* 영어의 경우, OKt에서 'Nouns'이 아닌 'Alpha'로 분류된다는 문제점이 있어 이를 명사로 변환하는 작업 또한 실시했다.

그 결과, 빈도가 잦은 댄스 장르 5개는 'Electro(152)', 'House(123)', 'Funk(122)', 'Hiphop(109)', 'Trap(62)' 으로 밝혀졌으며,
해당 장르가 맞으면 1 / 아니면 0으로 라벨링하여 새로운 5가지 feature를 데이터에 추가했다.

코드는 레포지토리에 첨부해두었다.

### iv. Algorithms
음악 데이터와 관련된 연구가 많이 없어 해당 분야의 대표적인 모델이 존재하지 않는다.
따라서, 이번 프로젝트에서는 지도학습(머신러닝, 딥러닝) 방법을 각각 사용해보고 성능을 살펴볼 계획이다.

트레인, 테스트 데이터 셋은 다음과 같다

![크기](https://user-images.githubusercontent.com/97533303/174097009-512abff3-fbfe-469d-926e-a24d000c04a1.png)

## Conclusion 
#### 지도학습(머신러닝) 모델 - 로지스틱 회귀분석, XGB
**1. 로지스틱 회귀분석**

librosa를 통해서 구한 feature 값이 연속형이고, 우리가 해결해야 하는 문제는 '분류'문제이기 때문에 머신러닝 모델 중 로지스틱 회귀분석(Logistic Regression)을 사용해보았다.

![로지스틱 결과](https://user-images.githubusercontent.com/97533303/174093573-805d5995-1e69-4dc6-81aa-ace00c68edae.png)

0.5822 라는 정확도를 얻을 수 있었다.

**2. XGBoost**
음악 장르를 분류하는 다른 연구에서 XGBoost를 사용했고, 앙상블 모델로서 다른 기법보다 성능이 좋을 것이라 판단해 XGBoost를 사용했다. 

![xgb 결과](https://user-images.githubusercontent.com/97533303/174093747-cf0466fc-cb24-49ac-94b7-29ccb4d95f7c.png)

0.5420 라는 정확도를 얻을 수 있었다.

#### 지도학습(딥러닝) 모델 - DNN
**3. DNN**
가장 기본적인 딥러닝 모델인 DNN을 사용했다.

![DNN 결과](https://user-images.githubusercontent.com/97533303/174096119-47b3a0a9-b1cf-45ac-a69b-3384c0ac1875.png)

0.5402 라는 정확도를 얻을 수 있었다.

유튜브 링크: https://youtu.be/I3qV5oLsqUM
