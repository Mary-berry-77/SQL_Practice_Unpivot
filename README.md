**#SQL Learning Notes**

## SQL_Practice_Unpivot + Applications

**##Goals**
-Understanding GROUP BY
-Using MIN function with GROUP BY
-Unpivoting columns

**##Problem (Adapted from HackerRank)**
-Task: Pivot the Occupation column in the OCCUPATIONS table so that each Name is sorted alphabetically and displayed under its corresponding Occupation. The output columns should be titled Doctor, Professor, Singer, and Actor, respectively.
-The first column should be an alphabetically ordered list of Doctor names.
-The second column should be an alphabetically ordered list of Professor names.
-The third column should be an alphabetically ordered list of Singer names.
-The fourth column should be an alphabetically ordered list of Actor names.
-For columns with fewer names than the maximum count per occupation, the remaining cells should be filled with NULL.

**##Database**
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


**##Solution Approach**

**###Step 1**: Assign Rank to Each Name with ROW_NUMBER()
First, we assign a ranking to each name within each occupation using the ROW_NUMBER() function, sorted alphabetically by name. Here’s the basic structure of the subquery:


```sql
SELECT name, occupation,
       ROW_NUMBER() OVER (PARTITION BY occupation ORDER BY name) AS row_num
FROM occupations
```

**###Step 2**: Group by row_num
Using GROUP BY row_num in the main query groups the results by each row_num value. This allows us to arrange the table so that each row_num group holds names with the same ranking across all occupations. 

For instance:
-row_num = 1: contains the first alphabetically ranked name for each occupation
-row_num = 2: contains the second alphabetically ranked name for each occupation, and so on.

This step ensures that we have all names aligned by their rank, regardless of occupation.

**###Step 3**: Using MIN(CASE WHEN occupation = 'Doctor' THEN name ELSE NULL END)

The expression MIN(CASE WHEN occupation = 'Doctor' THEN name ELSE NULL END) works by checking the occupation of each record:
-When the occupation is 'Doctor', it selects the name.
-Otherwise, it sets the value to NULL.

The MIN() function then finds the alphabetically smallest name **within each row_num group** for Doctor. 
This process repeats across occupations (e.g., Professor, Singer, Actor), ensuring we get the smallest (alphabetically first) name in each row_num group per occupation.

**###Example Explanation**:

In each row_num group, this function returns names as follows:
-row_num = 1 group: Contains first alphabetical names across all occupations.
Doctor = Alice, Professor = Daniel, Singer = James, Actor = Ethan
-row_num = 2 group: Contains the next alphabetical names across all occupations.
Doctor = John, Professor = Emily, Singer = Olivia, Actor = Mia

The GROUP BY row_num structure here ensures that we only select the smallest alphabetically ordered names for each row_num, providing a neatly pivoted output.

**##Final Query**

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

**##Query Results Table**

|Actor	|Doctor	|Professor	|Singer|
|------|-------------|-------------|-------|
|Ethan Harris|	Alice Johnson|	Daniel Walker|	Sophia Anderson|
|Mia Turner|	Bob Brown|	David Martinez|	James Thomas|
|Noah Green	|Charlie Davis|	Emily Lewis|	Olivia Robinson|
|Isabella Scott|	Jane Smith|	Michael Wilson|	NULL|







**#Application_1**

#Customer Interest Analysis by Age and Gender

This project analyses customer interests based on gender and age group data, drawing from the customer_interests table. By identifying the top interests in each demographic, we can gain insights into customer preferences, helping tailor marketing strategies effectively.

**##Problem Description**

**###The main goal is to:**

Aggregate customer interests by gender and age group.
Transform the top 5 interests for each demographic into separate columns, ordered by popularity.

For example, columns for Male_10s, Male_20s, Female_10s, Female_20s will display the most selected interests for each group. 
If a group has fewer recorded interests, remaining fields will display NULL.


**##Project Steps**

**###Step 1: Aggregate Customer Interests by Age and Gender**

