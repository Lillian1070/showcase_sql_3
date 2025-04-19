# [SQL] Most Active User & Top Rated Movie in a Specific Period

_This SQL practice problem is based on **[LeetCode SQL 50 - 1341. Movie Rating](https://leetcode.com/problems/movie-rating/description/?envType=study-plan-v2&envId=top-sql-50)**, used here for personal learning and educational purposes._ 


- **Objective:** Calculate the most active user and top-rated movie during February 2020.  
- **Practice Purpose:** Self-learning and reinforcement of SQL joins, aggregation, and window functions.
- **Outline:**
  - [**Practice**](#section-1) (practice problem and query output)
  - [**Solution**](#section-2) (step-by-step explanation)
  - [**Query Optimization**](#section-3) (refinement for efficiency and readability)


## <a name="section-1"></a>🧪 Practice

Table: `Movies`

| Column Name   | Type    |
|---------------|---------|
| movie_id      | int     |
| title         | varchar |


- `movie_id` is the primary key (column with unique values) for this table.
- `title` is the name of the movie.
 

Table: `Users`

| Column Name   | Type    |
|---------------|---------|
| user_id       | int     |
| name          | varchar |


- `user_id` is the primary key (column with unique values) for this table.
- The column `name` has unique values.


Table: `MovieRating`

| Column Name   | Type    |
|---------------|---------|
| movie_id      | int     |
| user_id       | int     |
| rating        | int     |
| created_at    | date    |


- (`movie_id`, `user_id`) is the primary key (column with unique values) for this table.
- This table contains the rating of a movie by a user in their review.
- `created_at` is the user's review date. 
 

Write a solution to:

Find the name of the user who has rated the greatest number of movies. In case of a tie, return the lexicographically smaller user name.

Find the movie name with the highest average rating in February 2020. In case of a tie, return the lexicographically smaller movie name.

The result format is in the following example.

 

### Example 

#### Input: 

`Movies` table:

| movie_id    |  title       |
|-------------|--------------|
| 1           | Avengers     |
| 2           | Frozen 2     |
| 3           | Joker        |

`Users` table:

| user_id     |  name        |
|-------------|--------------|
| 1           | Daniel       |
| 2           | Monica       |
| 3           | Maria        |
| 4           | James        |

`MovieRating` table:

| movie_id    | user_id      | rating       | created_at  |
|-------------|--------------|--------------|-------------|
| 1           | 1            | 3            | 2020-01-12  |
| 1           | 2            | 4            | 2020-02-11  |
| 1           | 3            | 2            | 2020-02-12  |
| 1           | 4            | 1            | 2020-01-01  |
| 2           | 1            | 5            | 2020-02-17  | 
| 2           | 2            | 2            | 2020-02-01  | 
| 2           | 3            | 2            | 2020-03-01  |
| 3           | 1            | 3            | 2020-02-22  | 
| 3           | 2            | 4            | 2020-02-25  | 



#### Output: 

| results      |
|--------------|
| Daniel       |
| Frozen 2     |



#####  Explanation: 

- Daniel and Monica have rated 3 movies ("Avengers", "Frozen 2" and "Joker") but Daniel is smaller lexicographically.
- Frozen 2 and Joker have a rating average of 3.5 in February but Frozen 2 is smaller lexicographically.






## <a name="section-2"></a>🧠 Solution

*This section outlines my thought process for solving the problem.*

### Step 1: Identify the Fields Required

To create the result table with the most active user and top-rated movie during February 2020, I need to find out the following info: 

* User ID with the count of rated movies (see temp table `T1`)
* Movie ID with the average rating during Feb 2020 (see temp table `T3`)


### Step 2a: Create a Temporary Table `T1` to Count Rated Movies for Each User



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
```

The temporary table `T1` should be similar to what we have below. 

| user_id | name   | movie_count |
| ------- | ------ | ----------- |
| 1       | Daniel | 3           |
| 2       | Monica | 3           |
| 3       | Maria  | 2           |
| 4       | James  | 1           |


### Step 2b: Create a Temporary Table `T2` to Filter the User with Top Amount of Rated Movies

Please note that I used `ORDER BY` and `LIMIT 1` to ensure that, in case of a tie in the count of rated movies, only the user name with the lexicographically smallest first letter is selected.

```sql
    ), 
    T2 AS (
        SELECT * 
        FROM T 
        WHERE movie_count = (SELECT MAX(movie_count) FROM T) 
        ORDER BY name LIMIT 1
    ), 
```

The temporary table `T2` should be similar to what we have below. 

| user_id | name   | movie_count |
| ------- | ------ | ----------- |
| 1       | Daniel | 3           |


### Step 2c: Create a Temporary Table `T3` to Calculate the Average Rating of Each Movie

Please note that I used `ORDER BY` and `LIMIT 1` to ensure that, while the average rating is tied, only the movie name with the lexicographically smallest first letter is selected.

```sql
    T3 AS (
        SELECT 
            mr.movie_id,
            m.title,
            AVG(mr.rating) AS avg_rate
        FROM MovieRating mr 
        LEFT JOIN Movies m ON mr.movie_id=m.movie_id
        WHERE mr.created_at BETWEEN '2020-02-01' AND '2020-02-29'
        GROUP BY mr.movie_id
        ORDER BY avg_rate DESC, title ASC LIMIT 1
    )
```

The temporary table `T3` should be similar to what we have below. 

| movie_id | title    | avg_rate |
| -------- | -------- | -------- |
| 2        | Frozen 2 | 3.5      |



### Step 3: Merge Two Results into One Table


Finally, I can use [`UNION ALL`](https://www.geeksforgeeks.org/sql-union-all/) to merge the results into one table within the same column `results`.

In this case, no matter whether I use `UNION ALL` or `UNION`, the result will be the same since `T2` and `T3` both return only one result.


```sql
SELECT name AS results
FROM T2
UNION ALL 
SELECT title AS results 
FROM T3;
```



#### Final Syntax and Output using MySQL

##### * Syntax

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
        WHERE mr.created_at BETWEEN '2020-02-01' AND '2020-02-29'
        GROUP BY mr.movie_id
        ORDER BY avg_rate DESC, title ASC LIMIT 1
    )

SELECT name AS results
FROM T2
UNION ALL 
SELECT title AS results 
FROM T3;
```

##### * Output

| results  |
| -------- |
| Daniel   |
| Frozen 2 |


## <a name="section-3"></a>🛠️ Query Optimization using MySQL

*Note: This section is updated on 02/18/2025.*

Upon review, I realized that the temp tables can be replaced with inline subqueries, removing the need for a `WITH` clause. This makes the query more concise and avoids the creation of temporary tables.


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

