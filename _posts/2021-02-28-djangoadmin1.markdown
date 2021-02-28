---
layout: post
title: "[Django]model - 테이블 정의"
subtitle: "[Django]model - 테이블 정의"
categories: Development Django
tags: Django Python Model
comments: true
---

## 테이블 정의

장고에서는 테이블을 하나의 클래스로 정의하고, 테이블의 컬럼은 클래스의 변수로 매핑합니다. 태이블 클래스는 `django.db.models.Model` 클래스를 상속받아 정의하고, 각 클래스 변수의 타입도 장고에서 미리 정의된 필드 클래스를 사용합니다.

설문조사를 가정하고 `질문(Question)`과 그 `대답(Choice)`를 DB에 저장한다고 합시다.


```python
## models.py
from django.db import models

class Question(models.Model):
    # id -> PK는 장고에서 자동 생성해줌.
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
    # pub_date에 대한 레이블컬럼. Admin사이트에서 이 문구를 보게될 것.

    def __str__(self):
        return self.question_text

    class Meta:
        db_table = "TestQuestion" # DB에 저장될 이름.
        verbose_name = "질문"
        verbose_name_plural = "질문"

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    # 항상 다른 테이블의 PK에 연결되므로, Question클래스의 id변수까지 지정할 필요없이 Question클래스만 지정하면 됨.
    # 실제 테이블에서 FK로 지정된 컬럼은 _id 접미사가 붙는다. ex) question_id
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self): # Admin 사이트나 장고 쉘등에서 테이블명으로 나타내기 위해.
        return self.choice_text

    class Meta:
        db_table = "TestChoice"
        verbose_name = "선택"
        verbose_name = "선택"
```

## Foreign Key
`외래키(ForeignKey)`란 테이블의 필드 중에서 다른 테이블의 행과 식별할 수 있는 키를 의미합니다.

일반적으로 `외래키`가 포함된 테이블을 자식 테이블이라 하며, `외래키` 값을 갖고 있는 테이블은 부모 테이블이라 합니다.

즉, `외래키`란 테이블과 테이블을 연결하기 위해 사용되는 키입니다.

`외래키`를 작성할때 필수적으로 포함되어야할 매개 변수는 다음과 같습니다.

- **참조한 테이블**
- **개체 삭제시 수행할 동작**

참조할 테이블이란 외래키에서 어떤 테이블을 참조할지 의미합니다. 예제에서는 `Question` 테이블입니다.

개체 삭제시 수행할 동작(on_delete)은 `외래키(ForeignKey)`가 바라보는 테이블의 값이 삭제될때 수행할 방법을 지정합니다.



|on_delete|의미|
|-|-|
|**models.CASCADE**|외래키를 포함하는 행도 함께 삭제|
|**models.PROTECT**|해당 요소가 함께 삭제되지 않도록 오류 발생(ProtectedError)|
|**models.SET_NULL**|외래키 값을 NULL값으로 변경(null=True일때 사용 가능)|
|**Models.DO_NOTHING**|아무 행동을 하지 않음|

## Meta 내부 클래스 옵션

- **db_table** : DB에 저장되는 테이블의 이름을 설정할 수 있습니다.

- **verbose_name** : 사용자가 읽기 쉬운 모델 객체의 이름으로 관리자 화면 등에서 표시됩니다. 설정하지 않을 시, class name인 `Choice`, `Question`로 표시됩니다.

- **verbose_name_plural** :  verbose_name의 복수형이다. 옵션을 지정하지 않을 시 verbose_name에 s를 붙인다. 위의 경우 `Choices` 와 `Questions`로 표시됩니다.



![django_model2](https://yunsikus.github.io/assets/img/post_img/django-model_2.jpg)


![django_model3](https://yunsikus.github.io/assets/img/post_img/django-model_3.jpg)


## Question 테이블 컬럼과 클래스 변수간의 매핑

테이블 컬럼과 클래스 변수 간의 매핑 관계는 다음과 같습니다.

|테이블 컬럼명|컬럼타입|장고의 클래스 변수|장고의 필드 클래스|
|-|-|-|-|
|id|integer|(id)|(PK는 장고에서 자동 생성해줌)
|question_text|varchar(200)|question_text|models.CharField(max_length=200)
|pub_date|datetime|pub_date|models.Date.TimeField("date published")


## Choice 테이블 컬럼과 클래스 변수간의 매핑

|테이블 컬럼명|컬럼타입|장고의 클래스 변수|장고의 필드 클래스|
|-|-|-|-|
|id|integer|(id)|(PK는 장고에서 자동 생성해줌)
|choice_text|varchar(200)|choice_text|models.CharField(max_length=200)
|votes|integer|votes|models.integerField(default=0)
|question_id|integer|question|models.ForeignKey(Question)

필드 클래스는 직관적을 이해가 되지만 유의할 사항은 다음과 같습니다.

- `PK(Primary Key)`는 클래스에 지정해주지 않아도, 장고는 항상 PK에 대한 속성을 NOT NULL 및 Autoincrement로 이름은 id로 해서 자동으로 만들어 줍니다.
- `DateTimeField()` 필드 클래스에 정의한 `date published`는 `pub_date`컬럼에 대한 레이블 문구입니다. Admin 사이트에서 이 문구를 보게 될 것입니다.
- `FK(Foreign Key)`는 항상 다른 테이블의 PK에 연결되므로, `Question` 클래스의 id 변수까지 지정할 필요 없이 `Question` 클래스만 지정하면 됩니다. 실제 테이블에서 FK로 지정된 컬럼은 \_id 접미사가 붙습니다.
- `__str__()` 메소드는 객체를 문자열로 표현할때 사용하는 함수입니다. Admin 사이트나 장고 쉘등에서 테이블명을 보여줘야 하는데 이때 __str__()메서드를 정의하지 않으면 테이블명이 제대로 표시되지 않습니다.

해당 메소드를 설정하면 다음과 같이 데이터 값들이 `object`에서 위에서 설정한 `choice_text`로 보인다.

![django_model4](https://yunsikus.github.io/assets/img/post_img/django-model_4.jpg)

![django_model5](https://yunsikus.github.io/assets/img/post_img/django-model_5.jpg)


## 데이터베이스 변경사항 반영

- 테이블의 신규 생성, 테이블의 정의 변경 등 데이터베이스에 변경이 필요한 사항이 있으면, 이를 데이터베이스에 실제로 반영해주는 작업을 해야 합니다.
- 아직까지는 클래스로 테이블 정의만 변경한 상태입니다. 다음 명령으로 변경사항을 데이터베이스에 반영합니다.


```Python
python3 manage.py makemigrations
python3 manage.py migrate
```

- `makemigrations`명령에 의해 프로젝트 파일 디렉토리 하위에 마이그레이션 파일들이 생기고, 이 마이크레이션 파일들을 이용해 `migrate` 명령으로 데이터베이스 테이블을 만들어줍니다.

- DB에서 `TestQuestion`, `TestChoice` 테이블이 생성된것을 확인할 수 있습니다.


![django_model1](https://yunsikus.github.io/assets/img/post_img/django-model_1.jpg)
