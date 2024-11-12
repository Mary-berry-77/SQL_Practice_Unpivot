#SQL Learning Notes

## SQL_Practice_Unpivot

##Goals
-Understanding GROUP BY
-Using MIN function with GROUP BY
-Unpivoting columns

##Problem (Adapted from HackerRank)
-Task: Pivot the Occupation column in the OCCUPATIONS table so that each Name is sorted alphabetically and displayed under its corresponding Occupation. The output columns should be titled Doctor, Professor, Singer, and Actor, respectively.
-The first column should be an alphabetically ordered list of Doctor names.
-The second column should be an alphabetically ordered list of Professor names.
-The third column should be an alphabetically ordered list of Singer names.
-The fourth column should be an alphabetically ordered list of Actor names.
-For columns with fewer names than the maximum count per occupation, the remaining cells should be filled with NULL.

##Database
-Table: occupations

|Name	|Occupation    |
|------|--------------|
|John Doe | Doctor |
|Jane Smith  |Doctor|
|Alice Johnson|Doctor|
|Bob Brown|	Doctor|
|Charlie Davis|	Doctor|
|Michael Wilson	|Professor|
|Sarah Clark|	Professor|
|David Martinez	|Professor|
|Emily Lewis|	Professor|
|Daniel Walker|	Professor|
|Sophia Anderson|	Singer|
|James Thomas|	Singer|
|Olivia Robinson|	Singer|
|Isabella Scott|	Actor|
|Ethan Harris|	Actor|
|Mia Turner|	Actor|
|Noah Green|	Actor|


##Solution Approach

###Step 1: Assign Rank to Each Name with ROW_NUMBER()
First, we assign a ranking to each name within each occupation using the ROW_NUMBER() function, sorted alphabetically by name. Here’s the basic structure of the subquery:


```sql
SELECT name, occupation,
       ROW_NUMBER() OVER (PARTITION BY occupation ORDER BY name) AS row_num
FROM occupations
```

###Step 2: Group by row_num
Using GROUP BY row_num in the main query groups the results by each row_num value. This allows us to arrange the table so that each row_num group holds names with the same ranking across all occupations. 

For instance:
-row_num = 1: contains the first alphabetically ranked name for each occupation
-row_num = 2: contains the second alphabetically ranked name for each occupation, and so on.

This step ensures that we have all names aligned by their rank, regardless of occupation.

###Step 3: Using MIN(CASE WHEN occupation = 'Doctor' THEN name ELSE NULL END)

The expression MIN(CASE WHEN occupation = 'Doctor' THEN name ELSE NULL END) works by checking the occupation of each record:
-When the occupation is 'Doctor', it selects the name.
-Otherwise, it sets the value to NULL.

The MIN() function then finds the alphabetically smallest name **within each row_num group** for Doctor. 
This process repeats across occupations (e.g., Professor, Singer, Actor), ensuring we get the smallest (alphabetically first) name in each row_num group per occupation.

###Example Explanation:

In each row_num group, this function returns names as follows:
-row_num = 1 group: Contains first alphabetical names across all occupations.
Doctor = Alice, Professor = Daniel, Singer = James, Actor = Ethan
-row_num = 2 group: Contains the next alphabetical names across all occupations.
Doctor = John, Professor = Emily, Singer = Olivia, Actor = Mia

The GROUP BY row_num structure here ensures that we only select the smallest alphabetically ordered names for each row_num, providing a neatly pivoted output.

##Final Query

```sql
SELECT  
    MIN(CASE WHEN occupation = 'Actor' THEN name ELSE NULL END) AS Actor,	     
    MIN(CASE WHEN occupation = 'Doctor' THEN name ELSE NULL END) AS Doctor,
    MIN(CASE WHEN occupation = 'Professor' THEN name ELSE NULL END) AS Professor,
    MIN(CASE WHEN occupation = 'Singer' THEN name ELSE NULL END) AS Singer
FROM (
    SELECT name, occupation,
           ROW_NUMBER() OVER (PARTITION BY occupation ORDER BY name) AS row_num
    FROM occupations
) AS ranked
GROUP BY row_num
ORDER BY row_num;
```

##Query Results Table

|Actor	|Doctor	|Professor	|Singer|
|------|-------------|-------------|-------|
|Ethan Harris|	Alice Johnson|	Daniel Walker|	Sophia Anderson|
|Mia Turner|	Bob Brown|	David Martinez|	James Thomas|
|Noah Green	|Charlie Davis|	Emily Lewis|	Olivia Robinson|
|Isabella Scott|	Jane Smith|	Michael Wilson|	NULL|
|NULL|	John Doe|	Sarah Clark|	NULL|


##Why Use GROUP BY row_num?
The ROW_NUMBER() function assigns ranks to rows but doesn’t aggregate them. By grouping on row_num, we can apply MIN() to **each rank level** across occupations, effectively letting us “pivot” the data by occupation. Thus, GROUP BY row_num allows us to align names by their rank within each occupation, creating the desired pivot effect.
