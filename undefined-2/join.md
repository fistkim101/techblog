# 데이터베이스 JOIN 원리

## JOIN 의 종류와 의미

### INNER, OUTER 용어의 유래

일단 대전제부터 인지하자. INNER, OUTER 는 모두 Nested Loop Join을 가정한 반복문의 바깥쪽, 안쪽의 위치에서 비롯된 용어이다. 아래에서 하나씩 알아보자.

### Inner Join

> For inner join, the outer loop would iterate over any relation and the inner loop would iterate over the other relation and create composite rows whenever join columns matched. Thus the output rows get created and populated in the inner loop. Hence this is called INNER JOIN.

위 인용구는 스택오버플로우에서 가져온 'INNER' 라는 명칭에 대한 추측(?) 인데, 벤다이어그램 상의 INNER 의 의미가 절대 아니고 뒤에 볼 Nested Loop Join 의 안쪽 Loop 에서 비롯된 INNER 라고 봐야 한다.

즉, NL Join 시에 바깥쪽 반복문에서 드라이빙 테이블을 가져와 순차적으로 진행하면서 안쪽 반복문에 드라이븐 테이블을 두고 이중 반복문을 돌리는데, 이때 안쪽 반복문에 두는 드라이븐 테이블과 공통적으로 존재하는 컬럼을 기준으로 최종 반환 결과 집합(composite rows)을 산출 하는 것이 INNER JOIN 이기 때문에 INNER 인 것이다.

<figure><img src="../.gitbook/assets/image (7) (3).png" alt=""><figcaption></figcaption></figure>

```sql
SELECT * FROM TableA
INNER JOIN TableB
ON TableA.name = TableB.name
```

### Left Outer Join

