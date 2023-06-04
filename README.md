# 청년 창업가를 위한 상권 분석 기반 블루오션 지역 추천 서비스
### 9조 한번해보조

## 목차
1. 프로젝트 소개
2. 사용한 데이터 소개
3. 프로그램 기능
4. 프로그램 소스 코드
5. 실행결과
6. 프로그램의 강점
7. 프로그램 고려사항


## 1. 프로젝트 소개
타겟은 창업에 대한 경험과 자원이 부족한 상태로 가게를 오픈하고 싶은 청년창업가다. 업종을 먼저 선택하고 창업하려는 사람들은, 어느 지역이 나의 창업에 성공을 가져다줄지 결정하는 데 상권 분석 정보가 필요하다. 소상공인시장진흥공단의 무료 상권정보 사이트를 통해 각 지역마다 업종 수, 유동인구, 매출 등 상권에 대한 정보 파악이 가능하지만, 각 지역에 대한 정보를 일일이 찾아 비교해야 하며, 창업자마다 중요하게 생각하는 요소가 다른 점을 반영하지 못한 일방적인 정보 전달 형식으로 창업자들은 유의미한 정보를 간편하게 얻기 어렵다. 

따라서 본 데이터 분석 프로젝트에서는 공공데이터를 활용해 서울시내 블루오션을 분석해 청년 창업가들에게 창업이 성공할 확률이 높은 최적의 조건을 가진 지역을 추천한다.

## 2. 사용한 데이터 소개

### 2-1. 소상공인시장진흥공단_상가(상권)정보 업종코드_20230228.csv
2023년 2월 기준 소상공인 상업 업종코드를 분류한 csv 파일을 소상공인시장진흥공단에서 다운받아 이용했다. 사용자가 직접 입력해야하기 때문에 입력 편의를 위해서 대분류명, 중분류명을 사용하였다. 
- 활용 목적: 사용자가 어떤 분야의 창업을 원하는지 공인된 기준으로 입력받기 위해 활용하였다. 이후 잠재소비자 수를 업종별로 저장할 때 이용된다.
- 전처리 과정: 전처리를 거치지 않고 이용하였다.

### 2-2. 소상공인시장진흥공단_상가(상권)정보_서울_202303.csv
2023년 3월 기준, 영업 중인 전국 상가업소 데이터를 제공한다. 소상공인시장진흥공단에서 제작하였으며, 공공데이터 포털에서 csv형식의 원본 데이터파일을 다운받아 이용했다. 각 상가업소의 상권업종코드와 위치한 주소, 행정동코드 등이 표시되어있다.
- 활용 목적: 해당 데이터는 블루오션 필터링 과정에서 입력받은 업종의 상가의 개수를 지역별로 알아내 동종업계 수를 파악하는 데에 사용된다. 또한 추천지역 제공이 끝나고, 추천지역의 동종업계 상가 업소의 수를 시각화하여 나타낼 때 이용된다.
- 전처리 과정: 데이터 검사결과 NaN값이 없는 것으로 확인되었고, 특정 수치가 아니라 각 상가업소 정보를 제공하기 때문에 전처리를 거치지 않고 이용하였다.

### 2-3. 행정동 단위 서울 생활인구.csv
서울시가 보유한 공공데이터와 통신데이터로 측정한 특정시점에 서울의 특정 지역에 존재하는 모든인구수 정보이다. 서울특별시에서 제작했으며, 서울열린데이터광장 홈페이지에서 csv파일 원본을 다운받아 이용했다.
- 활용 목적: 특정 지역에 존재하는 인구수는 해당 지역에서 거주하는 사람과 유동인구 등을 반영한다. 따라서 생활인구가 많을수록 상권이 활발할 가능성이 높다고 판단했고, 생활인구를 블루오션을 판단하기 위한 지표로 삼았다.
전처리 과정에서 행정동 코드별 총생활인구를 전부 더한 데이터는 블루오션 필터링 과정에서 동종업계 대비 생활인구가 높은 지역을 출력할 때 사용된다.
- 전처리 과정: 나잇대별 생활인구 열은 따로 이용되지 않으므로 제거했다. 이후 새롭게 만든 데이터프레임과 groupby함수를 이용해 행정동코드가 같은 행의 생활인구를 모두 더해주었다. 생활인구 데이터에는 기준일/시간대 별로 총생활인구가 나눠진다. 행정동코드를 기준으로 생활인구를 더하면 한 달 동안의 지역별(행정동별) 생활인구를 전부 알 수 있다. 또한 2023.02.01~23.02.28까지의 생활인구가 다 담겨있기 때문에, 총 생활인구를 28일로 나누어 하루 생활인구 평균을 이용했다.

