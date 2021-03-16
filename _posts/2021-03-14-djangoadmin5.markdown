---
layout: post
title: "[Django] admin - export기능 추가"
subtitle: "[Django] admin - export기능 추가"
categories: Development Django
tags: Django Model admin export
comments: true
---

## export 기능 추가하기

이번에는 admin페이지에서 export 기능을 추가하는 방법을 알아봅시다. django-import-export는 csv, xlsx등의 파일을 읽어 데이터베이스에 저장하거나, 데이터베이스의 내용을 csv, xlsx등의 파일로 저장하는 기능을 제공합니다. 지원되는 파일 포맷은 tablib가 지원하는 형식이며 tablib는 Exel, Json, YAML, HTML, Jira, TSV, ODS, CSV, DBF를 지원합니다.
(import export와 연동되는게 0.14 버전이라 그 이상을 설치하면 형식이 뜨지 않는다.)


## 1. import-export resource를 생성
django-import-export를 우리가 생성한 모델과 연도시키기 위해서 admin.py에 ModelResouce를 생성해줍니다. 이 class에서는 어떤 컬럼들을 대상으로 추출할지, 순서는 어떻게 추출할지 등 다양한 커스터마이징 기능등을 제공합니다.  

```python
class XprdMasterTagsetMeasureResource(resources.ModelResource):
    class Meta:
        model = XprdMasterTagsetMeasure
```

## 2. 다양한 커스터마이징 기능을 알아봅시다.

특정 fields만 추출하고 싶을 시 fields 옵션을 통해 지정 가능합니다.
name과 type만 추출하고 싶을 시 다음과 같이 지정해줍니다. 

```python
class XprdMasterTagsetMeasureResource(resources.ModelResource):
    class Meta:
        model = XprdMasterTagsetMeasure
        fields = ('name','type')
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
