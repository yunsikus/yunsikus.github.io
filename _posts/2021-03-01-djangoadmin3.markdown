---
layout: post
title: "[Django] admin - 데이터조작"
subtitle: "[Django] admin - 데이터조작"
categories: Development Django
tags: Django Model
comments: true
---

## Admin 데이터 조작하기

Admin 사이트가, UI로 보면서 데이터를 관리할 수 있어 편리하다면, 쉘 데이터 처리는 복잡한 조건 검색처럼, Admin 사이트보다는 더 다양한 데이터 관리 명령이 가능합니다.

따라서, 간단하거나 일반적인 데이터 관리 또는 UI에서 데이터 모습을 확인하고자 할 때는 주로 Admin 사이트를 이용하고, 복잡한 데이터 처리 또는 별도로 웹 브라우저로 접속할 필요가 없는 경우 쉘로 데이터 처리하는 것이 보통입니다.

장고 파이썬 쉘을 시작하려면 다음과 같은 명령을 실행하면 됩니다.

```python
python3 manage.py shell
```

## 1. Create - 데이터 생성/입력
데이터 입력 즉 테이블에 레코드를 생성하기 위해서는 필드값을 지정하여 객체를 생성한 후에 `save()` 메소드를 호출하면 됩니다. 이 명령은 내부적으로 SQL 용어의 `INSERT` 문장을 실행합니다.

```python
from test1.models import Question, Choice
from django.utils import timezone

q = Question(question_text="What's new?", pub_date=timezone.now())
q.save()
```

## 2. Read - 데이터 조회
데이터베이스로부터 데이터를 조회하기 위해서는 `QuerySet` 객체를 사용합니다. `QuerySet`은 데이터베이스 테이블로부터 꺼내 온 객체들의 콜렉션입니다. `QuerySet`은 필터를 가질 수 있으며, 필터를 사용하여 QuerySet 내의 항목 중에서 조건에 맞는 레코드만 다시 추출합니다. SQL 용어로 `QuerySet`은 `SELECT` 문장에 해당하며, `필터`는 `WHERE` 절에 해당합니다.

조회 결과를 담는 `QuerySet`을 얻으려면 `objects`객체를 사용합니다. `objects` 객체는 테이블 정보를 담고 있는 객체입니다. 만일 다음과 같은 명령을 실행하면 테이블에서 모든 Question 객체를 담고 있는 `QuerySet` 콜렉션을 반환합니다. 데이터베이스 용어로 Question 클래스는 테이블에 해당하고, Question 객채는 특정 레코드 하나에 해당하며, `objects.all()`은 레코드들 모두를 의미합니다.  

```python
Question.objects.all() # 'Question테이블, 레코드들 모두' 라고 해석하면 됨
```

모든 레코드가 아니라 조건에 맞는 일부 레코드만 검색할ㄷ 때는 `filter()` 와 `exclude()` 메소드를 사용합니다.
- **filter 메소드** : 주어진 조건에 맞는 객체들을 담고 있는 `QuerySet` 콜렉션을 반환함
- **exclude 메소드** : 주어진 조건에 맞는 객제들을 재외한 `QuerySet` 콜렉션을 반환함

이러한 `QuerySet` 메소드들은 실행결과 또한 `QuerySet` 콜렉션을 반환하므로, 다음과 같이 체인식 호출이 가능합니다.

```python
import datetime
Question.objects.filter(
  question_text__startswith="What"
).exclude(
  pub_date__gte=datetime.date.today()
)
```

한 개의 요소만 있는 것이 확실한 경우에는 get() 메소드를 호출하면 됩니다. 호출 결과는 `QuerySet`이 아니라 한 개의 객체입니다.

```python
one_entry = Question.objects.get(pk=1)
## <Question: 연식은 몇살입니까>
```
또한, `QuerySet` 요소의 개수를 제한하기 위하여 다음처럼 파이썬의 배열 슬라이싱 문접을 사용할 수도 있습니다. 이는 SQL 용어로 `OFFSET`, `LIMIT` 절에 해당합니다.

```python
Question.objects.all()[:2]
```

## 3. Update - 데이터 수정
이미 존재하는 객체에 대한 필드값을 수정하는 경우에도 필드 속성값을 수정한 후 `save()` 메소드를 호출하면 됩니다. 이는 SQL 용어로 `UPDATE`절에 해당합니다.

```python
q = Question.objects.get(pk=4)
q.question_text = "What is your favorite hobby?"
q.save()
```

여러 개의 객체를 한꺼번에 수정할 수도 있는데, 다음과 같이 `update()` 메소드를 사용합니다.

```python
Question.objects.filter(pub_date__gte=datetime.date.today()).update(question_text="updated question")
```

## 4. Delete - 데이터삭제

객체를 삭제하기 위해서는 `delete()` 메소드를 사용합니다. 아래 문장은 `pub_date` 필드의 날짜가 오늘인 모든 객체를 삭제하는 명령입니다. `delete()` 메소드는 SQL용어로 `DELETE`절에 해당합니다.

```python
Question.objects.filter(pub_date__gte=datetime.date.today()).delete()
```
