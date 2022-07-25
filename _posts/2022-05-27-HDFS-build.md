---
title: "HDFS 블록 파일 시스템[2] - 구축 및 간단 예제"
last_modified_at: 2022-05-27
author: 김수민
---

이번 포스팅에서는 지난 포스팅에 이어 HDFS 블록파일 시스템의 구축 과정과 python 라이브러리를 사용한 간단한 파일 업로드에 대해 설명하겠습니다.

지난 포스팅은 다음 링크를 참고해 주세요.

https://epozen-dt.github.io/2022/04/27/HDFS-basic.html

---

### HDFS 도커 배포

본 포스팅에서는 hdfs만을 각 이미지로 구축해 구성하는 방법 대신

big-data-europe에서 제공하는 파일을 사용해 docker container로 배포하는 방법을 설명하겠습니다.

우선 github에서 clone 합니다.

```bash
git clone https://github.com/big-data-europe/docker-hadoop.git
```

해당 파일 내부에는 다음과 같이 구성되어 있습니다.
![git-folder](https://user-images.githubusercontent.com/87166420/170618964-c258553a-8df8-461e-ba1d-b73f8369edd0.PNG)

각 디렉토리를 보면 datanode, namenode를 기본으로 이를 관리하는 manager와 내역을 관리하는 historyserver까지 제공하도록 구성되어 있습니다.



다운 받은 파일을 명령어를 통해 배포합니다.

```bash
docker-compose up -d
```

이때 주의해야할 점은, docker-compose 파일에서와 각 폴더 내부의 Dockerfile 에서, mount 되는 디렉토리 주소를 원하는 곳으로 지정이 되어 있는가 입니다.

docker-compose.yml 파일에 volumes를 확인하면 됩니다. 

또한 ports에 번호들이 현재 사용되지 않고 있는가도 미리 확인하는 것이 좋습니다.



모든 과정들을 정상적으로 수행하면 5개의 컨테이너가 올라온 것을 확인할 수 있습니다.

```bash
docker ps
```

![docker ps 결과](https://user-images.githubusercontent.com/87166420/170619022-8d5dd3d8-408b-4e18-8345-1f449624c1ff.PNG)


각 노드의 주소는 다음과 같습니다.

 local에서 구축하지 않은 경우 localhost대신 서버의 ip 주소를 넣으면 됩니다.

- namenode navigator : localhost:9870
- datanode navigator : localhost:9864
- resourcemanager : localhost:8088
- History server : localhost:8188
- nodemanager : localhost:8042 



이 중 nomenode의 주소에 접속하면, ui를 사용한 namenode에 대한 정보들이 나옵니다.

![namenode ui](https://user-images.githubusercontent.com/87166420/170619053-eeb9177f-ed67-439e-a1d6-a7f340cfe778.PNG)



여기에는 연결되어있는 Datanode에 대한 정보들도 나옵니다.

또한 탭 중 Browse the file system 탭을 통해 현재 업로드 되어있는 디렉토리 및 목록에 대한 정보가 올라와 있으며, upload까지 수행할 수 있습니다.

![upload ui](https://user-images.githubusercontent.com/87166420/170619072-6cf74326-dadd-4d7f-8612-2ca9f3257a98.PNG)


다만, 업로드 과정에서 local에서 작업하지 않은 경우에 간혹 upload가 되지 않는다고 오류가 뜨는 경우가 발생합니다.

이때는 두 가지를 확인해봐야 하는데,

1.  hadoop.env 파일에서 webhdfs가 허용되어 있는가와, permission을 꺼 둔 상태가 맞는가 설정파일을 확인한다.
   - HDFS_CONF_dfs_webhdfs_enabled=true
   - HDFS_CONF_dfs_permissions_enabled=false
2. local에서 해당 서버를 datanode로 인식할 수 있도록 hosts파일에 해당 주소를 추가한다.
   - Windows에 경우 C:\Windows\System32\drivers\etc 디렉토리에 있는 hosts 파일에 해당 주소를 매핑한다. (ex. x.x.x.x	datanode)
   - 리눅스 계열에 경우 /etc/hosts파일에 해당 주소를 매핑한다

이후 시도해보면 정상적으로 파일을 업로드 할 수 있습니다.

UI에서 업로드가 불가능한 경우, 아래의 python 라이브러리를 통한 업로드 진행 역시 불가할 가능성이 높습니다.

---

### Python library를 통한 기본 예제

우리는 기본적인 hdfs 연결 및 파일의 업로드, 다운로드, 삭제, 수정에 대한 내용을 진행하겠습니다.

우선 사용할 라이브러리를 추가합니다.

'pip install hdfs' 를 통해 설치된 상태에서 수행합니다.

```python
from hdfs import InsecureClient
```

1. **Client 생성**

```python
client = InsecureClient('http://x.x.x.x:9870', user='tester')
```

우리가 구축한 hdfs에서 namenode의 port를 9870으로 했으면, 9870을, 2.x 버전을 사용한 경우 50070가 기본적으로 셋팅되어 있을 것이므로 해당 포트 번호를 넣어 사용하면 됩니다.

user은 앞서 dfs_permissions_enabled를 false로 설정했기 때문에 아무 user을 사용해도 권한 오류가 발생하지 않을 겁니다.



2. **디렉토리**

우선 현재 만들어져있는 디렉토리가 어떤게 있는지 확인해보겠습니다.

```python
fnames = client.list('/')
print(fnames)
```

'/' 부분에 원하는 위치를 넣으면 해당 디렉토리 내부의 list를 출력합니다.

디렉토리를 생성하고자 한다면 makedirs()를 사용해 진행할 수 있습니다.

이때, 이미 만들어져있는 디렉토리를 생성하려고 해도 따로 오류가 발생하지 않고 다음으로 넘어갑니다.

```python
client.makedirs('/test')
```



3. **파일 업로드**

파일 업로드에는 가장 기본적인 upload() 를 사용하는 방법이 있습니다.

upload()는 json파일 외에도 다양한 파일들을 local에서부터 upload 할 수 있습니다.

```python
client.upload('/test','./TEST.npy')
```

hdfs에서의 위치 디렉토리와 현재 업로드할 파일의 위치 및 파일명을 넣어주면 됩니다.

다만 앞서 디렉토리와는 다르게, 이미 있는 파일을 다시 업로드하려고 하면 오류를 발생시킵니다.

```bash
hdfs.util.HdfsError: Remote path '/test/TEST.npy' already exists.
```

따라서 혹시 같은 위치에 같은 이름의 다른 파일을 올리고 싶다면, 기존 업로드 된 파일을 삭제 후 원하는 파일을 업로드 해야 합니다.



4. **파일 삭제**

```python
client.delete('/test/TEST.npy')
```

파일 삭제는 delete 명령어로, 원하는 파일의 위치를 정확하게 작성하면 됩니다.

해당 위치에 파일이 없으면 오류를 발생시키지 않고 다음으로 넘어갑니다.



5. **파일 다운로드**

이제 업로드된 파일을 다운로드 하고자 하는 경우 download() 명령어를 사용합니다.

다운로드 시에는 overwrite 여부를 지정할 수 있습니다.

```python
client.download('/test/TEST.npy','./download_test.npy')
```

download할 파일의 hdfs위치와 저장할 local의 위치를 지정해주면 완료됩니다.



6. **파일 읽기**

파일을 읽어와 사용하고자 할때는 임시로 저장할 변수가 필요합니다.

이를 위해 이번에는 TemporaryFile을 import시켜 사용할 예정입니다.

```python
from tempfile import TemporaryFile
import numpy as np

tp_npy_file = TemporaryFile()

with client.read('/test2/casting_data.npy') as reader:
    tp_npy_file.write(reader.read())
    tp_npy_file.seek(0)
np_arr = np.load(tp_npy_file, allow_pickle=True)
```

client에서 데이터를 읽어오기 위해서 read()를 사용합니다. 해당 함수는 반드시 with 구문을 사용하도록 설명되어 있습니다.

read() 결과의 타입은 <class 'urllib3.response.HTTPResponse'> 입니다.

따라서 해당 내용을 읽어와 미리 생성한 변수에 작성한 후 원하는 type으로 load시키는 과정을 가져야 이후 원하는 타입의 변수로 사용할 수 있습니다.

이번 예제는 npy파일로, numpy array로 불러오기 위해 작업했습니다.



앞서 설명한 6가지 이외에 추가적인 명령어에 대한 내용은 다음 링크에서 확인할 수 있습니다.

https://hdfscli.readthedocs.io/en/latest/api.html

---

이번 포스팅에서는 hdfs를 간단하게 docker-compose를 사용해서 구축하는 방법과 기본 예제를 통해 python client를 사용해 파일 및 디렉토리를 다루는 방법에 대해 알아보았습니다.

---

### 참고자료

[1] https://www.section.io/engineering-education/set-up-containerize-and-test-a-single-hadoop-cluster-using-docker-and-docker-compose/

[2] https://github.com/big-data-europe/docker-hadoop

[3] https://www.codedevlib.com/article/using-python-pyhdfs-to-operate-hadoop-connectionerror-httpconnectionpoolhostbigdatasenior03chybinmycom-90127
