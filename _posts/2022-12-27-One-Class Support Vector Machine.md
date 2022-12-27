---
title: "단일 클래스 서포트 벡터 머신(One-Class Support Vector Machine)"
last_modified_at: 2022-12-27
author: 김상민
---

-------------

이번 포스팅에서는 단일 클래스 분류 기법 중 **단일 클래스 서포트 벡터 머신(One-Class Support Vector Machine, OC-SVM)**에 관한 내용을 공유해보려고 합니다. 

---------------

![ocsvm-1](https://user-images.githubusercontent.com/102953592/209496898-3922e729-296e-4b84-9acf-bdfa860dc8e3.JPG)

> ## OC-SVM 이란?
Support Vector 기반의 Anomaly Detection 방법 중 하나로 Data를 N차원의 Feature space로 mapping 한 뒤, 분류 경계면(Hyper Plane)을 통해 mapping된 Data 중 normal data들이 원점으로부터 최대한 멀어지게 만들어서 분류하는 방법입니다.

![ocsvm-2](https://user-images.githubusercontent.com/102953592/209496923-bd2d4517-b808-4f8d-9ebc-beb5aa319b5c.JPG)

일반적인 SVM과는 차이를 가지고 있는데, Binary-Classification에 많이 쓰이는 SVM은 각 데이터들 속 Support Vector 간의 Margin을 기준으로 Hyper plane을 나누지만, OCSVM은 원점을 기준으로 하기에, 조건에 따라 데이터가 아무리 많아도 Class가 1개일 수 있습니다.


> ## Hyper Plane

![ocsvm-3](https://user-images.githubusercontent.com/102953592/209497733-55945637-6658-45b0-974e-8c930edd5df7.png)

위의 수식에 따라 Hyper plane을 구할 수 있으며 크게 3가지의 텀으로 구분됩니다.
첫 번째는 1/2*llwll^2 텀으로 각 데이터들에 따라 모델의 변동성이 최소화 되도록 보정하는 역할을 수행합니다. 두 번째는 1/vn * sigma(psi) 텀으로 psi는 데이터와 hyper plane간의 거리를 의미합니다. 추후 보정텀이 추가되어, Hyper plane 바깥(비정상)에 있는 점들의 거리의 SUM만 남게됩니다. (비정상일수록 Loss커짐) 세 번째는 p(lo) 텀으로 원점과 Hyper plane간의 거리, 이 값을 빼줆으로써 최소화 문제를 만족할 때 p가 최대가 되도록 합니다.(원점에서 Hyper plane이 최대한 멀어지도록)


다음으로 예제 코드를 통해 OC-SVM을 살펴보겠습니다.

> ## Step 1: Import Libraries
필요한 라이브러리를 임포트 합니다.
``` python
# Synthetic dataset
from sklearn.datasets import make_classification
# Data processing
import pandas as pd
import numpy as np
from collections import Counter
# Visualization
import matplotlib.pyplot as plt
# Model and performance
from sklearn.svm import OneClassSVM
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report
```

> ## Step 2: Create Imbalanced Dataset
불균형 데이터를 생성합니다.
``` python
# Create an imbalanced dataset
X, y = make_classification(n_samples=100000, n_features=2, n_informative=2,
                           n_redundant=0, n_repeated=0, n_classes=2,
                           n_clusters_per_class=1,
                           weights=[0.995, 0.005],
                           class_sep=0.5, random_state=0)
# Convert the data from numpy array to a pandas dataframe
df = pd.DataFrame({'feature1': X[:, 0], 'feature2': X[:, 1], 'target': y})
# Check the target distribution
df['target'].value_counts(normalize = True)
```
![ex-2](https://user-images.githubusercontent.com/102953592/209492799-6a8d46f7-bdd2-4889-9ec7-ccc002b1de1e.JPG)

> ## Step 3: Train Test Split
학습을 위해 훈련 데이터와 테스트 데이터를 분리합니다.
``` python
# Train test split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
# Check the number of records
print('The number of records in the training dataset is', X_train.shape[0])
print('The number of records in the test dataset is', X_test.shape[0])
print(f"The training dataset has {sorted(Counter(y_train).items())[0][1]} records for the majority class and {sorted(Counter(y_train).items())[1][1]} records for the minority class.")
```
![ex-3](https://user-images.githubusercontent.com/102953592/209492810-e7d6567b-fceb-4c06-b0c2-b147f48ec406.JPG)

> ## Step 4: Train One-Class Support Vector Machine (SVM) Model
OCSVM을 학습합니다.
``` python
# Train the one class support vector machine (SVM) model
one_class_svm = OneClassSVM(nu=0.01, kernel = 'rbf', gamma = 'auto').fit(X_train)
```

> ## Step 5: Predict Anomalies
학습한 모델을 바탕으로 결과 값을 예측합니다.
``` python
# Predict the anomalies
prediction = one_class_svm.predict(X_test)
# Change the anomalies' values to make it consistent with the true values
prediction = [1 if i==-1 else 0 for i in prediction]
# Check the model performance
print(classification_report(y_test, prediction))
```
![ex-5](https://user-images.githubusercontent.com/102953592/209492827-efbb8ca0-00ea-4ce0-8c43-228f7543d206.JPG)

> ## Step 6: Customize Predictions Using Scores
점수를 사용하여 예측합니다.
``` python
# Get the scores for the testing dataset
score = one_class_svm.score_samples(X_test)
# Check the score for 2% of outliers
score_threshold = np.percentile(score, 2)
print(f'The customized score threshold for 2% of outliers is {score_threshold:.2f}')
# Check the model performance at 2% threshold
customized_prediction = [1 if i < score_threshold else 0 for i in score]
# # Check the prediction performance
print(classification_report(y_test, customized_prediction))
```
![ex-6](https://user-images.githubusercontent.com/102953592/209492840-65268415-7450-4a96-bece-aeae59bd4b42.JPG)

> ## Step 7: Visualization
결과를 시각화 합니다.
``` python
# Put the testing dataset and predictions in the same dataframe
df_test = pd.DataFrame(X_test, columns=['feature1', 'feature2'])
df_test['y_test'] = y_test
df_test['one_class_svm_prediction'] = prediction
df_test['one_class_svm_prediction_cutomized'] = customized_prediction
# Visualize the actual and predicted anomalies
fig, (ax0, ax1, ax2)=plt.subplots(1,3, sharey=True, figsize=(20,6))
# Ground truth
ax0.set_title('Original')
ax0.scatter(df_test['feature1'], df_test['feature2'], c=df_test['y_test'], cmap='rainbow')
# One-Class SVM Predictions
ax1.set_title('One-Class SVM Predictions')
ax1.scatter(df_test['feature1'], df_test['feature2'], c=df_test['one_class_svm_prediction'], cmap='rainbow')
# One-Class SVM Predictions With Customized Threshold
ax2.set_title('One-Class SVM Predictions With Customized Threshold')
ax2.scatter(df_test['feature1'], df_test['feature2'], c=df_test['one_class_svm_prediction_cutomized'], cmap='rainbow')
```
![ex-7](https://user-images.githubusercontent.com/102953592/209492861-8a7722ad-668e-4593-9e60-85ef9ab91e03.JPG)


이상으로 단일 클래스 서포트 벡터 머신에 대해 알아보았습니다.

------------------------

> **참고 자료**  
* [sklearn](https://scikit-learn.org/stable/modules/generated/sklearn.svm.OneClassSVM.html)
* [Blog](https://limitsinx.tistory.com/147)
* [Blog](https://medium.com/grabngoinfo/one-class-svm-for-anomaly-detection-6c97fdd6d8af)
