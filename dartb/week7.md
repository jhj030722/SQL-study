## 목표

**: CASE문, ISNULL() 구문**

## 주제

같은 결과를 도출하더라도 여러 표현법을 사용하고, 효율성을 따져보는 것이 중요합니다.

**-[ISNULL] NULL처리하기 (SQL 고득점kit)**

[](https://school.programmers.co.kr/learn/courses/30/lessons/59410)

**동물의 생물 종, 이름, 성별 및 중성화 여부를 아이디 순으로 조회하는 SQL문을 작성합니다.**

이때, 이름이 없는 동물의 이름은 ‘No name’으로 합니다.

**문제1. IFNULL()으로 해결**

```sql
SELECT ANIMAL_TYPE, IFNULL(NAME, 'No name') as NAME, SEX_UPON_INTAKE
FROM ANIMAL_INS
```

같은 문제를, CASE WHEN 문법을 사용하여 해결해주세요

**문제2. CASE WHEN으로 해결**

```sql
SELECT ANIMAL_TYPE,
CASE WHEN NAME IS NULL THEN 'No name' ELSE NAME END AS NAME, SEX_UPON_INTAKE
FROM ANIMAL_INS
```
- select문에 해당 함수를 쓰는 것이 포인트



**-중성화 여부 파악하기**

[](https://school.programmers.co.kr/learn/courses/30/lessons/59409#qna)

**문제 3. 문제를 풀어주세요 (힌트: IF, LIKE를 사용할 수 있습니다)**

```sql
SELECT ANIMAL_ID, NAME, 
IF(SEX_UPON_INTAKE LIKE '%NEUTERED%' OR SEX_UPON_INTAKE LIKE '%SPAYED%', "O", "X")
FROM ANIMAL_INS
ORDER BY ANIMAL_ID;
```

**문제 4. 아래는 QnA에 올라온 질문입니다. 왜 풀이가 틀렸는지 답해주세요.**

[](https://school.programmers.co.kr/questions/80270)

```sql
각 조건에 대해 개별적으로 LIKE문을 써야하는데 하나로 써버렸음
```