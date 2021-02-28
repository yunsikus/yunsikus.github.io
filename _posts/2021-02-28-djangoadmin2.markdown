---
layout: post
title: "[Django] admin - admin 사이트 꾸미기"
subtitle: "[Django] admin - admin 사이트 꾸미기"
categories: Development Django
tags: Django Model
comments: true
---

## Admin 사이트 꾸미기

프로젝트를 개발하다 보면 진행 과정이나 개발 완료 후에도 데이터베이스를 다룰 일이 많이 발생합니다. 장고의 Admin 사이트는 데이터베이스에 들어있는 데이터를 쉽게 관리할 수 있도록 데이터의 `생성(Create)`, `조회(Read)`, `변경(Update)`, `삭제(Delete)` 등의 기능을 제공합니다.

장고의 Admin 기능은 데이터 관리를 쉽게 해줄 뿐 아니라, UI도 깔끔하고 자신의 취향에 맞게 꾸미는 것도 가능하여 많은 사람들이 장고를 사용하는 장점이 있습니다.

테이블을 보여주는 UI양식을 변경하려면 `admin.py` 파일을 변경하면 됩니다. 앞서 만든 `Choice`와 `Question`테이블을 가지고 admin의 기능을 살펴보겠습니다.

## 1. 필드 순서 변경하기  
`fields` 를 통해 변수들의 순서를 지정할 수 있습니다.

```python
## admin.py
from django.contrib import admin
from .models import Question, Choice

class QuestionAdmin(admin.ModelAdmin):
    fields = ['pub_date', 'question_text'] # 필드 순서 변경

admin.site.register(Question, QuestionAdmin)
admin.site.register(Choice)
```
![django_admin1](https://yunsikus.github.io/assets/img/post_img/django-admin_1.jpg)

## 2. 각 필드를 분리해서 보여주기
`fieldsets`으로 각 필드를 분리할 수 있습니다. `fieldsets`에 있는 각 튜플의 첫번째 인자가 해당 필드의 제목입니다.

```python
# 위의 내용 동일
class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        ('Question Statement', {'fields':['question_text']}),
        ('Date Statement', {'fields':['pub_date']})
    ]
# 아래 내용 동일
```
![django_admin2](https://yunsikus.github.io/assets/img/post_img/django-admin_2.jpg)

## 3. 필드 접기
필드 항목을 접을 수도 있습니다. 앞에서 살펴본 필드 순서 변경. 필드 분리 및 지금 진행하려고 하는 필드 접기 기능 등은 모두 필드 개수가 많아 폼이 길어진 경우에 유용하게 사용할 수 있습니다.

```python
# 위의 내용 동일
class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        ('Question Statement', {'fields':['question_text']}),
        ('Date Statement', {'fields':['pub_date'], 'classes':['collapse']})
    ]
# 아래 내용 동일
```
![django_admin3](https://yunsikus.github.io/assets/img/post_img/django-admin_3.jpg)

## 4.외래키 관계 화면

Question과 Choice 모델 클래스는 1:N 관계로 이루어져 있고, 서로 외래키로 연결되어 있습니다. 만약 질문 하나에 3개의 답변 항목이 있다면 Choice 레코드를 추가하는 동일한 작업을 3번 반복해야 됩니다. 이 작업을 한 화면에서 한번에 처리할 수 있도록 UI를 변경할 수 있습니다.

Question 레코드를 기준으로 여러개의 Choice 레코드가 연결되는 것이므로, ChoiceAdmin이 아니라 QuestionAdmin 클래스를 수정합니다.항목이 추가 될수록 한눈에 보기 힘들어져, 테이블 형식으로 보기 위해 `admin.TabularInline`을 상속받습니다.  

```python
# 위의 내용 동일
class ChoiceInline(admin.TabularInline):
    model = Choice
    extra = 2

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        ('Question Statement', {'fields':['question_text']}),
        ('Date Statement', {'fields':['pub_date']})
    ]
    inlines = [ChoiceInline]
# 아래 내용 동일
```
![django_admin4](https://yunsikus.github.io/assets/img/post_img/django-admin_4.jpg)


## 5. 레코드 리스트 컬럼 지정하기.

Admin 사이트의 첫 페이지에서 테이블명을 클릭하면, 해당 테이블의 레코드 리스트가 나타납니다. 이때 각 레코드의 제목은 `models.py`에서 정의한 `__str__()` 메소드의 리턴값을 레코드의 제목으로 사용합니다. `list_display` 속성을 한 줄 추가하면 레코드 리스트에 보여주는 컬럼 항목을 지정할 수 있습니다.  

```python
# 위의 내용 동일
class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        ('Question Statement', {'fields':['question_text']}),
        ('Date Statement', {'fields':['pub_date']})
    ]
    list_display = ('question_text', 'pub_date') # 레코드 리스트 컬럼 항목 지정
# 아래 내용 동일
```
![django_admin5](https://yunsikus.github.io/assets/img/post_img/django-admin_5.jpg)

## 6. 필터 사이드바 붙이기

`list_filter` 속성을 한 줄 추가하면, UI 화면 우측에 필터 사이드 바를 붙일 수도 있습니다.
```python
# 위의 내용 동일
class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        ('Question Statement', {'fields':['question_text']}),
        ('Date Statement', {'fields':['pub_date']})
    ]
    list_display = ('question_text', 'pub_date') # 레코드 리스트 컬럼 항목 지정
    list_filter = ['pub_date'] # 필터 사이드 바 추가
# 아래 내용 동일
```
![django_admin6](https://yunsikus.github.io/assets/img/post_img/django-admin_6.jpg)

## 7. 검색 박스 표시하기
`search_fileds` 속성을 한 줄 추가하면 UI화면에 검색 박스를 표시할 수 있습니다.
```python
# 위의 내용 동일
class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        ('Question Statement', {'fields':['question_text']}),
        ('Date Statement', {'fields':['pub_date']})
    ]
    list_display = ('question_text', 'pub_date') # 레코드 리스트 컬럼 항목 지정
    list_filter = ['pub_date'] # 필터 사이드 바 추가
    search_fields = ['question_text'] # 검색 박스 추가
# 아래 내용 동일
```
![django_admin7](https://yunsikus.github.io/assets/img/post_img/django-admin_7.jpg)
