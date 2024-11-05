## 문제 1; 즐겨찾기가 가장 많은 식당 정보 출력하기


```sql
SELECT FOOD_TYPE, REST_ID, REST_NAME, FAVORITES
FROM REST_INFO
WHERE (FOOD_TYPE, FAVORITES) IN (SELECT FOOD_TYPE, MAX(FAVORITES) 
                              FROM REST_INFO
                              GROUP BY FOOD_TYPE)
ORDER BY FAVORITES DESC;
```

<개선된 쿼리>
- row_number 윈도우 함수 사용
```sql
--WITH 절로 CTE 정의 
--RankedRest: CTE의 이름 => 이후에 이 이름을 사용해 데이터에 접근

WITH RankedRest AS(
    SELECT FOOD_TYPE, REST_ID, REST_NAME, FAVORITES, ROW_NUMBER() -- 각 행에 고유번호를 부여 
    OVER 
    (PARTITION BY FOOD_TYPE
    -- 푸드 타입 기준으로 데이터 그룹화(Partition)
    ORDER BY FAVORITES DESC, REST ID
    -- food type 그룹 내에서 즐겨찾기수 내림차순 정렬 후, 즐겨찾기수 값이 동일할 경우 rest_id 기준으로 오름차순 정렬 
    ) 
    AS 
    FROM REST_INFO
)
SELECT FOOD_TYPE, REST_ID, REST_NAME, FAVORITES
FROM RankedRest
-- CTE에서 가져오기
WHERRE rnk = 1
-- RANK 값이 1인 = FAVORITES이 가장 높은  행만 선택. 
ORDER BY FOOD_TYPE DESC;