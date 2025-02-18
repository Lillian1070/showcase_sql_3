# [SQL] Top Movie Rating

https://leetcode.com/problems/movie-rating/description/?envType=study-plan-v2&envId=top-sql-50

```sql
WITH 
    T AS (
        SELECT 
            mr.user_id, 
            u.name, 
            COUNT(mr.movie_id) AS movie_count
        FROM MovieRating mr
        LEFT JOIN Users u ON mr.user_id=u.user_id  
        GROUP BY user_id
    ), 
    T2 AS (
        SELECT * 
        FROM T 
        WHERE movie_count = (SELECT MAX(movie_count) FROM T) 
        ORDER BY name LIMIT 1
    ), 
    T3 AS (
        SELECT 
            mr.movie_id,
            m.title,
            AVG(mr.rating) AS avg_rate
        FROM MovieRating mr 
        LEFT JOIN Movies m ON mr.movie_id=m.movie_id
        WHERE mr.created_at >= '2020-02-01' AND mr.created_at <= '2020-02-28'
        GROUP BY mr.movie_id
        ORDER BY avg_rate DESC, title ASC LIMIT 1
    )

SELECT name AS results
FROM T2
UNION ALL 
SELECT title AS results 
FROM T3;
```

```sql
(SELECT u.name AS results
 FROM MovieRating mr
 JOIN Users u ON u.user_id = mr.user_id
 GROUP BY u.name
 ORDER BY COUNT(*) DESC, u.name ASC
 LIMIT 1)

UNION ALL

(SELECT m.title AS results
 FROM MovieRating mr
 JOIN Movies m ON m.movie_id = mr.movie_id
 WHERE mr.created_at BETWEEN '2020-02-01' AND '2020-02-29'
 GROUP BY m.title
 ORDER BY AVG(mr.rating) DESC, m.title ASC
 LIMIT 1);
```
