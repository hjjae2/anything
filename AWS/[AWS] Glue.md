![](../images/[AWS]%20Glue_43.png)

<br>

Crawler 는 Data Stores(S3, ...) 를 분석한다. <br>
파티션, 스키마를 파악하여 (Data Catalog)Table 을 생성한다.
- Table 은 파티션 정보를 가지고 있는데, 이 파티션 정보가 중요하다.
  - 데이터를 조회할 때 스캔한 파티션을 기반으로(?) 데이터 조회가 가능하다.

데이터를 조회할 때 스캐한 파티션을 기반으로, DataStore 를 조회 & 결과를 반환하는 것 같다.
- 따라서, 한번 스캔된 파티션(= DataStore 쪽에)에 데이터를 추가/삭제하면 바로바로 확인할 수 있다.

<br>

**[예시 : 스캔된 파티션]**

Data Store(ex: S3의 Path A) 해당 파티션(Path)에 대해 이미 스캔되었다면, (해당 파티션, path 위치에)S3에 데이터를 삽입,삭제하면 바로바로 조회가 가능하다.

<br>

**[예시 : 스캔되지 않은 파티션]**

반면에 DataStore(ex: S3의 Path B) 해당 파티션(Path)에 대해 한번이라도 스캔되지 않았다면, 이는 크롤러에 의해 최초 한번은 스캔되어야 한다. 그렇지 않으면 사용할 수 없다.

<br><br>

### Deprecated

![](../images/[AWS]%20Glue_44.png)

> https://docs.aws.amazon.com/glue/latest/dg/crawler-configuration.html

- 처음에는 위(`DeleteBehavior`)에서 언급한 'a deleted object' 가 S3의 object 인 줄 알았는데, Data Store 를 의미하는 것 같다.

- Data Store 에 정상적으로 접근하지 못하는 경우를 의미하는 것 같다. (해당 Path 를 찾지 못해 접근하지 못하거나, 권한이 없을 때? 등의 상황이 있을 것 같다.)

**[예시: 로그]**

```sh
2022-01-01T00:00:00.000+09:00	[0123456-7890-1234-5678-1234567890] BENCHMARK : Running Start Crawl for Crawler {크롤러명}
2022-01-01T00:00:00.000+09:00	[0123456-7890-1234-5678-1234567890] BENCHMARK : Classification complete, writing results to database {DB명}
2022-01-01T00:00:00.000+09:00	[0123456-7890-1234-5678-1234567890] INFO : Crawler configured with SchemaChangePolicy {"UpdateBehavior":"LOG","DeleteBehavior":"DEPRECATE_IN_DATABASE"}.
2022-01-01T00:00:00.000+09:00	[0123456-7890-1234-5678-1234567890] INFO : Table {테이블명} is marked deprecated because a matching schema was not found at the table's location.
2022-01-01T00:00:00.000+09:00	[0123456-7890-1234-5678-1234567890] INFO : Found partition of table {테이블명} with partition values [1, A, C, 2] with no matching schema at the partition's S3 location
2022-01-01T00:00:00.000+09:00	[0123456-7890-1234-5678-1234567890] BENCHMARK : Finished writing to Catalog
2022-01-01T00:00:00.000+09:00	[0123456-7890-1234-5678-1234567890] INFO : Run Summary For TABLE:
2022-01-01T00:00:00.000+09:00	[0123456-7890-1234-5678-1234567890] INFO : UPDATE: 1
2022-01-01T00:00:00.000+09:00	[0123456-7890-1234-5678-1234567890] BENCHMARK : Crawler has finished running and is in state READY
```

<br><br>

(테스트 해보니)한번 `Deprecated` 마킹되면 정상적으로 돌아와도 마킹이 풀리지 않는 것 같다.

![](../images/[AWS]%20Glue_25.png)

> https://docs.aws.amazon.com/glue/latest/dg/console-tables.html
> 
> ㄴ (깔끔하게 유지하기 위해서) 삭제 후 재생성하는 것도 방법이 될 수 있을 것 같다.