### 2-4. 서울시 우리마을가게 상권분석서비스(상권-소득소비).csv
서울 열린데이터 광장에 올라와 있는 상권분석서비스의 csv 파일로, 상권마다의 분야별 지출의 총 금액 데이터를 담고 있다. 2023년 3월에 올라온 2022년 4분기까지의 데이터이다. 
- 활용 목적: 상권의 분야마다 소비자 지출 수준을 파악하여 잠재소비자 계산에 활용한다. 선택한 분야의 총 지출 금액이 크면 잠재소비자가 많은 것으로 가정하고 점수화 과정에서 요인으로 활용한다. 
- 전처리 과정:  해당 데이터는 상권별로 분류되어 있어 각 상권을 행정동으로 분류해주는 과정을 거쳤다. 먼저 ‘서울시 우리마을가게 상권분석서비스(상권영역)’ 데이터를 불러왔고, 이 데이터를 판다스의 merge 함수를 사용하여 대상 데이터와 병합시켰다. 행정동명 컬럼을 추가하기 위해 행정동 데이터를 한 번 더 합쳤다. 필요없는 컬럼을 제거한 후 groupby() 메서드를 활용하여 분기별, 상권별로 세분화되어있는 값을 읍면동명(행정동명과 동일함)을 기준으로 합하였다. 

### 2-5. 서울시 우리마을가게 상권분석서비스(상권영역).csv

서울 열린데이터 광장에 올라와 있는 상권분석서비스의 csv 파일로, 2021년 12월 기준의 상권코드와 행정동코드 데이터를 담고 있다. 
- 활용 목적: 상권 코드로 분류되어 있는 데이터들에 행정동코드를 연결해주기 위해 상권코드와 행정동코드를 모두 포함하고 있는 상권영역 데이터가 필요하다.
- 전처리 과정: 소득소비 데이터와 신 상권 점포 데이터 전처리 과정에서 불필요한 열을 제거했다. 

### 2-6. 서울시_우리마을가게_상권분석서비스(신_상권_점포)_2021년.csv

서울 열린데이터 광장에 올라와 있는 상권분석서비스의 csv 파일로, 상권마다의 개・폐업 데이터를 담고 있다. 2022년 4월에 올라온 2021년 4분기까지의 데이터이다.
- 활용 목적: 지역별로 폐업률을 계산하여 점수화 요인으로 사용한다. 폐업 점포 수 총합을 점포 수 총합으로 나누어 행정동의 폐업률을 알아낸다.
- 전처리 과정: 신 상권 점포 데이터를 상권 코드 기준으로 상권영역 데이터와 병합했다. 사용하지 않을 컬럼은 모두 제거하고 컬럼명은 활용하기 편한 이름으로 변경하였다. 상권별로 구분되어 있는 이 데이터에서 행정동의 폐업률을 구하기 위해 행정동별로 폐업 점포 수와 전체 점포 수를 각각 더하였다.  기존 데이터프레임에 폐업률 컬럼을 새롭게 추가하여 폐업 점포 수/점포 수*100의 공식을 적용한 값을 저장하였다. 행정동 컬럼을 추가하기 위해 생활인구 데이터 전처리에 사용했던 행정동 데이터를 병합했다. 하나의 행정동이 여러 구에 걸쳐있어 행정동코드는 다르지만 행정동명은 같은 동들로 인해 오류가 발생한다. 따라서 중복되는 행정동들은 그 값을 평균내어 통일하였다.


