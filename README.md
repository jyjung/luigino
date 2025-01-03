# luigino

luigino는 **luigi** 작업을 좀더 편리하게 쓰고자 계속 개선중인 라이브러리입니다. 또한 Celery와 연동하여 작업을 실행할 수 있으며, 공통적인 작업 기능을 제공합니다.

## 설치

```
pip install luigino
```

## 사용법

luigi.task 대신 CommonLuigiTask 클래스를 사용합니다.  

```python
from luigino import  CommonLuigiTask

class MyTask(CommonLuigiTask):
    def run(self):
        # 작업 구현
        pass
```

# CommonLuigiTask 클래스

`CommonLuigiTask` 클래스는 Luigi 작업을 위한 기본 클래스이며, 파일 경로 처리, 출력 쓰기 및 이전 출력 읽기와 같은 공통 기능을 제공합니다.

## 메서드

### `work_file_path(self, file_name)`

파일 이름을 기반으로 출력 디렉토리에서 전체 파일 경로를 생성합니다.

- **매개변수:**
  - `file_name` (str): 파일 이름.
- **반환값:**
  - `str`: 전체 파일 경로.

#### 예제

```python
task = CommonLuigiTask()
file_path = task.work_file_path("example.json")
print(file_path)  # 출력: output_dir/example.json
```

### `write_output(self, all_info)`
제공된 정보를 작업의 출력 파일에 JSON 형식으로 씁니다.

- 매개변수:
    - all_info (dict 또는 list): 출력 파일에 쓸 정보.

#### 예제

```python
task = CommonLuigiTask()
data = {"key": "value"}
task.write_output(data)
```

### `get_output_from_path(self, file_name: str) -> dict | list | None`
지정된 파일 이름의 JSON 파일 내용을 읽어 반환합니다.

- 매개변수:
    - file_name (str): 읽을 파일의 이름.
- 반환값:
    - dict | list | None: 파일의 내용이 딕셔너리 또는 리스트로 파싱된 결과.

#### 예제

```python
task = CommonLuigiTask()
data = task.get_output_from_path("example.json")
print(data)  # 출력: 파일의 JSON 내용
```

### `get_previous_output(self) -> dict | list | None`
이전 requires 에 정의된  작업의 입력 파일 내용을 JSON 형식으로 읽어 반환합니다.

- 반환값:
    - dict | list | None: 입력 파일의 내용이 딕셔너리 또는 리스트로 파싱된 결과.
#### 예제

```python 
class ExampleTask(CommonLuigiTask):
    def requires(self):
        return AnotherTask()

    def run(self):
        previous_output = self.get_previous_output()
        print(previous_output)  # 출력: 이전 작업의 JSON 내용

task = ExampleTask()
task.run()
```

### `output_dir(self)`
생성시 json config 값으로 받은 dir을 출력합니다.  

- 반환값:
    - str: 출력 디렉토리.
#### 예제
```python 
config = {
    "output_dir": "output/ababab"
}
sst = ShelveSetTask(config = config)
print(sst.output_dir())  #"output/ababab" 
```

# celery 연동 방법

CommonLuigiTask 를 상속받아 작성한 task는 아래와 같이 execute_task 로 동작시킬수 있습니다. 
celery의 특징 때문인지 파라미터는 str 로 전달되어야 합니다. execute_task는 내부에서 json.loads를 사용해 str을 dict 형식으로 변경합니다. 


```python
import os
from celery import Celery
from celery import states
from celery.exceptions import TaskError,Ignore
from .any_task import ParamExtractionTask, UnzipTask

app = Celery('anyproject',
             broker=os.getenv('CELERY_BROKER_URL', 'redis://redis:6379/0'),
             backend=os.getenv('CELERY_BROKER_URL', 'redis://redis:6379/0')
)
app.config_from_object('celeryconfig')

@app.task(bind=True)
def param_extraction(self, json_param: str):
    return execute_task(self, ParamExtractionTask, json_param)

@app.task(bind=True)
def unzip_file(self, json_param: str):
    return execute_task(self, UnzipTask, json_param)            


```

## `celeryconfig.py` 내용

```python
worker_send_task_events = True
task_send_sent_event = True
```