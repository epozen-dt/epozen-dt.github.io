---
title: "Flask-APScheduler 사용법"
last_modified_at: 2022-11-29
author: Gun-ha, KANG
---

> 이번 포스팅에서는 플라스크 스케줄러 라이브러리인 **Flask-APScheduler**에 대해 공유해보려고 합니다.


## **Flask-APScheduler**

> **Features**   

Advanced Python Scheduler 의 Flask 확장 버전으로 보면 될 것 같습니다.  

* 플라스크 구성에서 스케줄러나 작업 정의를 로드 가능
* 스케줄러가 실행될 호스트 이름을 지정할 수 있음
* REST API 에 대한 인증을 제공
* 예약된 작업을 관리하기 위한 REST API 를 제공


> **예시. Job Schedules**

Python Package Index(PyPI) 를 통해 Flask-APSchduler 를 설치할 수 있습니다.  

```
pip install Flask-APScheduler
```  

* 1.schduler 를 Flask APP 객체에 등록합니다.

```  
def create_app() -> Flask:
    """
    Application Factory: app 객체를 생성하는 함수
    즉, Flask가 create_app이라는 이름의 함수를 자동으로 팩토리(factory) 함수로 인식해서 해당 함수를 통해서 Flask를 실행
    :return: app:  Application Instance 반환
    """

    # 전역 사용 시, 순환 참조 오류 발생 위험 존재
    app = Flask(__name__)
    app.config.from_object(get_flask_env())
    reg_router(app)

    # Scheduler 설정
    app.config.from_object(SchedulerStatusCheck())
    scheduler = APScheduler()
    scheduler.init_app(app)
    scheduler.start()
    return app
# End of create_app
```  

* 2.스케줄러 클래스 작성
  + 수행 방식으로는 cron, date, interval 등이 존재합니다.

```  
class SchedulerStatusCheck:
    """
    주기적으로 상태 체크
    """

    def __init__(self):
        """
        스케쥴러 설정
        """

        # 환경변수 설정
        self._id = "StatusCheck"
        self._trigger = "interval"
        self._period = 5  # 초단위
        self._max_instances = 10

        # 작업 설정
        self.JOBS = [
            {
                'id': self._id,  # 작업 id
                'func': self._jobs,  # 작업 수행 함수
                'trigger': self._trigger,  # 작업 실행 형식 -> 주기적 실행, 타이머
                'seconds': self._period,  # 작업 수행 시간(heartBeat Period), 단위: sec
                'max_instances': self._max_instances,  # 비동기 instance
            },
        ]
    # End of __init__

    def _jobs(self):
        """
        작업 수행 함수
        :param self:
        :return:
        """

        log.debug("====> Run Scheduler")

    # End of _jobs
# End of SchedulerContainerCheck
```  

* 3.실행결과  
  + 5초마다 (interval) 되는 것을 확인할 수 있습니다.

![1](https://user-images.githubusercontent.com/92897860/204421826-8ce8b5aa-33f4-4a6d-8267-a40f393601a8.png)


> **참고 자료**  

* [[PyPI] Flask-APScheduler](https://pypi.org/project/Flask-APScheduler/)
* [[github] Flask-APScheduler](https://github.com/viniciuschiele/flask-apscheduler)
* [[Document] Flask-APScheduler](https://viniciuschiele.github.io/flask-apscheduler/)

---

<details>
  <summary><b>Contact</b></summary>

<b>Author. </b>KangGunha

<b>Email. </b>zxcvbnm9931@epozen.com

</details>
