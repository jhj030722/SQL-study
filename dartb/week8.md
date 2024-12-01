## 1. PIVOT과 UNPIVOT

### 1. **pivot과 unpivot에 관해 자유로운 리소스를 이용해 학습하고 요약해주세요**.

    (1) PIVOT : Pivot은 데이터 행(row)을 열(column)로 변환하여 데이터를 재구조화하는 작업

    (2) UNPIVOT : Pivot의 반대 작업으로, 열(column)을 행(row)으로 변환하여 데이터를 재구조화하는 작업


|제품명	|월	|매출액|
|---------|---------|---------|
|ProductA|	1월	|100|
|ProductA|  2월	|120|
|ProductB|	1월	|200|
|ProductB|  2월 |180|


을 PIVOT 하면 

``` SQL
SELECT 
    제품명,
    [1월] AS January,
    [2월] AS February
FROM 
    (SELECT 제품명, 월, 매출액 FROM sales) AS SourceTable
PIVOT 
    (SUM(매출액) FOR 월 IN ([1월], [2월])) AS PivotTable;

```

| 제품명   | 1월   | 2월   |
|---------|---------|---------|
| ProductA | 100 | 120 |
| ProductB | 200 | 180 |

결과가 이렇게 됨. 




### 2. **다음 문제를 풀어주세요**
LINK: https://www.hackerrank.com/challenges/occupations/problem



```SQL
WITH RankedOccupations AS (
    SELECT 
        Name,
        Occupation,
        ROW_NUMBER() OVER (PARTITION BY Occupation ORDER BY Name) AS RowNumber
    FROM Occupations
)
SELECT 
    MAX(CASE WHEN Occupation = 'Doctor' THEN Name END) AS Doctor,
    MAX(CASE WHEN Occupation = 'Professor' THEN Name END) AS Professor,
    MAX(CASE WHEN Occupation = 'Singer' THEN Name END) AS Singer,
    MAX(CASE WHEN Occupation = 'Actor' THEN Name END) AS Actor
FROM RankedOccupations
GROUP BY RowNumber
ORDER BY RowNumber;
```

1. ROW_NUMBER()를 사용하여 각 직업별로 이름을 알파벳순 정렬 -> 행번호 부여
(Doctor는 의사그룹 내에서 1,2,3 번호부여)

2. CASE WHEN을 사용해 PIVOT 구조 구현 - 직업을 COLUMN으로 변환하고 MAX를 이용해 동일한 Rownumber에 해당한느 이름 선택

3. Rownumber를 기준으로 데이터 그룹화하여 같은 순번 데이터를 행으로 묶기

4. rownumber로 최종 ORDER BY 하기 


## 2. 성능 최적화 기법

**1) 아래 칼럼을 읽고 요약해주세요.** 

칼럼: https://community.heartcount.io/ko/query-optimization-tips/


1. **인덱스 컬럼에 함수 사용 지양**: 원본 데이터를 직접 비교하는 조건을 사용하는 것이 인덱스를 최대한 활용하고 쿼리 성능을 높이는 핵심 비결. 예를 들어, 날짜 필터링에서는 함수 대신 범위 조건(BETWEEN 등)을 사용하는 것이 효과적.

2. **OR 대신 UNION 사용**: OR 조건은 전체 테이블 스캔을 유발할 수 있음. 이를 대체하기 위해 UNION을 활용하면 각 조건이 독립적으로 인덱스를 사용할 수 있어 성능이 향상됨.

3. **필요한 열과 행만 선택**: 쿼리에서 사용하지 않는 데이터를 조회하는 것은 리소스 낭비. 필요한 열과 조건에 해당하는 행만 조회하도록 쿼리를 설계

4. **불필요한 정렬 회피**: ORDER BY를 사용한 정렬은 비용이 크므로 꼭 필요한 경우에만 사용

5. **효율적인 JOIN 활용**: JOIN 조건에서는 인덱스가 있는 컬럼을 사용하고, 상황에 맞는 JOIN 유형(INNER JOIN, OUTER JOIN 등)을 선택하는 것이 중요

6. **효율적인 서브쿼리 작성**: 서브쿼리는 필요에 따라 최적화되어야 하며, 인덱스가 설정된 필드와 결합해 불필요한 계산을 줄여야 함


