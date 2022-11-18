# Exercise 1
## a
![스크린샷 2022-10-29 오전 9 14 17](https://user-images.githubusercontent.com/78265252/198752845-27d0a692-7968-47ca-91a5-9862d4b8bdf9.png)

```sql
select * from table1 where unsorted between 967 and 969;
```

## b
![스크린샷 2022-10-29 오전 9 16 22](https://user-images.githubusercontent.com/78265252/198752932-8d793139-8d36-4c09-b1d7-ddd10140fcff.png)

```sql
select * from table1 where unsorted in (select unsorted from table1 where unsorted between 967 and 969);
```
## c
![스크린샷 2022-10-29 오전 9 17 36](https://user-images.githubusercontent.com/78265252/198752973-f2eaeb07-b552-4bf6-9c33-2ece88c5af41.png)

```sql
select * from table1 where unsorted = 967 or unsorted = 968 or unsorted = 969;
```
## d
![스크린샷 2022-10-29 오전 9 50 20](https://user-images.githubusercontent.com/78265252/198754319-7b9528a4-74cd-4e32-a736-37e8c511c5f9.png)

```sql
select * from table1 where unsorted = 967 union all select * from table1 where unsorted = 968 union all select * from table1 where unsorted = 969;
```
# Exercise 2
## a
![스크린샷 2022-10-29 오전 10 36 37](https://user-images.githubusercontent.com/78265252/198755933-83a709b5-fa0b-4228-8c59-728d1b9481f0.png)

```sql
explain analyze select * from table1 where unsorted between 967 and 969;
explain analyze select * from table1 where unsorted in (select unsorted from table1 where unsorted between 967 and 969);
explain analyze select * from table1 where unsorted = 967 or unsorted = 968 or unsorted = 969;
explain analyze select * from table1 where unsorted = 967 union all select * from table1 where unsorted = 968 union all select * from table1 where unsorted = 969;
```
## b
![스크린샷 2022-10-29 오전 10 38 05](https://user-images.githubusercontent.com/78265252/198756294-56bc4adf-cab8-413b-8ab4-e44bda158d64.png)

```sql
create index btreeindex on table1 using btree (unsorted);
```
## c
![스크린샷 2022-10-29 오전 10 39 00](https://user-images.githubusercontent.com/78265252/198756677-d5b57f48-7615-42ed-9951-22020475933f.png)

```sql
drop index btreeindex;
create index hashindex on table1 using hash (unsorted);
```

## 비교
- between의 경우 btree가 가장 빠르고 no index와 hash index가 거의 동일한 것을 볼 수 있다. hash index의 경우 equal join이나 natural join만 가능하기 때문에 seq scan을 한 것을 볼 수 있다.
- in query의 경우 btree -> hash -> no index 순으로 빠른 것을 볼 수 있다. btree의 경우 index only scan 다음 index scan이 사용된 것을 볼 수 있다. hash는 seq scan 이후에 in query를 위한 index scan이 사용된 것을 볼 수 있다.
- or query의 경우는 모두 seq scan이 사용된 것을 볼 수 있다. btree의 경우 여러 개의 or 조건을 탐색하기 위해 여러 번의 disk I/O가 발생하고 hash도 bucket 별로 disk I/O 및 bucket 탐색이 필요하기 때문이다.
- union all의 경우 hash -> btree -> no index 순으로 빠른 것을 볼 수 있다. equal join에서 hash index는 높은 성능을 보이고 각 결과를 append 하면 되기 때문이다.

# Exercise 3
## a
![스크린샷 2022-10-29 오후 12 05 07](https://user-images.githubusercontent.com/78265252/198794617-49f3fd64-3303-4671-bb04-2cb4eb4acf6a.png)

```sql
explain analyze select count(*) from (select * from pool1 union all select * from pool2) as tmp;
```
## b
![스크린샷 2022-10-29 오후 12 09 24](https://user-images.githubusercontent.com/78265252/198796609-ca9af71b-01a8-44a7-8378-23454e632b01.png)

```sql
explain analyze select sum(val) from (select count(val) as val from pool1 union all select count(val) as val from pool2) as tmp;
```

## 비교
a의 경우 seq scan 이후에 append를 먼저 하고 count를 하는 것을 볼 수 있다. b의 경우는 각 테이블별로 먼저 count를 하고 2개의 결과값을 append 해서 최종적으로 sum을 하는 것을 볼 수 있다. b의 경우가 append해야 되는 data의 크기가 훨씬 작아서 더 빠른 것이라고 생각된다.

# Exercise 4
## a
![스크린샷 2022-10-29 오후 12 32 35](https://user-images.githubusercontent.com/78265252/198807618-124d2a6b-c768-43fc-a72a-2eaf10f34dc6.png)

```sql
explain analyze select * from pool1 where val >= 250 union select * from pool2 where val >= 250;
```
## b
![스크린샷 2022-10-29 오후 12 32 55](https://user-images.githubusercontent.com/78265252/198807813-f77e3470-04c7-4d54-ae7c-a8655ed45468.png)

```sql
explain analyze select * from (select * from pool1 union select * from pool2) as tmp where tmp.val >= 250;
```

## 비교
a와 b의 plan이 동일하다. b의 경우 filter 없이 먼저 append를 하고 최종 테이블에서 filter를 할 줄 알았는데 a와 동일하게 먼저 where 조건에 따른 filter가 적용된 것을 볼 수 있다. append의 양을 줄이기 위해 내부적으로 최적화하여 where 조건의 filter를 먼저 적용한 것으로 보인다.