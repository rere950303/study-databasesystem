# Exercise 1
![스크린샷 2022-10-01 오후 12 18 12](https://user-images.githubusercontent.com/78265252/193392399-4d2e0e97-d5a4-4b1b-a0b2-5cb109cab609.png)

```sql
create index index_sorted on table1(sorted);
create index index_unsorted on table1(unsorted);
```
- `sorted`의 경우 정렬되어 있으므로 clustered index
- `unsorted`의 경우 정렬되어 있지 않으므로 non-clustered index

# Exercise 2
## a
### seq scan
![스크린샷 2022-10-01 오후 12 40 01](https://user-images.githubusercontent.com/78265252/193392405-bb9829b7-9674-4eb2-b43f-7f5f2cd1b22d.png)

```sql
explain analyze select rndm from table1 where rndm = 1005;
```

### index scan
![스크린샷 2022-10-01 오후 12 41 07](https://user-images.githubusercontent.com/78265252/193392415-395a289d-400c-41ec-b06a-6e17bd8b3f0b.png)

```sql
explain analyze select sorted, rndm from table1 where sorted = 1999231;
```

### index only scan
![스크린샷 2022-10-01 오후 12 41 45](https://user-images.githubusercontent.com/78265252/193392432-9476f69b-e160-40b6-b33e-9f5dc7143e81.png)

```sql
explain analyze select sorted from table1 where sorted = 1999231;
```

## b
### clustered index
![스크린샷 2022-10-01 오후 12 44 32](https://user-images.githubusercontent.com/78265252/193392448-658e4ba4-57fc-444f-ac77-071ec8ca0de8.png)

```sql
explain analyze select sorted from table1 where sorted > 1999231;
```

### non-clustered index
![스크린샷 2022-10-01 오후 12 45 07](https://user-images.githubusercontent.com/78265252/193392458-21a066cc-c8eb-4962-9f67-02008dda1589.png)

```sql
explain analyze select unsorted from table1 where unsorted > 1999231;
```

### 시간 비교
clustered index의 경우 1999231을 찾은 다음 relation에서 linear scan을 하면 되지만 non-clustered index의 경우 계속해서 B-tree를 탐색하면서 레코드에 대한 포인터를 가져와야 하므로 시간이 상대적으로 더 소요된다.

## c
![스크린샷 2022-10-01 오후 12 52 47](https://user-images.githubusercontent.com/78265252/193392491-a39e56e4-9e99-4586-8381-9b1821859aa7.png)

```sql
explain analyze SELECT sorted, rndm FROM table1 WHERE sorted>1999231 AND rndm=1005;
```

![스크린샷 2022-10-01 오후 12 53 07](https://user-images.githubusercontent.com/78265252/193392499-78e5e40a-0416-4989-86be-0016f6435469.png)

```sql
explain analyze SELECT sorted, rndm FROM table1 WHERE sorted<1999231 AND rndm=1005;
```

![스크린샷 2022-10-01 오후 12 57 31](https://user-images.githubusercontent.com/78265252/193392506-fafb126c-8cc3-4587-9a0c-611ccd532378.png)

```sql
select sorted from table1 where sorted = (select max(sorted) from table1);
```

### query plan 다른 이유
`sorted` 컬럼의 최댓값이 1999999인 것을 볼 수 있다. 첫 번째 쿼리의 경우 index를 통한 레코드를 찾은 이후 linear 탐색해야 되는 범위가 매우 적지만 두번째 쿼리의 경우 index를 이용해 레코드를 찾은 이후 linear 탐색해야 되는 범위가 매우 넓다. 따라서 seq scan과 비교하였을 때 큰 효과가 없으며 오히려 B-tree 탐색을 위한 오버헤드 발생으로 효율이 더 떨어질 수도 있다.

# Exercise 3
## inserting first
![스크린샷 2022-10-01 오후 1 16 38](https://user-images.githubusercontent.com/78265252/193392515-cb1d20d2-1881-40b5-9052-96494c7445bc.png)

```sql
create table table10 (val integer);
insert into table10 (select * from pool);
create index tmp on table10(val);
```
- 소요시간: 8244.707 ms

## creating index first
![스크린샷 2022-10-01 오후 1 20 10](https://user-images.githubusercontent.com/78265252/193392519-aa31a423-bc5e-4eb0-8e4c-3f12effc245a.png)

```sql
create table table20 (val integer);
create index tmp2 on table20(val);
insert into table20 (select * from pool);
```
- 소요시간: 12058.196 ms

## 시간이 차이 나는 이유
insert 이후에 index를 생성하면 정렬되어 있는 컬럼에서 bottom-up 방식으로 B-tree를 빠르게 생성할 수 있다. 하지만 index를 생성한 이후 insert를 하게 되면 레코드가 삽입될 때마다 B-tree를 탐색해야 되고 노드의 크기가 일정 수준 넘어가게 되면 split 연산까지 추가적으로 발생한다. 이와 같은 이유로 insert를 먼저 하는 것이 효율적이다.