The first step is to group interests by gender and age range, categorised as 10s, 20s, 30s, etc. 
This is achieved by using a CASE statement to define age groups and count the number of customers with each interest.


```sql
select gender,
       age_group,
       interest,
       count(*) as num_customers,
       sum(count(*)) over (partition by gender, age_group) as total_in_age_group
from (
    select
        gender,
        case 
            when age between 10 and 19 then '10s'
            when age between 20 and 29 then '20s'
            when age between 30 and 39 then '30s'
            when age between 40 and 49 then '40s'
            when age between 50 and 59 then '50s'
            else null 
        end as age_group,
        interest
    from customer_interests
) as age_in_group
group by gender, age_group, interest
order by gender, age_group, num_customers desc;
---

**###Step 2: Rank Interests within Each Demographic**

Using the data from Step 1, we calculate each interest’s popularity percentage and rank them within each demographic using row_number(). 
This makes it easy to identify the top interests by popularity.

---
with interest_percentage as (
    select 
        gender,
        age_group,
        interest,
        num_customers,
        round(num_customers / total_in_age_group * 100) as percentage,
        row_number() over (partition by gender, age_group order by num_customers desc) as rnum
    from age_group_num_customers
)
```

**###Step 3: Transform Interests into Columns for Output**

Finally, we use conditional aggregation to pivot the top interests into separate columns for each demographic group, listing interests and their respective percentages. 
Any group with fewer interests will display NULL in the corresponding cells.

```sql
select 
    concat(min(case when gender = 'M' and age_group = '10s' then interest else null end), ' (', 
           min(case when gender = 'M' and age_group = '10s' then percentage else null end), '%)') as male_10s,
    concat(min(case when gender = 'M' and age_group = '20s' then interest else null end), ' (', 
           min(case when gender = 'M' and age_group = '20s' then percentage else null end), '%)') as male_20s,
    concat(min(case when gender = 'M' and age_group = '30s' then interest else null end), ' (',  
           min(case when gender = 'M' and age_group = '30s' then percentage else null end), '%)') as male_30s,
    concat(min(case when gender = 'M' and age_group = '40s' then interest else null end), ' (',  
           min(case when gender = 'M' and age_group = '40s' then percentage else null end), '%)') as male_40s,
    concat(min(case when gender = 'M' and age_group = '50s' then interest else null end), ' (',  
           min(case when gender = 'M' and age_group = '50s' then percentage else null end), '%)') as male_50s,
    concat(min(case when gender = 'F' and age_group = '20s' then interest else null end), ' (',  
           min(case when gender = 'F' and age_group = '20s' then percentage else null end), '%)') as female_20s,
    concat(min(case when gender = 'F' and age_group = '30s' then interest else null end), ' (',  
           min(case when gender = 'F' and age_group = '30s' then percentage else null end), '%)') as female_30s,
    concat(min(case when gender = 'F' and age_group = '40s' then interest else null end), ' (',  
           min(case when gender = 'F' and age_group = '40s' then percentage else null end), '%)') as female_40s,
    concat(min(case when gender = 'F' and age_group = '50s' then interest else null end), ' (',  
           min(case when gender = 'F' and age_group = '50s' then percentage else null end), '%)') as female_50s
from interest_percentage 
group by rnum
limit 5;
```

**##Final Query**

