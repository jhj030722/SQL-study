## [JOIN] ROOT 아이템 구하기

```sql
SELECT A.ITEM_ID, A.ITEM_NAME
FROM ITEM_INFO A JOIN ITEM_TREE B 
ON A.ITEM_ID = B.ITEM_ID
WHERE B.PARENT_ITEM_ID IS NULL 
ORDER BY A.ITEM_ID ASC
```
- root 아이템이란 parent_item이 없는 아이템이기 때문에 (본인이 가장 parent) 
- item_tree 테이블에서 parent_item 이 null인 것을 조회해서
- 두 테이블을 조인시켜서 풀면됨.

## [CONCAT 사용] 노선별 평균 역 사이 거리 조회하기

<문제>
SUBWAY_DISTANCE 테이블에서 노선별로 노선, 총 누계 거리, 평균 역 사이 거리를 노선별로 조회하는 SQL문을 작성해주세요.

총 누계거리는 테이블 내 존재하는 역들의 역 사이 거리의 총 합을 뜻합니다. 총 누계 거리와 평균 역 사이 거리의 컬럼명은 각각 TOTAL_DISTANCE, AVERAGE_DISTANCE로 해주시고, 총 누계거리는 소수 둘째자리에서, 평균 역 사이 거리는 소수 셋째 자리에서 반올림 한 뒤 단위(km)를 함께 출력해주세요.
결과는 총 누계 거리를 기준으로 내림차순 정렬해주세요.

```SQL
SELECT ROUTE, CONCAT(ROUND(SUM(D_BETWEEN_DIST),1),'km') as TOTAL_DISTANCE, 
CONCAT(ROUND(AVG(D_BETWEEN_DIST),2),'km') as AVERAGE_DISTANCE
FROM SUBWAY_DISTANCE 
GROUP BY ROUTE
ORDER BY SUM(D_BETWEEN_DIST) DESC
```

> CONCAT 함수: CONCAT 함수는 둘 이상의 문자열을 입력한 순서대로 합쳐서 반환해주는 함수.  구분자를 사용하지 않고 문자열을 단순히 연결한다. 

(예제)

```SQL 
CONCAT(string1, string2, ...)

SELECT CONCAT('Hello', ' ', 'World') AS result;  

//출력 : Hello World
```
- D_BETWEE_DIST의 합계에 'km'를 붙여주기 위해 concat 함수를 써서 문자열로 만든다. 
- average_distance를 만들때도 동일하게.
- route 별로 그룹화
- total_distance와 average_distance는 **문자열화된** 것이기 때문에 order by되지 않으므로 다시 sum(D_BETWEEN_DIST)로 정렬해주기. 



## [WINDOW] 헤비 유저가 소유한 장소

<문제>
이 서비스에서는 공간을 둘 이상 등록한 사람을 "헤비 유저"라고 부릅니다. 헤비 유저가 등록한 공간의 정보를 아이디 순으로 조회하는 SQL문을 작성해주세요.

[문제링크](https://school.programmers.co.kr/learn/courses/30/lessons/77487)

```SQL
SELECT *
FROM PLACES 
WHERE HOST_ID IN (SELECT HOST_ID FROM PLACES GROUP BY HOST_ID HAVING COUNT(HOST_ID) >=2)
```
- `서브쿼리`를 사용하여 HOST_ID가 2번 이상 집계된 HOST_ID별로 그룹화한다.

- 그 서브쿼리가 포함된 (`IN`함수 사용) HOST_ID를 조회하여출력