### 2-7. 한국부동산원_상업용부동산 임대동향조사_임대정보_분기별 지역별 임대료(소규모상가).csv
한국부동산원에서 제공한 해당 데이터의 원형태는 상권 인덱스에 대해 2022_1분기, 2022_2분기 컬럼만 존재하는 csv 파일이었기에, 직접 행정구 컬럼을 추가해 데이터를 추가해 새로 데이터셋을 구축했다.
- 활용 목적: 지역별로 임대료를 계산하여 점수화 요인으로 사용한다. 
- 전처리 과정: 행정구 코드에 행정동 코드를 결합하기 위해, 컬럼명을 ‘시군구명'으로 바꾸어준다. 필요없는 컬럼을 삭제하고, 같은 행정구를 공유하는 행끼리 평균 내어 하나의 행으로 합쳐준다. df_dong에 있는 행정동 데이터 프레임과 합쳐 하나의 데이터 프레임을 만들고, ‘읍면동'을 인덱스화 해주었다. 중복 데이터를 제거하고, 동일 행정동명을 공유하는 경우, 평균 내어 하나의 행으로 합쳐주었다. 


### 2-8. 서울시_우리마을가게_상권분석서비스(신_상권_추정매출)_2021년.csv
서울 열린데이터 광장에 올라와 있는 상권분석서비스의 csv 파일로, 2021년 기준 업종별 분기당 매출 데이터다. 데이터 원본을 csv파일로 다운받아 이용하였다.
- 활용 목적: 지역별로 매출을 계산하여 점수화 요인으로 사용한다. 
- 전처리 과정: 사용할 '상권_코드', '상권_코드_명', '서비스_업종_코드', '서비스_업종_코드_명', '주중_매출_금액', '주말_매출_금액' 컬럼만 남기고, '주중매출_금액'과 '주말_매출_금액' 컬럼을 더한 '총_매출_금액' 컬럼을 생성한다. 행정동 코드를 컬럼으로 더해주기 위해, 행정동 코드를 컬럼으로 가지고 있는 다른 상권 코드 데이터셋을 불러와 상권 코드를 기준으로 두 데이터를 붙여준다. 인덱스를 행정동 코드로 지정해주고, '행정동_코드'를 기준으로 그룹화하여 '총_매출_금액'  평균 계산한다. 행정동 데이터를 가져와 행정동명을 더해준다. 중복되는 행정동은 그 값을 평균내어 통일한다.


### 2-9. KIKmix
행정안전부 공식 홈페이지 업무안내-지방자치균형발전실-주민등록, 인감 페이지에 게시된 2023. 2. 24. 행정기관(행정동) 및 관할구역(법정동) 현황이다. 행정동코드, 시도명, 시군구명, 읍면동명 등으로 전국의 지역을 나타낸다. 데이터 원본을 csv파일로 다운받아 이용하였다.
- 활용 목적: 이 데이터는 전처리과정에서 행정동코드에 따른 시도명, 시군구명, 읍면동명을 데이터프레임에 붙이는 데 사용된다.
- 전처리 과정: 데이터를 불러오고, 법정동코드, 동리명, 생성일자, 말소일자는 전부 제거했다. 또한 전국의 지역 이름을 담고 있기 때문에 프로그램에서 사용할 서울특별시 지역 외의 지역은 모두 삭제했다. NaN값이 포함된 행은 읍면동명이 없는 데이터인데, 프로그램에서는 읍면동별로 지역을 나누었기 때문에 전부 삭제해주었다. 또한 법정동코드와 합쳐진 데이터이기 떄문에 여러 개의 법정동코드가 한 행정동코드에 속하는 경우도 있었다. 이 경우 중복된 행정동코드 값들을 모두 지워주었다. 다음으로 데이터를 다른 데이터와 합칠 수 있게 형식 등을 맞추고, merge함수를 이용해 행정동 코드가 일치하는 곳에 데이터프레임을 합쳤다.


## 3. 프로그램 기능

프로그램의 목적은 사용자에게 원하는 업종을 입력받아, 생활인구, 동종업계 수, 예상매출, 폐업률, 상권 소득소비 값을 고려한 블루오션 추천 서비스를 제공하는 것이다. 사용자에게 업종을 입력받고, 추천지역을 보여주는 과정에서 다음 기능을 사용한다.

- 업종 대분류 입력시 중분류 출력
- 블루오션 필터링 상위지역 출력
- 점수화 상위 3개 지역 출력
- 상위 3개 지역에 대한 분석 시각화 출력
- 상위 3개 지역에 대한 업종 지도에 표시







