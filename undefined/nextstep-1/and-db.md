# 인덱스 이해하기 & DB 튜닝

## 수직적 탐색 vs 수평적 탐색 <a href="#vs" id="vs"></a>



<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

**index 구조 - B tree**

index를 이용한 탐색은 수직적 탐색과 수평적 탐색으로 나뉜다. 수직적 탐색은 스캔의 시작점을 찾는 과정이고 수평적 탐색은 수직적 탐색으로 찾은 스캔의 시작점부터 스캔을 진행하면서 데이터를 찾는 과정을 의미한다.

각각의 자세한 사항은 강의 자료에 잘 나와 있어서 첨부한다.

> a. 수직적 탐색\
> 인덱스 수직적 탐색은 루트 블록에서부터 시작합니다. 루트를 포함해 브랜치 블록에 저장된 각 인덱스 레코드는 하위 블록에 대한 주소값을 갖습니다. 수직적 탐색 과정에 찾고자 하는 값보다 크거나 같은 값을 만나면, 바로 직전 레코드가 가리키는 하위 블록으로 이동합니다.
>
> \
>
>
> b. 수평적 탐색\
> 수직적 탐색을 통해 스캔 시작점을 찾았으면 수평적 탐색을 통해 데이터를 찾습니다. 인덱스 리프블록끼리는 양방향 Linked List이므로 서로 앞뒤 블록에 대한 주소값을 갖습니다. 필요한 컬럼을 인덱스가 모두 갖고 있어 인덱스만 스캔하고 끝나는 경우도 있지만, 그렇지 않을 경우 테이블도 액세스해야 합니다. 이 경우 ROWID가 필요합니다. ROWID는 데이터블럭 주소 + 로우 번호(블록내 순번)로 구성됩니다.
>
> \
>
>
> c. 테이블 액세스\
> ROWID가 가리키는 테이블 블록을 버퍼캐시에서 먼저 찾아보고 못찾을 때만 디스크에서 블록을 읽습니다. 우선, 리프블록에서 읽은 ROWID를 분해해서 DBA(데이터 파일 번호 + 블록번호) 정보를 얻습니다. 그리고 DBA를 해시함수에 입력해서 해시 체인을 찾고 거기서 버퍼 헤더를 찾습니다. 실제 데이터가 담긴 버퍼블록은 매번 다른 위치에 캐싱되는데, 그 메모리 주소값을 버퍼 헤더가 가지고 있습니다. 즉, 모든 데이터가 캐싱돼 있더라도 테이블 레코드를 찾기 위해 매번 DBA 해싱과 래치 획득 과정을 반복해야 합니다. 동시 액세스가 심할 때는 캐시버퍼 체인 래치와 버퍼 Lock에 대한 경합까지 발생합니다.

\
\


## Index Range Scan vs Index Full Scan vs Index Unique Scan <a href="#index-range-scan-vs-index-full-scan-vs-index-unique-scan" id="index-range-scan-vs-index-full-scan-vs-index-unique-scan"></a>

### **1) Index Range Scan**

<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

인덱스 루트 블록에서 리프 블록까지 수직적으로 탐색한 후에 리프 블록을 필요한 범위만 스캔하는 방식이다. 여기서 말하는 ‘필요한 범위’란 조건에 맞지 않는 데이터를 만나면 탐색을 멈춘다는 의미이다.

```
where 생년월일 between '20200101' and '20200131'
```

가령 위와 같은 조건 식에서 생년월일 순으로 정렬해두었다고 가정하면, 2020년 1월에 태어난 사람을 찾으려면 2020년 1월 1일 이후부터 찾다가 2020년 2월 1일 이후에 태어난 사람을 만나는 순간 멈추는 것을 의미한다.

조건에 맞지 않는 데이터를 만났을 때에 멈춰도 누락이 없기 위해서는 데이터 자체가 정렬이 되어 있어야 하고 이를 위해서는 인덱스 컬럼을 가공하지 않아야한다.

\
\


### **2) Index Full Scan**

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

수직적 탐색 없이 인덱스 리프 블록을 처음부터 끝까지 수평적으로 탐색하는 방식이며 최적의 인덱스가 없을 때 차선으로 선택한다.

```
SELECT COUNT(*) FROM subway.programmer
```

\
\


**3) Index Unique Scan**

<figure><img src="../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

