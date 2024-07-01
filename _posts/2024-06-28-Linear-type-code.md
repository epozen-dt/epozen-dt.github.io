---
title: "Linear-type"
last_modified_at: 2024-06-28
author: 김혜원
---

본 포스팅은 Linear 모델이 구현된 코드에 대한 것입니다.

---


&nbsp;
다음 코드는 선형 모델을 구현한 것입니다.


&nbsp;

## Linear

    class Linear(nn.Module):
        def __init__(self, input_dim, output_dim):
            super(Linear, self).__init__()
            self.linear = nn.Linear(input_dim, output_dim)

        def forward(self, input_data):
            return self.linear(input_data)

## Nlinear
    class Nlinear(nn.Module):
        def __init__(self, input_dim, output_dim, kernel_size=3):
            super(Nlinear, self).__init__()
            self.kernel_size = kernel_size
            self.avg_pool = nn.AvgPool1d(kernel_size=self.kernel_size, stride=1, padding=(kernel_size - 1) // 2)
            self.linear = nn.Linear(input_dim, output_dim)

        def forward(self, input_data):
            x = []
            x_outputs = []
            for i in range(len(input_data)):
                x.append(input_data[i] - input_data[-1])

            x_normal = self.linear(x)

            for i in range(len(input_data)):
                x_outputs.append(x_normal[i] + input_data[-1])

            return x_outputs


## Dlinear
    class MovingAvg(nn.Module):
        def __init__(self, kernel_size, stride):
            super(MovingAvg, self).__init__()
            self.kernel_size = kernel_size
            self.avg = nn.AvgPool1d(kernel_size=kernel_size, stride=stride, padding=0)

        def forward(self, x):
            front = x[:, 0:1, :].repeat(1, (self.kernel_size - 1) // 2, 1)
            end = x[:, -1:, :].repeat(1, (self.kernel_size - 1) // 2, 1)
            x = torch.cat([front, x, end], dim=1)  # 원래 데이터와 연결
            x = self.avg(x.permute(0, 2, 1)).permute(0, 2, 1)  # 이동평균 구함
            return x


    class decomp(nn.Module):
        def __init__(self, kernel_size):
            super(decomp, self).__init__()
            self.moving_avg = MovingAvg(kernel_size, stride=1)

        def forward(self, x):
            moving_mean = self.moving_avg(x)
            res = x - moving_mean
            return res, moving_mean


    class Dlinear(nn.Module):
        def __init__(self, input_dim, output_dim, kernel_size):
            super(Dlinear, self).__init__()
            self.decomposition = decomp(kernel_size)
            self.Linear_Seasonal = nn.Linear(input_dim, output_dim)
            self.Linear_Trend = nn.Linear(input_dim, output_dim)

        def forward(self, x):
            residual, moving_mean = self.decomposition(x)
            seasonal_output = self.Linear_Seasonal(residual)
            trend_output = self.Linear_Trend(residual)
            return seasonal_output + trend_output
&nbsp;



&nbsp;


------
> 참고

[1] [논문](https://arxiv.org/pdf/2205.13504)



