---
title: "Keras 모델 학습 커스터마이징"
last_modified_at: "2021-12-30"
author: "김수민"

---

이번 포스팅에서는 keras에서 학습을 커스터마이징하는 방법에 대해 이야기 해보도록 하겠습니다.

------
## 목차
[1. 훈련루프 직접 정의](#1-훈련루프-직접-정의)

[2. Model 클래스 재 정의](#2-model-클래스-재-정의)

[3. References](#3-references)

------

우선 keras에서 모델을 학습하는 데에는 fit()을 사용해 간단하게 진행할 수 있습니다.

이때 fit에서 수행되는 방식 외에 사용자가 정의할 수 있도록 커스터마이징이 필요한 경우 <u>훈련루프를 처음부터 정의해 작성하는 방법</u>과 <u>Model 클래스를 재 정의하는 방법</u>이 있습니다.



## 1. 훈련루프 직접 정의

학습을 커스터마이징 할 때 처음부터 정의하여 작성하는 경우 모든 세부사항을 사용자가 원하는대로 정의할 수 있다는 장점이 있습니다.

하지만 학습 과정에서 필요로 하는 각 연산의 미분값을 세세히 설정하는데는 많은 노력이 필요합니다. 

이에 tensorflow에서는 특정 구문 속의 연산들을 기록하고 자동으로 미분할 수 있도록 GradientTape API를 제공합니다.

---

### - GradientTape을 사용한 직접 구현

GradientTape은 선언된 구문 안의 연산들에 대해 어떤 연산이 어떤 순서로 발생하는지 기록합니다.

이후 gradient() 수행 시 구문 안에 작성된 연산들을 자동 미분합니다.

```python
x = tf.constant(3.0)
with tf.GradientTape() as g:
	g.watch(x)
	y = x * x
dy_dx = g.gradient(y, x)
print(dy_dx)
```

예를 들어 다음과 같이 x=3, y=x^2인 경우 gradient값을 구하면 gradient(y, x)에서는 y식의 미분 값인 2x 값 6을 출력하게 됩니다.

---

이제 앞서 설명한 GradientTape을 이용해 학습을 직접 구현하는 예시를 보겠습니다.

우선 모델을 tf.keras.Model로 생성합니다.

본 예제에서는 MSE(MeanSquareError)을 loss로, SGD를 optimizer로 하는 모델을 생성했습니다.

```python
def new_model():
	input = tf.keras.layers.Input((2,2,1))
	n1 = NewLayer(1)(input)
	output = tf.keras.Model(inputs=input, outputs=n1)
	return output

model = new_model()
loss_function = tf.keras.losses.MeanSquaredError()
optimizer = tf.keras.optimizers.SGD(learning_rate=0.01)
train_loss = tf.keras.metrics.Mean()
```

그 후 아래의 함수를 tf.function 데코레이션을 붙여 선언합니다.

```python
@tf.function
def train_step(images, labels):
	# GradientTape 적용
	with tf.GradientTape() as tape:
		# 1. 예측
        	predictions = model(images)
		tf.print("pred:",predictions,"\n")
        	# 2. loss 계산
        	loss = loss_function(labels, predictions)
	# 3. gradients 계산
	gradients = tape.gradient(loss, model.trainable_variables)
	tf.print("gradients:",gradients,"\n")
	# 4. weight 업데이트
	optimizer.apply_gradients(zip(gradients, model.trainable_variables))
	# 5. loss 업데이트
	train_loss(loss)
```

함수 내부에서는 모델에서 1차적으로 예측을 하고 예측한 값을 기반으로 정답과의 loss를 계산, 이후 오차 역전파를 수행합니다.

오차 역전파의 경우 앞서 말했던 TapeGradient의 gradient()를 통해 자동 미분 함수를 적용한 값을 지정한 optimizer에 적용함으로써 수행합니다.

gradient 계산 시 trainable_variables는 레이어에서 생성한 변수들 중 trainable 설정값이 True(default값)인 변수들을 자동으로 불러와 입력합니다.

계산된 gradient값들을 설정한 optimizer에 적용하면 optimizer의 종류에 따라 다양하게 계산되어 다음 weight값이 설정됩니다.

optimizer에 batch에 따라 적용하는 방법, 방향성을 고려한 방법 등 다양한 종류가 있으며, 이에 따른 성능 차이가 존재해 모델 학습 성능 향상을 위해 다양하게 적용해 볼 필요가 있습니다.

![img](https://blog.kakaocdn.net/dn/bQ934t/btqASyVqeeD/ozNDSKWvAbxiJb7VtgLkSk/img.png)

<출처 : https://www.slideshare.net/yongho/ss-79607172 >

학습 수행 코드는 위에 정의한 함수를 호출하면 됩니다.

```python
EPOCHS = 4
for epoch in range(EPOCHS):
	for images, labels in zip(ex,sy):
		train_step(images, labels)
	template = '에포크: {}, 손실: {:.5f}'
	print(template.format(epoch + 1, train_loss.result()))
```

---

### - 직접 정의 방법

GradientTape을 사용할 때, 역전파에 적용하고 싶은 수식이 다른 경우가 발생할 때 좀 더 낮은 레벨에서 수정이 가능합니다.

역전파 적용시 직접적인 수식을 변경하는 방법으로는 @tf.custom_gradient 를 통해 수식에서 순전파, 역전파 수식 모두를 정의하는 방법이 있습니다.

```python
@tf.custom_gradient
def bar(x, y):
	def grad(upstream):
		dz_dx = y
		dz_dy = x
		return upstream * dz_dx, upstream * dz_dy
	z = x * y
	return z, grad
```

다음과 같이 @tf.custom_gradient 데코레이션을 붙인 함수를 선언할 때, 내부에 원하는 역전파 수식을 갖는 함수를 선언하고 반환하면 역전파 연산 시 수정된 내용으로 연산이 진행됩니다.

```python
x = tf.constant([2.0, 5.0], dtype=tf.float32)
y = tf.constant([3.0, 8.0], dtype=tf.float32)
with tf.GradientTape(persistent=True) as tape:
	tape.watch(x)
	tape.watch(y)
	z = bar(x, y)
print(z)                  # tf.Tensor([ 6. 40.], shape=(2,), dtype=float32)
print(tape.gradient(z,x)) # tf.Tensor([3. 8.], shape=(2,), dtype=float32)
print(tape.gradient(z,y)) # tf.Tensor([2. 5.], shape=(2,), dtype=float32)
```

<출처 : https://www.tensorflow.org/api_docs/python/tf/custom_gradient>

---

## 2. Model 클래스 재 정의

모델 학습을 커스터마이징하는 또 하나의 방법으로는 Model 클래스를 재정의해 fit()을 사용하는 방법입니다. 

fit()을 그대로 사용하게 된다면 콜백, 내장 배포 지원 또는 단계 융합같은 편리한 특성들은 계속해서 사용할 수 있고, 훈련 내용만 재 정의 할 수 있다는 장점이 있습니다.

fit()을 재 정의 하기 위해서는 Model 클래스의 훈련 단계 함수인 train_step()을 재정의 해야 합니다.

해당 함수를 커스터마이징 하는 방법은 Model 클래스를 하위 클래스화 하는 새 클래스를 만드는 방법이 있습니다.

### - Custom Model 생성

```python
class CustomModel(keras.Model):
    def train_step(self, data):
        # Unpack the data. Its structure depends on your model and
        # on what you pass to `fit()`.
        x, y = data

        with tf.GradientTape() as tape:
            y_pred = self(x, training=True)  # Forward pass
            # Compute the loss value
            # (the loss function is configured in `compile()`)
            loss = self.compiled_loss(y, y_pred, regularization_losses=self.losses)

        # Compute gradients
        trainable_vars = self.trainable_variables
        gradients = tape.gradient(loss, trainable_vars)
        # Update weights
        self.optimizer.apply_gradients(zip(gradients, trainable_vars))
        # Update metrics (includes the metric that tracks the loss)
        self.compiled_metrics.update_state(y, y_pred)
        # Return a dict mapping metric names to current value
        return {m.name: m.result() for m in self.metrics}
```

<출처 : https://tensorflow.org/guide/keras/customizing_what_happens_in_fit?hl=ko>

train_step 함수에서는 fit()에서 입력받은 데이터를 가지고 내부 연산 결과를 적용하게 되어 내부 내용은 위에 직접 구현하는 내용과 동일합니다.

다만 이때 loss와 optimizer은 self.compiled_loss, self.optimizer을 통해 손실을 계산하며, compile()로 전달된 손실함수와 optimizer을 래핑해 사용하게 됩니다.

해당 모델을 사용할 때는 아래와 같이 원래 keras.Model 대신 CustomModel을 사용해 모델을 생성하면 됩니다.

```python
import numpy as np

# Construct and compile an instance of CustomModel
inputs = keras.Input(shape=(32,))
outputs = keras.layers.Dense(1)(inputs)
model = CustomModel(inputs, outputs)
model.compile(optimizer="adam", loss="mse", metrics=["mae"])

# Just use `fit` as usual
x = np.random.random((1000, 32))
y = np.random.random((1000, 1))
model.fit(x, y, epochs=3)
```

<출처 : https://tensorflow.org/guide/keras/customizing_what_happens_in_fit?hl=ko>

---

이번 포스팅에서는 keras에서 학습을 커스터마이징 하는 방법에 대해 알아보았습니다.

GradientTape는 연산을 기록해 gradient()로 자동 미분을 도와주어 오차 역전파 시에 구현에 편의성을 높였습니다.

역전파 시의 수식을 직접 변경하기 위한 @tf.custom_gradients에 대해서도 간단한 예시를 살펴보았으며

학습 수행 시 fit()을 그대로 사용하기 위한 모델 커스터마이징 방안도 살펴보았습니다.

이상입니다.

---

## 3. References

https://www.tensorflow.org/guide/keras/customizing_what_happens_in_fit?hl=ko

https://www.tensorflow.org/api_docs/python/tf/GradientTape

https://www.tensorflow.org/guide/function

https://www.tensorflow.org/api_docs/python/tf/custom_gradient

https://teddylee777.github.io/tensorflow/gradient-tape

