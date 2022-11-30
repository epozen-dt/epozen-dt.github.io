---
title: "custom error handler"
last_modified_at: 2022-11-30
author: 이한솔
---

이번 시간에는 사용자가 에러 처리를 정의하는 custom error handler에 대해 알아보도로 하겠습니다.

---
프로그램 실행시 에러가 발생하면 그 상황에 대해 적절한 처리가 필요합니다. <br>
API의 에러 응답(error response)을 정의할 수 있는 Custom exception handler에 대해 설명하려고 합니다. <br>
>Flask에서는 데코레이터를 통해 error handling이 가능합니다. <br>
이 포스팅에서는 Flask로 구현해보도록 하겠습니다.


``` python 
def error_handler(app):

    @app.errorhandler(500)
    def data_error(e):

        log_error_message = e
        return error_response("500 에러 발생", log_error_message, 500)


def error_response(error_message, log_error_message, status_code):
    """
    :param error_message: 사용자 에러 메시지
    :param log_error_message: 로그 기록용 에러 메시지
    :param status_code: 상태 코드
    :return:
    """

    result = {
        'code': status_code,
        'message': error_message
    }
    logs.error(log_error_message)

    return json.dumps(result, ensure_ascii=False)
```
500 에러가 발생 했을 때 에러 처리를 하는 함수입니다.<br>
에러 내용을 로그에 기록하고 에러 응답하는 error_response 함수를 return하도록 했습니다. <br>
에러를 응답하는 error_response 함수에서는 API의 에러 응답을 정의하도록 하였습니다.

``` python 
from flask import abort


@app.route('/', methods=['GET'])
def index():
    abort(500)  # 에러를 발생시키는 abort함수
```
다음과 같이 500 에러를 발생 시킬 경우 
정의한대로 응답하는 것을 확인 할 수 있습니다.

<img src="https://user-images.githubusercontent.com/96156882/204720905-d26b77b8-60f2-40f5-bc78-9be4955ae2be.png" width="400">


> 이번에는 지난 시간에 포스팅 했던 flask request validator를 이용해
유효성 검사를 통과하지 못한 값이 들어왔을 경우 에러를 정의해보도록 하겠습니다.

[flask request validator로 파라미터 유효성 검사하기](https://epozen-dt.github.io/flask_request_validator/)

```python

@app.errorhandler(RequestError)
    def data_error(e):
        """
        validate_params 에러
        - null값 체크, rule(정규식, MaxLength 등) 검사
        """
        log_error_message = request_validation_error(e) # 어떤 유효성에 어긋나는지 확인하는 함수 정의함
        return error_response(error_message, log_error_message, 10)

@img.route('/', methods=['GET'])
@validate_params(
    Param('id', JSON, str, required=True),
    Param('ip', JSON, str, rules=[Pattern(r'(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\.){3}([0-9]|[1-9]['
                                          r'0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])$')], required=True),
)
def index(valid: ValidRequest):
    info = valid.get_json()

```
id에 아무것도 입력하지 않았을 경우, ip를 정규식에 맞지 않는 값을 입력해 요청한 결과
다음과 같이 정의한대로 응답하는 것을 확인할 수 있었습니다.

<img src="https://user-images.githubusercontent.com/96156882/204724430-652f5541-3343-4813-ac09-0bf38d530a1f.png" width="400">

---

이번 시간에는 사용자가 에러 처리를 정의하는 custom error handler에 대해 알아보았습니다. <br>
custom error handler를 이용하면 원하는대로 response 할 수 있고, 전체적인 에러 관리를 한 곳에서 모여 처리할 수 있다는 장점이 있는 것 같습니다.

---

