# IPL-Sports-Analytics-using-SQL
create Table matches (
id int primary key ,
	city varchar(40),
	date DATE ,
	player_of_match varchar(40),
	venue varchar ,
	neutral_venue int,
	
team1 varchar(80),
	team2 varchar(80),
	toss_winner varchar(80),
	toss_decision varchar(20),
	winner varchar(80),
	result varchar(40),
	result_margin int,
	eliminator varchar(10) ,
	method VARCHAR(10),
	umpire1 VARCHAR(40),
	umpire2 varchar(40)
	
	
	
);

explain select*from matches;

CREATE TABLE deliveries1 (
    id INT,
    inning INT,
    over INT,
    ball INT,
    batsman VARCHAR(90),
    non_striker VARCHAR(90),
    bowler VARCHAR(90),
    batsman_runs INT,
    extra_runs INT,
    total_runs INT,
    is_wicke INT,
    dismissal_kind VARCHAR(90),
    player_dismissed VARCHAR(90),
    fielder VARCHAR(90),
    extras_type VARCHAR(90),
    batting_team VARCHAR(90),
    bowling_team VARCHAR(90),
     FOREIGN KEY(id) REFERENCES matches(id)
);  


-- 4Import data from CSV file ’IPL_Ball.csv’ attached in resources to ‘deliveries’-- 


COPY matches FROM 'C:\Pg Admin\Data_For_Final_ProjectIPLMatches_IPLBall\IPLMatches+IPLBall\IPL_matches.csv' DELIMITER ',' CSV HEADER;


COPY deliveries1 FROM 'C:\Pg Admin\Data_For_Final_ProjectIPLMatches_IPLBall\IPLMatches+IPLBall\IPL_Ball.csv' DELIMITER ',' CSV HEADER;


--5 Select the top 20 rows of the deliveries table after ordering them by id, inning, over, ball in ascending order.


SELECT * FROM deliveries1
ORDER BY id, inning, over, ball
LIMIT 20;

--6 Select the top 20 rows of the matches table--

select * from matches
limit 20 ;



--7 Fetch data of all the matches played on 2nd May 2013 from the matches table--


select *
from matches
where date = '2013-05-02';



-- 8Fetch data of all the matches where the result mode is ‘runs’ and margin of victory is more than 100 runs--


SELECT count(*) 
FROM matches
WHERE result = 'runs' AND result_margin > 100;


--9Fetch data of all the matches where the final scores of both teams tied and order it in descending order of the date.--

SELECT *
FROM matches
WHERE result = 'tie'
ORDER BY date DESC;


--10Get the count of cities that have hosted an IPL match.--

SELECT COUNT(DISTINCT city)
FROM matches;

