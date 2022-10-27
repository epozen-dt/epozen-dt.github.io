---
title: "flask request validator로 파라미터 유효성 검사하기"
last_modified_at: 2022-10-27
author: 이한솔
---

이번 시간에는 Flask Request 파라미터의 유효성을 검사해주는 flask request validator 패키지에 대해 알아보도로 하겠습니다.

---

API서버를 만드는 과정에서 HTTP Request 파라미터가 유효한 값인지 확인해주는 작업이 필요합니다.<br>
> **flask request validator**는 Flask에서 HTTP Request 파라미터의 유효성을 검사해주는 편리한 오픈 소스로

``` python 
pip install flask request validator
```
다음과 같이 설치 후 이용할 수 있습니다.<br>
파라미터 유효성 체크는 대표적으로 <br>
1. request parameter의 타입 (GET, FORM 등)
2. parameter 값의 타입 (str, int 등)
3. 필수값인지 여부
4. 유효성 방식
   
등이 있습니다.

기존에는 유효성을 체크하기 위해 메소드 안에서 일일히 확인했어야 하지만

``` python 
@app.route('/user', methods=["POST"])
def get_user():
    # request parameter 타입 확인 - JSON
    body = request.get_json()
    # parameter 값의 타입 확인 
    user_id = str(body.get("user_id"))
    name = str(body.get("name"))
    phone = str(body.get("phone"))
    # 필수 여부
    if user_id is None:
        return {"error": f"{user_id}값이 없습니다."}, 400
    # 유효성 방식 확인
    if re.match(r'^[0-9]{2,3}-[0-9]{3,4}-[0-9]{4}$',phone) is None:
        return {"error": f"{phone}값이 유효하지 않습니다."}, 400

    response = {
        "id": user_id,
        "name": name,
        "phone": phone,
    }

    return response, 200
```
flask request validator를 이용하기 위해서는 @validate_params 데코레이터를 실행 할 함수 위에 지정해주면 됩니다.

사용 방법은 Param(param 이름, request param의 타입, 값의 타입, 필수 여부, 유효성 방식)으로 각 파라미터 값들은 다음과 같습니다.
- request param의 타입 - GET, PATH, FORM, JSON, HEADER
- 값의 타입 - str, bool, int, float, dict, list
- 필수 여부 - True, False
- 유효성 방식 
  - Pattern : 정규식
  - Enum : 허용된 값
  - MaxLength, MinLenth : 길이
  - NotEmpty
  - IsDatetimeIsoFormat, IsEmail, Datetime, Number

``` python 
@app.route('/user', methods=["POST"])
@validate_params(
    # param 이름, request param 타입, 값의 타입, 필수 여부, 유효성 방식
    Param('phone', JSON, str, required=False, rules=[Pattern(r'^[0-9]{2,3}-[0-9]{3,4}-[0-9]{4}$')]), 
    Param('desc', FORM, str, rules=[MaxLength(512)], required=False),
)
def get_user(valid: ValidRequest):
```
만약 유효성에 검사를 통과하지 못한 값이 들어올 경우에는 다음과 같이 에러 발생 이유를 에러 메세지로 출력해줍니다.
``` python 
flask_request_validator.exceptions.InvalidRequestError: ({}, {}, {'phone': RulesError(ValuePatternError('^[0-9]{2,3}-[0-9]{3,4}-[0-9]{4}$'))}, {})
```

이번 시간에는 Flask Request 파라미터의 유효성을 검사해주는 flask request validator에 대해 알아보았습니다.
flask request validator를 이용하면 통해 더 편리하고 깔끔하게 유효성을 검사할 수 있을 것 같습니다.<br>
자세한 소스는 https://github.com/d-ganchar/flask_request_validator 여기서 확인하실 수 있습니다.

---

