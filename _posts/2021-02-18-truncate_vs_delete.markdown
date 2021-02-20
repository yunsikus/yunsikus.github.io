---  
layout: post  
title: "DB Isolation Level"  
subtitle: "DB Isolation Level"  
categories: DB
tags: Mysql MariaDB InnoDB Transaction
comments: true  


---  
## Isolation Level이란?

동시에 여러 `트랜잭션`이 처리될 때, `트랜잭션`끼리  얼마나 서로 독립되어있는지 나타내는 레벨입니다. 즉, 간단하게 말해 특정 `트랜잭션`이 다른 `트랜잭션`에 변경한 데이터를 볼 수 있도록 허용할지 말지를 결정하는 것

예를 들어, 한 사용자가 어떠한 데이터를 수정하고 있는 경우 다른 사용자들이 그 데이터에 접근하는 것을 차단함으로써 완전한 데이터만을 사용자들에게 제공할 수 있습니다. (`무결성`)

반면 많은 사용자들의 수정 작업으로 인하여 통계 자료를 작성할 수 없는 사용자를 위하여 읽기 작업을 수행할 수 있도록 Isolation Level을 변경할 수 있습니다.(`동시성`)

Isoaltion Level을 완화하는 경우 `동시성`이 증가되는데 반해, 데이터의 `무결성`에 문제가 발생할 수 있거나, 데이터의 `무결성`을 완벽하게 유지하는 데 반하여 `동시성`이 떨어질 수 있습니다. → `무결성`과 `동시성`은 `trade off` 관계에 있음

isolation level 확인 쿼리

```sql
show variables like 'tx_isolation';
```

## 4 Types of Transaction Isolation Level

### Read Uncommited (동시성 GOOD, 무결성 BAD)

- 아직 다른 트랜잭션에서 `COMMIT` 되지 않은 데이터들을 읽어올 수 있는 level
- `Dirty Read` 문제가 발생
    - 어떤 `트랜잭션`에서 아직 실행이 끝나지 않은 다른 `트랜잭션`에 의한 변경사항을 보게 되는 경우.
    - 만약 원래 `트랜잭션`에서 그 변경 사항을 `롤백`하면 그 데이터를 읽은 `트랜잭션`은 dirty 데이터를 가지고 있다고 말한다.

    1. A 트랜잭션에서 10번 사원의 나이를 27살에서 28로 바꿈
    2. 아직 COMMIT하지 않음
    3. B 트랜잭션에서 10번 사원의 나이를 조회함
    4. 28살이 조회됨

     5. A 트랜잭션에서 RollBack됨 (27살이 되고 28살은 없는 데이터가 됨)



### Read Committed

- 어떤 트랜잭션의 변경내용이 COMMIT 되어야만 다른 트랜잭션에서 조회할 수 있는 LEVEL.
- `Nonrepeatable Read` 문제가 발생
    - 한 트랜잭션에서 SELECT 쿼리의 결과가 동일해야 하지만 그 결과가 다른 경우

1. B 트랜잭션에서 10번 사원의 나이를 조회
2. 27살이 조회됨
3. A 트랜잭션에서 10번 사원의 나이를 27살에서 28살로 바꾸고 COMMIT
4. 28살이 조회됨

### Repeatable Read

- 트랜잭션이 시작되기 전에 커밋된 내용에 대해서만 조회할 수 있는 LEVEL
- 자신의 트랜잭션 번호보다 낮은 트랜잭션 번호에서 변경된(COMMIT) 것만 보게된다.
    - 모든 InnoDB의 트랜잭션은 고유한 트랜잭션 번호(순차적으로 증가하는)를 가지고 있으며 언두 영역에 백업된 모든 레코드는 발생시킨 트랜잭션 번호가 포함되어 있다.

1. 10번 트랜잭션이 500000번 사원을 조회
2. 12번 트랜잭션이 500000번 사원의 이름을 변경하고 커밋
3. 10번 트랜잭션이 500000번 사원을 다시 조회
4. 언두 영역에 백업된 데이터 반환(직전 트랜잭션의 결과값)

- `Phantom Read` 문제가 발생
    - 한 트랜잭션 내에서 쿼리를 두 번 실행하였는데, 첫 번째 쿼리에서 없던 유령(Phantom) 레코드가 두 번째 쿼리에서 나타나는 현상.
    - Repeatable READ 이하에서만 발생하고(SERIALIZABLE은 발생하지 않음), INSERT에 대해서만 발생
    - 예를 들어 잔고가 100달러 미만인 계좌를 모두 찾아내는 `트랜잭션`이 있고, 이 `트랜잭션`에서 그 데이터를 두 번 읽는다고 가정. 처음 데이터를 읽어들이고 난 후에 다른 `트랜잭션`에서 잔고가 0달러인 계좌를 새로 만들면(INSERT) 이 계좌도 잔고가 100달러 이하라는 조건에 맞게 됩니다.

```sql
START TRANSACTION; -- transaction id : 1
SELECT id FROM Balance_under_100; -- 0건 조회

		START TRANSACTION; -- transaction id : 2
		INSERT FROM Balance_under_100 id = 1;
		COMMIT;

SELECT * FROM Balance_under_100; -- 여전히 0건 조회
UPDATE Balance_under_100 SET name = 'yunsik' where id = 1';
SELECT * from Balance_under_100; --  1건 조회
COMMIT;
```

### Serializable (무결성 GOOD, 동시성 BAD)

- 가장 단순하고 가장 엄격한 LEVEL
- InnoDB에서 기본적으로 순수한 SELECT 작업은 아무런 잠금을 걸지 않고 동작하는데 Serializable일 경우 다른 트랜잭션에서 이 레코드를 변경하지 못하게 한다.
## DB type별 기본 Isolation Level

|이름|Isolation Level|특징|
|--|--|--|
|Mysql(InnoDB)| Repeatable-Read|Repeatable-Read에서도 Phantom Read 현상이 발생하지 않음(update시는 생김)|
|PostgreSQL|Read-Commited|
|Oracle|Read-Commited|


참고
- [https://effectivesquid.tistory.com/entry/데이터베이스-Isolation-Level](https://effectivesquid.tistory.com/entry/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-Isolation-Level)
- [https://jupiny.com/2018/11/30/mysql-transaction-isolation-levels/](https://jupiny.com/2018/11/30/mysql-transaction-isolation-levels/)
- [https://joont92.github.io/db/트랜잭션-격리-수준-isolation-level/](https://joont92.github.io/db/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EA%B2%A9%EB%A6%AC-%EC%88%98%EC%A4%80-isolation-level/)
- mariadb 사이트 isolation level 설명 : [https://mariadb.com/kb/en/mariadb-transactions-and-isolation-levels-for-sql-server-users/](https://mariadb.com/kb/en/mariadb-transactions-and-isolation-levels-for-sql-server-users/)
- [https://jsonobject.tistory.com/427](https://jsonobject.tistory.com/427)
- [https://suhwan.dev/2019/06/09/transaction-isolation-level-and-lock/](https://suhwan.dev/2019/06/09/transaction-isolation-level-and-lock/)
