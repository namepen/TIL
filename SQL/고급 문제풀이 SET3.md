# 고급 SET 3 문제

1. Ollivander's Inventory

```sql
# Solution 1. 
SELECT w.id
     , wp.age
     , w.coins_needed
     , w.power
FROM wands AS w
INNER JOIN  (
    SELECT code
         , power
         , MIN(coins_needed) AS coins_needed
    FROM wands
    GROUP BY code, power
    ) as mw
ON w.code = mw.code
AND w.power = w.power 
AND w.coins_needed = mw.coins_needed
INNER JOIN wands_property AS wp
ON w.code = wp.code
AND wp.is_evil = 0
ORDER BY w.power DESC, wp.age DESC;

# Soultion 2.: window function with MIN
SELECT id, age, coins_needed, power
FROM (
    SELECT w.id
         , wp.age
         , w.coins_needed
         , w.power
         , MIN(coins_needed) OVER (PARTITION BY power, age) min_coins
        FROM wands w
            INNER JOIN wands_property wp ON w.code = wp.code
        WHERE wp.is_evil = 0
        ) w_list
WHERE coins_needed = min_coins
ORDER BY power DESC, age DESC;

# Solution 3. : window function with ROW_NUM
SELECT id, age, coins_needed, power
FROM (
    SELECT w.id
         , wp.age
         , w.coins_needed
         , w.power
         , ROW_NUMBER() OVER (PARTITION BY power, age ORDER BY coins_needed) rn
        FROM wands w
            INNER JOIN wands_property wp ON w.code = wp.code
        WHERE wp.is_evil = 0
        ) w_list
WHERE rn = 1
ORDER BY power DESC, age DESC;
```

2. [Weather Observation Station 11](https://www.hackerrank.com/challenges/weather-observation-station-11/problem)

```sql
# Solution 1.
SELECT DISTINCT city
FROM station
WHERE city NOT REGEXP '^[aeiou]'
OR city NOT REGEXP '[aeiou]$';
```