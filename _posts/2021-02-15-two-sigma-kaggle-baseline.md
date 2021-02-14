---  
layout: post  
title: "[Kaggle]Two Sigma Connect:Rental Listing Inquiries Baseline 잡기"  
subtitle: "[Kaggle]Two Sigma Connect:Rental Listing Inquiries Baseline 잡기"  
categories: Data
tags: ML Kaggle Classification
comments: true  


---  
## [Two Sigma Connect:Rental Listing Inquiries](https://www.kaggle.com/c/two-sigma-connect-rental-listing-inquiries/overview)

## 문제 설명
- New York 시의 부동산 선호도 예측 문제.


## Data fields
- bathrooms: number of bathrooms
- bedrooms: number of bathrooms
- building_id
- created
- description
- display_address
- features: a list of features about this apartment
- latitude
- listing_id
- longitude
- manager_id
- photos: a list of photo links. You are welcome to download the pictures yourselves from renthop's site, but they are the same as imgs.zip.
- price: in USD
- street_address
- interest_level: this is the `target variable`. It has 3 categories: '`high`', '`medium`', '`low`'

# Make Quick Baseline

- 간단한 파생변수, 통계량, 날짜변수 추가.
- Text data를 tf-idf, count-vectorizer, SVD로 유효한 피쳐로 변환
- catbosot로 baseline모델 구축

## 1. 데이터 가져오기
- 캐글내의 노트북을 이용할 경우 다음 경로에 train, test, submission data가 저장되어있다.
- data 전처리에 필요한 pandas와 numpy를 가져온다.
- train, test 데이터를 펼쳐볼 시 컬럼이 잘릴 수 있다. display option에서 보여지는 컬럼의 수를 늘려준다.


```python
# This Python 3 environment comes with many helpful analytics libraries installed
# It is defined by the kaggle/python Docker image: https://github.com/kaggle/docker-python
# For example, here's several helpful packages to load

import numpy as np # linear algebra
import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)

# Input data files are available in the read-only "../input/" directory
# For example, running this (by clicking run or pressing Shift+Enter) will list all files under the input directory

import os
for dirname, _, filenames in os.walk('/kaggle/input'):
    for filename in filenames:
        print(os.path.join(dirname, filename))

# You can write up to 20GB to the current directory (/kaggle/working/) that gets preserved as output when you create a version using "Save & Run All"
# You can also write temporary files to /kaggle/temp/, but they won't be saved outside of the current session
```

    /kaggle/input/two-sigma-connect-rental-listing-inquiries/images_sample.zip
    /kaggle/input/two-sigma-connect-rental-listing-inquiries/Kaggle-renthop.torrent
    /kaggle/input/two-sigma-connect-rental-listing-inquiries/sample_submission.csv.zip
    /kaggle/input/two-sigma-connect-rental-listing-inquiries/test.json.zip
    /kaggle/input/two-sigma-connect-rental-listing-inquiries/train.json.zip



```python
pd.options.display.max_columns = 999


train = pd.read_json("/kaggle/input/two-sigma-connect-rental-listing-inquiries/train.json.zip")
test = pd.read_json("/kaggle/input/two-sigma-connect-rental-listing-inquiries/test.json.zip")
```

- train 과 test를 합치는 작업을 먼저 한다. 이는 전처리를 일괄적으로 적용하기 위해서이다.


```python
all_data = pd.concat([train, test])
```

## 2. categorical 변수들을 dummy로 변환
- LabelEncoder를 활용하여 `building_id`, `manager_id`, `display_address`, `street_address`를 숫자처리
- NaN의 경우 input을 list로 변환 시 똑같이 숫자처리된다.


```python
from sklearn.preprocessing import LabelEncoder

le = LabelEncoder()
C = ['building_id','manager_id','display_address','street_address']
## LE

for i in C:
    all_data[i] = le.fit_transform(list(all_data[i]))  # Nan 은 숫자처리 # list를 만나면 카테고리로 변함

```

## 3. 날짜 변수 추가
- 날짜 변수가 있는경우 pd.to_datatime과 dt로 다양한 시간 변수를 생성할 수 있다.
- 변수중의 created로 `year`, `month`, `day`, `hour`, `weekday`, `minute`, `second`등을 추가


```python
## 날짜 변수 추가

all_data['created'] = pd.to_datetime(all_data['created'])
# all_data['year'] = all_data['created'].dt.year # year는 2016으로 모두 동일
all_data['month'] = all_data['created'].dt.month
all_data['day'] = all_data['created'].dt.day
all_data['hour'] = all_data['created'].dt.hour
all_data['weekday'] = all_data['created'].dt.weekday
all_data['minute'] = all_data['created'].dt.minute
all_data['second'] = all_data['created'].dt.second
```

- categorical variable vs categorical variable의 경우 countplot을 통해 유효한 변수인지 확인할 수 있다.
- 각 시간과 interst_level과의 관계를 확인할 수 있다.


```python
import matplotlib.pyplot as plt
import seaborn as sns
plt.figure(figsize=(20,12))

sns.countplot(all_data['weekday'], hue = all_data['interest_level'])
```

