---
title: "Nelson Rule 개념 정리 및 간단한 예제"
last_modified_at: 2022-05-31
author: 심건우
---

이번 포스팅에선, Nelson Rule 관련 개념을 정리하고, 간단한 예제를 진행하겠습니다.

관련 정보는 아래 주소를 참고했습니다.

[Nelson Rule 관련 정보](https://en.wikipedia.org/wiki/Nelson_rules)

## Nelson Rules
 Nelson Rule은 일부 측정 데이터가 관리 이탈 상태에 있는지 여부를 결정하는 공정 관리 방법입니다.
 Nelson Rule은 1984년 10월 Journal of Quality Technology에 Lloyd S Nelson의 기사에서 처음 발표되었습니다.
 Nelson Rule은 시계열 데이터 기반 관리도에 적용됩니다.
 Nelson Rule은 데이터들의 평균과 표준편차를 기반으로 합니다.
 
 * +n시그마(σ) = 평균 + (n x 표준편차)
 * -n시그마(σ) = 평균 - (n x 표준편차)
 * UCL = +3시그마(σ)
 * LCL = -3시그마(σ)
 
### Rule 1
 - 1개의 데이터가 +3시그마보다 크거나 -3시그마보다 작은지 확인


![image](https://user-images.githubusercontent.com/87160438/171092983-d08bd793-b3ea-4190-8ca4-ae980ef34746.png)

[이미지 출처](https://leedakyeong.tistory.com/entry/%EB%84%AC%EC%8A%A8-%EB%B2%95%EC%B9%99%EC%9D%B4%EB%9E%80-What-is-the-Nelson-Rules)

### Rule 2
 - 연속된 9개의 데이터 모두 평균보다 크거나 작은지 확인


![image](https://user-images.githubusercontent.com/87160438/171093045-26b46187-d3a9-455b-abb9-cdfd4c3939d9.png)

[이미지 출처](https://leedakyeong.tistory.com/entry/%EB%84%AC%EC%8A%A8-%EB%B2%95%EC%B9%99%EC%9D%B4%EB%9E%80-What-is-the-Nelson-Rules)

### Rule 3
 - 연속된 6개의 데이터가 추세적으로 증가 혹은 감소하는지 확인


![image](https://user-images.githubusercontent.com/87160438/171093059-6e494e59-9bbe-4d43-a880-1eef9bb47383.png)

[이미지 출처](https://leedakyeong.tistory.com/entry/%EB%84%AC%EC%8A%A8-%EB%B2%95%EC%B9%99%EC%9D%B4%EB%9E%80-What-is-the-Nelson-Rules)

### Rule 4
 - 연속된 14개의 데이터가 진동하는지 확인


![image](https://user-images.githubusercontent.com/87160438/171093090-86e153bc-b418-4c37-b8b2-fd7f08b9f6ca.png)

[이미지 출처](https://leedakyeong.tistory.com/entry/%EB%84%AC%EC%8A%A8-%EB%B2%95%EC%B9%99%EC%9D%B4%EB%9E%80-What-is-the-Nelson-Rules)

### Rule 5
 - 연속된 3개의 데이터 중 2개 이상의 데이터가 +2시그마보다 크거나 -2시그마보다 작은지 확인
 - 2개 이상의 데이터는 같은 방향성을 띔


![image](https://user-images.githubusercontent.com/87160438/171093101-93d99293-5ee0-4459-bcb9-9ba20a7e5840.png)

[이미지 출처](https://leedakyeong.tistory.com/entry/%EB%84%AC%EC%8A%A8-%EB%B2%95%EC%B9%99%EC%9D%B4%EB%9E%80-What-is-the-Nelson-Rules)

### Rule 6
 - 연속된 5개의 데이터 중 4개 이상의 데이터가 +1시그마보다 크거나 -1시그마보다 작은지 확인
 - 4개 이상의 데이터는 같은 방향성을 띔


![image](https://user-images.githubusercontent.com/87160438/171093125-f808df91-1db6-44c0-9656-36ad64624769.png)

[이미지 출처](https://leedakyeong.tistory.com/entry/%EB%84%AC%EC%8A%A8-%EB%B2%95%EC%B9%99%EC%9D%B4%EB%9E%80-What-is-the-Nelson-Rules)

### Rule 7
 - 연속된 15개의 데이터가 +1시그마보다 작고 -1시그마보다 큰지 확인


![image](https://user-images.githubusercontent.com/87160438/171093140-78342583-484d-44ad-914f-a830b826a32f.png)

[이미지 출처](https://leedakyeong.tistory.com/entry/%EB%84%AC%EC%8A%A8-%EB%B2%95%EC%B9%99%EC%9D%B4%EB%9E%80-What-is-the-Nelson-Rules)

### Rule 8
 - 연속된 8개의 데이터가 +1시그마보다 크거나 -1시그마보다 작은지 확인
 - 8개의 데이터는 각각 다른 방향성을 가질 수 있음


![image](https://user-images.githubusercontent.com/87160438/171093151-1f041f6b-5958-46ed-9782-4004638d990f.png)

[이미지 출처](https://leedakyeong.tistory.com/entry/%EB%84%AC%EC%8A%A8-%EB%B2%95%EC%B9%99%EC%9D%B4%EB%9E%80-What-is-the-Nelson-Rules)

## 예제
 Nelson Rule 1, 3, 5, 7을 java로 간단하게 구현해보겠습니다.
 
 - Rule 1

```java
public Boolean nsr1(double data, Double UCL, Double LCL) {
	// 위반
	if (data > UCL || data < LCL) {
		return false;
	}

	// 통과
	else {
		return true;
	}
}

```

 - Rule 3

```java
public Boolean nsr3(double data) {
	// 데이터 임시 저장
	nsrData3.add(data);

	// 데이터 확보 (6개)
	if (nsrData3.size() == 6) {
		// 위반 (연속 증가)
		if (nsrData3.get(0) < nsrData3.get(1) && nsrData3.get(1) < nsrData3.get(2) &&
		  nsrData3.get(2) < nsrData3.get(3) && nsrData3.get(3) < nsrData3.get(4) &&
		  nsrData3.get(4) < nsrData3.get(5)) {
			nsrData3.remove(0);
			return false;
		}

		// 위반 (연속 감소)
		else if (nsrData3.get(0) > nsrData3.get(1) && nsrData3.get(1) > nsrData3.get(2) &&
			   nsrData3.get(2) > nsrData3.get(3) && nsrData3.get(3) > nsrData3.get(4) &&
			   nsrData3.get(4) > nsrData3.get(5)) {
			nsrData3.remove(0);
			return false;
		}

		// 통과
		else {
			nsrData3.remove(0);
			return true;
		}
	}

	// 데이터 미확보
	else {
		return true;
	}
}
```

 - Rule 5

```java
public Boolean nsr5(double data, Double UCL, Double CL) {
	// 시그마 간격
	Double sigma = (UCL - CL) / 3;
	// + 2시그마
	Double sigma2Plus = CL + (2 * sigma);
	// - 2시그마
	Double sigma2Minus = CL - (2 * sigma);
	// + 2시그마를 넘어선 횟수
	Integer countPlus = 0;
	// - 2시그마를 넘어선 횟수
	Integer countMinus = 0;
	// 데이터 임시 저장
	nsrData5.add(data);
	
	// 데이터 확보 (3개)
	if (nsrData5.size() == 3) {
		for (int i = 0; i <= 2; i++) {
			// +2시그마 초과
			if (nsrData5.get(i) > sigma2Plus) {
				countPlus += 1;
			}
			// -2시그마 미만
			else if (nsrData5.get(i) < sigma2Minus) {
				countMinus += 1;
			}
		}
		
		// 위반
		if (countPlus >= 2 || countMinus >= 2) {
			nsrData5.remove(0);
			return false;
		}
		// 통과
		else {
			nsrData5.remove(0);
			return true;
		}
	}
	
	// 데이터 미확보
	else {
		return true;
	}
}
```

 - Rule 7

```java
public Boolean nsr7(double data, Double UCL, Double CL) {
	// 시그마 간격
	Double sigma = (UCL - CL) / 3;
	// + 1시그마
	Double sigma1Plus = CL + (1 * sigma);
	// - 1시그마
	Double sigma1Minus = CL - (1 * sigma);
	// +- 1시그마 이내에 존재하는 경우
	Integer count = 0;
	// 데이터 임시 저장
	nsrData7.add(data);
	
	// 데이터 확보 (15개)
	if (nsrData7.size() == 15) {
		// +1시그마보다 작고, -1시그마보다 큰 경우
		for (int i = 0; i <= 14; i++) {
			if (sigma1Minus <= nsrData7.get(i) && nsrData7.get(i) <= sigma1Plus) {
				count += 1;
			}
		}
		// 위반
		if (count >= 15) {
			nsrData7.remove(0);
			return false;
		} 
		// 정상
		else {
			nsrData7.remove(0);
			return true;
		}
	}
	// 데이터 미확보
	else {
		return true;
	}
}
```