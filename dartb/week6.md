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
```

- `ROW_NUMBER()`를 사용하면 동일한 논리를 간단히 확장할 수 있습니다. 예를 들어, 두 번째로 높은 FAVORITES 값을 선택하거나, 순위를 기반으로 추가 분석을 할 때 더 쉽게 확장할 수 있습니다.
- 서브쿼리 없이도 여러 기준을 적용해 정렬하고 순위를 부여할 수 있으므로 향후 요구사항 변경에 대비하기가 더 쉽습니다.

# 문제 2; 

```sql
-- 2022년도 한해 평가 점수가 가장 높은 사원 정보를 조회
SELECT G.SCORE, G.EMP_NO, E.EMP_NAME, E.POSITION, E.EMAIL
FROM HR_DEPARTMENT AS D
-- 부서 테이블과 사원 테이블 조인
JOIN HR_EMPLOYEES AS E
-- 각 직원이 어느 부서에 속해있는지
ON D.DEPT_ID = E.DEPT_ID
-- GRADE 테이블로부터 점수 계산하는 서브쿼리(G)와 조인
JOIN ( -- RANK 윈도우 함수 사용: SCORE을 Order 하여 내림차순 순위 매김
    SELECT EMP_NO, SCORE, RANK() OVER (ORDER BY SCORE DESC) AS RANKING
    FROM ( -- 점수를 계산하기 위한 두번째 서브쿼리 : temp 
        SELECT EMP_NO, SUM(SCORE) AS SCORE
        FROM HR_GRADE
        WHERE YEAR = 2022
        GROUP BY EMP_NO
    ) AS TEMP
) AS G
ON E.EMP_NO = G.EMP_NO
WHERE G.RANKING = 1 #점수 가장 높은 직원
ORDER BY G.SCORE DESC
```