> When we want all rows in left side relation\table to be retained, the outer loop will have to iterate on the left table and rows would be added not only in the inner loop for matching cases but also in the outer loop for non-matching cases(where left table doesn't have a matching row in right table based on join columns). In this case, the left table needs to go to the outer loop, so it is called LEFT OUTER JOIN.

그대로 번역을 하자면 아래와 같다.

> 왼쪽편의 테이블이 모두 결과에 포함 되길 원할때 바깥쪽 반복문은 왼쪽편의 테이블을 대상으로 순회해야하며 결과적으로 추가되는 행들은 안쪽 반복문에서 매칭되는 행 뿐만이 아니라 바깥쪽 반복문에서 매칭되지 않는 행들까지 포함한다.(즉, 바깥쪽 반복문에서 순회하는 모든 행과 안쪽 반복문에서 매칭되는 행들이 그 대상이다.)
>
> 이 경우 왼쪽편 테이블이 바깥쪽 반복문에 있어야 하므로 LEFT OUTER JOIN 이다.

<figure><img src="../.gitbook/assets/image (9) (2) (2) (3).png" alt=""><figcaption></figcaption></figure>

```sql
SELECT * FROM TableA
LEFT OUTER JOIN TableB
ON TableA.name = TableB.name
```

### Right Outer Join

> When we want all rows in right side relation\table to be retained, right table will need to go into outer loop, so it is called RIGHT OUTER JOIN.

Left Outer Join 과 같은 맥락이다. 벤다이어 그램은 생략한다.

### Full Outer Join

> When we want non matching rows of both tables to be retained, in the simplest approach, we would have two nested loops. One nested loop would have left table in the outer loop and the other nested loop would have right table in the outer loop. So both tables go to outer loop, hence it is called FULL OUTER JOIN.

> 만약 두 테이블간에 매칭되지 않는 행들의 집합을 결과물로 원할때 가장 간단한 접근은 두 개의 네스티드 루프를 두는 것이다. 하나는 왼쪽 테이블을 바깥쪽 반복문에 두는 것이고 하나는 오른쪽 테이블을 바깥쪽 반복문에 두는 것이다. 여기서 보면 결국 두 테이블이 모두 바깥쪽 반복문에 위치한다. 그래서 FULL OUTER JOIN 이다.

<figure><img src="../.gitbook/assets/image (15) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### 전체 벤다이어그램

누군가 정리를 잘 해놓은 것을 찾아서 아래에 옮겨왔다.

<figure><img src="../.gitbook/assets/image (16) (1) (1).png" alt=""><figcaption><p>출처 : <a href="https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-JOIN-%EC%A1%B0%EC%9D%B8-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EA%B8%B0%EC%89%BD%EA%B2%8C-%EC%A0%95%EB%A6%AC">https://inpa.tistory.com/entry/MYSQL-%F0%9F%93%9A-JOIN-%EC%A1%B0%EC%9D%B8-%EA%B7%B8%EB%A6%BC%EC%9C%BC%EB%A1%9C-%EC%95%8C%EA%B8%B0%EC%89%BD%EA%B2%8C-%EC%A0%95%EB%A6%AC</a></p></figcaption></figure>

## JOIN 알고리즘

### Nested Loop Join <a href="#nested-loop-join" id="nested-loop-join"></a>

\


<figure><img src="../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (26) (1) (1).png" alt=""><figcaption></figcaption></figure>

중첩 반복문(시간 복잡도 O(N\*M))(외부 테이블 행의 수 N, 내부 테이블 행의 수 M) 방식으로 동작하는 조인 방식이다. 그래서 대용량 데이터에는 적합하지 않다. 반복문의 외부에 있는 테이블을 선행 테이블 또는 외부 테이블(Outer Table)이라고 하며 반복문 내부에 있는 테이블을 후행 테이블 또는 내부 테이블(Inner Table)이라고 한다.

1. 외부 테이블(선행 테이블, 드라이빙 테이블)에서 조건을 만족하는 첫 번째 행을 찾는다. 이때 첫번째 그림처럼 인덱스가 존재한다면 이를 활용할 수 있다.
2. 내부 테이블(후행 테이블, 드리븐 테이블)을 처음부터 끝까지 순차적으로 스캔하면서 외부 테이블의 현재 행과 조인 조건을 비교한다.(이때 내부 테이블을 순차적으로 조회할때 인덱스가 존재하지 않으면 랜덤 액세스로 동작한다. 내부 테이블 검색 동작시 인덱스 존재하면 순차 액세스, 인덱스 존재하지 않으면 랜덤 액세스)
3. 조인 조건에 맞는 행을 찾으면 결과 집합에 추가한다.
4. 내부 테이블을 모두 스캔한 후, 외부 테이블의 다음 행을 가져와서 위의 과정을 반복한다.
5. 모든 외부 테이블의 행을 처리할 때까지 반복한다.

### Sort Merge Join <a href="#sort-merge-join" id="sort-merge-join"></a>

<figure><img src="../.gitbook/assets/image (8) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (13) (2).png" alt=""><figcaption></figcaption></figure>

정렬할 데이터가 많은 경우 임시 영역(디스크)을 사용하기 때문에 정렬 비용으로 인해 성능이 떨어질 수도 있다.&#x20;

1. 조인 대상 테이블의 데이터를 정렬한다. 이를 위해 각 테이블에서 조인에 사용할 열(또는 열의 조합)에 대한 인덱스를 활용할 수 있다.
2. 정렬된 데이터셋을 기반으로 두 데이터셋을 병합한다. 병합은 두 데이터셋을 순차적으로 스캔하면서 조인 조건을 비교한다.(네스티드 루프 조인이 기본적으로 내부 테이블에 대해서 랜덤 액세스를 하는 것과 대비되는 부분이다. 정렬이 되어있기 때문에 순차 스캔이 되는 것이다. 두번째 그림을 참고하자.)
3. 조인 조건을 만족하는 행을 찾으면 결과에 추가한다.
4. 병합 작업이 완료될 때까지 위의 단계를 반복한다.

### Hash Join <a href="#hash-join" id="hash-join"></a>

<figure><img src="../.gitbook/assets/image (10) (2) (2).png" alt=""><figcaption></figcaption></figure>

Hash Join 은 네스티드 루프 조인의 랜덤 액세스 문제와 Sort 머지 조인의 정렬 비용 문제를 해결한 방식이다. 네스티드 루프 조인에서 인덱스가 따로 존재하지 않아서 랜덤 액세스가 발생해야 하는 경우나 소트 머지 조인이 부적합한 경우 사용할 수 있다.

하지만 해시 조인은 해시 테이블을 메모리에 저장하고 조회하기 때문에 메모리 사용량에 영향을 받는다. 작은 테이블이 메모리에 저장될 수 있을 만큼 충분한 메모리가 필요하다. 만약 메모리에 모든 데이터를 저장할 수 없는 경우에는 일부 데이터를 디스크에 저장하게 되어 성능 저하가 발생할 수 있다.

1. 조인 대상 테이블 중 작은 테이블(일반적으로 내부 테이블)을 선택한다. 작은 테이블을 선택하는 이유는 해시 테이블의 크기를 줄이고 메모리 사용을 최적화하기 위함이다. 그렇게 선택된 작은 테이블에 대해 해시 함수를 적용하여 해시 테이블을 생성한다. 해시 테이블은 해시 값과 해당 값에 대한 포인터(또는 연결 리스트)로 구성된다.
2. 큰 테이블(일반적으로 외부 테이블)을 스캔하면서 해시 함수를 적용하여 해시 테이블에서 일치하는 해시 값을 찾는다.
3. 해시 값이 일치하는 경우에는 실제 값들을 비교하여 조인 조건을 확인하고, 조인 조건을 만족하는 행을 결과에 추가한다. 모든 행을 처리할 때까지 위의 단계를 반복한다.
