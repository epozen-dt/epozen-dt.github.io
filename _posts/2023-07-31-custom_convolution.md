---
title: "데이터 중심의 AI"
last_modified_at: 2023-07-31
author: 최선빈
---

본 포스팅은 Custom Convolutionn에 대한 내용입니다.

---
&nbsp;

> ## 커스텀 컨볼루션(CustomConvolution)

&nbsp;

커스텀 컨볼루션은 사용자가 직접 컨볼루션 레이어를 정의하여 원하는 방식으로 컨볼루션 연산을 수행하는 기법을 말합니다. 일반적으로 컨볼루션 연산은 이미 정해진 커널과 입력 이미지 간의 합성곱 연산을 수행하는데, 커스텀 컨볼루션은 사용자가 원하는 형태의 커널을 정의하고 이를 사용하여 컨볼루션 연산을 수행할 수 있도록 합니다.

커스텀 컨볼루션을 사용하면, 기존의 컨볼루션 연산과는 다른 방식으로 입력 이미지와 커널 간의 연산을 조정할 수 있습니다. 예를 들어, 특정 패턴이나 특징을 강조하는 커널을 정의하거나, 특정 문제에 더 적합한 컨볼루션 기법을 사용할 수 있습니다.

커스텀 컨볼루션은 파이토치와 같은 딥러닝 프레임워크에서 제공하는 기본적인 컨볼루션 레이어를 확장하거나, 새로 정의하여 구현 합니다.

커스텀 컨볼루션을 이용하여 이미지 분류 모델을 설계를 해보겠습니다.

---
&nbsp;

> ## 구현

&nbsp;

다음 코드는 모델의 구현 예시입니다.

    class CustomConv2d(nn.Module):
        def __init__(self, in_channels, out_channels, kernel_size, stride=1, padding=0):
            super(CustomConv2d, self).__init__()

            self.in_channels = in_channels
            self.out_channels = out_channels
            self.kernel_size = kernel_size
            self.stride = stride
            self.padding = padding

            # 가중치 텐서와 편향 텐서를 정의합니다.
            self.weight = nn.Parameter(torch.Tensor(out_channels, in_channels, kernel_size, kernel_size))
            self.bias = nn.Parameter(torch.Tensor(out_channels))

            # 가중치와 편향을 초기화하는 메서드를 호출하여 초기화합니다.
            self.reset_parameters()

        def reset_parameters(self):
            # 가중치를 Kaiming He 초기화 방법으로 초기화합니다.
            nn.init.kaiming_uniform_(self.weight, a=math.sqrt(5))

            # 편향이 None이 아닌 경우에만 편향을 초기화합니다.
            if self.bias is not None:
                # 편향 초기화를 위해 가중치의 fan_in 값을 계산합니다.
                fan_in, _ = nn.init._calculate_fan_in_and_fan_out(self.weight)

                # bound 값으로 -1/sqrt(fan_in)부터 1/sqrt(fan_in) 사이의 값을 가지도록 편향을 초기화합니다.
                bound = 1 / math.sqrt(fan_in)
                nn.init.uniform_(self.bias, -bound, bound)

        def forward(self, input,get_features=False):
            # 주어진 입력(input)에 대해 컨볼루션 연산을 수행하여 결과를 반환합니다.
            return torch.nn.functional.conv2d(input, self.weight, self.bias, self.stride, self.padding)
            

    class CustomCNN(nn.Module):
        def __init__(self):
            super(CustomCNN, self).__init__()
            self.conv1 = CustomConv2d(in_channels=3, out_channels=16, kernel_size=3, stride=1, padding=1)
            self.relu1 = nn.ReLU()
            self.pool1 = nn.MaxPool2d(kernel_size=2, stride=2)

            self.conv2 = CustomConv2d(in_channels=16, out_channels=32, kernel_size=3, stride=1, padding=1)
            self.relu2 = nn.ReLU()
            self.pool2 = nn.MaxPool2d(kernel_size=2, stride=2)
            self.fc = nn.Linear(32 * 8 * 8, 10)


            # self.features = nn.Sequential(
            #      nn.Conv2d(3, 16, kernel_size=3, stride=1, padding=1),
            #      nn.ReLU(inplace=True),
            #      nn.MaxPool2d(kernel_size=2, stride=2),
            #      nn.Conv2d(16, 32, kernel_size=3, stride=1, padding=1),
            #      nn.ReLU(inplace=True),
            #      nn.MaxPool2d(kernel_size=2, stride=2)
            #  )
            # self.classifier = nn.Sequential(
            #      nn.Linear(32 * 8 * 8, 10),
            #  )
        def forward(self, x, get_features=False):
        x = self.pool1(self.relu1(self.conv1(x)))
        x = self.pool2(self.relu2(self.conv2(x)))
        x = x.view(-1, 32 * 8 * 8)
        logits = self.fc(x)

        # x = self.features(x)
        # x = x.view(x.size(0), -1)
        # logits = self.classifier(x)

        if get_features:
            return x, logits
        else:
            return logits


------
> 참고문헌

[1] [커스텀 컨볼루션](http://discuss.pytorch.org/t/custom-convolution-layer/45979)
