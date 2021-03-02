---
layout: post
title: "[Django] model - 기존 테이블 연동"
subtitle: "[Django] model - 기존 테이블 연동"
categories: Development Django
tags: Django Model inspectdb
comments: true
---

## 기존 테이블 연동하기

학습하고 있는 강의와 책에서는 모두 테이블을 새롭게 생성합니다. 하지만 실제 장고를 적용하는 환경에서는 기존에 가지고 있는 테이블을 활용하는 경우가 많을 것입니다. 이번 장에서는 기존 테이블을 연동하는 방법을 알아보겠습니다.

## 1. inspectdb로 연동할 예정인 테이블의 구조를 확인합니다.

```python
python3 manage.py inspectdb
```

다음 명령어를 날리면, db의 테이블들을 model.py 형식으로 가져옵니다. 제가 연동하고자 하는 테이블명은 `XprdMasterTagsetMeasure`이고 이 테이블의 구조를 다음과 같이 읽어옵니다.

![django_model_inspectdb1](https://yunsikus.github.io/assets/img/post_img/django-model-inspectdb1.jpg)


## 2. model.py에 연결하려는 테이블을 붙여넣습니다.

연결하고 싶은 테이블 구조를 복사해서 model.py로 붙여넣습니다.
이때 2가지를 주의합니다.
- 첫번째 변수인 `id`에 `primary_key`를 설정해줍니다.

장고는 기본적으로 각각의 모델에 `id`필드를 자동으로 추가해줍니다. 만약 `id`필드를 `primary_key`로 사용하지 않을 시 해당 필드를 삭제해도 됩니다.

- `managed = True` 로 변경해줍니다.

`managed = False`이면 django를 통해서 각 테이블의 생성, 수정 삭제등이 적용되지 않습니다. 따라서 `True`로 변경해줍니다.  

```python
class XprdMasterTagsetMeasure(models.Model):
    id = models.IntegerField(primary_key=True) # primary_key 옵션 추가
    name = models.CharField(max_length=180)
    type = models.CharField(max_length=180, blank=True, null=True)
    replace_to = models.CharField(max_length=180)
    create_date = models.DateTimeField(blank=True, null=True)
    update_date = models.DateTimeField(blank=True, null=True)
    memo = models.CharField(max_length=2000, blank=True, null=True)

    class Meta:
        managed = False # True로 변경
        db_table = 'xprd_master_tagset_measure'
```

## 3. Makemigrations Migrate 합니다.

이후에는 똑같이 `makemigrations`, `migrate`  해줍니다.

```python
python3 manage.py makemigrations
python3 manage.py migrate
```

다만, 해당 테이블이 이미 db에 존재하기 때문에 다음과 같은 에러 메시지를 마주칩니다.

```
"Table 'xprd_master_tagset_measure' already exists"
```
 이 상태로 데이터를 확인해도, 테이블이 연동된것을 확인할 수 있습니다. 다만, 앞으로 다른 테이블을 생성하여 `migrate`해도 해당 `migration`의 영향을 받기 때문에 거짓으로 `migrate`을 했다는 명령문을 입력해줍니다.

 ```python
 python3 manage.py migrate --fake
 ```
