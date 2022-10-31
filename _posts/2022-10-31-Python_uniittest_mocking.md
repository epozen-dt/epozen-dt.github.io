---
title: "python unittest - mocking"
last_modified_at: 2022-10-31
author: Gun-ha, KANG
---

> 이번 포스팅에서는 **python unittest - mocking**에 대해 공유해보려고 합니다.


## **unittest Mocking**

> **unittest.mock**   

Python 에서는 unittest, pytest 를 활용하여 단위테스트를 작성합니다.  
본 포스팅에서는 그 중 Python에  내장된unittest의 unittest.mock 의 patch 활용에 대해서 작성했습니다.

(참고) 환경으로는 다음과 같습니다.
* IDE: Pycharm  
* Python: 3.10  
* flask: 2.2.2  
* psycopg2: 2.9.5  


> **Mocking = Patching**

테스트 코드를 작성할 때, 실제로 실행하고 싶지 않은 코드(외부 의존 API, 기타 등등)들이 있을 수 있습니다.  
이럴 때, Mocking 기법을 사용하여 일부분을 임의의 가짜로 대체하여 테스트를 진행할 수 있습니다.

unittest.mock 모듈에서는 patch() 를 이용하여 다음과 같은 작업을 할수 있습니다.  
함수나 클래스 데코레이터로 활용하거나 with 문 내부에서 활용 가능합니다.    
* 외부에서 import 해야하는 외부 라이브러리의 행위를 mocking 합니다.  

```  
import unittest
from unittest.mock import patch
```  

자세한 정의에 대해서는 여기서 확인하실 수 있습니다.  
https://docs.python.org/ko/3/library/unittest.mock.html#patch


> **사전 파이썬 코드**


**(1) router.test_router.py**  

[DELETE] http://localhost:5000/<no>  
ex. http://localhost:5000/5

```  
from utils import common

@bp.route("/<no>", methods=['DELETE'])
def del_data(no):

    # DB Table 에서 no 에 해당하는 row 가 있는지 확인 (있으면 True)
    if not common.is_check_data("TEST_TABLE", no):
        return json.dumps({'code': '', 'message': '--> no 에 해당하는 테이블 데이터가 없음'})

    try:
        # DB Table 에서 no 에 해당하는 row delete
        db_manager = common.DBManager()
        db_manager.del_data(str(no))
    except Exception as e:
        print(e)
        return json.dumps({'code': '', 'message': e})
    else:
        return json.dumps({'code': '00', 'message': '성공'})

# End of del_data
``` 


**(2) utils.common.py 의 is_check_data(),  DBManager.del_data()**

```  
def is_check_data(tb: str, tb_id: str) -> bool:
    """
    현재 tb_id 에 해당하는 데이터가 있는 경우 True,
    없으면 False
    :param tb: 테이블
    :param tb_id: 테이블 고유 id
    :return: Boolean 
    """

    query_sql = f"""
         SELECT title FROM public."TEST_TABLE" WHERE "no" = {tb_id}
    """
    <SELECT 관련 코드 작성>
# End of is_check_data

class DBManager:
    def __init__(self):
        self._conn_pool = get_db_connection_pool()
    # End of __init__

    def del_data(self, no: str):

        query_sql = f"""
            DELETE FROM public."TEST_TABLE"
            WHERE "no" = {no}
        """
        <삭제 관련 코드 작성>

    # End of del_data
# End of DBManager
``` 
  

> **단위테스트 적용**
  
**Unittest 작성**

작성 과정은 다음과 같습니다.  
* router.test_router.py 의 del_data(no) 메소드에 대한 REST API 테스트를 진행합니다.
* 이때, setUp() 을 통해 TESTING 모드로 수행합니다.
* @patch('utils.common.DBManager.del_data') 를 통해 실제 psycopg2를 통해 DB delete 하는 과정을 가짜로 수행합니다. 해당 줄이 주석처리 되면, 실제로 DB 테이블의 row 가 삭제됩니다.
* case 1 과 2를 통해 ist_check_data 또한 가짜로 수행하여, return 값을 True와 False 을 반환하는 경우에 대해서 테스트를 수행합니다.


```  
class TestAPI(unittest.TestCase):

    def setUp(self) -> None:
        # Flask provides utilities for testing an Application
        create_app().config["TESTING"] = True
        self.app = create_app().test_client()
        # domain
        self.domain = "http://localhost:5000"

    @patch('utils.common.DBManager.del_data')
    def test_check_data(self, del_data):
        # given
        no = "5"

        # case1. is_check_data() 의 리턴을 True 로 설정
        with patch('utils.common.is_check_data', return_value=True):

            # when
            right_resp = self.app.delete(self.domain + '/'+no)

            # then
            assert right_resp.status_code == 200
            assert {'code': '00', 'message': '성공'} == json.loads(right_resp.get_data())

        # case2. is_check_data() 의 리턴을 False 로 설정
        with patch('utils.common.is_check_data', return_value=False):
            # when
            right_resp = self.app.delete(self.domain + '/' + no)

            # then
            assert right_resp.status_code == 200
            assert {'code': '', 'message': '--> no 에 해당하는 테이블 데이터가 없음'} == json.loads(right_resp.get_data())
            assert json.dumps({'code': '', 'message': '--> no 에 해당하는 테이블 데이터가 없음'}) == right_resp.get_data(as_text=True)
``` 

Patch() 를 사용하여 특정 메소드에 대해서 가짜 return 값을 사용함으로써, 원하는 부분에 대해서만 단위테스트를 작성할 수 있습니다.


>  **DB Connection - patch**

추가적으로, DB Connection 관련해서도 다음과 같이 테스트가 가능합니다.

```  
class DBTest(unittest.TestCase):
    @patch('utils.common.psycopg2.pool.SimpleConnectionPool')
    def test_get_db_connection_pool(self, mock_pool):
        # connect
        get_db_connection_pool()
        mock_pool.assert_called_with(minconn=1, maxconn=20, user=config.DB_USERNAME,
                                     password=config.DB_PASSWORD,
                                     host=config.DB_HOST,
                                     database=config.DB_NAME)

    @patch('utils.common.psycopg2.pool.SimpleConnectionPool.getconn')
    def test_get_db_connection_pool_disconnect(self, mock_connect):
        # disconnect
        mock_conn = mock_connect.return_value
        mock_close = mock_conn.close
        conn = get_db_connection_pool().getconn().close()
        mock_close.assert_called_once()
``` 

> **참고 자료**  

* [unittest - Unit testing framework](https://docs.python.org/3/library/unittest.html)
* [unittest.mock - mock object library](https://docs.python.org/3/library/unittest.mock.html)  


---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
