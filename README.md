# Django로 홈페이지 만들기

## 참고
- [Django 문서](https://docs.djangoproject.com/ko/2.0/)
- [Django 문서(한국어판)](https://django-doc-test-kor.readthedocs.io/en/old_master/index.html)

## 진행과정

### Django 설치
```bash
$ pip3 install Django
$ python -m django --version
```

### 프로젝트 만들기
```bash
$ django-admin startproject [projectName]
$ cd [projectName]
```

### 명령어들
```bash
# 개발서버 실행
$ python package.py runserver 8080
# 대화식 쉘 실행
$ python package.py shell
```

### 데이터베이스 설치(MySQL)
[Django 문서/topics/templates](https://docs.djangoproject.com/ko/2.0/ref/databases/)
* mysqlclient(MySQL DB API Driver) 설치
```bash
$ sudo apt-get install mysql-server mysql-client libmysqlclient-dev
$ sudo pip3 install mysqlclient
```
* [settings.py](./myhome/settings.py) 설정
```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'pleiweb',
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': ''
    }
}
```
or
```python
# settings.py
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'OPTIONS': {
            'read_default_file': '/path/to/my.cnf',
        },
    }
}

# my.cnf
[client]
database = NAME
user = USER
password = PASSWORD
default-character-set = utf8
```
* settings 의 INSTALLED_APPS 에 필요한 데이터베이스 테이블 생성
```bash
$ python manage.py migrate
```

### 모델 생성
* [models.py](./polls/models.py) 편집
INSTALLED_APPS에 'polls.apps.PollsConfig' 추가
* migrate 하기
```bash
# polls/migrations에 모델에 대한 migration 저장
$ python manage.py makemigrations polls
# migration SQL 스크립트 출력(optional)
$ python manage.py sqlmigrate polls 0001
# 모델에 대한 데이터베이스 테이블 생성
$ python manage.py migrate
```

### 관리자 생성
```bash
$ python manage.py createsuperuser
Username (leave blank to use 'admin'): 
Email address: 
Password:
Password (again):
Superuser created successfully.

$ python manage.py runserver 8080
```
http://127.0.0.1:8080/admin 에서 로그인

### view 추가하기
* [views.py](./polls/views.py) 에 detail, results, vote view 추가
* [urls.py](./polls/urls.py) 에 각 view에 대한 path 추가
* django.shortcuts 활용
```python
from django.shortcuts import render, get_object_or_404

objectName = get_object_or_404(ClassName, pk=id)
return render(request, 'path/templateName', args_dict(optional))
```
* request: HttpRequest 개체
    - request.POST['name']: POST로 전송된 value
* django.http.HttpResponseRedirect(reverse('app_name:path_name', args=(..., )))

### 템플릿 만들기
* [./polls/templates/polls/](./polls/templates/polls)
    - index.html: Question들에 대한 ListView
    - detail.html: 선택된 Question에 대한 choice들을 보여주는 DetailView
    - results.html: 선택된 Question에 대한 투표결과를 보여주는 DetailView
* Django template language
( [Django 문서/topics/templates](https://docs.djangoproject.com/ko/2.0/topics/templates/) | [Django 문서/ref/templates](https://docs.djangoproject.com/ko/2.0/ref/templates/) )
    * {{ ... }}: 변수
        - {{ ...|filter }}: 필터
        - {{ forloop.counter }}: for 태그 반복 횟수
    * {% ... %}: 태그
        - {% url 'path_name' args %}: URL 경로의존성 제거
        - {% extends "....html" %}{% block ... %}{% endblock %}: 템플릿 상속
        - {% csrf_token %}: 내부 URL 향하는 POST 폼에 사용
    - {# ... #}: 주석

### 제너릭 뷰(Class-based views)
( [Django 문서/topics/class-based-views](https://docs.djangoproject.com/ko/2.0/topics/class-based-views/) )
* [urls.py](./polls/urls.py) 수정 (이전 코드는 """ 주석 처리)
* [views.py](./polls/views.py) 수정 (이전 코드는 """ 주석 처리)
    - from django.views import generic 의 클래스들을 상속받아 새로운 view 클래스 정의
    - generic.ListView: 개체 목록 표시
    - generic.DetailView: 특정 개체 유형에 대한 세부 정보 페이지 표시

### test
* 참고
    - [Django 문서/tipics/testing](https://docs.djangoproject.com/ko/2.0/topics/testing/)
    - [wiki - Test-driven development](https://en.wikipedia.org/wiki/Test-driven_development)

* [test.py](./polls/tests.py) 수정
```python
from django.test import TestCase

class ClassName(TestCase):
    def func(self):
        pass
```
TestCase 클래스를 상속받은 새로운 클래스에 테스트용 함수를 작성한다.

* 테스트 실행
```bash
python manage.py test [appName]
```

### 정적 파일 사용
* CSS
    - [style.css](./polls/static/polls/style.css) 작성
    - 템플릿에 CSS 추가
    ```html
    {% load static %}
    <link rel="stylesheet" type="text/css" href="{% static 'appName/style.css' %}" />
    ```
* image
    - [background.jpg]를 ./appName/static/appName/ 에 추가
    - style.css에 항목 추가
    ```css
    body {
        background: white url("images/background.gif") no-repeat;
    }
    ```

### 관리자 사이트 커스터마이징
* 참고
    - [Django 문서/intro/tutorial07](https://docs.djangoproject.com/ko/2.0/intro/tutorial07/)
    - [Django 문서/ref/contrib/admin](https://docs.djangoproject.com/ko/2.0/ref/contrib/admin/)
* 폼 커스터마이징
    - [admin.py](./polls/admin.py) 수정
* 템플릿 커스터마이징
    - [settings.py](./myhome/settings.py)의 TEMPLATES.'DIRS' 수정
    - 장고 설치 위치 확인
    ```bash
    $ python -c "import django; print(django.__path__):
    ```
    - Django 자체 소스코드(django/contrib/amdin/templates)를 [project 내](./templates)에 복사
    - [base_site.html](./templates/admin/base_site.html) 수정

