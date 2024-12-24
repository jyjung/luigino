아래는 **`pyproject.toml`** 없이, **`setup.py`**만을 사용하여 전통적인 방식으로 Python 패키지를 빌드하고 PyPI에 업로드하는 기본 흐름입니다.

---

일단 업로드된 파일은 아래와 같은 명령어로 다시 업로드 가능
```bash
 python setup.py sdist bdist_wheel
 twine upload dist/*
```


## 1. 기본 폴더 구조

예시 폴더 구조를 가정해 보겠습니다. 패키지 이름이 `my_package`라고 할 때:

```
my_package/
├─ my_package/
│  ├─ __init__.py
│  └─ core.py
├─ tests/
│  └─ test_core.py
├─ setup.py
├─ LICENSE
└─ README.md
```

- **`my_package/`** 디렉토리(동명 폴더)에 실제 Python 소스 코드  
  - `__init__.py`가 있으면 이를 **패키지**로 인식  
- **`tests/`** (선택): 테스트 코드  
- **`setup.py`**: 패키징/설치 스크립트 (필수)  
- **`README.md`** (선택): 패키지 설명  
- **`LICENSE`** (선택): 라이선스 정보

---

## 2. `setup.py` 작성

다음은 **setuptools**를 이용한 전형적인 `setup.py` 예시입니다.

```python
# setup.py
import setuptools

setuptools.setup(
    name="my_package",                   # PyPI에 게시될 패키지 이름
    version="0.1.0",                     # 버전
    author="홍길동",
    author_email="hong@example.com",
    description="An example Python package",
    long_description=open("README.md", encoding="utf-8").read(),
    long_description_content_type="text/markdown",
    url="https://github.com/yourname/my_package",  # 프로젝트 홈 혹은 GitHub
    license="MIT",                                  # 혹은 "Apache-2.0" 등
    packages=setuptools.find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "Operating System :: OS Independent",
    ],
    python_requires=">=3.6",             # 최소 파이썬 버전
    install_requires=[
        # 필요 라이브러리가 있으면 여기에
        # 예: "requests>=2.0"
    ],
    # entry_points 등 추가 설정이 필요하면 아래 작성
    # entry_points={
    #     "console_scripts": [
    #         "my-script = my_package.core:main",
    #     ],
    # },
)
```

### 주요 항목

- **name**: 패키지명 (PyPI에 등록될 이름)  
- **version**: 패키지 버전 (의미 있게 관리)  
- **author, author_email**: 패키지 저작자 정보  
- **description**: 짧은 설명  
- **long_description**: `README.md` 파일을 읽어서 상세 설명으로 사용 (선택)  
- **long_description_content_type**: `text/markdown` 등을 지정해 PyPI에서 Markdown 렌더링 가능  
- **url**: 프로젝트 홈페이지/저장소 링크  
- **packages**: `setuptools.find_packages()` 사용 시, `__init__.py`가 있는 디렉토리를 자동 인식  
- **install_requires**: 패키지를 설치할 때 함께 설치해야 하는 종속 라이브러리  
- **entry_points**: 콘솔 스크립트 제공 시 사용  

---

## 3. 패키지 빌드

1) **필수 라이브러리 설치**  
   ```bash
   pip install --upgrade setuptools wheel
   ```
2) **빌드 수행**  
   ```bash
   cd my_package  # setup.py가 있는 디렉토리로 이동
   python setup.py sdist bdist_wheel
   ```
3) **결과 확인**  
   - `dist/` 폴더가 생기며, 예를 들어  
     - `my_package-0.1.0.tar.gz` (sdist)  
     - `my_package-0.1.0-py3-none-any.whl` (wheel)  
     이렇게 두 파일이 생성됨.

이렇게 생성된 `.whl` 또는 `.tar.gz` 파일을 다른 환경에서도 `pip install`로 설치할 수 있습니다.

---

## 4. PyPI 업로드 (선택)

작성한 패키지를 **공개 PyPI**에 등록하려면, **Twine**을 사용합니다.

1) **Twine 설치**  
   ```bash
   pip install --upgrade twine
   ```
2) **PyPI 계정 생성**  
   - [PyPI](https://pypi.org/) 회원 가입 후, **Account settings**에서 **API token** 발급
3) **패키지 업로드**  
   ```bash
   # dist 폴더에 빌드된 파일들이 있다고 가정
   twine upload dist/*
   ```
4) **설치 테스트**  
   ```bash
   pip install my_package
   ```
   (PyPI에 정상 업로드되었다면 `my_package`를 설치 가능)

---

## 5. 로컬 설치 테스트 (PyPI 업로드 전)

PyPI에 올리기 전에, 로컬에서 빌드된 Wheel 파일을 직접 설치해 볼 수 있습니다.

```bash
# dist/*.whl 파일 기준
pip install dist/my_package-0.1.0-py3-none-any.whl
```

이후 `python` 쉘이나 스크립트에서 `import my_package`가 잘 동작하는지 확인해보세요.

---

## 정리

- **구조**: `setup.py`와 패키지 소스(`my_package/`)를 준비  
- **`setup.py`**: `setuptools.setup()` 내에서 패키지 정보와 설정을 기술  
- **빌드**: `python setup.py sdist bdist_wheel`  
- **업로드**: `twine upload dist/*` → PyPI에 공개 배포  

이 과정을 거치면, **`pyproject.toml` 없이도** 기존 전통적인 방식으로 pip 설치 및 PyPI 업로드가 가능합니다.  
(단, 최근에는 `pyproject.toml`를 통한 **PEP 517/PEP 621** 방식이 권장되고 있지만, 기존 프로젝트들은 위 방식도 여전히 유효하게 동작합니다.)