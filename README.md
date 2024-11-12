## 한국 식당들의 지역별 업종 분석

### 사용 데이터 

행정안전부 LOCALDATA <https://www.localdata.go.kr/datafile/each/07_24_04_P_CSV.zip>

전국 일반음식점 데이터 (~2024-08-31)

---
### 데이터 가공

#### 전체 데이터(2,163,574건) 중 폐업한 음식점들의 데이터 drop -> 현재 영업중인 음식점 데이터 추출(697,989건)
#### 전체 Column 중 필요한 Column의 데이터만 추출

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

결과
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
결과
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
결과
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
결과
||소재지전체주소|...|세부주소|
|-----|---|---|---|
|0|경기도 하남시 선동 427 미사강변 더샵 리버포레|...|경기도 하남시|
|1|경기도 안산시 상록구 수암동 499-1 1층 101호|...|경기도 안산시|
|2|경기도 안산시 상록구 본오동 873-7 리치스빌딩 1층 일부 |...|경기도 안산시|
|3|경기도 하남시 감일동 514-4|...|경기도 하남시|
|4|경기도 파주시 금릉동 434-10|...|경기도 파주시|
---
### 데이터 시각화