--11Create table deliveries_v02 with all the columns of the table ‘deliveries’ and an additional column ball_result containing values boundary, dot or other depending on the total_run (boundary for >= 4, dot for 0 and other for any other number)
--(Hint 1 : CASE WHEN statement is used to get condition based results)
--(Hint 2: To convert the output data of select statement into a table, you can use a subquery. Create table table_name as [entire select statement].

CREATE TABLE deliveries_v02 AS
SELECT *,
       CASE
           WHEN total_runs >= 4 THEN 'boundary'
           WHEN total_runs = 0 THEN 'dot'
           ELSE 'other'
       END AS ball_result
FROM deliveries1;

--12Write a query to fetch the total number of boundaries and dot balls from the deliveries_v02 table.--


SELECT
    ball_result,
    COUNT(*) AS count
FROM deliveries_v02
WHERE ball_result IN ('boundary', 'dot')
GROUP BY ball_result;

--13Write a query to fetch the total number of boundaries scored by each team from the deliveries_v02 table and order it in descending order of the number of boundaries scored.--

SELECT
    batting_team,
    SUM(CASE WHEN ball_result = 'boundary' THEN 1 ELSE 0 END) AS total_boundaries
FROM deliveries_v02
GROUP BY batting_team
ORDER BY total_boundaries DESC;


-- 14Write a query to fetch the total number of dot balls bowled by each team and order it in descending order of the total number of dot balls bowled--

SELECT
    bowling_team,
    SUM(CASE WHEN ball_result = 'dot' THEN 1 ELSE 0 END) AS total_dot_balls
FROM deliveries_v02
GROUP BY bowling_team
ORDER BY total_dot_balls DESC;

--15Write a query to fetch the total number of dismissals by dismissal kinds where dismissal kind is not NA--

SELECT
    dismissal_kind,
    COUNT(*) AS total_dismissals
FROM deliveries_v02
WHERE dismissal_kind != 'NA'
GROUP BY dismissal_kind;


--16Write a query to get the top 5 bowlers who conceded maximum extra runs from the deliveries table--

SELECT
    bowler,
    SUM(extra_runs) AS total_extra_runs
FROM deliveries1
GROUP BY bowler
ORDER BY total_extra_runs DESC
LIMIT 5;

--17Write a query to create a table named deliveries_v03 with all the columns of deliveries_v02 table and two additional column (named venue and match_date) of venue and date from table matches--




CREATE TABLE deliveries_v03 AS
SELECT a.*, b.venue, b.date AS match_date
FROM deliveries_v02 as a
left JOIN matches as b ON a.id = b.id ;


--18 Write a query to fetch the total runs scored for each venue and order it in the descending order of total runs scored.--


SELECT
    venue,
    SUM(total_runs) AS total_runs_scored
FROM deliveries_v03
GROUP BY venue
ORDER BY total_runs_scored DESC;

--19 Write a query to fetch the year-wise total runs scored at Eden Gardens and order it in the descending order of total runs scored.--


SELECT
    EXTRACT(YEAR FROM match_date) AS year,
    SUM(total_runs) AS total_runs_scored
FROM deliveries_v03
WHERE venue = 'Eden Gardens'
GROUP BY year
ORDER BY total_runs_scored DESC;

/*20 Get unique team1 names from the matches table, you will notice that there are two entries for
Rising Pune Supergiant one with Rising Pune Supergiant and another one with Rising Pune Supergiants
Your task is to create a matches_corrected table with two additional columns team1_corr and team2_corr containing team 
names with replacing Rising Pune Supergiants with Rising Pune Supergiant. Now analyse these newly created columns*/



CREATE TABLE matches_corrected AS
SELECT *,
       CASE WHEN team1 = 'Rising Pune Supergiants' THEN 'Rising Pune Supergiant' ELSE team1 END AS team1_corr,
       CASE WHEN team2 = 'Rising Pune Supergiants' THEN 'Rising Pune Supergiant' ELSE team2 END AS team2_corr
FROM matches;


select*from matches_corrected;

/*21Create a new table deliveries_v04 with the first column as ball_id containing information of match_id, inning, 
over and ball separated by ‘-’ (For ex. 335982-1-0-1 match_id-inning-over-ball) and rest of the columns same as deliveries_v03)*/




CREATE TABLE deliveries_v04 AS
SELECT
    CONCAT(id, '-', inning, '-', over, '-', ball) AS ball_id,
    deliveries_v03.*
FROM deliveries_v03;


select*from deliveries_v04;

--22Compare the total count of rows and total count of distinct ball_id in deliveries_v04;--

SELECT
    COUNT(*) AS total_rows,
    COUNT(DISTINCT ball_id) AS distinct_ball_ids
FROM deliveries_v04;

--23 Create table deliveries_v05 with all columns of deliveries_v04 and an additional column for row number partition over ball_id. (HINT : Syntax to add along with other columns,  row_number() over (partition by ball_id) as r_num)


CREATE TABLE deliveries_v05 AS
SELECT
    dv4.*,
    ROW_NUMBER() OVER (PARTITION BY ball_id) AS r_num
FROM deliveries_v04 dv4;

select *from deliveries_v05;

--24Use the r_num created in deliveries_v05 to identify instances where ball_id is repeating. (HINT : select * from deliveries_v05 WHERE r_num=2;)


SELECT *
FROM deliveries_v05
WHERE r_num > 1;


/*25Use subqueries to fetch data of all the ball_id which are repeating.
(HINT: SELECT * FROM deliveries_v05 WHERE ball_id in (select BALL_ID from deliveries_v05 WHERE r_num=2);*/


SELECT *
FROM deliveries_v05
WHERE ball_id IN (
    SELECT ball_id
    FROM deliveries_v05
    WHERE r_num > 1
);