```sql
with age_group_num_customers as(
select gender,
	   age_group,
	   interest,
	   count(*) as num_customers,
	   sum(count(*)) over (partition by gender, age_group) as total_in_age_group
from (
		select
			gender,
			case when age between 10 and 19 then '10s'
				 when age between 20 and 29 then '20s'
				 when age between 30 and 39 then '30s'
				 when age between 40 and 49 then '40s'
				 when age between 50 and 59 then '50s'
				 else null 
			end as age_group,
			interest
		from customer_interests
		) as age_in_group
group by gender,
		 age_group,
		 interest
order by gender, age_group, num_customers desc
),
interest_percentage as(
select gender,
	   age_group,
	   interest,
	   num_customers,
	   round(num_customers/total_in_age_group *100) as percentage,
	   row_number () over (partition by gender, age_group order by num_customers DESC) as rnum
from age_group_num_customers 
group by gender, age_group, interest
order by gender, age_group, rnum
)
select 
	concat(min(case when gender = 'M' and age_group = '10s' then interest else null end), ' (', 
	       min(case when gender = 'M' and age_group = '10s' then percentage else null end), '%)') as male_10s,
	concat(min(case when gender = 'M' and age_group = '20s' then interest else null end),  ' (', 
		   min(case when gender = 'M' and age_group = '20s' then percentage else null end), '%)')as male_20s,
	concat(min(case when gender = 'M' and age_group = '30s' then interest else null end), ' (',  
		   min(case when gender = 'M' and age_group = '30s' then percentage else null end), '%)')as male_30s,
	concat(min(case when gender = 'M' and age_group = '40s' then interest else null end), ' (',  
		   min(case when gender = 'M' and age_group = '40s' then percentage else null end), '%)')as male_40s,
	concat(min(case when gender = 'M' and age_group = '50s' then interest else null end), ' (',  
		   min(case when gender = 'M' and age_group = '50s' then percentage else null end), '%)')as male_50s,
	concat(min(case when gender = 'F' and age_group = '20s' then interest else null end),  ' (',  
		   min(case when gender = 'F' and age_group = '20s' then percentage else null end), '%)')as female_20s,
	concat(min(case when gender = 'F' and age_group = '30s' then interest else null end), ' (',  
		   min(case when gender = 'F' and age_group = '30s' then percentage else null end), '%)')as female_30s,
	concat(min(case when gender = 'F' and age_group = '40s' then interest else null end),  ' (',  
		   min(case when gender = 'F' and age_group = '40s' then percentage else null end), '%)')as female_40s,
	concat(min(case when gender = 'F' and age_group = '50s' then interest else null end),  ' (',  
		   min(case when gender = 'F' and age_group = '50s' then percentage else null end), '%)')as female_50s
from interest_percentage 
group by rnum
limit 5;
```

**##Output **


|male_10s   |     male_20s        |       male_30s            |      male_40s            |     male_50s       |  female_20s|       female_30s             |  female_40s|       female_50s|
|---------------|---------------------|---------------------------|-------------------------|-----------------------|-----------------|----------------------------|---------------|-----------------|
|Music (33%)	|Gaming (33%)	     |Technology (28%)| Technology (29%)	|Sports (33%)	        |Fitness (22%)	|Cooking (22%)	|Cooking (33%)	|Cooking (33%)|
|Gaming (33%)	|Sports (28%)	     |Gaming (26%)	|Gaming (25%)	     |Fitness (25%)	           |Beauty (20%)  |	Fitness (21%)	|Fitness (29%)	|Beauty (33%)|
|Sports (17%)|	Music (23%)	     |Music (24%)	       |Music (23%)	     |Technology (25%)  	|Cooking (16%)	|Beauty (20%)	|Gaming (14%)	|Fashion (33%)|
|Technology (17%)|Technology (13%)   |Sports (22%)	       |Sports (19%)|        	Music (8%)   	|Technology (15%)	|Technology (17%)	|Technology (10%)|	NULL|
|NULL|	       Fitness (2%)|                        NULL       |Fitness (4%)|                Gaming (8%)	|Fashion (12%)    |	Gaming (8%)	|Beauty (10%)	|  NULL                  |
|NULL|	John Doe|	Sarah Clark|	NULL|


**##Why Use GROUP BY row_num?**
The ROW_NUMBER() function assigns ranks to rows but doesn’t aggregate them. By grouping on row_num, we can apply MIN() to **each rank level** across occupations, effectively letting us “pivot” the data by occupation. Thus, GROUP BY row_num allows us to align names by their rank within each occupation, creating the desired pivot effect.
