## 한국 식당들의 지역별 업종 분석
한국의 다양한 음식점들, 예를 들어 양식, 한식, 중식, 일식 등 업종별 분포를 시각화하여 지역마다 어떤 종류의 음식점이 많은지 한눈에 알아볼 수 있도록 하는 프로젝트입니다.

### 사용 데이터 

행정안전부 LOCALDATA <https://www.localdata.go.kr/datafile/each/07_24_04_P_CSV.zip>

전국 일반음식점 데이터 (~2024-08-31)

---
### 데이터 가공 및 정제

1. 전체 데이터(2,163,574건) 중 폐업한 음식점들의 데이터 drop -> 현재 영업중인 음식점 데이터 추출(697,989건)
2. 전체 Column 중 필요한 Column의 데이터만 추출

```
import pandas as pd
df=pd.read_csv('fulldata.csv', encoding='cp949')

df = df[['상세영업상태코드', '소재지전체주소', '도로명전체주소', '사업장명', '업태구분명', '위생업태명']]

df = df[df.상세영업상태코드 == 1]
df = df.drop('상세영업상태코드', axis=1)

df = df.reset_index(drop=True)
df.to_csv('data.csv')
```

전체 Column ['번호', '개방서비스명', '개방서비스아이디', '개방자치단체코드', '관리번호', '인허가일자', '인허가취소일자',
       '영업상태구분코드', '영업상태명', '상세영업상태코드', '상세영업상태명', '폐업일자', '휴업시작일자', '휴업종료일자',
       '재개업일자', '소재지전화', '소재지면적', '소재지우편번호', '소재지전체주소', '도로명전체주소', '도로명우편번호',
       '사업장명', '최종수정시점', '데이터갱신구분', '데이터갱신일자', '업태구분명', '좌표정보(x)', '좌표정보(y)',
       '위생업태명', '남성종사자수', '여성종사자수', '영업장주변구분명', '등급구분명', '급수시설구분명', '총직원수',
       '본사직원수', '공장사무직직원수', '공장판매직직원수', '공장생산직직원수', '건물소유구분명', '보증액', '월세액',
       '다중이용업소여부', '시설총규모', '전통업소지정번호', '전통업소주된음식', '홈페이지']

추출한 Column ['상세영업상태코드', '소재지전체주소', '도로명전체주소', '사업장명', '업태구분명', '위생업태명']

결과 (data.csv)
||소재지전체주소|도로명전체주소|사업장명|업태구분명|위생업태명|
|-----|---|---|---|---|---|
|0|경기도 하남시 선동 427...|경기도 하남시 미사강변북로...|33떡볶이&꼬마김밥|분식|분식|
|1|인천광역시 중구 운서동...|인천광역시 중구 흰바위로...|중화요리 미식재|중국식|중국식|
|2|부산광역시 동래구 온천동...|부산광역시 동래구 금강공원로...|웨이브라운지 바|호프/통닭|호프/통닭|
|3|부산광역시 동래구 낙민동...|부산광역시 동래구 온천천로...|4242샌드위치 동래점|경양식|경양식|
|4|충청북도 증평군 증평읍 송산리...|충청북도 증평군 증평읍 송산로...|백세장수촌 증평점|한식|한식|


　

 
3. ‘소재지전체주소’, ‘도로명전체주소’ column을 parsing해서 음식점의 ‘시/도’ 정보를 저장하는 ‘주소‘ column을 생성
```
import pandas as pd
df=pd.read_csv('data_ver1.csv')
df.drop('Unnamed: 0', axis=1, inplace=True)

df['주소'] = df['소재지전체주소'].fillna(df['도로명전체주소']).apply(lambda x: x.split()[0])
df = df.reset_index(drop=True)

df.to_csv('data.csv')
```
결과 (data.csv)
||소재지전체주소|...|주소|
|-----|---|---|---|
|0|경기도 하남시 선동 427...|...|경기도|
|1|인천광역시 중구 운서동...|...|인천광역시|
|2|부산광역시 동래구 온천동...|...|부산광역시|
|3|부산광역시 동래구 낙민동...|...|부산광역시|
|4|충청북도 증평군 증평읍 송산리...|...|충청북도|


　

 
4. ‘주소‘ column을 활용해 서울특별시의 음식점을 구 단위로 분류하기 위한 ‘세부주소’ column 생성 후 data_seoul 파일로 저장
```
import pandas as pd
df=pd.read_csv('data_ver2.csv')
df.drop('Unnamed: 0', axis=1, inplace=True)

df = df[df.주소 == '서울특별시']

df['세부주소'] = df['소재지전체주소'].fillna(df['도로명전체주소']).apply(lambda x: x.split()[0] + ' ' + x.split()[1])
df = df.reset_index(drop=True)

df.to_csv('data_seoul.csv')
```
결과 (data_seoul.csv)
||소재지전체주소|...|세부주소|
|-----|---|---|---|
|0|서울특별시 마포구 서교동 346-34|...|서울특별시 마포구|
|1|서울특별시 마포구 망원동 416-14|...|서울특별시 마포구|
|2|서울특별시 마포구 서교동 332-27 |...|서울특별시 마포구|
|3|서울특별시 마포구 연남동 515-12 미스바|...|서울특별시 마포구|
|4|서울특별시 강북구 우이동 72-65 |...|서울특별시 강북구|


　

 
5. ‘주소‘ column을 활용해 경기도의 음식점을 시 단위로 분류하기 위한 ‘세부주소’ column 생성 후 data_gyeonggi 파일로 저장
```
import pandas as pd
df=pd.read_csv('data_ver2.csv')
df.drop('Unnamed: 0', axis=1, inplace=True)

df = df[df.주소 == '경기도']

df['세부주소'] = df['소재지전체주소'].fillna(df['도로명전체주소']).apply(lambda x: x.split()[0] + ' ' + x.split()[1])
df = df.reset_index(drop=True)

df.to_csv('data_gyeonggi.csv')
```
결과 (data_gyeonggi.csv)
||소재지전체주소|...|세부주소|
|-----|---|---|---|
|0|경기도 하남시 선동 427 미사강변 더샵 리버포레|...|경기도 하남시|
|1|경기도 안산시 상록구 수암동 499-1 1층 101호|...|경기도 안산시|
|2|경기도 안산시 상록구 본오동 873-7 리치스빌딩 1층 일부 |...|경기도 안산시|
|3|경기도 하남시 감일동 514-4|...|경기도 하남시|
|4|경기도 파주시 금릉동 434-10|...|경기도 파주시|
---
### 데이터 시각화
지역별 음식점들의 업종 비율을 한눈에 확인하기 위해 Stack Bar Chart를 사용
지역과 업종을 기준으로 그룹화한 데이터를
Y축은 '지역명'으로 설정하고
X축은 '업장들의 수'로 설정하여 시각화 진행

