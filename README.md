# 현재의 음악 장르 분류기를 개선하는 모델


류다윤, 경영학부, <ryudy4977@naver.com>

최재훈, 경영학부, <bang8649@naver.com>

---


## I. Proposal
### A.	Motivation: Why are you doing this?
최근, 음악은 셀 수 없이 많은 장르가 쏟아져 나오고 있고 다양한 장르간 결합도 생겨나고 있다. 그 예로, Alternative R&B는 R&B를 기반으로 일렉트로닉, 록, 힙합 등 다양한 장르의 요소가 결합된 몽환적이고 감성적이며 우울한 멜로디라인과 울림있는 무드가 특징인 2000년대 중후반에 등장한 새로운 장르다. 하지만 현재의 음악 장르 분류기는 단순히 이를 R&B로 분류한다. 우리는 Alternative R&B라는 새로운 장르가 탄생했음에도 불구하고 기존의 분류 체제를 따르는 것이 현 시대를 잘 반영한다고 볼 수 있을지 의문이 들었다. 또한 힙합에서도 재즈랩, 이모 랩, 트랩, 이스트 코스트 힙합 등의 다양한 세부 장르가 있다. 소울도 마찬가지로 어반 소울, 시카고 소울, 네오 소울 등의 세부 장르가 있다. 그러나, 현재의 음악 장르 분류기는 발라드, 댄스, 힙합, 알앤비, 소울 등으로 장르를 분류한다. 따라서 본 토크나이
프로젝트는 기존의 광범위한 음악 장르 분류기를 현 시대에 맞춘 세분화된 분류기로 개선시키는 것을 목적으로 한다.
### B. What do you want to see at the end?
궁극적으로, 이를 통해 사용자가 자신이 원하는 음악을 더욱 디테일하게 찾을 수 있게 함을 목표로 한다.
## II. Datasets 
~년 ~월부터 ~년 ~월까지의 멜론 월간 차트 장르별 TOP100 음악 데이터와 앨범 소개를 사용한다. 또한, 모델의 더욱 정확한 훈련을 위해 ‘LOOPERMAN’의 ‘Popular Music Loop Search Tags’ 데이터도 사용한다. (링크)
## III. Methodology 
### A. Explaining your choice of algorithms and features
#### i. 멜론 월간 차트 크롤링
먼저 멜론 월간 차트 장르별 TOP100에서 노래 제목, 앨범명, 가수의 정보를 크롤링한다. 크롤링은 ...를 사용해...

#### ii. 음원 추출
해당 데이터에서 각각의 노래 제목을 유튜브에서의 검색 키워드로 설정한다. 뮤직비디오의 경우, 음원 뿐만 아니라 여러 다양한 잡음(이걸 뭐라고 해야할지..)도 있기 때문에 흔히 유튜브에서 음원과 가사만 포함된 ‘lyrics' 형태의 데이터만을 추출한다. 그리고, 각각의 키워드를 검색한 후 나타나는 콘텐츠 제목과 url을 크롤링한다. ‘노래 제목+lyrics’ 형태의 검색에서 최상단에 있는 콘텐츠가 거의 대부분 우리가 찾고자 하는 순수하게 음원만 있는 데이터이기 때문에 가장 첫번째의 콘텐츠의 제목과 url만 추출한다. 그리고 YT-DLP를 통해 유튜브 영상을 다운로드하고 이를 wav로 포맷을 변경한다. 

#### iii. 기존 음악 장르 분류기 구현
각 장르별 TOP100 음원의 wav 파일로 기존 음악 장르 분류기를 구현한다. 장르별로 크롤링했기 때문에 y값은 각 장르로 설정하면 된다. 예를 들어, 2022년 4월 멜론 월간 차트 발라드 TOP100 음악은 모두 '발라드'를 y값으로 한다. feature는 각 wav 파일을 Librosa를 통해 분석한 특징으로 한다. 

##### - Librosa 란?
Librosa는 음원 데이터를 분석하는 라이브러리다. Tempo(BPM), 음파가 양에서 음으로 또는 음에서 양으로 바뀌는 비율인 Zero Crossing Rate, 소리의 "무게 중심"이 어딘지를 알려주는 지표인 Spectral Centroid, 총 스펙트럴 에너지 중 낮은 주파수(85% 이하)에 얼마나 많이 집중되어 있는가를 알려주는 Spectral Rolloff 등의 다양한 기능들을 제공한다.

#### iv. 개선된 음악 장르 분류기
댄스나 힙합의 경우에는 세분화된 장르가 많이 존재하기 때문에 이들의 분류를 더욱 구체화할 것이다. 먼저, 댄스에 해당하는 데이터만을 따로 뽑고 해당 곡들의 앨범 소개 글을 분석한다. 앨범 소개 글을 크롤링하고 TOP 100에 있는 곡의 소개만을 추출한다. 그리고 해당 소개 글을 n 개의 token으로 토크나이징한다. (장르의 명칭 특성상 최대 n=4로 하는 것이 적당하다고 판단했다.) 모든 앨범 소개에 n-gram을 적용하고 이들 중 가장 빈도수가 높은 5개의 세부 장르를 본 프로젝트의 새로운 분류 기준으로 정한다. 최종적으로 Librosa를 통해 분석한 각 음원별 특징을 feature로, 새로운 분류 기준을 y값으로 설정해 기존에 댄스로 분류되었던 곡들의 재분류한다. 예를 들어 레드벨벳의 'Feel My Rhythm' 의 경우에는 댄스 중에서도 트랩 비트를 가지고 있어 댄스-트랩으로 분류를 할 수 있을 것이다.