![png1](https://yunsikus.github.io/assets/img/post_img/two-sigma-text_1.jpg)




```python
import matplotlib.pyplot as plt
import seaborn as sns
plt.figure(figsize=(20,12))

sns.countplot(all_data['day'], hue = all_data['interest_level'])
```

![png2](https://yunsikus.github.io/assets/img/post_img/two-sigma-text_2.jpg)



## 4. 파생변수 추가 (단순 길이 추가)

- 사진의 길이 (사진이 없는 경우와 있는 경우의 차이가 큼)
- feature의 개수 (feature가 많을 수록 interest가 높아짐)
- description의 길이 (성의있는 description일수록 interest가 높아짐)


```python
## 개수 변수 추가

all_data['num_photos'] = all_data['photos'].apply(len)
all_data['num_features'] = all_data['features'].apply(len)
all_data['num_description'] = all_data['description'].apply(lambda x: len(x.split()))
```

## 5. text data를 학습 가능한 수치형 데이터로 변환
- countvector와 tf-idf vector로 변환.


```python
from sklearn.feature_extraction.text import CountVectorizer, TfidfVectorizer

cv = CountVectorizer()
cv_description = cv.fit_transform(all_data['description'])

tf = TfidfVectorizer()
tf_description = tf.fit_transform(all_data['description'])
```

- 10개의 vector로 축소하여 all_data에 병합.


```python
## 차원축소 5만개 -> 10개
from sklearn.decomposition import TruncatedSVD

svd = TruncatedSVD(n_components = 10)
svd_tf_description = svd.fit_transform(tf_description)
svd_cv_description = svd.fit_transform(cv_description)

all_data = pd.concat([pd.DataFrame(svd_tf_description),pd.DataFrame(svd_cv_description), all_data.reset_index(drop=True)],1) # all_data에 계속 추가
```

## 6. 통계량 추가
- groupby agg로 mean, count, min, max, std등 다양한 통계량을 추가


```python
## 통계량 추가

manager_price = all_data.groupby('manager_id')['price'].agg(['mean', 'count', 'min', 'max', 'std']) # 통계량 추가
display_address_price = all_data.groupby('display_address')['price'].agg(['mean', 'count', 'min', 'max', 'std'])
street_address_price = all_data.groupby('street_address')['price'].agg(['mean','count', 'min', 'max', 'std'])  

all_data = pd.merge(all_data, manager_price, how = 'left', on = 'manager_id')
all_data = pd.merge(all_data, display_address_price, how = 'left', on = 'display_address')
all_data = pd.merge(all_data, street_address_price, how = 'left', on = 'street_address')
```

# 7. train전 간단한 전처리
- all_data에서 수치형을 제외한 다른 컬럼 삭제
- 결측치 처리는 0보다는 -1이 효과가 더 높음(0이 들어있는 값과 의미가 겹칠수가 있다.)
- 다시 train data와 test data로 분리


```python
# 결측치 처리
all_data2 = all_data[all_data.columns[all_data.dtypes != 'object']]
all_data2 = all_data2.drop(['created'],1)
all_data2 = all_data2.fillna(-1)
```


```python
train2 = all_data2[:len(train)]
test2 = all_data2.iloc[len(train):]
```

# 8. RF 혹은 Boosting모델을 사용
- rf, xgboost, lgbm, catboost 를 mean stack
- 일단 lgbm만 적용
- classification의 경우 metric에 따라 porbability를 제출하는게 유리한 경우가 있음


```python
# from lightgbm import LGBMClassifier
# lgc = LGBMClassifier(learning_rate = 0.1)
# lgc.fit(np.array(train2), train['interest_level'])
# result = lgc.predict_proba(np.array(test2))
```


```python
# from xgboost import XGBClassifier
# xgb = XGBClassifier(learning_rate = 0.1)
# xgb.fit(np.array(train2), train['interest_level'])
# result = xgb.predict_proba(np.array(test2))
```


```python
from catboost import CatBoostClassifier
cbc = CatBoostClassifier(learning_rate = 0.1, verbose=20)
cbc.fit(np.array(train2), train['interest_level'])
result = cbc.predict_proba(np.array(test2))
```



# 9. submission파일 제출
- 대회에서 제공해주는 sub파일을 가져와서 제출용 파일로 활용
- 제출용 파일의 컬럼순서는 꼭 sub파일의 순서일 필요는 없다.


```python
sub = pd.read_csv("/kaggle/input/two-sigma-connect-rental-listing-inquiries/sample_submission.csv.zip")
sub.columns = ['listing_id', 'high', 'low', 'medium'] # 꼭 sub 순서일 필요는 없다.
sub.iloc[:,1:] = result
sub.head()
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
      <th>listing_id</th>
      <th>high</th>
      <th>low</th>
      <th>medium</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7142618</td>
      <td>0.104585</td>
      <td>0.533640</td>
      <td>0.361775</td>
    </tr>
    <tr>
      <th>1</th>
      <td>7210040</td>
      <td>0.079495</td>
      <td>0.791218</td>
      <td>0.129288</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7174566</td>
      <td>0.003735</td>
      <td>0.969333</td>
      <td>0.026932</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7191391</td>
      <td>0.154144</td>
      <td>0.369938</td>
      <td>0.475918</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7171695</td>
      <td>0.013626</td>
      <td>0.784977</td>
      <td>0.201398</td>
    </tr>
  </tbody>
</table>
</div>




```python
sub.to_csv('result_cbc.csv',index = 0) # baseline 코드로 public score 0.56537
```