\# 1단계 실행 결과
```
import pandas as pd
import matplotlib.pyplot as plt

# 데이터 준비
df=pd.read_csv('data_ver2.csv')
region_data = df.groupby(['주소', '업태구분명']).size().unstack(fill_value=0)

# 그래프 그리기
region_data.plot(kind='barh', stacked=True, figsize=(10, 6))
plt.title('지역별 음식점의 종류 분포')
plt.xlabel('업장의 수')
plt.ylabel('지역')
plt.legend(title='업장의 종류', bbox_to_anchor=(1.05, 1), loc='upper left')
plt.tight_layout()
plt.show()
```
![image](https://github.com/user-attachments/assets/abba8287-408b-4484-b979-a7b782a633ad)

-> 업장의 종류가 너무 다양해 제대로 된 정보 수집이 불가능하다 판단하여 상위 10개의 종류만 추출하고 나머지는 기타로 묶어서 시각화 진행

\# 2단계 실행 결과
```
import pandas as pd
import matplotlib.pyplot as plt

# 데이터 준비
df = pd.read_csv('data_ver2.csv')

# 업태구분명별로 전체 개수 계산
top_categories = df['업태구분명'].value_counts().head(10).index  # 상위 10개만 선택

# 상위 10개만 남기고 나머지는 "기타"로 묶기
df['업태구분명'] = df['업태구분명'].apply(lambda x: x if x in top_categories else '기타')

# 지역별 업태구분명 계산
region_data = df.groupby(['주소', '업태구분명']).size().unstack(fill_value=0)

# 그래프 그리기
region_data.plot(kind='barh', stacked=True, figsize=(12, 6))
plt.title('지역별 음식점의 종류 분포 (상위 10개 + 기타)')
plt.xlabel('업장의 수')
plt.ylabel('지역')
plt.legend(title='업장의 종류', bbox_to_anchor=(1.05, 1), loc='upper left')
plt.tight_layout()
plt.show()
```
![image](https://github.com/user-attachments/assets/3931b5a7-6239-4445-99ee-cef67651a169)

-> 서울특별시와 경기도의 음식점 수가 너무 많아 다른 지역의 원활한 정보 수집을 위해 서울특별시와 경기도 제외

\# 3단계 실행 결과
```
import pandas as pd
import matplotlib.pyplot as plt

# 데이터 준비
df = pd.read_csv('data_ver2.csv')

# 업태구분명별로 전체 개수 계산
top_categories = df['업태구분명'].value_counts().head(10).index  # 상위 10개만 선택

# 상위 10개만 남기고 나머지는 "기타"로 묶기
df['업태구분명'] = df['업태구분명'].apply(lambda x: x if x in top_categories else '기타')

# 서울특별시와 경기도 제외
filtered_df = df[~df['주소'].str.contains('서울특별시|경기도', na=False)]

# 지역별 업태구분명 계산
region_data = filtered_df.groupby(['주소', '업태구분명']).size().unstack(fill_value=0)

# 그래프 그리기
region_data.plot(kind='barh', stacked=True, figsize=(12, 6))
plt.title('지역별(서울, 경기 제외) 음식점의 종류 분포 (상위 10개 + 기타)')
plt.xlabel('업장의 수')
plt.ylabel('지역')
plt.legend(title='업장의 종류', bbox_to_anchor=(1.05, 1), loc='upper left')
plt.tight_layout()
plt.show()
```
![image](https://github.com/user-attachments/assets/dc6bf049-8cb9-4b7a-878a-99152ac06628)

\# 서울특별시 각 구별 음식점들의 종류 분포
```
import pandas as pd
import matplotlib.pyplot as plt

# 서울특별시 데이터 준비
df = pd.read_csv('data_seoul.csv')

# '세부주소'에서 '서울특별시'를 제거하고 구 이름만 추출
df['세부주소'] = df['세부주소'].str.replace('서울특별시 ', '', regex=False).str.split().str[0]

# 업태구분명별로 전체 개수 계산
top_categories = df['업태구분명'].value_counts().head(10).index  # 상위 10개만 선택

# 상위 10개만 남기고 나머지는 "기타"로 묶기
df['업태구분명'] = df['업태구분명'].apply(lambda x: x if x in top_categories else '기타')

# 지역별 업태구분명 계산
region_data = df.groupby(['세부주소', '업태구분명']).size().unstack(fill_value=0)

# 그래프 그리기
region_data.plot(kind='barh', stacked=True, figsize=(12, 9))
plt.title('서울 각 구별 음식점의 종류 분포 (상위 10개 + 기타)')
plt.xlabel('업장의 수')
plt.ylabel('서울특별시')
plt.legend(title='업장의 종류', bbox_to_anchor=(1.05, 1), loc='upper left')
plt.tight_layout()
plt.show()
```
![image](https://github.com/user-attachments/assets/81b1df9a-0f0e-4b11-b933-9fd583cc33a3)

\# 경기도 각 시별 음식점들의 종류 분포
```
import pandas as pd
import matplotlib.pyplot as plt

# 경기도 데이터 준비
df = pd.read_csv('data_gyeonggi.csv')

# '세부주소'에서 '경기도'를 제거하고 시 이름만 추출
df['세부주소'] = df['세부주소'].str.replace('경기도 ', '', regex=False).str.split().str[0]

# 업태구분명별로 전체 개수 계산
top_categories = df['업태구분명'].value_counts().head(10).index  # 상위 10개만 선택

# 상위 10개만 남기고 나머지는 "기타"로 묶기
df['업태구분명'] = df['업태구분명'].apply(lambda x: x if x in top_categories else '기타')

# 지역별 업태구분명 계산
region_data = df.groupby(['세부주소', '업태구분명']).size().unstack(fill_value=0)

# 그래프 그리기
region_data.plot(kind='barh', stacked=True, figsize=(12, 11))
plt.title('경기도 각 시별 음식점의 종류 분포 (상위 10개 + 기타)')
plt.xlabel('업장의 수')
plt.ylabel('경기도')
plt.legend(title='업장의 종류', bbox_to_anchor=(1.05, 1), loc='upper left')
plt.tight_layout()
plt.show()
```
![image](https://github.com/user-attachments/assets/43be6eea-ddc6-43c1-bc94-ca024459195d)


### 프로젝트 결과

1. 전국 기준
● 한국 각 지역에서 한식이 가장 많은 비중을 차지하고 있으며, 분식과 호프/통닭 업종이 그 뒤를 이어 높은 비율로 분포하고 있음.
● 지역별로 특화된 업종(예: 횟집, 중식 등)이 분명히 드러나, 지역 특성과 소비 트렌드를 반영.

2. 서울특별시 기준
● 강남구는 음식점 수에서 압도적으로 많으며, 다양한 업종이 분포.
● 관악구, 송파구, 마포구 등에서도 음식점 수가 많고 업종 다양성이 나타남.
● 외국 음식 전문점(인도, 태국 등)이 포함되어 용산구 이태원과 같은 지역별 차별화된 특징이 보임.

3. 경기도 기준
● 전국과 마찬가지로 한식과 호프/통닭, 분식이 주요 업종으로 여러 시에 고르게 분포되어 있음.
● 화성시, 수원시, 성남시, 고양시 등 대규모 도시에서는 업종 분포가 다양하며 음식점 수가 상대적으로 많음.

### 결론
지역별 음식점 분포를 분석한 결과, 한식이 대부분의 지역에서 높은 비중을 차지하며, 대도시일수록 음식점 수와 업종의 다양성이 두드러지는 경향을 확인할 수 있었습니다.

