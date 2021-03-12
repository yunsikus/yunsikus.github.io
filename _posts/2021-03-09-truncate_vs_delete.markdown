---  
layout: post  
title: "Truncate VS Delete"  
subtitle: "Truncate VS Delete"  
categories: DB
tags: Mysql MariaDB InnoDB Truncate Delete Drop
comments: true  

---  

join update를 하기 위한 테이블을 임시 생성 후 삭제하는 작업이 필요했습니다. 테이블을 삭제하는 방법에는 크게 `Truncate`, `Delete`이 있어 각각의 특징을 이번 기회에 정리해보았습니다. (Drop은 따로 정리)

## Truncate VS Delete

| 특성 |Truncate|Delete|
|----|-----|-----|
|DDL과 DML|DDL(데이터를 정의하는 언어로서 개체를 만들고 변경 삭제하는 CREATE ALTER DROP문과 같은 것들)로  `AUTOCOMMIT`되기 때문에 `Rollback` 이 불가능하다|DML(데이터 조작 언어로서 데이터를 가공하는 SELECT, INSERT, UDPATE문과 같음)로 `사용자 COMMIT` 이라 `Rollback`이 가능하다.
|트랜잭션 로그|truncate Table은 테이블의 데이터를 저장하는데 사용되는 페이지의 할당을 취소하는 방식으로 데이터 제거. `삭제된 페이지`만을 트랜잭션 로그에 기록하기 때문에 로그 공간을 덜 사용함. → 속도가 빠름|Delete Table은 행을 한번에 하나씩 제거하고 `삭제된 각 행`에 대해 트랜잭션 로그를 기록.
|잠금|`테이블`과 `페이지`를 잠그기 때문에 상대적으로 적은 수의 잠금이 사용됨.|삭제를 위해 테이블의 각 `행`이 잠긴다.
|테이블 정의|테이블을 최초 생성된 초기 상태로 만듬. 테이블 정의는 삭제되지 않음.|데이터만 삭제. 테이블 정의는 삭제되지 않음.
|저장공간|테이블이 사용했던 Storage중 최초 테이블 생성 시 할당된 Storage만 남기고 Release처리 된다. 즉 저장공간이 유지되지 않는다.|데이터를 모두 Delete해도 사용했던 Storage는 Release처리 되지 않는다. 즉 저장공간이 유지된다.
|일부삭제 가능여부|`where`절 사용하여 일부 삭제 불가. 전체 삭제만 가능|`where`절 사용하여 일부 삭제 가능

## 테이블 정의와 저장공간 비교
![truncate_delete1](https://yunsikus.github.io/assets/img/post_img/truncate_delete.jpg)


`DELETE`는 데이터만 삭제하여 저장공간이 남아있고 테이블 정보는 삭제되지 않음.

`TRUNCATE`는 `DROP` 후에 `CREATE`하는 것과 같아 저장공간이 남아있지 않으며, 테이블 정보는 삭제되지 않음

Join을 위한 임시테이블의 경우, 테이블 정보를 남길 필요가 없기 때문에 `DELETE` 나 `TRUNCATE` 보다는 `DROP`으로 제거가 필요.

---

## Reference

Delete와 Truncate의 차이점([https://pointnet.tistory.com/334](https://pointnet.tistory.com/334))

SQL 데이터 삭제 TRUNCATE, DROP, TABLE([https://pathas.tistory.com/93](https://pathas.tistory.com/93))

Difference between TRUNCATE, DELETE and DROP commands([https://jhnyang.tistory.com/56](https://jhnyang.tistory.com/56))
