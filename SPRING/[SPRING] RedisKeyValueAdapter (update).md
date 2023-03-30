## update

### element 가 컬렉션인 경우

(컬렉션 요소들) 전체 삭제 후 다시 insert

```sql
// 1
127.0.0.1:6379> hgetall test
1) "_class"
2) "~~~"
3) "ids.[0]"
4) "1"
3) "ids.[1]"
4) "2"
```

```sql
// 2
127.0.0.1:6379> hgetall test
1) "_class"
2) "~~~"
```

```sql
// 3
127.0.0.1:6379> hgetall test
1) "_class"
2) "~~~"
3) "ids.[0]"
4) "3"
```

<br>

### 그 외

`hmset`, `hset` 커맨드로 overwrite

<br>

### 참고

```java
public void update(PartialUpdate<?> update) {

    RedisPersistentEntity<?> entity = this.converter.getMappingContext()
            .getRequiredPersistentEntity(update.getTarget());

    String keyspace = entity.getKeySpace();
    Object id = update.getId();

    byte[] redisKey = createKey(keyspace, converter.getConversionService().convert(id, String.class));

    RedisData rdo = new RedisData();
    this.converter.write(update, rdo);

    redisOps.execute((RedisCallback<Void>) connection -> {

        RedisUpdateObject redisUpdateObject = new RedisUpdateObject(redisKey, keyspace, id);

        for (PropertyUpdate pUpdate : update.getPropertyUpdates()) {

            String propertyPath = pUpdate.getPropertyPath();

            if (UpdateCommand.DEL.equals(pUpdate.getCmd()) || pUpdate.getValue() instanceof Collection
                    || pUpdate.getValue() instanceof Map
                    || (pUpdate.getValue() != null && pUpdate.getValue().getClass().isArray()) || (pUpdate.getValue() != null
                            && !converter.getConversionService().canConvert(pUpdate.getValue().getClass(), byte[].class))) {

                redisUpdateObject = fetchDeletePathsFromHashAndUpdateIndex(redisUpdateObject, propertyPath, connection);
            }
        }

        if (!redisUpdateObject.fieldsToRemove.isEmpty()) {
            connection.hDel(redisKey,
                    redisUpdateObject.fieldsToRemove.toArray(new byte[redisUpdateObject.fieldsToRemove.size()][]));
        }

        for (Index index : redisUpdateObject.indexesToUpdate) {

            if (ObjectUtils.nullSafeEquals(DataType.ZSET, index.type)) {
                connection.zRem(index.key, toBytes(redisUpdateObject.targetId));
            } else {
                connection.sRem(index.key, toBytes(redisUpdateObject.targetId));
            }
        }

        if (!rdo.getBucket().isEmpty()) {
            if (rdo.getBucket().size() > 1
                    || (rdo.getBucket().size() == 1 && !rdo.getBucket().asMap().containsKey("_class"))) {
                connection.hMSet(redisKey, rdo.getBucket().rawMap());
            }
        }
        
    ...
```