수직적 탐색만으로 데이터를 찾는 스캔 방식이다. unique 인덱스를 통해 “=” 조건으로 탐색하는 경우에 작동하며, 중복되지 않은 unique한 값을 “=”조건으로 검색할 경우 해당 데이터 한 건을 찾는 순간 더 이상 탐색 하지 않는 방식이다.

```
SELECT * FROM subway.programmer WHERE id = 10
```



## 인덱스 튜닝 <a href="#undefined" id="undefined"></a>

인덱스 스캔 후 테이블 레코드를 엑세스 할 때, ROWID를 가지고 엑세스를 하게 되는데 이 때 랜덤 I/O가 발생한다.

```
select /* index(emp emp_x01) */
*
from emp
where deptno =30
  and sal >= 2000 ;
```

\


예를 들어서 위와 같은 조건에서 탐색이 발생하면 deptno=30 에 해당하는 ROWID들을 가지고 랜덤하게 테이블 레코드를 탐색하게 된다.

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

**deptno 가 30을 만족하는 ROWID를 가지고 랜덤하게 조회해보는 그림**

\


그래서 성능 향상을 위해선 random access 를 줄이고 sequential read 를 높여야 한다. 아래와 같이 sal에도 인덱스를 걸어주면 결과적으로 random access를 줄이게 된다. 다시 말하면 random access 를 줄이고 sequential read 를 높힌다는 것의 의미는 ‘테이블 엑세스를 최소화’한다는 의미이다. 그게 곧 속도 향상이고 성능 향상을 가능하게 하니까.

```
create index emp_x01 on emp(deptno,job,sal);

select /* index(emp emp_x01) */
*
from emp
where deptno =30
  and sal >= 2000 ;
```

\


다음은 인덱스 운영시 사용할 수 있는 쿼리로 강의 자료에 첨부되어 있던 내용이다.

```sql
## 테이블 / 인덱스 크기 확인
SELECT
    table_name,
    table_rows,
    round(data_length/(1024*1024),2) as 'DATA_SIZE(MB)',
    round(index_length/(1024*1024),2) as 'INDEX_SIZE(MB)'
FROM information_schema.TABLES
where table_schema = 'subway';

## 중복 인덱스 확인

select table_schema, table_name, `1st_col`, `2st_col`, `3st_col`, `4st_col`, count(*) as cnt
from (
select `information_schema`.`statistics`.`TABLE_SCHEMA` AS `table_schema`
      ,`information_schema`.`statistics`.`TABLE_NAME` AS `table_name`
      ,`information_schema`.`statistics`.`INDEX_NAME` AS `index_name`
      ,max((case when (`information_schema`.`statistics`.`SEQ_IN_INDEX` = 1) then `information_schema`.`statistics`.`COLUMN_NAME` else '0' end)) AS `1st_col`
      ,max((case when (`information_schema`.`statistics`.`SEQ_IN_INDEX` = 2) then `information_schema`.`statistics`.`COLUMN_NAME` else '0' end)) AS `2st_col`
      ,max((case when (`information_schema`.`statistics`.`SEQ_IN_INDEX` = 3) then `information_schema`.`statistics`.`COLUMN_NAME` else '0' end)) AS `3st_col`
      ,max((case when (`information_schema`.`statistics`.`SEQ_IN_INDEX` = 4) then `information_schema`.`statistics`.`COLUMN_NAME` else '0' end)) AS `4st_col`
from `information_schema`.`statistics`
where (`information_schema`.`statistics`.`TABLE_SCHEMA` not in ('performance_schema','mysql','common_schema','information_schema','ps_helper','sys'))
group by `information_schema`.`statistics`.`TABLE_SCHEMA`,`information_schema`.`statistics`.`TABLE_NAME`,`information_schema`.`statistics`.`INDEX_NAME`,`information_schema`.`statistics`.`INDEX_TYPE`
) A
GROUP BY table_schema, table_name, 1st_col, 2st_col, 3st_col, 4st_col
HAVING count(*) > 1;
```

강의 자료에서 인덱스 설계시 주의 사항 두 가지를 강조하고 있었고 내용은 아래와 같다.

> 조건절에 항상 사용하거나, 자주 사용하는 컬럼을 선정합니다.\
> ‘=’ 조건으로 자주 조회하는 컬럼을 앞쪽에 둡니다.

\


둘 이상의 컬럼에 인덱스를 걸 경우 주의해야할 점이 있는데, 순서가 Index Range Scan의 발동에 영향을 준다는 것이다.

