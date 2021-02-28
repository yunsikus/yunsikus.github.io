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

질문 테이블에는 질문인 `question` 그리고 질문의 생성 시간인 `pub_date`를 저장할 것입니다.

대답 테이블에는 질문에 해당하는 답변을 저장해야 하므로 질문인 `question`을 Foreignkey로 가져옵니다. 그리고 그 답변인 `choice_text` 그리고 그 대답이 몇표를 얻었는지를 기록하는 `votes`가 있습니다.

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
