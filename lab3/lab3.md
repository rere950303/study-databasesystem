# Exercise 1
## a
![스크린샷 2022-10-15 오후 7 30 21](https://user-images.githubusercontent.com/78265252/195981696-5886d7d7-fd82-4cf4-8fe0-adbbe6f0e99c.png)

```sql
explain analyze select * from supplier join nation on s_nationkey = n_nationkey;
```

## b
![스크린샷 2022-10-15 오후 7 31 26](https://user-images.githubusercontent.com/78265252/195981733-7022a2f0-5590-4a9c-a0aa-2115da9268d1.png)

```sql
explain analyze select * from supplier join nation on s_nationkey = n_nationkey order by s_nationkey;
```
## 결과
첫 번째는 hash join, 두 번째는 merge join이 사용되었다. equi join에서 hash를 이용하면 같은 해쉬값을 갖는 partition만 비교하면 되므로 더 효율적이다. join의 결과를 search key를 기준으로 정렬을 해야 한다면 정렬 이후에 join을 진행하는 merge join이 더욱 효과적이다.

# Exercise 2
## a
![스크린샷 2022-10-15 오후 7 38 28](https://user-images.githubusercontent.com/78265252/195982073-ae1583ad-96d3-4381-b272-2a1ce1238ddb.png)

```sql
explain analyze select * from nation as n1 join nation as n2 on n1.n_nationkey = n2.n_nationkey;
```

## b
![스크린샷 2022-10-15 오후 7 39 18](https://user-images.githubusercontent.com/78265252/195982103-bd2cceca-7b8d-4390-93d4-50b783f8e012.png)

```sql
explain analyze select * from nation as n1 join nation as n2 on n1.n_nationkey != n2.n_nationkey;
```

## 결과
첫 번째는 hash join, 두 번째는 nested loop join이 사용되었다. hash join의 경우 equal 조인에서 매우 효과적이지만 not eqaul 연산에서는 사용될 수 없다. 작은 relation의 전체 크기가 메모리 버퍼에 들어갈 수 있다면 nested loop join의 효율이 높아지는데 예제는 이와 같은 케이스에 해당한다고 생각한다.

# Exercise 3
## a
![스크린샷 2022-10-15 오후 8 16 06](https://user-images.githubusercontent.com/78265252/195983616-6d3588a4-2c45-418d-8e96-0edbe9396f4b.png)

```sql
explain analyze select * from supplier join table1 on s_suppkey = sorted;
```

## b
![스크린샷 2022-10-15 오후 8 16 26](https://user-images.githubusercontent.com/78265252/195983629-f9583f65-ae72-4f5e-86ba-896d0292576d.png)

```sql
explain analyze select * from supplier join table1 on s_suppkey = unsorted;
```

## 결과
둘 다 equi join이므로 hash join이 사용되었다. sorted의 경우 정렬이 되어 있므로 bucket에서 일정 부분만 탐색하면 되지만 unsorted의 경우 버켓의 모든 레코드를 탐색해야 되므로 시간이 조금 더 소요되는 것을 볼 수 있다.

# Exercise 4
## a
![스크린샷 2022-10-15 오후 8 22 24](https://user-images.githubusercontent.com/78265252/195983829-7b9892bb-ab4a-4b77-b01b-8676bc60bfe4.png)

```sql
explain analyze select * from supplier join table1 on s_suppkey = sorted;
```

## b
![스크린샷 2022-10-15 오후 8 22 46](https://user-images.githubusercontent.com/78265252/195983853-67603dba-a49d-4157-9959-7133005e81cb.png)

```sql
explain analyze select * from supplier join table1 on s_suppkey = unsorted;
```

## 결과
둘다 merge join이 사용된 것을 볼 수 있다. 정렬되어 있는 sorted와 join 하는 경우 seq scan이 가능하므로 cost가 상대적으로 낮다. 하지만 unsorted와 join 하는 경우 hybrid merge-join으로써 btree의 leaf entry와 merge 한 이후 physical address로 정렬을 하여 여러 번 block을 seek + transfer를 해야 하므로 성능이 낮다.