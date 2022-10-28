# Exercise 1
![스크린샷 2022-10-12 오후 10 06 07](https://user-images.githubusercontent.com/78265252/195350585-2aeabdcb-b5c2-49ee-8e71-f179ac1e7109.png)

```sql
create index hash_index on table_hash using hash (recordid);
create index btree_index on table_btree using btree (recordid);
```

# Exercise 2
## a
![스크린샷 2022-10-12 오후 10 08 29](https://user-images.githubusercontent.com/78265252/195351170-f4037838-f7f7-4fbd-bbd4-c87e0f52ee1a.png)

```sql
explain analyze SELECT * FROM table_btree WHERE recordid=10001;
```

![스크린샷 2022-10-12 오후 10 09 04](https://user-images.githubusercontent.com/78265252/195351248-3773f558-6aad-4413-89e6-4c5b29d7bc73.png)

```sql
explain analyze SELECT * FROM table_hash WHERE recordid=10001;
```

### 비교
둘 다 똑같이 index scan을 이용하는 것을 볼 수 있다. btree의 경우 tree의 높이만큼 디스크 탐색을 해야 되고 hash의 경우 bucket의 overflow가 없다면 hash 값 계산 후 한 번만 디스크 탐색을 하면 되므로 hash index의 경우가 조금 더 빠른 것을 볼 수 있다.

## b
![스크린샷 2022-10-12 오후 10 17 10](https://user-images.githubusercontent.com/78265252/195352912-af5fd118-573e-450a-97f9-03652bd01490.png)

```sql
explain analyze SELECT * FROM table_btree WHERE recordid>250 AND recordid<550;
```

![스크린샷 2022-10-12 오후 10 17 34](https://user-images.githubusercontent.com/78265252/195353046-a9ab7ab4-104e-4289-9f48-e6a453b6e7cb.png)
```sql
explain analyze SELECT * FROM table_hash WHERE recordid>250 AND recordid<550;
```

### 비교
btree의 경우 index scan을, hash의 경우 parallel seq scan을 타는 것을 볼 수 있다. 시간도 hash가 훨씬 많이 소요된다. 현재 index가 cluster이므로 btree의 경우 btree의 leaf node를 linear 탐색을 할 수 있다. 하지만 hash의 경우 record의 분산으로 인해 약 300번에 가까운 hash 값 계산 및 디스크 탐색이 필요하다. 따라서 위와 같은 차이를 보인다.

# Exercise 3
## a
![스크린샷 2022-10-12 오후 10 23 16](https://user-images.githubusercontent.com/78265252/195354334-f72166a2-5d72-40c9-ac7a-74d283506cfd.png)

```sql
explain analyze update table_btree set recordid=9999998 where recordid=9999997;
```

![스크린샷 2022-10-12 오후 10 24 24](https://user-images.githubusercontent.com/78265252/195354575-8ecdf345-3ffe-499e-8ffd-fe1604f2f716.png)

```sql
explain analyze update table_noindex set recordid=9999998 where recordid=9999997;
```
### 비교
btree의 경우 index scan을, noindex의 경우 seq scan을 이용한다. btree의 경우 index의 수정 및 btree의 모양을 다시 변경해야 되기 때문에 index scan을 이용한다. noindex의 경우 seq scan을 해야 되기 때문에 btree에 비해 많은 시간이 소요되는 것을 볼 수 있다.

## b
![스크린샷 2022-10-12 오후 10 30 22](https://user-images.githubusercontent.com/78265252/195356075-b5b28496-1457-4651-b081-90ae7d836dea.png)

```sql
explain analyze update table_btree set recordid=recordid * 2 where recordid>8000000;
```

![스크린샷 2022-10-12 오후 10 32 08](https://user-images.githubusercontent.com/78265252/195356272-0b4959c6-f717-4800-9011-ad55aa4dcb60.png)

```sql
explain analyze update table_noindex set recordid=recordid * 2 where recordid>8000000;
```

### 비교
각각 index scan, seq scan을 타는 이유는 위와 동일하다. 하지만 index scan이 훨씬 더 많은 시간이 소요되는 것을 볼 수 있다. recordid의 값을 2배로 해서 8000000~10000000의 범위가 16000000~20000000로 변경되었는데 btree의 leaf node와 internal node는 정렬이 되어 있어야 하므로 btree의 재생성을 위한 연산이 추가적으로 소요되기 때문이다.

## c
![스크린샷 2022-10-12 오후 10 37 37](https://user-images.githubusercontent.com/78265252/195357530-71874111-2c51-4651-854e-234badbf5509.png)

```sql
explain analyze update table_btree set recordid=recordid * 1.1;
```

![스크린샷 2022-10-12 오후 10 38 57](https://user-images.githubusercontent.com/78265252/195357907-cd77b240-10ac-4e24-93e3-5b0b1e3be12c.png)

```sql
explain analyze update table_noindex set recordid=recordid * 1.1;
```

### 비교
이번에는 b와 다르게 모든 컬럼을 변경하므로 굳이 index를 이용할 필요가 없다. 따라서 모두 seq scan을 이용하는 것을 볼 수 있다. 하지만 위와 마찬가지로 btree의 변경으로 이해 noindex보다 시간이 더 소요되는 것을 볼 수 있다.

# Exercise 4
## a
### indexscan=true
![스크린샷 2022-10-12 오후 11 33 07](https://user-images.githubusercontent.com/78265252/195371362-d0d4ae0d-70c9-449a-96c4-01240b609df0.png)

```sql
explain analyze SELECT * from test0 where x >= 1 and x <= 10 and y >= 1 and y <= 10;
```

![스크린샷 2022-10-12 오후 11 41 53](https://user-images.githubusercontent.com/78265252/195373650-7dc75fe5-5f80-4c0e-a4ce-ea6be9d66439.png)

```sql
explain analyze SELECT * from test1 where test1.p <@ box '((1, 1), (10, 10))';
```

### indexscan=false
![스크린샷 2022-10-12 오후 11 43 08](https://user-images.githubusercontent.com/78265252/195373922-af33d1d1-022c-473d-8be5-20ee31f8e2c2.png)

```sql
explain analyze SELECT * from test0 where x >= 1 and x <= 10 and y >= 1 and y <= 10;
```

![스크린샷 2022-10-12 오후 11 43 40](https://user-images.githubusercontent.com/78265252/195374061-2f8dd93c-8c74-45c1-a560-23f8e5b5e83c.png)

```sql
explain analyze SELECT * from test1 where test1.p <@ box '((1, 1), (10, 10))';
```

### 비교
indexscan이 가능한 경우 Point보다 x, y를 이용하는 것이 더 효율적인 것을 볼 수 있다. rtree는 영역이 겹치는 경우에 한 번에 여러 개의 경로를 탐색해야 될 수도 있기 때문이다.
indexscan이 불가능한 경우 Point를 이용하는 것이 조금 더 효율적이지만 큰 차이는 없는 것을 알 수 있다.

## b
### indexscan=true
![스크린샷 2022-10-13 오전 12 03 05](https://user-images.githubusercontent.com/78265252/195378684-836353c5-c009-49cf-b9d9-0c2709f0d9a1.png)

```sql
explain analyze SELECT * from test2 where testbox && box '((0,0), (1,1))' and testbox && box '((9,9), (10,10))';
```

### indexscan=false
![스크린샷 2022-10-13 오전 12 03 49](https://user-images.githubusercontent.com/78265252/195378854-72968c7c-15c1-44b4-85f5-1a902d6a6ed4.png)

```sql
explain analyze SELECT * from test2 where testbox && box '((0,0), (1,1))' and testbox && box '((9,9), (10,10))';
```

## c
### indexscan=true
![스크린샷 2022-10-13 오전 12 38 26](https://user-images.githubusercontent.com/78265252/195387144-4c506303-469c-4d73-8dc6-a5b350c9bed1.png)

```sql
explain analyze SELECT x * x + y * y as distance from test0 order by distance asc limit 10;
```

![스크린샷 2022-10-13 오전 12 40 09](https://user-images.githubusercontent.com/78265252/195387502-c8e390be-26a8-4ffe-805f-65df585f2134.png)

```sql
explain analyze SELECT p <-> point(0, 0) as distance from test1 order by distance asc limit 10;
```

### indexscan=false
![스크린샷 2022-10-13 오전 12 40 56](https://user-images.githubusercontent.com/78265252/195387692-712211b7-3282-4a1a-809d-781800103464.png)

```sql
explain analyze SELECT x * x + y * y as distance from test0 order by distance asc limit 10;
```

![스크린샷 2022-10-13 오전 12 41 19](https://user-images.githubusercontent.com/78265252/195387793-1c448beb-6148-4c69-a11e-e5a582085d7b.png)

```sql
explain analyze SELECT p <-> point(0, 0) as distance from test1 order by distance asc limit 10;
```

### 비교
indexscan이 가능한 경우 index를 이용하면 index only scan을 하므로 시간이 훨씬 단축되는 것을 볼 수 있다.
indexscan이 불가능한 경우에는 모든 Point에 대해서 거리를 계산해야 하므로 단순히 x, y를 이용한 쿼리가 더 효율적인 것을 볼 수 있다.