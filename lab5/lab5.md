# Exercise 1
## a
![스크린샷 2022-11-05 오후 12 45 20](https://user-images.githubusercontent.com/78265252/200099495-1b7696a1-c9ae-4e3e-9737-ab10f34a6fcd.png)
```sql
SELECT reltuples FROM pg_class WHERE relname='supplier';
```

![스크린샷 2022-11-05 오후 12 47 56](https://user-images.githubusercontent.com/78265252/200099512-01d2f042-f472-412a-85c7-6f55568f6bbe.png)
```sql
SELECT count(*) FROM supplier WHERE s_suppkey<=350;
```

### 분석
레코드의 개수가 10000개이고 primary key이며 범위가 1 ~ 10000이므로 350을 예상할 수 있다.

## b
![스크린샷 2022-11-05 오후 12 49 51](https://user-images.githubusercontent.com/78265252/200099554-e8ec3a34-a48a-4fe1-8c0c-777ab156cf16.png)
```sql
SELECT histogram_bounds FROM pg_stats WHERE tablename='supplier' AND attname = 's_acctbal';
```

![스크린샷 2022-11-05 오후 12 50 51](https://user-images.githubusercontent.com/78265252/200099587-aa4fbc7b-b8d9-4304-b9a2-addd238b5cd5.png)
```sql
SELECT most_common_vals, most_common_freqs FROM pg_stats WHERE tablename='supplier' AND attname='s_acctbal';
```

![스크린샷 2022-11-05 오후 12 51 31](https://user-images.githubusercontent.com/78265252/200099603-7b4ce6bb-8feb-4b14-9b71-af642c9443ec.png)
```sql
SELECT count(*) FROM supplier WHERE s_acctbal<=405.68;
```

### 분석
먼저 histrogram에는 most common vals가 빠져있다. 결과를 보면 most common vals가 2개씩 45개가 존재하므로 histogram에서 총 90개가 빠지게 된다. 따라서 num_bucket이 100이라고 했을 때 하나의 구간에는 99.1개가 존재하게 된다. 또한 most common vals에서 405.68보다 작은 값은 총 5개가 존재한다. 이를 종합해 보면 답을 구하는 수식은 (12 * 100 + (405.68 - 322.65) / (433.93 - 322.65) * 100 + 2 * 5)가 된다. 실제 결과와 비교해 봤을 때 그 답이 일치하는 것을 볼 수 있다.

# Exercise 2
## a
![스크린샷 2022-11-05 오후 1 01 48](https://user-images.githubusercontent.com/78265252/200099903-c0de8afe-4420-4e71-a30c-a24903c654c0.png)
```sql
explain analyze SELECT * FROM supplier WHERE s_suppkey<=350;
```

## b
![스크린샷 2022-11-05 오후 1 02 33](https://user-images.githubusercontent.com/78265252/200099940-40fb96ef-a3f2-4313-9621-a58deba7e3e5.png)
```sql
explain analyze SELECT * FROM supplier WHERE s_suppkey>350;
```

## 비교
s_suppkey의 경우 clustering index이다. a와 같은 경우 seq scan을 한다고 하면 filter(s_suppkey <= 350)를 계속 검사해야 되지만 index scan을 이용한다면 350에 해당되는 pointer를 찾고 뒤의 방향으로 seq scan을 하면 더 효율적이다. b와 같은 경우는 탐색해야 되는 범위가 매우 넓은데 index scan의 경우 index 탐색을 위한 더 많은 disk I/O를 요구하므로 seq scan이 더 효율적이다.

# Exercise 3
## a
![스크린샷 2022-11-05 오후 1 42 14](https://user-images.githubusercontent.com/78265252/200101479-5d2e20ab-af78-46b4-8c79-b67fa62ff006.png)
```sql
explain analyze SELECT count(*) FROM t1 NATURAL JOIN t2 NATURAL JOIN t3 NATURAL JOIN t4;
```

## b
![스크린샷 2022-11-05 오후 1 42 51](https://user-images.githubusercontent.com/78265252/200101506-67537ea6-9fef-4dd3-8c04-03b8bb399b09.png)
```sql
explain analyze SELECT count(*) FROM t4 NATURAL JOIN t3 NATURAL JOIN t2 NATURAL JOIN t1;
```

## 비교
- a의 경우 t1과 t4를 hash join, t2와 t3를 merge join, 최종적으로 t1과 t2를 merge join 하게 된다. t2와 t3을 먼저 merge join 하는 것이 중간 결과 relation을 저장하는데 적은 용량을 필요로 한다. 또한 레코드의 개수를 고려할 때 t2, t3를 이용하는 것이 정렬하는데 적은 비용이 든다.
- t1과 t2를 hash join, t2와 t3를 hash join, t3와 t4를 hash join 하게 된다. 중간 결과 relation의 크기가 작을수록 유리하므로 레코드 크기가 작은 t1, t2, t3, t4 순서대로 hash join을 한 것으로 보인다.

# Exercise 4
## a
![스크린샷 2022-11-05 오후 2 08 38](https://user-images.githubusercontent.com/78265252/200102281-658f42c0-7a48-4eb9-9d0d-ce858186304e.png)
```sql
explain analyze SELECT count(*) FROM t1 NATURAL JOIN t2 NATURAL JOIN t3 NATURAL JOIN t4;
```

## b
![스크린샷 2022-11-05 오후 2 09 16](https://user-images.githubusercontent.com/78265252/200102305-85195254-76be-4559-b164-abedd2ac792a.png)
```sql
explain analyze SELECT count(*) FROM t4 NATURAL JOIN t3 NATURAL JOIN t2 NATURAL JOIN t1;
```

## 비교
`SET join_collapse_limit=1;`를 통해 주어진 query를 그대로 실행해야 한다. 결과를 보면 a가 훨씬 빠른 것을 볼 수 있다. 위에서 설명한 바와 같이 중간 결과 relation의 크기가 작을수록 저장 용량과 계산 비용이 적게 소요된다. 따라서 레코드 개수가 적은 t1, t2, t3, t4 순서대로 join을 하는 것이 훨씬 효율적이다.