```
CREATE INDEX `idx_user_Country_OpenSource`  ON `subway`.`programmer` (Country, OpenSource)
```

\


위와 같이 인덱스를 설정할 경우 아래 쿼리중 첫번째 쿼리에서는 Full Table Scan이 발생한다. 따라서 Index Range Scan을 하기 위해서는 인덱스 선두 컬럼이 조건절에 있어야 한다. 즉 위와 같이 인덱스를 걸 경우 Country 이 조건절에 있어야 한다.

```
SELECT * FROM subway.programmer WHERE OpenSource = 'Yes'
SELECT * FROM subway.programmer WHERE OpenSource = 'Yes' AND Country LIKE 'Nigeria'
```

\


다음은 강의 자료에 첨부된 성능 개선 대상 식별을 위한 sql문이다.

```sql
## 프로세스 목록
SHOW PROCESSLIST;

## 성능 개선 대상 식별
SELECT DIGEST_TEXT AS query,
             IF(SUM_NO_GOOD_INDEX_USED > 0 OR SUM_NO_INDEX_USED > 0, '*', '') AS full_scan,
             COUNT_STAR AS exec_count,
             SUM_ERRORS AS err_count,
             SUM_WARNINGS AS warn_count,
             SEC_TO_TIME(SUM_TIMER_WAIT/1000000000000) AS exec_time_total,
             SEC_TO_TIME(MAX_TIMER_WAIT/1000000000000) AS exec_time_max,
             SEC_TO_TIME(AVG_TIMER_WAIT/1000000000000) AS exec_time_avg_ms, SUM_ROWS_SENT AS rows_sent,
             ROUND(SUM_ROWS_SENT / COUNT_STAR) AS rows_sent_avg, SUM_ROWS_EXAMINED AS rows_scanned,
             DIGEST AS digest
   FROM performance_schema.events_statements_summary_by_digest ORDER BY SUM_TIMER_WAIT DESC


## 최근 실행된 쿼리 이력 기능 활성화
UPDATE performance_schema.setup_consumers SET ENABLED = 'yes' WHERE NAME = 'events_statements_history'
UPDATE performance_schema.setup_consumers SET ENABLED = 'yes' WHERE NAME = 'events_statements_history_long'

## 최근 실행된 쿼리 이력 확인
SELECT * FROM performance_schema.events_statements_history
```

\
\


## DB 서버 튜닝 <a href="#db" id="db"></a>

강의에서는 DB 설정값들이 어떤 의미를 가지는지 알아보고 튜닝포인트를 분석해봤다.

```
mysql> SHOW VARIABLES LIKE '%max_connection%';
mysql> SHOW STATUS LIKE '%CONNECT%';
mysql> SHOW STATUS LIKE '%CLIENT%';
```

> * connect\_timeout\
>   MySQL이 클라이언트로부터 접속 요청을 받는 경우 몇 초까지 기다릴지를 설정하는 변수로, 기본 값은 5초이며 일반적으로 수정할 필요는 없습니다.
> * Interactive\_timeout\
>   ‘mysql>’과 같은 콘솔이나 터미널 상에서의 클라이언트 접속을 의미하며, 기본 값으로 8시간이 잡혀 있으나 1시간 정도로 낮추는 것이 좋습니다.
> * wait\_timeout\
>   접속한 후 쿼리가 들어올 때까지 기다리는 시간으로, 접속이 많은 DBMS에서는 이 값을 낮춰 sleep 상태의 Connection들을 정리하여 전체 성능을 향상시킬 수 있습니다. 하지만 값을 너무 낮추게 되면 지나치게 잦은 커넥션이 발생할 수 있으므로, 보통 15\~20 사이의 값을 설정합니다. Aborted client는 2% 아래인 것이 바람직한 상태입니다.
> * max\_connections\
>   서버가 허용하는 최대한의 커넥션 수입니다. 서버의 사양에 따라 달라질 수 있으며 일반적으로 120\~250개 정도로 설정합니다. 하지만 접속이 많고 고용량 서버의 경우 1000개 정도의 높은 값을 설정하는 것도 가능하니, Too many connection 에러가 발생하지 않도록 적절한 값을 설정하는 것이 중요합니다.
> * back\_log\
>   max\_connection 설정값 이상의 접속이 발생할 때 얼마만큼의 커넥션을 큐에 보관할지에 대한 설정 값으로, 기본 값은 50이며 접속이 많은 서버의 경우 이 값을 늘릴 필요가 있습니다.

\