**2) 문제를 풀어주세요.**

## 문제

여러분은 `customer_orders`라는 테이블을 관리하는 데이터베이스 관리자로 일하고 있습니다. 이 테이블에는 고객의 주문 정보가 저장되어 있으며, 각 고객이 주문한 제품과 수량, 가격 정보가 포함되어 있습니다. 또한, 고객들이 특정 제품을 재구매한 비율을 계산하려고 합니다.

### 테이블 구조:

1. **customers** 테이블
    - `customer_id` (고객 ID, PRIMARY KEY)
    - `name` (고객 이름)
2. **orders** 테이블
    - `order_id` (주문 ID, PRIMARY KEY)
    - `customer_id` (고객 ID, FOREIGN KEY)
    - `order_date` (주문 날짜)
3. **order_details** 테이블
    - `order_id` (주문 ID, FOREIGN KEY)
    - `product_id` (제품 ID)
    - `quantity` (수량)
    - `unit_price` (단가)

---

### 요구 사항:

1. **`avg_order_value`**: 고객별 평균 주문 금액을 계산하여 `customers` 테이블에 업데이트하세요.
    - `avg_order_value`는 각 고객이 한 번의 주문에서 지출한 평균 금액입니다.
2. **`total_spent`**: 고객별 총 지출 금액을 계산하여 `customers` 테이블에 업데이트하세요.
    - `total_spent`는 고객이 지금까지 지출한 총 금액입니다.
3. **`num_orders`**: 고객이 총 몇 번의 주문을 했는지 계산하여 `customers` 테이블에 업데이트하세요.
    - `num_orders`는 고객이 주문한 총 개수입니다.
4. **`repurchase_rate`**: 고객의 재구매 비율을 계산하여 `customers` 테이블에 업데이트하세요.
    - `repurchase_rate`는 각 고객이 2번 이상 주문한 제품 비율을 의미합니다. (즉, 재구매한 제품이 전체 구매 제품 중 몇 퍼센트를 차지하는지)

### 예시:

- 고객 A는 3번 주문을 했고, 그 중 2개의 제품을 재구매했습니다.
    - 평균 주문 금액: 100,000원
    - 총 지출 금액: 300,000원
    - 주문 횟수: 3번
    - 재구매 비율: 66.67%

### 답.
```sql
WITH order_summary AS (
    SELECT
        o.customer_id,
        AVG(od.quantity * od.unit_price) AS avg_order_value,
        SUM(od.quantity * od.unit_price) AS total_spent,
        COUNT(DISTINCT o.order_id) AS num_orders,
        COUNT(DISTINCT CASE 
            WHEN od.product_id IN (
                SELECT product_id
                FROM order_details od2
                JOIN orders o2 ON od2.order_id = o2.order_id
                WHERE o2.customer_id = o.customer_id
                GROUP BY od2.product_id
                HAVING COUNT(od2.product_id) > 1
            ) THEN od.product_id END) AS repurchased_products
    FROM orders o
    JOIN order_details od ON o.order_id = od.order_id
    GROUP BY o.customer_id
)
UPDATE customers c
SET
    avg_order_value = os.avg_order_value,
    total_spent = os.total_spent,
    num_orders = os.num_orders,
    repurchase_rate = CASE
        WHEN os.num_orders > 0 THEN os.repurchased_products * 1.0 / COUNT(DISTINCT od.product_id)
        ELSE 0
    END
FROM order_summary os
JOIN orders o ON os.customer_id = o.customer_id
JOIN order_details od ON o.order_id = od.order_id
WHERE c.customer_id = os.customer_id;

```
### 추가 질문:

1. 정답 쿼리에서 `repurchase_rate`를 구할 때 사용한 `HAVING COUNT(order_details.product_id) > 1`의 의미는 무엇인가요?
```
테이블에서 고객이 2회 이상 구매한 제품을 필터링하는 조건
```
2. 이 문제에서 사용될 수 있는 성능을 최적화하기 위한 방법은 무엇일까요?
```
JOIN이나 **CTE를 사용하여 중복된 계산을 한 번만 수행하고, 그 결과를 재사용하는 것
```
